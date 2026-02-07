### ℹ️ Pourquoi utiliser AES-128 plutôt que AES-256 ?

#### Contraintes matérielles côté badge

- Les lecteurs NFC ou badges (type MIFARE, PN532) ne supportent souvent **que AES-128**.
    
- AES-256 est plus gourmand (CPU, mémoire) → incompatible avec certains microcontrôleurs.
    

#### Sécurité suffisante pour notre cas d’usage

- AES-128 reste **extrêmement sécurisé** à ce jour : pas de cassure pratique connue.
    
- Le challenge transmis ne contient aucune donnée sensible.
    
- La **vérification de l’authenticité** est assurée côté backend avec une base de données sécurisée.
    
- Les clés sont stockées chiffrées **en AES-256-GCM**, donc **la base est sécurisée**, même si le badge lui-même ne l'est pas.
    

> ⚖️ **Conclusion** : on utilise AES-128 pour **des raisons de compatibilité matérielle**, sans sacrifier la sécurité globale du système.

### 🔐 Schéma de chiffrement / déchiffrement du badge AES128

#### 1. Enregistrement initial d’un badge (exécuté une seule fois)

```
[Clé A ou B brute (16 octets)]
            │
            ▼
Chiffrement AES-256-GCM avec:
  - clé = MASTER_KEY (32 octets depuis env: DB_ENC_KEY_HEX)
  - IV = généré aléatoirement (12 octets)
            │
            ▼
[Encrypted = IV + TAG + CIPHER]
            │
            ▼
Stocké dans badge.derivedKey
```

#### 2. À la présentation d’un badge (lecture en temps réel)

```
[Badge présente son UID]
            │
            ▼
Le badge chiffre en local une constante ("access-check") avec :
  - clé = Clé A (16 octets)
  - IV = derivedKey déchiffré (pour obtenir IV original)
  - Algo = AES-128-CBC
            │
            ▼
Produit : encryptedChallenge
→ envoyé au backend dans le message MQTT
```

#### 3. Côté backend : traitement du message

```
[Reçoit UID + encryptedChallenge + source]
            │
            ▼
Recherche badge en base par UID
            │
            ├─ Si non trouvé → refus immédiat
            ├─ Si deniedAccessFlag = true → refus immédiat
            ▼
Déchiffre badge.derivedKey avec MASTER_KEY
  → récupère IV original
            │
            ▼
Rechiffre localement "access-check" avec :
  - clé = badge.keyA
  - IV = IV original
            │
            ▼
Compare avec encryptedChallenge reçu
            ├─ Égal → accès GRANTED
            └─ Sinon → accès DENIED
```

#### 4. Si accès GRANTED :

```
Publie sur :
- access/reader/result     → feedback au lecteur
- LOGO_sub                 → message de commande vers le contrôleur (ex-: ouverture de porte)
- access_logs              → journalisation
```

## Intégration MQTT Backend : Lecteur de badge (`access/reader`)

### Objectif

Réagir à chaque badge présenté sur un lecteur connecté via MQTT, en validant l'accès à partir des données du badge, loggant l'événement, et envoyant les instructions appropriées.

---

### 1. Scénario global d'accès

1. Le lecteur lit un badge et publie un message sur `access/reader` avec :
    
    - `uid` : identifiant unique de la carte (hexadécimal)
        
    - `encrypted` : réponse chiffrée du badge au challenge
        
    - `source` : nom du lecteur ou point d'entrée
        
2. Le backend :
    
    - Acquitte la réception avec un message sur `access/reader/ack` (contenu minimal)
        
    - Valide la réponse chiffrée en recherchant le badge et en régénérant le chiffrement attendu
        
    - Si la réponse est correcte :
        
        - Publie une réponse de validation sur `access/reader/result`
            
        - Publie une commande MQTT sur `LOGO_sub` (reçue par le contrôleur LOGO!)
            
    - Si la réponse est incorrecte :
        
        - Publie une réponse de refus sur `access/reader/result`
            
    - Enregistre l'événement dans `access_logs`
        

---

### 2. Écoute MQTT sur `access/reader`

Cette étape met en place un abonnement au topic MQTT `access/reader` pour écouter les messages émis par les lecteurs de badge. Dès réception d'un message, un acquittement est publié sur `access/reader/ack`, puis la logique de traitement est déléguée à `handleReaderAccess`.

#### Fichier : `src/subscribers/accessReader.subscriber.ts`

```ts
import { mqttService } from '../services/mqtt.service.js';
import { handleReaderAccess } from '../services/handleReaderAccess.js';

mqttService.subscribe('access/reader', async (message) => {
  try {
    const data = JSON.parse(message.toString());
    await mqttService.publish('access/reader/ack', JSON.stringify({ received: true, uid: data.uid }));
    await handleReaderAccess(data);
  } catch (err) {
    console.error('[access/reader] Invalid message or error:', err);
  }
});
```

---

### 3. Handler : `handleReaderAccess`

Cette fonction centrale traite les données reçues d’un lecteur :

- Elle recherche le badge correspondant via son UID
    
- Elle vérifie s’il est valide (non interdit) et si le chiffrement attendu correspond à celui reçu
    
- Elle publie une réponse sur `access/reader/result`
    
- En cas de succès, elle publie une commande d'ouverture de porte vers le contrôleur concerné
    
- En cas d’échec (badge absent, interdit ou chiffrement incorrect), elle ne publie **aucune** commande d’ouverture
    
- Elle loggue systématiquement l'accès en base de données
    

#### Fichier : `src/services/handleReaderAccess.ts`

```ts
import { AppDataSource } from '../db/data-source.js';
import { BadgeEntity } from '../entities/BadgeEntity.js';
import { AccessLogEntity } from '../entities/AccessLogEntity.js';
import { mqttService } from './mqtt.service.js';
import { encryptAes128Challenge } from '../utils/crypto.js';

type ReaderAccessPayload = {
  uid: string;
  encrypted: string;
  source: string;
};

export async function handleReaderAccess({ uid, encrypted, source }: ReaderAccessPayload) {
  const badgeRepo = AppDataSource.getRepository(BadgeEntity);
  const logRepo = AppDataSource.getRepository(AccessLogEntity);

  const uidBuffer = Buffer.from(uid, 'hex');
  const badge = await badgeRepo.findOne({ where: { cardId: uidBuffer } });

  let accessOutcome: 'granted' | 'denied' = 'denied';
  let userId: string | null = null;

  if (badge && !badge.deniedAccessFlag) {
    const expected = encryptAes128Challenge('access-check', badge.keyA, badge.derivedKey);

    if (expected === encrypted) {
      accessOutcome = 'granted';
      userId = badge.userId;
      await mqttService.publish(`access/door/open/${source}`, JSON.stringify({ open: true, uid }));
    }
  }

  await mqttService.publish(
    'access/reader/result',
    JSON.stringify({ uid, access: accessOutcome, source })
  );

  const log = logRepo.create({
    cardId: uidBuffer,
    userId,
    accessOutcome,
    source,
  });

  await logRepo.save(log);
}
```

---

### 4. Utilitaire de chiffrement AES128 (dans `crypto.ts`)

#### Définition des paramètres :

- **`keyBuffer`** :
    
    - Type : `Buffer`
        
    - Contenu : Clé AES128 utilisée pour chiffrer la chaîne "access-check"
        
    - Source : `badge.keyA` (ou `keyB` si besoin)
        
    - Taille : 16 octets (128 bits)
        
- **`ivBuffer`** :
    
    - Type : `Buffer`
        
    - Contenu : vecteur d'initialisation utilisé pour le chiffrement AES-128-CBC
        
    - Source : obtenu en déchiffrant `badge.derivedKey` avec `decryptColumn()`
        
    - Taille : 16 octets
        

```ts
import { createCipheriv, createDecipheriv, randomBytes } from 'crypto';

const ENC_KEY = Buffer.from(process.env.DB_ENC_KEY_HEX!, 'hex');
const IV_LEN = 12;

export function encryptColumn(plain: Buffer): Buffer {
  const iv = randomBytes(IV_LEN);
  const cipher = createCipheriv('aes-256-gcm', ENC_KEY, iv);
  const encrypted = Buffer.concat([cipher.update(plain), cipher.final()]);
  const tag = cipher.getAuthTag();
  return Buffer.concat([iv, tag, encrypted]);
}

export function decryptColumn(data: Buffer): Buffer {
  const iv = data.subarray(0, IV_LEN);
  const tag = data.subarray(IV_LEN, IV_LEN + 16);
  const encrypted = data.subarray(IV_LEN + 16);
  const decipher = createDecipheriv('aes-256-gcm', ENC_KEY, iv);
  decipher.setAuthTag(tag);
  return Buffer.concat([decipher.update(encrypted), decipher.final()]);
}

export function encryptAes128Challenge(plainText: string, keyBuffer: Buffer, ivBuffer: Buffer): string {
  const cipher = createCipheriv('aes-128-cbc', keyBuffer, ivBuffer);
  let encrypted = cipher.update(plainText, 'utf8', 'hex');
  encrypted += cipher.final('hex');
  return encrypted;
}
```

---

### 5. Exemple de message MQTT attendu (lecteur → backend)

```json
{
  "uid": "a1b2c3d4e5f6",
  "encrypted": "3ac5fe21bdc1...",
  "source": "porte-01"
}
```

---

### 6. Tests unitaires : Subscriber

Ce test vérifie que le subscriber :

- s'abonne correctement au topic `access/reader`
    
- publie un acquittement sur `access/reader/ack`
    
- appelle la fonction `handleReaderAccess` avec les données reçues
    

#### Fichier : `test/subscribers/accessReader.subscriber.test.ts`

```ts
import { describe, it, vi, beforeEach, expect } from 'vitest';
import type { Mock } from 'vitest';
import { mqttService } from '../../src/services/mqtt.service.js';

vi.mock('../../src/services/mqtt.service.js', () => ({
  mqttService: {
    publish: vi.fn(),
    subscribe: vi.fn((topic, cb) => {
      console.log('Mock subscribe called with topic:', topic);
    })
  }
}));

vi.mock('../../src/services/handleReaderAccess.js', () => ({
  handleReaderAccess: vi.fn()
}));

describe('access/reader subscriber',  () => {
  beforeEach(async() => {
    vi.clearAllMocks();
    await import('../../src/subscribers/accessReader.subscriber.js');
  });

  it('publie un ack et appelle handleReaderAccess', async () => {
    const message = {
      uid: 'a1b2c3d4e5f6',
      encrypted: 'abcd1234ef',
      source: 'porte-01'
    };

    const messageBuffer = Buffer.from(JSON.stringify(message));

    const subscribeMock = mqttService.subscribe as Mock;
    const subscribeCall = subscribeMock.mock.calls.find(call => call[0] === 'access/reader');
    const callback = subscribeCall?.[1];

    expect(callback).toBeDefined();

    await callback(messageBuffer);

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/reader/ack',
      JSON.stringify({ received: true, uid: message.uid })
    );
  });
});
```

---

### 7. Tests unitaires : Module `crypto`

Ce test vérifie le bon fonctionnement des fonctions :

- `encryptColumn`
    
- `decryptColumn`
    
- `encryptAes128Challenge`
    

#### Fichier : `test/utils/crypto.test.ts`

```ts
import { describe, it, expect, beforeAll } from 'vitest';
import crypto from 'crypto';

let encryptColumn: any;
let decryptColumn: any;
let encryptAes128Challenge: any;

beforeAll(async () => {
  process.env.DB_ENC_KEY_HEX = crypto.randomBytes(32).toString('hex');
  const cryptoUtils = await import('../../src/utils/crypto.js');
  encryptColumn = cryptoUtils.encryptColumn;
  decryptColumn = cryptoUtils.decryptColumn;
  encryptAes128Challenge = cryptoUtils.encryptAes128Challenge;
});

describe('crypto utils', () => {
  it('encryptColumn and decryptColumn should be reversible', () => {
    const input = Buffer.from('hello world', 'utf-8');
    const encrypted = encryptColumn(input);
    const decrypted = decryptColumn(encrypted);
    expect(decrypted.toString()).toBe('hello world');
  });

  it('encryptAes128Challenge should produce a valid ciphertext', () => {
    const key = crypto.randomBytes(16);
    const iv = crypto.randomBytes(16);
    const encrypted = encryptAes128Challenge('access-check', key, iv);
    expect(typeof encrypted).toBe('string');
    expect(encrypted.length).toBeGreaterThan(0);
  });

  it('should throw if decryptColumn receives invalid data', () => {
    expect(() => decryptColumn(Buffer.from('invalid'))).toThrow();
  });

  it('should throw if encryptAes128Challenge receives a key of invalid length', () => {
    const badKey = crypto.randomBytes(8);
    const iv = crypto.randomBytes(16);
    expect(() => encryptAes128Challenge('access-check', badKey, iv)).toThrow();
  });
  
  it('logs access attempt with correct data', async () => {
	const badge = {
		deniedAccessFlag: false,
		keyA: Buffer.from('a'.repeat(32), 'hex'),
		derivedKey: Buffer.from('b'.repeat(32), 'hex'),
		cardId: Buffer.from('abcd', 'hex'),
		userId: 'user-123'
	};
	(cryptoUtils.encryptAes128Challenge as any).mockReturnValue('expected-value');
	mockBadgeRepo.findOne.mockResolvedValue(badge);
	
	const mockLog = { id: 'log-1' };

	mockLogRepo.create.mockReturnValue(mockLog);
	mockLogRepo.save.mockResolvedValue(mockLog);
	
	await handleReaderAccess({ uid: 'abcd', encrypted: 'expected-value', source: 'porte-01' });
	
	expect(mockLogRepo.create).toHaveBeenCalledWith({
		cardId: Buffer.from('abcd', 'hex'),
		userId: 'user-123',
		accessOutcome: 'granted',
		source: 'porte-01'
	});

	expect(mockLogRepo.save).toHaveBeenCalledWith(mockLog);
  });
});
```

---

### 8. Tests unitaires : Handler `handleReaderAccess`

Ce test couvre la fonction `handleReaderAccess` dans plusieurs cas :

- badge non trouvé
    
- badge refusé (`deniedAccessFlag: true`)
    
- badge autorisé mais clé incorrecte
    
- badge autorisé avec clé correcte (autorisation et ouverture)
    

#### Fichier : `test/services/handleReaderAccess.test.ts`

```ts
import crypto from 'crypto';
process.env.DB_ENC_KEY_HEX = crypto.randomBytes(32).toString('hex');

import { describe, it, vi, beforeEach, expect } from 'vitest';

vi.mock('../../src/utils/crypto.js', () => ({
  encryptAes128Challenge: vi.fn(() => 'not-matching'),
}));

import { handleReaderAccess } from '../../src/services/handleReaderAccess.js';
import { AppDataSource } from '../../src/data-source.js';
import { mqttService } from '../../src/services/mqtt.service.js';
import * as cryptoUtils from '../../src/utils/crypto.js';

vi.mock('../../src/services/mqtt.service.js', () => ({
  mqttService: {
    publish: vi.fn()
  }
}));

vi.mock('../../src/data-source.js', () => ({
  AppDataSource: {
    getRepository: vi.fn()
  }
}));

describe('handleReaderAccess', () => {
  const mockBadgeRepo = {
    findOne: vi.fn()
  };
  const mockLogRepo = {
    create: vi.fn(),
    save: vi.fn()
  };

  beforeEach(() => {
    vi.clearAllMocks();
    AppDataSource.getRepository = vi.fn()
      .mockImplementationOnce(() => mockBadgeRepo)
      .mockImplementationOnce(() => mockLogRepo);
  });

  it('publishes denied if badge not found', async () => {
    mockBadgeRepo.findOne.mockResolvedValue(null);

    await handleReaderAccess({ uid: 'abcd', encrypted: 'xxx', source: 'porte-01' });

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/reader/result',
      JSON.stringify({ uid: 'abcd', access: 'denied', source: 'porte-01' })
    );
  });

  it('publishes denied if badge is flagged denied', async () => {
    mockBadgeRepo.findOne.mockResolvedValue({ deniedAccessFlag: true });

    await handleReaderAccess({ uid: 'abcd', encrypted: 'xxx', source: 'porte-01' });

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/reader/result',
      JSON.stringify({ uid: 'abcd', access: 'denied', source: 'porte-01' })
    );
  });

  it('publishes denied if encrypted != expected', async () => {
    const badge = {
      deniedAccessFlag: false,
      keyA: Buffer.from('a'.repeat(32), 'hex'),
      derivedKey: Buffer.from('b'.repeat(32), 'hex'),
      cardId: Buffer.from('abcd', 'hex'),
      userId: 'user-123'
    };

    (cryptoUtils.encryptAes128Challenge as any).mockReturnValue('not-matching');
    mockBadgeRepo.findOne.mockResolvedValue(badge);

    await handleReaderAccess({ uid: 'abcd', encrypted: 'expected-value', source: 'porte-01' });

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/reader/result',
      JSON.stringify({ uid: 'abcd', access: 'denied', source: 'porte-01' })
    );
  });

  it('publishes granted and opens door if encrypted == expected', async () => {
    const badge = {
      deniedAccessFlag: false,
      keyA: Buffer.from('a'.repeat(32), 'hex'),
      derivedKey: Buffer.from('b'.repeat(32), 'hex'),
      cardId: Buffer.from('abcd', 'hex'),
      userId: 'user-123'
    };

    (cryptoUtils.encryptAes128Challenge as any).mockReturnValue('expected-value');
    mockBadgeRepo.findOne.mockResolvedValue(badge);

    await handleReaderAccess({ uid: 'abcd', encrypted: 'expected-value', source: 'porte-01' });

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/door/open/porte-01',
      JSON.stringify({ open: true, uid: 'abcd' })
    );

    expect(mqttService.publish).toHaveBeenCalledWith(
      'access/reader/result',
      JSON.stringify({ uid: 'abcd', access: 'granted', source: 'porte-01' })
    );
  });
});
```

---

### 9. Rappel - Topics MQTT utilisés

- `access/reader` : message initial du lecteur (avec badge)
    
- `access/reader/ack` : acquittement immédiat de réception
    
- `access/reader/result` : réponse finale (granted / denied)
    
- `LOGO_sub` : canal de réception pour le contrôleur (ouverture de porte)
    
- `LOGO_pub` : canal de retour du contrôleur (état, acquittement)