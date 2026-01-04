## PasswordService – Fiche Technique

### Objectif

Gérer de façon sécurisée le hachage et la vérification des mots de passe en utilisant **Argon2id**, conforme aux recommandations **OWASP 2025**.

---

## Étapes opérationnelles

### ✅ Étape 1 – Définition de la structure utilisateur

#### Structure de l'entité UserEntity

```typescript
import "reflect-metadata";
import { Entity, Column, BeforeInsert, BeforeUpdate } from "typeorm";
import { BaseEntity } from "../base.entity.js";
import Role from "../../enums/roles-enum.js";
import { PasswordService } from "../../services/password.service.js";

@Entity("users")
export class UserEntity extends BaseEntity {
  @Column({ type: "varchar" })
  firstname: string;

  @Column({ type: "varchar" })
  lastname: string;

  @Column({ type: "varchar" })
  username: string;

  @Column({ type: "varchar" })
  password: string;

  @Column({ type: "varchar", unique: true })
  mail: string;

  @Column({ type: "enum", enum: Role })
  role: Role;

  @BeforeInsert()
  @BeforeUpdate()
  private async hashPassword() {
    if (this.password) {
      this.password = await PasswordService.hash(this.password);
    }
  }
}
```

---

### ✅ Étape 2 – Définition des points d’entrée et de sortie

#### Endpoint POST `/badges`

- **Entrée :**
    
    - `uid` (carte scannée)
        
    - `role` (admin, security, employee)
        
- **Traitement :**
    
    - Calcul de la clé dérivée (AES)
        
    - Insertion en base de données
        
- **Sortie :**
    
    - Renvoi de la clé calculée
        
- **Règle métier :**
    
    - Admin et Security sont acceptés
        
    - Employee est refusé
        

#### Jeux de test prévus

- Badge Admin
    
- Badge Security
    
- Badge Employee (refusé)
    

---

## Services

### 🛠️ Installation

```bash
pnpm i argon2
```

### 🏗️ Fonctionnalités de PasswordService

#### 1. Hachage de mot de passe

- Algorithme : **Argon2id**
    
- Exemple :
    
    ```ts
    const hashed = await PasswordService.hash('monMotDePasse');
    ```
    

#### 2. Vérification d’un mot de passe

- Exemple :
    
    ```ts
    const isValid = await PasswordService.verify(hashed, 'monMotDePasse');
    ```
    

### ⚙️ Paramètres par défaut

- Type : Argon2id
    
- Options avancées : non définies pour l'instant
    

### ✅ Pourquoi Argon2id ?

- Recommandé par OWASP
    
- Meilleure protection que bcrypt
    
- Utilisation simple avec [npm argon2](https://www.npmjs.com/package/argon2)
    

### 🧑‍💻 Intégration prévue

- UserFactory : hachage des mots de passe des seeders
    
- Authentification : vérification des identifiants à la connexion
    

### 📌 Bonnes pratiques

- Ne jamais stocker le mot de passe en clair
    
- Toujours utiliser le PasswordService
    
- Mettre à jour les paramètres si OWASP publie de nouvelles recommandations
    

---

## Implémentation du PasswordService

```typescript
import argon2 from 'argon2';

/**
 * Service utilitaire pour le hachage et la vérification des mots de passe
 * utilisant l'algorithme Argon2id.
 */
export class PasswordService {
  /**
   * Génère un hash sécurisé pour un mot de passe donné.
   * @param plainPassword - Le mot de passe en clair à hasher.
   * @returns Le hash Argon2id généré.
   */
  static async hash(plainPassword: string): Promise<string> {
    return argon2.hash(plainPassword, {
      type: argon2.argon2id,
    });
  }

  /**
   * Vérifie si un mot de passe en clair correspond à un hash existant.
   * @param hashedPassword - Le hash stocké en base.
   * @param plainPassword - Le mot de passe à vérifier.
   * @returns true si le mot de passe correspond, false sinon.
   */
  static async verify(hashedPassword: string, plainPassword: string): Promise<boolean> {
    return argon2.verify(hashedPassword, plainPassword);
  }
}
```

---

## 📑 Notes d’utilisation pour les seeders

### Objectif

Permettre la création d'utilisateurs de test en fournissant des mots de passe **en clair** dans les seeders tout en garantissant qu'ils sont **automatiquement hachés** avant d'être enregistrés en base de données.

### Fonctionnement

- Le champ `password` est saisi **en clair** dans le seeder (par exemple : `password: 'password'`).
    
- L'écouteur `@BeforeInsert` et `@BeforeUpdate` de l'entité `UserEntity` utilise **PasswordService** pour **hasher automatiquement** ce mot de passe avant l'insertion en base.
    
- Cela permet d'assurer une **cohérence de sécurité** tout en facilitant l'écriture de seeders pour les environnements de développement.
    

### Exemple dans un seeder

```typescript
await UserFactory.make({
  firstname: 'Alice',
  lastname: 'Durand',
  username: 'alice.d',
  mail: 'alice@demo.local',
  role: Role.ADMIN,
  password: 'password', // En clair ici, hashé automatiquement par l'entité
});
```

### Bonnes pratiques

- **Utiliser des mots de passe simples uniquement pour les environnements de test ou de développement.**
    
- **Ne jamais réutiliser ces mots de passe en production.**
    
- **Toujours s'assurer que le hachage est effectif avant d’utiliser les données en production.**

## UserSeeder – Définition des personas de test

### Objectif

Créer trois utilisateurs représentatifs des rôles supportés par le système : `admin`, `security`, `employee`.

### Implémentation

```
import { Seeder } from '@jorgebodega/typeorm-seeding';
import UserFactory from '../factories/user.factory.js';
import Role from '../../enums/roles-enum.js';

class UserSeeder extends Seeder {
  async run() {
    await UserFactory.make({
      firstname: 'Alice',
      lastname: 'Durand',
      username: 'alice.d',
      mail: 'alice@demo.local',
      role: Role.ADMIN,
      password: 'password',
    });

    await UserFactory.make({
      firstname: 'Bruno',
      lastname: 'Martin',
      username: 'bruno.m',
      mail: 'bruno@demo.local',
      role: Role.SECURITY,
      password: 'password',
    });

    await UserFactory.make({
      firstname: 'Chloé',
      lastname: 'Bernard',
      username: 'chloe.b',
      mail: 'chloe@demo.local',
      role: Role.EMPLOYEE,
      password: 'password',
    });
  }
}

export default UserSeeder;
```

### Résumé des personas

|Prénom|Nom|Email|Rôle|Mot de passe|
|---|---|---|---|---|
|Alice|Durand|alice@demo.local|Admin|password|
|Bruno|Martin|bruno@demo.local|Security|password|
|Chloé|Bernard|chloe@demo.local|Employee|password|

### Bonnes pratiques

- Utiliser uniquement dans les environnements de développement.
    
- Supprimer ou désactiver ces utilisateurs avant passage en production.
    
- Ces utilisateurs permettent de tester les accès et les comportements métiers liés aux rôles définis.