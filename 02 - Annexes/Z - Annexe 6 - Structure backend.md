
# Architecture : Différence entre Route, Service et Repository

Dans une architecture backend bien structurée (comme avec Node.js, Fastify, TypeScript, etc.), il est important de bien séparer les responsabilités entre les différentes couches de l’application : **Route**, **Service**, et **Repository**.

---

## 1. Rôles des différentes couches

|Couche|Rôle principal|Contenu typique|
|---|---|---|
|**Route**|Interface HTTP. Reçoit la requête, valide, appelle le service|Parsing de `req`, validation (`zod`), `reply.send()`|
|**Service**|Logique métier. Intermédiaire entre route et repo|Orchestration, logique conditionnelle, appels aux repos|
|**Repository**|Accès brut à la base de données (CRUD)|`findOneBy`, `save`, `create`, requêtes personnalisées|

---

## 2. Schéma de fonctionnement logique

```
[ Client HTTP ]
       ↓
[ Route / Controller ]
       ↓
[ Service (logique métier) ]
       ↓
[ Repository (base de données) ]
       ↓
[ Base de données réelle ]
```

---

## 3. Exemple simplifié

### Route (`routes/badge.routes.ts`)

```ts
app.post('/badges/add', async (req, res) => {
  const { userId } = req.body;
  const badgeService = new BadgesService();
  const result = await badgeService.add(userId);
  res.code(result.status).send(result);
});
```

### Service (`services/badges.service.ts`)

```ts
export class BadgesService {
  async add(userId: string) {
    const cardId = await this.prepareCard();
    const keys = deriveKeysAB(cardId);

    const badge = await this.badgeRepo.findByCardId(cardId);
    if (badge) throw new Error("Card already exists");

    return this.badgeRepo.createBadge({ cardId, userId, ...keys });
  }
}
```

### Repository (`repositories/badges.repository.ts`)

```ts
export class BadgesRepository {
  async findByCardId(cardId: Buffer) {
    return badgeRepo.findOneBy({ cardId });
  }

  async createBadge(data: Partial<BadgeEntity>) {
    const badge = badgeRepo.create(data);
    return badgeRepo.save(badge);
  }
}
```

---

## 4. Pourquoi respecter cette structure ?

- ✅ **Lisibilité** : chaque fichier a un rôle clair.
    
- ✅ **Testabilité** : les services sont testables sans serveur ni base.
    
- ✅ **Évolutivité** : la logique métier peut changer sans impacter les routes ou la BDD.
    

---

## 5. Faut-il toujours suivre cette structure ?

Non, ce n’est pas rigide. Quelques exceptions peuvent apparaître :

- Un **script** ou une **cron** peut appeler directement un repository si la logique est simple.
    
- Un service peut utiliser plusieurs repositories.
    

Mais **dans un contexte HTTP/API, il est fortement recommandé de garder cette séparation nette.**

---

## 6. Conclusion

Le service agit comme un **middleware logique** entre la route et le repository. C’est lui qui garantit que la logique métier est centralisée, réutilisable et testable. Cela permet d’éviter des duplications, de clarifier le code et de faciliter les tests unitaires.+