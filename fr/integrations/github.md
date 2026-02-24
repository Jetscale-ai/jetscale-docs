# Intégration GitHub

JetScale s'intègre avec GitHub pour créer automatiquement des pull requests contenant du code Terraform prêt pour la production pour vos recommandations d'optimisation des coûts.

## Aperçu

L'intégration GitHub permet à JetScale de :
- Pousser du code Terraform généré vers vos dépôts
- Créer des pull requests avec des recommandations d'optimisation détaillées
- S'intégrer de manière transparente avec votre flux de travail de révision de code existant
- Suivre l'état de mise en œuvre des recommandations

## Prérequis

Avant de connecter GitHub, vous aurez besoin de :

1. **Compte GitHub** avec accès à votre dépôt d'infrastructure
2. **Accès au dépôt** avec permissions de push
3. **Personal Access Token** (PAT) avec les portées suivantes :
   - `repo` - Contrôle total des dépôts privés
   - `read:org` - Lire l'appartenance aux organisations et équipes
   - `user:email` - Accéder aux adresses email des utilisateurs

## Guide de configuration

### Étape 1 : Créer un Personal Access Token GitHub

1. Accédez à [GitHub Settings → Developer settings → Personal access tokens](https://github.com/settings/tokens)
2. Cliquez sur **Generate new token** → **Generate new token (classic)**
3. Donnez à votre jeton un nom descriptif (ex : "JetScale Integration")
4. Sélectionnez les portées requises :
   - ✓ `repo` (Contrôle total des dépôts privés)
   - ✓ `read:org` (Lire l'appartenance aux organisations et équipes)
   - ✓ `user:email` (Accéder aux adresses email des utilisateurs)
5. Cliquez sur **Generate token**
6. **Important** : Copiez votre jeton immédiatement - vous ne pourrez plus le voir

> **Note de sécurité** : Stockez votre jeton en toute sécurité. JetScale chiffre les jetons au repos et ne les enregistre ou ne les expose jamais.

### Étape 2 : Connecter GitHub dans JetScale

1. Accédez à **Paramètres** → **Intégrations** dans JetScale
2. Cliquez sur **Connect GitHub**
3. Collez votre Personal Access Token
4. Cliquez sur **Verify and Connect**

JetScale vérifiera votre jeton et affichera les informations de votre compte GitHub.

### Étape 3 : Sélectionner votre dépôt

1. Dans le menu déroulant **Repository**, sélectionnez votre dépôt d'infrastructure Terraform
2. JetScale vérifiera que vous avez un accès en push
3. Le dépôt sélectionné sera utilisé pour toutes les futures pull requests

Vous pouvez changer le dépôt sélectionné à tout moment depuis les paramètres d'intégration.

## Fonctionnement

### Création de pull request

Lorsque vous approuvez une recommandation d'optimisation des coûts dans JetScale :

1. **Création de branche** : JetScale crée une nouvelle branche de fonctionnalité depuis votre branche par défaut
   - Format de nommage de branche : `jetscale/optimization-{resource-type}-{date}`
   - Exemple : `jetscale/optimization-rds-2024-01-15`

2. **Push de fichiers** : Les fichiers Terraform générés sont poussés vers le répertoire `terraform/`
   - Les fichiers incluent toutes les définitions de ressources nécessaires
   - Les commentaires expliquent la justification de l'optimisation
   - Les variables sont extraites lorsque approprié

3. **Pull Request** : Une PR est automatiquement créée avec :
   - **Titre** : Résumé descriptif de l'optimisation
   - **Description** : Analyse détaillée de l'impact sur les coûts, considérations de performance et notes de mise en œuvre
   - **Labels** : Automatiquement tagué avec `jetscale` et le type de ressource

### Structure de la pull request

```markdown
## Optimisation des coûts : Redimensionnement d'instance RDS

### Résumé
Réduire l'instance RDS `production-db` de db.r5.2xlarge à db.r5.xlarge

### Impact sur les coûts
- **Coût mensuel actuel** : $730.00
- **Coût mensuel projeté** : $365.00
- **Économies mensuelles** : $365.00 (réduction de 50%)
- **Économies annuelles** : $4,380.00

### Analyse des performances
- Utilisation CPU actuelle : 15-25% en moyenne
- Utilisation mémoire actuelle : 30-40% en moyenne
- L'instance recommandée offre 2x l'utilisation de pointe actuelle
- Aucune dégradation de performance attendue

### Fichiers modifiés
- `terraform/rds_production_db.tf` - Classe d'instance mise à jour
- `terraform/variables.tf` - Variable instance_class ajoutée

### Recommandations de test
1. Appliquer les modifications d'abord dans l'environnement de staging
2. Surveiller les métriques de performance pendant 48 heures
3. Vérifier que les temps de réponse de l'application restent cohérents
4. Vérifier que les alarmes CloudWatch sont correctement configurées

---
🤖 Généré par [JetScale](https://jetscale.ai)
```

### Exemple de code Terraform

```hcl
resource "aws_db_instance" "production_db" {
  identifier     = "production-db"
  engine         = "postgres"
  engine_version = "14.7"

  # Optimisé de db.r5.2xlarge à db.r5.xlarge
  # Réduction des coûts : $365/mois (50%)
  # Basé sur 15-25% CPU moyen, 30-40% utilisation mémoire moyenne
  instance_class = "db.r5.xlarge"

  allocated_storage     = 100
  storage_type          = "gp3"
  storage_encrypted     = true

  # ... autre configuration inchangée
}
```

## Permissions de dépôt

JetScale nécessite un accès **push** à votre dépôt pour créer des branches et des pull requests. L'intégration va :

✓ Créer des branches de fonctionnalité
✓ Pousser des fichiers Terraform
✓ Créer des pull requests
✓ Lire les métadonnées du dépôt

✗ Ne peut pas fusionner les pull requests
✗ Ne peut pas supprimer les branches
✗ Ne peut pas modifier les fichiers existants en dehors des PR créées
✗ Ne peut pas accéder aux secrets du dépôt

## Gestion de l'intégration

### Voir l'état de l'intégration

Depuis **Paramètres** → **Intégrations** → **GitHub**, vous pouvez voir :
- Compte GitHub connecté
- Dépôt sélectionné
- Heure de dernière vérification
- Pull requests récentes créées par JetScale

### Changer de dépôt

1. Accédez à **Paramètres** → **Intégrations** → **GitHub**
2. Cliquez sur **Change Repository**
3. Sélectionnez un dépôt différent dans le menu déroulant
4. Cliquez sur **Update**

### Déconnecter GitHub

Pour supprimer l'intégration GitHub :

1. Accédez à **Paramètres** → **Intégrations** → **GitHub**
2. Cliquez sur **Disconnect**
3. Confirmez la déconnexion

**Important** : La déconnexion va :
- Supprimer votre jeton d'accès stocké de JetScale
- Arrêter la création automatique de PR pour les nouvelles recommandations
- Préserver les pull requests existantes (elles restent ouvertes dans GitHub)

Vous pouvez vous reconnecter à tout moment en utilisant un Personal Access Token nouveau ou existant.

## Dépannage

### Échec de vérification du jeton

**Problème** : Erreur "Invalid GitHub Token" lors de la connexion

**Solutions** :
- Vérifiez que toutes les portées requises sont sélectionnées (`repo`, `read:org`, `user:email`)
- Vérifiez que le jeton n'a pas expiré
- Assurez-vous d'avoir copié le jeton complet sans espaces supplémentaires
- Essayez de générer un nouveau jeton avec les portées correctes

### Dépôt non listé

**Problème** : Impossible de trouver votre dépôt dans le menu déroulant de sélection

**Solutions** :
- Vérifiez que vous avez un accès en push au dépôt
- Vérifiez si le dépôt appartient à une organisation - assurez-vous que votre jeton a la portée `read:org`
- Essayez de vous reconnecter avec un nouveau jeton
- Confirmez que le dépôt existe et n'est pas archivé

### Échec de création de pull request

**Problème** : Recommandation approuvée mais la PR n'a pas été créée

**Solutions** :
- Vérifiez qu'une branche n'existe pas déjà avec le même nom
- Vérifiez que vous avez toujours un accès en push au dépôt
- Assurez-vous que la branche par défaut n'a pas été supprimée ou renommée
- Consultez les [limites de taux](https://docs.github.com/en/rest/overview/resources-in-the-rest-api#rate-limiting) de GitHub - vous devrez peut-être attendre brièvement

### La branche existe déjà

**Problème** : Erreur "Branch already exists" lors de la création de PR

**Solutions** :
- Supprimez ou renommez la branche existante dans GitHub
- Réessayez la recommandation d'optimisation - JetScale générera un nouveau nom de branche avec horodatage
- Fusionnez ou fermez la PR existante si elle provient d'une recommandation précédente

## Bonnes pratiques

### Gestion des jetons

- **Créez un jeton dédié** pour JetScale au lieu de réutiliser des jetons
- **Définissez une date d'expiration** et renouvelez avant l'expiration
- **Utilisez des permissions granulaires** quand elles seront disponibles pour votre organisation
- **Alternez les jetons régulièrement** (tous les 90 jours recommandé)

### Configuration du dépôt

- **Utilisez un dépôt dédié** pour l'infrastructure Terraform
- **Activez la protection de branche** sur votre branche par défaut pour exiger des révisions
- **Configurez CODEOWNERS** pour demander automatiquement des révisions de votre équipe
- **Configurez des vérifications CI/CD** pour valider les modifications Terraform avant la fusion

### Flux de travail des pull requests

1. **Examinez la description de la PR** pour l'impact sur les coûts et l'analyse des performances
2. **Testez en staging** avant d'appliquer en production
3. **Surveillez les métriques** après avoir appliqué les modifications
4. **Fermez les PR que vous n'implémenterez pas** plutôt que de les laisser ouvertes indéfiniment

## Considérations de sécurité

### Sécurité des jetons

- JetScale chiffre les Personal Access Tokens au repos avec AES-256
- Les jetons ne sont jamais enregistrés ou exposés dans les réponses API
- Tous les appels API vers GitHub utilisent HTTPS avec TLS 1.2+
- Les jetons sont stockés séparément des données utilisateur avec un accès restreint

### Piste d'audit

Chaque action effectuée par JetScale est enregistrée :
- Tous les appels API GitHub sont enregistrés
- La création de pull request est suivie par recommandation
- Les tentatives échouées sont capturées avec les détails d'erreur
- Les journaux d'audit sont disponibles dans votre tableau de bord JetScale

### Révocation de l'accès

Pour révoquer immédiatement l'accès de JetScale :

1. **Dans GitHub** : Accédez à [Settings → Applications → Personal access tokens](https://github.com/settings/tokens)
2. Cliquez sur **Delete** à côté du jeton JetScale
3. **Dans JetScale** : Accédez à Paramètres → Intégrations → GitHub et cliquez sur **Disconnect**

Une fois révoqué, JetScale perd immédiatement tout accès à vos dépôts.

## Référence API

Pour l'accès programmatique à l'intégration GitHub, consultez notre [Documentation API](../api-reference.md#github-integration).

Points de terminaison clés :
- `POST /api/v1/github/action/connect` - Connecter un compte GitHub
- `POST /api/v1/github/action/select-repo` - Sélectionner un dépôt
- `GET /api/v1/github/repos` - Lister les dépôts accessibles
- `POST /api/v1/github/action/pr/create` - Créer une pull request
- `POST /api/v1/github/action/disconnect` - Déconnecter GitHub

## Support

Besoin d'aide avec l'intégration GitHub ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Problèmes GitHub** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)
