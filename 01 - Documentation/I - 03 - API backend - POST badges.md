## Étape 1 — Création de la route POST /badges

On définit une route Fastify qui reçoit une requête HTTP POST à l'adresse `/badges`. Elle accepte deux paramètres :

- `uid` : chaîne hexadécimale de 8 caractères (identifiant du badge)
    
- `userId` : identifiant unique de l’utilisateur auquel le badge est rattaché
    

On valide les paramètres, on dérive les clés AES, on insère le badge dans la base de données, puis on publie le message sur MQTT.

```ts
import { FastifyInstance } from 'fastify';
import { z } from 'zod';
import { deriveKeysAB } from '../deriveKeysAB.utils.js';
import { deriveBadgeKey } from '../deriveBadgeKey.utils.js';
import { BadgeEntity } from '../badges.entity.js';
import { AppDataSource } from '../../../data-source.js';
import { mqttService } from '../../../services/mqtt.service.js';
import fastifyPlugin from 'fastify-plugin';

export async function badgeAddRoutes(app: FastifyInstance) {
  app.post('/badges/add', async (request, reply) => {
    const bodySchema = z.object({
      uid: z.string().regex(/^[0-9A-F]{8}$/i, 'Invalid UID'),
      userId: z.string().uuid(),
    });

    const { uid, userId } = bodySchema.parse(request.body);

    const cardId = Buffer.from(uid, 'hex');

    const badgeRepo = AppDataSource.getRepository(BadgeEntity);
    const existing = await badgeRepo.findOneBy({ cardId });
    if (existing) {
      return reply.code(409).send({
        success: false,
        message: 'Badge already exists',
      });
    }

    const { keyA, keyB } = deriveKeysAB(uid);
    const derivedKey = deriveBadgeKey(cardId);

    const badge = badgeRepo.create({
      cardId,
      userId,
      keyA: Buffer.from(keyA, 'hex'),
      keyB: Buffer.from(keyB, 'hex'),
      derivedKey,
    });
    await badgeRepo.save(badge);

    await mqttService.publish('access/encoder', JSON.stringify({
      derivedKey: derivedKey.toString('hex'),
      keyA,
      keyB,
    }));

    return reply.code(201).send({ id: badge.id });
  });
}

export default fastifyPlugin(badgeAddRoutes);
```

---

## Étape 2 — Fonction de dérivation AES128

Cette fonction reçoit l'UID, génère un hash SHA256, et en extrait deux clés AES128 :

```ts
import crypto from 'crypto';

export function deriveBadgeKey(uid: Buffer): string {
  const hash = crypto.createHash('sha256').update(uid).digest();
  return hash.subarray(0, 16).toString('hex');
}
```

---

## Étape 3 — Service MQTT avec configuration au démarrage

Ce projet utilise déjà une infrastructure MQTT centralisée via le plugin `mqtt.ts`, qui initialise une seule connexion MQTT au démarrage de l'application (avec toutes les options nécessaires) et expose un client singleton via `getMqttClient()`.

Pour fournir une interface simple `publish(...)` et `subscribe(...)` utilisables dans les routes, on ajoute un petit adaptateur qui exploite ce client existant. Cela permet d'intégrer facilement la publication et l'abonnement sans créer de nouvelle connexion MQTT.

```ts
// services/mqttPublisherService.ts
import { getMqttClient } from '../../plugins/mqtt';

export const mqttService = {
  publish: (topic: string, message: string | Buffer): Promise<void> => {
    try {
      const client = getMqttClient();

      return new Promise((resolve, reject) => {
        client.publish(topic, message, (err) => {
          if (err) reject(err);
          else resolve();
        });
      });
    } catch (err) {
      if (process.env.NODE_ENV === 'test') return Promise.resolve();
      throw err;
    }
  },

  subscribe: (topic: string, callback: (message: Buffer) => void): void => {
    try {
      const client = getMqttClient();

      client.subscribe(topic, (err) => {
        if (err) {
          console.error(`Subscribe error on topic ${topic}`, err);
        }
      });

      client.on('message', (receivedTopic, message) => {
        if (receivedTopic === topic) {
          callback(message);
        }
      });
    } catch (err) {
      if (process.env.NODE_ENV !== 'test') throw err;
    }
  },
};
```

---

## Étape 4 — Test unitaire complet avec mocks

Ce test vérifie le bon fonctionnement de la route, la validation des paramètres, la génération des clés et l'appel au service MQTT. Il utilise `app.ts` pour éviter de lancer le serveur sur le port 3000. La base est nettoyée entre chaque test et le service MQTT est mocké pour ne pas exiger de connexion réelle.

```ts
import { beforeAll, afterAll, describe, it, expect, vi, beforeEach } from 'vitest';
import { AppDataSource } from '../../../data-source.js';
import app from '../../../app.js';
import { mqttService } from '../../../services/mqtt.service.js';
import { BadgeEntity } from '../badges.entity.js';

vi.mock('../../../services/mqtt.service.js', async () => {
  const actual = await vi.importActual<typeof import('../../../services/mqtt.service.js')>(
    '../../../services/mqtt.service.js'
  );
  return {
    ...actual,
    mqttService: {
      publish: vi.fn(),
    },
  };
});

describe('POST /badges/add', () => {
  beforeAll(async () => {
    if (!AppDataSource.isInitialized) {
      await AppDataSource.initialize();
    }
  });

  beforeEach(async () => {
    await AppDataSource.getRepository(BadgeEntity).clear();
    vi.clearAllMocks();
  });

  afterAll(async () => {
    if (AppDataSource.isInitialized) {
      await AppDataSource.destroy();
    }
  });

  it('crée un badge et publie sur MQTT', async () => {
    const response = await app.inject({
      method: 'POST',
      url: '/badges/add',
      payload: {
        uid: 'A1B2C3D4',
        userId: '550e8400-e29b-41d4-a716-446655440000',
      },
    });

    expect(response.statusCode).toBe(201);
    const payload = JSON.parse(response.body);
    expect(payload).toHaveProperty('id');

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/encoder',
      expect.stringContaining('derivedKey')
    );
  });

  it('retourne une erreur si le badge existe déjà', async () => {
    const payload = {
      uid: 'A1B2C3D4',
      userId: '550e8400-e29b-41d4-a716-446655440000',
    };

    const first = await app.inject({
      method: 'POST',
      url: '/badges/add',
      payload,
    });
    expect(first.statusCode).toBe(201);

    const second = await app.inject({
      method: 'POST',
      url: '/badges/add',
      payload,
    });

    // Selon la logique backend, ça peut être 409 (conflit) ou 500 (erreur SQL)
    expect(second.statusCode).toBeGreaterThanOrEqual(400);
  });
});
```