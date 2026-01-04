## 🔐 

Cette documentation explique comment mettre en place la dérivation d'une clé AES128 à partir d'un identifiant UID unique (badge) et d'une clé maîtresse `MASTER_KEY` secrète.

---

### 1. Ajout de `MASTER_KEY` dans `.env`

Ajoutez une variable secrète dans votre fichier `.env` à la racine du projet :

```env
MASTER_KEY_HEX=xxx
```

#### Génération de la clé maîtresse (macOS / Linux uniquement)

Utilisez la commande suivante pour générer une clé aléatoire de 32 octets (256 bits) encodée en hexadécimal :

```bash
openssl rand -hex 32
```

Exemple de résultat :

```env
MASTER_KEY_HEX=49d1a7ff2a8a3edc6ddcf9b37824308e4a6a0e9344ac6f8e3e79833bd06a9172
```

> Ne jamais commit cette clé dans un dépôt public. Utilisez un outil comme `dotenv` pour la charger.

---

### 2. Fonction `deriveBadgeKey(uid: Buffer): Buffer`

La fonction suivante dérive une clé AES128 unique (16 octets) à partir de l'UID du badge et de la `MASTER_KEY_HEX`, en utilisant HKDF avec SHA-256.

#### Fichier : `src/features/badges/deriveBadgeKey.utils.ts`

```ts
import { hkdfSync } from 'node:crypto';

export function deriveBadgeKey(uid: Buffer): Buffer {
  const masterKeyHex = process.env.MASTER_KEY_HEX;
  if (!masterKeyHex) throw new Error('MASTER_KEY_HEX env var manquante');

  const masterKey = Buffer.from(masterKeyHex, 'hex'); // 32 octets (256 bits)

  if (uid.length !== 4 && uid.length !== 7) {
    throw new Error('UID must be 4 or 7 bytes');
  }

  const info = Buffer.from('BADGE_KEY', 'utf8');
  const salt = Buffer.alloc(0);
  const ab = hkdfSync('sha256', masterKey, salt, Buffer.concat([uid, info]), 16);
  return Buffer.from(ab);
}
```

---

### 3. Tests unitaires de validation et d'erreur

Des tests permettent de valider les cas d'usage normaux ainsi que les erreurs attendues.

#### Fichier : `src/features/badges/deriveBadgeKey.utils.test.ts`

```ts
import { describe, it, expect, beforeEach } from 'vitest';
import { deriveBadgeKey } from './deriveBadgeKey.utils';

describe('deriveBadgeKey', () => {
  beforeEach(() => {
    process.env.MASTER_KEY_HEX = '49d1a7ff2a8a3edc6ddcf9b37824308e4a6a0e9344ac6f8e3e79833bd06a9172';
  });

  it('should derive a stable AES128 key from UID', () => {
    const uid = Buffer.from('04A1B2C3', 'hex');
    const key = deriveBadgeKey(uid);
    expect(key.toString('hex')).toBe('02f64e1a3e2f85982a17694c12ebea44');
  });

  it('should throw on UID shorter than 4 bytes', () => {
    const shortUid = Buffer.from([0x01, 0x02]);
    expect(() => deriveBadgeKey(shortUid)).toThrow('UID must be 4 or 7 bytes');
  });

  it('should throw on UID longer than 7 bytes', () => {
    const longUid = Buffer.alloc(8);
    expect(() => deriveBadgeKey(longUid)).toThrow('UID must be 4 or 7 bytes');
  });

  it('should throw if MASTER_KEY_HEX is not defined', () => {
    delete process.env.MASTER_KEY_HEX;
    expect(() => deriveBadgeKey(Buffer.alloc(4))).toThrow('MASTER_KEY_HEX env var manquante');
  });
});
```

> Modifiez la valeur attendue dans le test principal si vous utilisez une autre `MASTER_KEY_HEX`.

---

### Annexe : Liens utiles

- [RFC 5869 - HKDF: HMAC-based Extract-and-Expand Key Derivation Function](https://datatracker.ietf.org/doc/html/rfc5869)
    
- [Node.js `crypto.hkdfSync` documentation](https://nodejs.org/api/crypto.html#crypto_crypto_hkdfsync_digest-key-salt-info-keylen)