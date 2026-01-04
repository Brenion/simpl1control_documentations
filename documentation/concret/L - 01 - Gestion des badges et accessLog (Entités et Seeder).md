
Ce document présente l’implémentation complète et déterministe du système de badges dans le projet domotique. Il inclut l’entité `BadgeEntity`, l’entité `AccessLogEntity`, le `BadgeFactory` et le `UserSeeder` enrichi pour générer un badge par utilisateur.

---

### 1. Entité `BadgeEntity`

```ts
import {
  Entity,
  PrimaryColumn,
  Column
} from 'typeorm';
import { BaseEntity } from '../base.entity.js';

@Entity({ name: 'badges' })
export class BadgeEntity extends BaseEntity {
  @PrimaryColumn({ type: 'bytea' })
  cardId!: Buffer;

  @Column({ type: 'boolean', default: false })
  deniedAccessFlag!: boolean;

  @Column({ type: 'uuid', unique: true })
  userId!: string;

  @Column({ type: 'bytea', nullable: false })
  derivedKey!: Buffer;

  @Column({ type: 'bytea', nullable: false })
  keyA!: Buffer;

  @Column({ type: 'bytea', nullable: false })
  keyB!: Buffer;
}
```

Cette entité correspond à un badge physique de type MIFARE Classic. L’identifiant `cardId` est stocké en binaire. Chaque badge est associé à un `userId` unique. Les clés `derivedKey`, `keyA` et `keyB` sont des valeurs binaires nécessaires pour l’authentification et le chiffrement. `deniedAccessFlag` permet de bloquer un badge perdu ou volé.

---

### 2. Entité `AccessLogEntity`

```ts
import { Entity, Column } from 'typeorm';
import { BaseEntity } from '../base.entity.js';

@Entity({ name: 'access_logs' })
export class AccessLogEntity extends BaseEntity {
  @Column({ type: 'bytea', nullable: true })
  cardId!: Buffer | null;

  @Column({ type: 'uuid', nullable: true })
  userId!: string | null;

  @Column({ type: 'varchar' })
  accessOutcome!: 'granted' | 'denied';

  @Column({ type: 'varchar' })
  source!: string;
}
```

Cette entité enregistre chaque tentative d’accès. `cardId` et `userId` sont optionnels pour gérer les cartes inconnues ou utilisateurs supprimés. Le champ `accessOutcome` est une chaîne représentant le résultat de l’accès. `source` décrit le point d’entrée ou le lecteur utilisé.

---

### 3. Factory `badge.factory.ts`

```ts
import { FactorizedAttrs, Factory } from '@jorgebodega/typeorm-factory'
import { AppDataSource as dataSource } from '../../data-source.js'
import { faker } from '@faker-js/faker'
import { BadgeEntity } from '../../entities/badge.entity.js'
import { randomUUID } from 'crypto'

class BadgeFactory extends Factory<BadgeEntity> {
  protected entity = BadgeEntity
  protected dataSource = dataSource

  protected attrs(): FactorizedAttrs<BadgeEntity> {
    return {
      cardId: Buffer.from(faker.string.hexadecimal({ length: 8, casing: 'lower' }).replace('0x', ''), 'hex'),
      deniedAccessFlag: false,
      userId: randomUUID(),
      derivedKey: Buffer.from(faker.string.hexadecimal({ length: 32, casing: 'lower' }).replace('0x', ''), 'hex'),
      keyA: Buffer.from(faker.string.hexadecimal({ length: 32, casing: 'lower' }).replace('0x', ''), 'hex'),
      keyB: Buffer.from(faker.string.hexadecimal({ length: 32, casing: 'lower' }).replace('0x', ''), 'hex'),
    }
  }
}

export default BadgeFactory
```

Cette factory permet de créer dynamiquement des badges pour les tests. Dans un contexte de seed déterministe, seules les clés secondaires peuvent être aléatoires. `userId` est toujours remplacé manuellement dans le seeder pour correspondre à un utilisateur réel.

---

### 4. Seeder `user.seeder.ts`

```ts
import { Seeder } from '@jorgebodega/typeorm-seeding'
import UserFactory from '../factories/user.factory.js'
import BadgeFactory from '../factories/badge.factory.js'
import Role from '../../enums/roles-enum.js'
import { hash } from 'argon2'
import { NIL as NIL_UUID } from 'uuid'

class UserSeeder extends Seeder {
  async run() {
    const userFactory = new UserFactory()
    const badgeFactory = new BadgeFactory()

    const admin = await userFactory.create({
      firstname: 'admin',
      lastname: 'admin',
      username: 'alice.d',
      mail: 'admin@demo.local',
      password: await hash('admin'),
      role: Role.ADMIN,
    })

    const security = await userFactory.create({
      firstname: 'Bruno',
      lastname: 'Martin',
      username: 'bruno.m',
      password: await hash('bruno'),
      mail: 'security@demo.local',
      role: Role.SECURITY,
    })

    const employee = await userFactory.create({
      firstname: 'Chloé',
      lastname: 'Bernard',
      username: 'chloe.b',
      password: await hash('chloe'),
      mail: 'employee@demo.local',
      role: Role.EMPLOYEE,
    })

    const test = await userFactory.create({
      id: NIL_UUID,
      firstname: 'test',
      lastname: 'test',
      username: 'test',
      password: await hash('test'),
      mail: 'test@demo.local',
      role: Role.SECURITY,
    })

    await badgeFactory.create({
      userId: admin.id,
      cardId: Buffer.from('a1b2c3d4', 'hex'),
      derivedKey: Buffer.from('00112233445566778899aabbccddeeff', 'hex'),
      keyA: Buffer.from('aaaaaaaaaaaaaaaaaaaaaaaaaaaaaaaa', 'hex'),
      keyB: Buffer.from('bbbbbbbbbbbbbbbbbbbbbbbbbbbbbbbb', 'hex'),
    })

    await badgeFactory.create({
      userId: security.id,
      cardId: Buffer.from('22334455', 'hex'),
      derivedKey: Buffer.from('11223344556677889900aabbccddeeff', 'hex'),
      keyA: Buffer.from('cccccccccccccccccccccccccccccccc', 'hex'),
      keyB: Buffer.from('dddddddddddddddddddddddddddddddd', 'hex'),
    })

    await badgeFactory.create({
      userId: employee.id,
      cardId: Buffer.from('33445566', 'hex'),
      derivedKey: Buffer.from('ffeeddccbbaa99887766554433221100', 'hex'),
      keyA: Buffer.from('eeeeeeeeeeeeeeeeeeeeeeeeeeeeeeee', 'hex'),
      keyB: Buffer.from('ffffffffffffffffffffffffffffffff', 'hex'),
    })

    await badgeFactory.create({
      userId: test.id,
      cardId: Buffer.from('44556677', 'hex'),
      derivedKey: Buffer.from('1234567890abcdef1234567890abcdef', 'hex'),
      keyA: Buffer.from('abababababababababababababababab', 'hex'),
      keyB: Buffer.from('cdcdcdcdcdcdcdcdcdcdcdcdcdcdcdcd', 'hex'),
    })
  }
}

export default UserSeeder
```

Ce seeder crée 4 utilisateurs fixes avec leurs badges pré-définis. Tous les `cardId` et clés sont constants, assurant une base de tests répétable et fiable. Chaque badge est unique et directement lié à un utilisateur par son `userId`.