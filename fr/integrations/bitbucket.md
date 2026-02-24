# Intégration Bitbucket

JetScale s'intègre avec Bitbucket pour créer automatiquement des pull requests contenant du code Terraform prêt pour la production pour vos recommandations d'optimisation des coûts.

## Vue d'ensemble

L'intégration Bitbucket permet à JetScale de :
- Pousser le code Terraform généré vers vos dépôts Bitbucket
- Créer des pull requests avec des recommandations d'optimisation détaillées
- S'intégrer de manière transparente à vos workflows existants de revue de code et CI/CD
- Suivre le statut d'implémentation des recommandations via les pipelines Bitbucket

## Prérequis

Avant de connecter Bitbucket, vous aurez besoin de :

1. **Compte Bitbucket** (Cloud ou Server)
2. **Accès au dépôt** avec permissions en écriture
3. **Mot de passe d'application** ou **Jeton d'accès personnel** avec les portées appropriées

### Versions Bitbucket prises en charge

- **Bitbucket Cloud** (bitbucket.org) - Recommandé
- **Bitbucket Server** / **Data Center** (auto-hébergé)

## Guide de configuration

### Étape 1 : Créer un mot de passe d'application Bitbucket

![Création de mot de passe d'application Bitbucket](#)
*Emplacement de capture d'écran : paramètres de mot de passe d'application Bitbucket*

#### Pour Bitbucket Cloud :

1. Accédez à [Paramètres Bitbucket → Mots de passe d'application](https://bitbucket.org/account/settings/app-passwords/)
2. Cliquez sur **Créer un mot de passe d'application**
3. Donnez-lui un label descriptif (par exemple, "Intégration JetScale")
4. Sélectionnez les permissions requises :
   - ✓ **Compte : Lecture**
   - ✓ **Appartenance à l'espace de travail : Lecture**
   - ✓ **Projets : Lecture**
   - ✓ **Dépôts : Écriture**
   - ✓ **Pull requests : Écriture**
5. Cliquez sur **Créer**
6. **Important** : Copiez votre mot de passe d'application immédiatement - vous ne le reverrez plus

#### Pour Bitbucket Server :

1. Accédez à **Paramètres du profil → Jetons d'accès personnels**
2. Cliquez sur **Créer un jeton**
3. Donnez-lui un nom (par exemple, "JetScale")
4. Sélectionnez les permissions :
   - ✓ **Lecture du dépôt**
   - ✓ **Écriture du dépôt**
5. Définissez la date d'expiration
6. Cliquez sur **Créer**
7. Copiez votre jeton

> **Note de sécurité** : Stockez votre mot de passe d'application ou jeton de manière sécurisée. JetScale chiffre les identifiants au repos.

### Étape 2 : Connecter Bitbucket dans JetScale

![Écran de connexion Bitbucket](#)
*Emplacement de capture d'écran : page d'intégration Bitbucket de JetScale*

1. Accédez à **Paramètres** → **Intégrations** dans JetScale
2. Cliquez sur **Connecter Bitbucket**
3. Entrez votre configuration :
   - **Type Bitbucket** : Cloud ou Server
   - **URL du serveur** : (Pour Bitbucket Server uniquement) par exemple, `https://bitbucket.votreentreprise.com`
   - **Nom d'utilisateur** : Votre nom d'utilisateur Bitbucket
   - **Mot de passe d'application/Jeton** : L'identifiant créé à l'étape 1
4. Cliquez sur **Vérifier et connecter**

JetScale vérifiera vos identifiants et affichera vos dépôts accessibles.

### Étape 3 : Sélectionner votre dépôt

![Sélection de dépôt](#)
*Emplacement de capture d'écran : menu déroulant de sélection de dépôt Bitbucket*

1. Dans le menu déroulant **Dépôt**, sélectionnez votre dépôt d'infrastructure Terraform
2. Format : `espace-de-travail/slug-depot` (Cloud) ou `projet/depot` (Server)
3. JetScale vérifiera que vous avez un accès en écriture
4. Cliquez sur **Enregistrer**

Le dépôt sélectionné sera utilisé pour toutes les futures pull requests.

## Comment ça fonctionne

### Création de pull request

Lorsque vous approuvez une recommandation d'optimisation des coûts dans JetScale :

1. **Création de branche** : JetScale crée une nouvelle branche de fonctionnalité à partir de votre branche par défaut (généralement `main` ou `master`)
   - Nommage des branches : `jetscale/optimization-{type-ressource}-{horodatage}`
   - Exemple : `jetscale/optimization-rds-20240115-143022`

2. **Push de fichier** : Les fichiers Terraform générés sont commités vers la branche
   - Fichiers placés dans le répertoire `terraform/`
   - Le message de commit inclut le résumé d'optimisation et l'impact sur les coûts

3. **Pull Request** : Une PR est automatiquement créée avec :
   - **Titre** : Description concise de l'optimisation
   - **Description** : Analyse détaillée des coûts, considérations de performance, étapes d'implémentation
   - **Réviseurs** : Ajoutés automatiquement selon les paramètres du dépôt (optionnel)

### Structure de la pull request

```markdown
# Optimisation des coûts : Dimensionnement approprié de l'instance RDS

## Résumé
Rétrograder l'instance RDS `production-db` de **db.r5.2xlarge** à **db.r5.xlarge**

## Impact sur les coûts

| Métrique | Actuel | Projeté | Économies |
|--------|---------|-----------|---------|
| Coût mensuel | $730.00 | $365.00 | **$365.00 (50%)** |
| Coût annuel | $8,760.00 | $4,380.00 | **$4,380.00** |

## Analyse de performance

✓ Utilisation CPU actuelle : moyenne de 15-25%
✓ Utilisation mémoire actuelle : moyenne de 30-40%
✓ L'instance recommandée fournit 2x l'utilisation maximale actuelle
✓ **Évaluation des risques** : Faible - Aucune dégradation de performance attendue

## Fichiers modifiés

- `terraform/rds_production_db.tf` - Mise à jour de la classe d'instance de db.r5.2xlarge à db.r5.xlarge
- `terraform/variables.tf` - Ajout de la variable configurable instance_class

## Liste de vérification d'implémentation

- [ ] Examiner les modifications dans la sortie du plan Terraform
- [ ] Appliquer d'abord à l'environnement de staging
- [ ] Surveiller les performances pendant 24-48 heures
- [ ] Vérifier que les contrôles de santé de l'application réussissent
- [ ] Appliquer à la production avec l'approbation de la gestion du changement

## Recommandations de test

1. Exécuter `terraform plan` pour prévisualiser les modifications
2. Vérifier les ressources dépendantes
3. Vérifier les politiques de sauvegarde/snapshot
4. Confirmer que les tableaux de bord de surveillance sont mis à jour

---
🤖 Généré par [JetScale](https://jetscale.ai) | [Voir dans le tableau de bord JetScale](#)
```

### Exemple de code Terraform

```hcl
# Optimized RDS instance configuration
# Generated by JetScale on 2024-01-15

resource "aws_db_instance" "production_db" {
  identifier     = "production-db"
  engine         = "postgres"
  engine_version = "14.7"

  # Previous: db.r5.2xlarge ($730/month)
  # New: db.r5.xlarge ($365/month)
  # Cost Reduction: 50% ($365/month, $4,380/year)
  #
  # Justification:
  # - Average CPU utilization: 15-25%
  # - Average memory utilization: 30-40%
  # - New instance provides 2x headroom above peak usage
  instance_class = var.rds_production_instance_class

  allocated_storage     = 100
  storage_type          = "gp3"
  storage_encrypted     = true

  # ... other configuration unchanged
}
```

## Permissions du dépôt

JetScale nécessite un accès en **écriture** à votre dépôt pour créer des branches et des pull requests. L'intégration va :

✓ Créer des branches de fonctionnalité
✓ Commiter des fichiers Terraform
✓ Créer des pull requests
✓ Lire les métadonnées et branches du dépôt

✗ Ne peut pas fusionner les pull requests
✗ Ne peut pas supprimer les branches
✗ Ne peut pas modifier les paramètres ou webhooks
✗ Ne peut pas accéder aux secrets du dépôt ou variables de pipeline

## Intégration avec Bitbucket Pipelines

### Validation automatique

Les pull requests JetScale déclenchent vos Bitbucket Pipelines existants :

```yaml
# bitbucket-pipelines.yml example
pipelines:
  pull-requests:
    '**':
      - step:
          name: Validate Terraform
          script:
            - terraform init
            - terraform fmt -check
            - terraform validate
            - terraform plan
```

### Contrôles de statut

JetScale surveille le statut du pipeline :
- ✅ Tous les contrôles réussis : Prêt à fusionner
- ⚠️ Contrôles en attente : En attente du pipeline
- ❌ Contrôles échoués : Examiner les journaux du pipeline

## Gérer l'intégration

### Voir le statut de l'intégration

Depuis **Paramètres** → **Intégrations** → **Bitbucket** :
- Compte Bitbucket connecté
- URL du serveur (si Bitbucket Server)
- Espace de travail/projet et dépôt sélectionnés
- Heure de la dernière vérification
- Pull requests récentes créées

### Changer de dépôt

1. Accédez à **Paramètres** → **Intégrations** → **Bitbucket**
2. Cliquez sur **Changer de dépôt**
3. Sélectionnez un dépôt différent
4. Cliquez sur **Mettre à jour**

### Mettre à jour les identifiants

Si votre mot de passe d'application ou jeton expire :

1. Générez un nouveau mot de passe d'application/jeton dans Bitbucket
2. Accédez à **Paramètres** → **Intégrations** → **Bitbucket**
3. Cliquez sur **Mettre à jour les identifiants**
4. Entrez les nouveaux identifiants
5. Cliquez sur **Enregistrer et vérifier**

### Déconnecter Bitbucket

Pour supprimer l'intégration :

1. Accédez à **Paramètres** → **Intégrations** → **Bitbucket**
2. Cliquez sur **Déconnecter**
3. Confirmez la déconnexion

**Important** : La déconnexion va :
- Supprimer les identifiants stockés de JetScale
- Arrêter la création automatique de PR
- Préserver les pull requests existantes dans Bitbucket

## Dépannage

### Échec d'authentification

**Problème** : Erreur "Identifiants invalides" lors de la connexion

**Solutions** :
- Vérifiez que le nom d'utilisateur est correct (sensible à la casse)
- Vérifiez que le mot de passe d'application/jeton n'a pas expiré
- Assurez-vous que toutes les portées requises sont sélectionnées
- Pour Bitbucket Server, vérifiez que l'URL du serveur est correcte et accessible
- Essayez de créer un nouveau mot de passe d'application

### Dépôt introuvable

**Problème** : Impossible de trouver le dépôt dans le menu déroulant de sélection

**Solutions** :
- Vérifiez que vous avez un accès en écriture au dépôt
- Vérifiez que le dépôt n'est pas archivé
- Assurez-vous que l'appartenance à l'espace de travail/projet est active
- Pour Bitbucket Server, confirmez les permissions du projet
- Essayez d'actualiser la liste des dépôts

### Échec de création de pull request

**Problème** : Recommandation approuvée mais la PR n'a pas été créée

**Solutions** :
- Vérifiez qu'une branche n'existe pas déjà avec le même nom
- Vérifiez que vous avez toujours un accès en écriture
- Assurez-vous que la branche par défaut existe et n'est pas verrouillée
- Confirmez que le dépôt n'est pas en mode lecture seule
- Examinez les limites de taux de l'API Bitbucket

### Le pipeline ne se déclenche pas

**Problème** : Bitbucket Pipelines ne s'exécute pas sur les PR JetScale

**Solutions** :
- Vérifiez que les Pipelines sont activés pour le dépôt
- Vérifiez que `bitbucket-pipelines.yml` inclut les déclencheurs de pull-request
- Assurez-vous que le quota de pipeline n'est pas épuisé
- Examinez la correspondance des modèles de branche dans la configuration du pipeline

## Bonnes pratiques

### Gestion des identifiants

- **Mot de passe d'application dédié** : Créez des identifiants séparés pour JetScale
- **Définir l'expiration** : Utilisez des dates d'expiration raisonnables (90-180 jours)
- **Rotation régulière** : Mettez à jour les identifiants avant expiration
- **Révoquer les inutilisés** : Supprimez les anciens mots de passe d'application immédiatement

### Configuration du dépôt

- **Protection de branche** : Exiger des revues sur la branche par défaut
- **Réviseurs requis** : Configurez CODEOWNERS pour l'attribution automatique des revues
- **Validation du pipeline** : Activez les contrôles de pipeline requis avant la fusion
- **Stratégies de fusion** : Utilisez squash ou no-fast-forward pour un historique propre

### Workflow des pull requests

1. **Validation automatisée** : Laissez les pipelines valider les modifications Terraform
2. **Revue par les pairs** : Faites réviser les optimisations de coûts par l'équipe infrastructure
3. **Déploiement en staging** : Testez les modifications en non-production d'abord
4. **Surveillance** : Surveillez les métriques après avoir appliqué les modifications
5. **Documentation** : Mettez à jour les runbooks si l'infrastructure change

## Bitbucket Cloud vs Server

### Comparaison des fonctionnalités

| Fonctionnalité | Bitbucket Cloud | Bitbucket Server |
|---------|-----------------|------------------|
| Authentification | Mot de passe d'application | Jeton d'accès personnel |
| Format de dépôt | espace-de-travail/slug-depot | projet/depot |
| Version de l'API | REST 2.0 | REST 1.0 |
| Webhooks | Support natif | Support natif |
| Pipelines | Intégré | Plugin requis |

### Différences de configuration

**Bitbucket Cloud** :
- Utilise des permissions basées sur l'espace de travail
- Les mots de passe d'application ont des portées granulaires
- Points de terminaison HTTPS automatiques

**Bitbucket Server** :
- Utilise des permissions basées sur les projets
- Nécessite l'URL complète du serveur
- Peut nécessiter une configuration de pare-feu/VPN

## Considérations de sécurité

### Sécurité des identifiants

- JetScale chiffre les mots de passe d'application et jetons au repos (AES-256)
- Les identifiants n'apparaissent jamais dans les journaux ou réponses API
- Tous les appels API Bitbucket utilisent HTTPS avec TLS 1.2+
- Identifiants stockés séparément avec accès restreint

### Piste d'audit

Toutes les actions JetScale sont journalisées :
- Appels API Bitbucket
- Création de pull request par recommandation
- Tentatives d'authentification échouées
- Événements d'accès au dépôt

Journaux d'audit disponibles dans le tableau de bord JetScale sous **Paramètres** → **Journal d'audit**.

### Révoquer l'accès

Pour révoquer l'accès immédiatement :

**Bitbucket Cloud** :
1. Allez sur [Mots de passe d'application](https://bitbucket.org/account/settings/app-passwords/)
2. Cliquez sur **Révoquer** à côté du mot de passe JetScale

**Bitbucket Server** :
1. Accédez à **Jetons d'accès personnels**
2. Cliquez sur **Révoquer** à côté du jeton JetScale

**JetScale** :
1. Allez dans **Paramètres** → **Intégrations** → **Bitbucket**
2. Cliquez sur **Déconnecter**

## Référence API

Pour un accès programmatique, consultez notre [Documentation API](../api-reference.md#bitbucket-integration).

Points de terminaison clés :
- `POST /api/v1/bitbucket/action/connect` - Connecter le compte Bitbucket
- `POST /api/v1/bitbucket/action/select-repo` - Sélectionner le dépôt
- `GET /api/v1/bitbucket/repos` - Lister les dépôts accessibles
- `POST /api/v1/bitbucket/action/pr/create` - Créer une pull request
- `POST /api/v1/bitbucket/action/disconnect` - Déconnecter Bitbucket

## Support

Besoin d'aide avec l'intégration Bitbucket ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **GitHub Issues** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)
