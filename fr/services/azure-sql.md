# Optimisation Azure SQL

JetScale fournit une optimisation des coûts pilotée par IA pour Azure SQL Database, incluant les bases de données uniques, les pools élastiques et Azure SQL Managed Instance. Nos agents spécialisés analysent vos charges de travail de bases de données pour identifier les opportunités de redimensionnement et d'amélioration de configuration.

## Vue d'ensemble

JetScale optimise les ressources Azure SQL en analysant :
- **Utilisation des bases de données** : Utilisation DTU/vCore, CPU, mémoire et modèles d'E/S
- **Analyse des coûts** : Dépenses actuelles vs configuration optimale
- **Métriques de performance** : Performance des requêtes, modèles de connexion, utilisation du stockage
- **Alignement des niveaux de service** : Basic, Standard, Premium, Business Critical, Hyperscale

## Types Azure SQL pris en charge

### Azure SQL Database (Base de données unique)

Les bases de données uniques sont notre cible d'optimisation principale.

**Architecture de base de données :**
```
SqlDatabase (Cible d'optimisation, Facturable)
├── Calcul (DTU ou vCore)
├── Stockage (Données + Journal)
└── Stockage de sauvegarde
```

**Ce que nous optimisons :**
- **Niveau de service** : Redimensionnement entre Basic, Standard, Premium, Business Critical, Hyperscale
- **Niveau de calcul** : Provisionné vs Serverless
- **Dimensionnement DTU/vCore** : Correspondance avec les exigences réelles de charge de travail
- **Configuration du stockage** : Taille maximale des données et niveau de performance

### Azure SQL Elastic Pools

Les pools élastiques partagent les ressources entre plusieurs bases de données.

**Architecture du pool :**
```
SqlElasticPool (Cible d'optimisation, Facturable)
├── Base de données 1 - Partage les ressources du pool
├── Base de données 2 - Partage les ressources du pool
└── Base de données N - Partage les ressources du pool
```

**Ce que nous optimisons :**
- **Dimensionnement du pool** : Total de DTU/vCores pour toutes les bases de données
- **Niveau de service** : Standard, Premium, Business Critical
- **Densité des bases de données** : Nombre optimal de bases de données par pool

### Azure SQL Managed Instance

Instance SQL Server entièrement gérée.

**Architecture de l'instance :**
```
SqlManagedInstance (Cible d'optimisation, Facturable)
├── Calcul (vCores)
├── Stockage
└── Plusieurs bases de données
```

**Ce que nous optimisons :**
- **Dimensionnement de l'instance** : Nombre de vCores (4, 8, 16, 24, 32, 40, 64, 80)
- **Niveau de service** : General Purpose vs Business Critical
- **Configuration du stockage** : Dimensionnement du stockage des données et du journal

## Niveaux de service

### Modèle basé sur DTU (Base de données unique & Pools élastiques)

**Niveau Basic**
- **Utiliser pour** : Petites bases de données, développement, tests, charges de travail légères
- **Plage DTU** : 5 DTU
- **Taille maximale de base de données** : 2 Go
- **Coût typique** : ~5 $/mois
- **Idéal quand** : Apprentissage, prototypage, utilisation minimale en production

**Niveau Standard**
- **Utiliser pour** : La plupart des charges de travail de production, applications web, applications métier
- **Plage DTU** : 10, 20, 50, 100, 200, 400, 800, 1600, 3000 DTU
- **Taille maximale de base de données** : Jusqu'à 1 To
- **Coût typique** : 15-1 500 $/mois
- **Idéal quand** : Exigences de performance prévisibles, charges de travail équilibrées

**Niveau Premium**
- **Utiliser pour** : Applications critiques, taux de transaction élevé, faible latence
- **Plage DTU** : 125, 250, 500, 1000, 1750, 4000 DTU
- **Taille maximale de base de données** : Jusqu'à 4 To
- **Coût typique** : 465-14 500 $/mois
- **Idéal quand** : Performance, disponibilité et exigences de récupération les plus élevées

### Modèle basé sur vCore (Tous types)

**General Purpose**
- **Utiliser pour** : La plupart des charges de travail de production, calcul et E/S équilibrés
- **vCores** : 2, 4, 6, 8, 10, 12, 14, 16, 18, 20, 24, 32, 40, 80, 128
- **Mémoire maximale** : 625 Go (128 vCore)
- **Stockage** : Stockage distant (SSD), jusqu'à 4 To (unique) ou 16 To (instance gérée)
- **Coût typique** : 300-8 000 $/mois
- **Idéal quand** : Charges de travail métier standard

**Business Critical**
- **Utiliser pour** : Applications critiques, faible latence, haute disponibilité
- **vCores** : Même plage que General Purpose
- **Mémoire maximale** : 625 Go (128 vCore)
- **Stockage** : SSD local, jusqu'à 4 To (unique) ou 16 To (instance gérée)
- **HA intégrée** : Groupe de disponibilité Always-On avec réplicas secondaires lisibles
- **Coût typique** : 2,5x le prix de General Purpose
- **Idéal quand** : Performance et disponibilité les plus élevées requises

**Hyperscale** (Base de données unique uniquement)
- **Utiliser pour** : Très grandes bases de données (100 To+), montée/descente en charge rapide
- **vCores** : 2-80 vCores
- **Stockage** : Hautement évolutif, jusqu'à 100 To
- **Architecture** : Multi-niveaux avec serveurs de pages et nœuds de calcul
- **Coût typique** : Similaire à General Purpose, stockage facturé séparément
- **Idéal quand** : Base de données > 4 To, besoin de mise à l'échelle rapide

### Niveaux de calcul

**Provisionné (Provisioned)**
- **Facturation** : Par seconde pour les ressources de calcul
- **Idéal pour** : Charges de travail prévisibles, utilisation continue
- **Minimum** : Aucun calcul minimum

**Serverless** (General Purpose uniquement)
- **Facturation** : Par seconde quand actif, pause automatique en cas d'inactivité
- **Idéal pour** : Modèles d'utilisation intermittents et imprévisibles
- **Pause automatique** : Délai configurable (1 heure - 7 jours)
- **Économies** : Jusqu'à 90% pendant les périodes d'inactivité
- **Note** : Délai de démarrage à froid lors de la reprise depuis l'état en pause

## Comment JetScale optimise Azure SQL

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques Azure Monitor** (fenêtre glissante de 14 jours) :
- `cpu_percent` - Utilisation du CPU
- `dtu_consumption_percent` - Utilisation DTU (modèle DTU)
- `storage_percent` - Utilisation du stockage
- `connection_successful` / `connection_failed` - Modèles de connexion
- `sessions_percent` - Sessions actives
- `workers_percent` - Utilisation des threads de travail
- `deadlock` - Occurrences de blocage
- `log_write_percent` - Utilisation d'écriture du journal de transactions

**Données Azure Resource Manager** :
- Niveau de service et niveau de calcul
- Configuration DTU/vCore
- Taille de stockage actuelle et taille maximale
- État de la géo-réplication
- Paramètres de rétention de sauvegarde

**Données Cost Management** :
- Dépense mensuelle actuelle par base de données
- Ventilation des coûts de calcul, stockage et sauvegarde
- Tendances historiques des coûts

**Azure Advisor** (si disponible) :
- Recommandations de redimensionnement générées par Azure
- Suggestions d'optimisation des performances

### 2. Analyse

Nos agents IA effectuent une analyse approfondie :

**Modèles d'utilisation :**
- Utilisation DTU/vCore de pointe vs moyenne
- Modèles horaires (identifier les périodes d'inactivité pour serverless)
- Modèles jour de la semaine (charge week-end vs jour de semaine)
- Tendances de croissance au fil du temps

**Évaluation des performances :**
- Métriques de performance des requêtes
- Modèles de connexion et échecs
- Fréquence des blocages
- Pression du journal de transactions

**Modélisation des coûts :**
- Ventilation des coûts actuels (calcul, stockage, sauvegarde)
- Coût projeté pour les configurations alternatives
- Comparaison des coûts Provisionné vs Serverless
- Opportunités de capacité réservée

**Sélection du niveau :**
- Alignement Basic vs Standard vs Premium
- Exigences General Purpose vs Business Critical
- Pertinence d'Hyperscale pour les grandes bases de données
- Efficacité du modèle DTU vs vCore

### 3. Recommandations

JetScale génère des recommandations spécifiques et exploitables :

#### Réduction du niveau de service

**Exemple de recommandation :**
```
Ressource : app-database-prod
Actuel : Premium P2 (250 DTU, 930 $/mois)
Recommandé : Standard S3 (100 DTU, 300 $/mois)

Impact sur les coûts :
- Actuel : 930 $/mois
- Projeté : 300 $/mois
- Économies : 630 $/mois (68%), 7 560 $/an

Analyse de performance :
- Utilisation DTU moyenne : 35%
- Utilisation DTU de pointe : 58%
- Modèles de connexion : Normal
- Performance des requêtes : Bien dans les capacités du niveau Standard

Risque : Faible - Utilisation de pointe bien en dessous des limites du nouveau niveau
Note : Fonctionnalités du niveau Premium (latence plus faible, plus d'IOPS) pas pleinement utilisées
```

#### Redimensionnement DTU

**Exemple de recommandation :**
```
Ressource : customer-db-01
Actuel : Standard S7 (800 DTU, 600 $/mois)
Recommandé : Standard S3 (100 DTU, 300 $/mois)

Impact sur les coûts :
- Actuel : 600 $/mois
- Projeté : 300 $/mois
- Économies : 300 $/mois (50%), 3 600 $/an

Analyse de performance :
- Utilisation DTU moyenne : 18%
- Utilisation DTU de pointe : 42%
- 95e percentile : 35%
- Le niveau recommandé fournit une marge de 2,4x au-dessus de la pointe

Risque : Très faible - Marge substantielle maintenue
```

#### Migration Serverless

**Exemple de recommandation :**
```
Ressource : reporting-database
Actuel : General Purpose (4 vCores, Provisionné, 730 $/mois)
Recommandé : General Purpose (4 vCores, Serverless, ~290 $/mois)

Impact sur les coûts :
- Actuel : 730 $/mois (facturation 24/7)
- Projeté : 290 $/mois (temps actif moyen 40%)
- Économies : 440 $/mois (60%), 5 280 $/an

Analyse d'utilisation :
- Heures actives : ~10 heures/jour (heures ouvrables uniquement)
- Temps d'inactivité : Nuits et week-ends (60% du temps)
- Modèles de requêtes : Traitement par lots et rapports
- Tolérance au démarrage à froid : Acceptable (cas d'usage de rapports)

Configuration :
- Délai de pause automatique : 1 heure
- vCores min : 1 (quand actif)
- vCores max : 4 (pendant la pointe)

Risque : Très faible - Modèle d'utilisation intermittent idéal pour serverless
Note : Temps de reprise d'environ 2 minutes depuis l'état en pause
```

#### Optimisation vCore

**Exemple de recommandation :**
```
Ressource : api-database-prod
Actuel : General Purpose (16 vCores, 1 460 $/mois)
Recommandé : General Purpose (8 vCores, 730 $/mois)

Impact sur les coûts :
- Actuel : 1 460 $/mois
- Projeté : 730 $/mois
- Économies : 730 $/mois (50%), 8 760 $/an

Analyse de performance :
- CPU moyen : 22%
- CPU de pointe : 38%
- Utilisation de la mémoire : 45%
- Modèles d'E/S : Bien dans les limites de General Purpose

Risque : Faible - Marge de 2,6x au-dessus de l'utilisation CPU de pointe
```

#### Business Critical → General Purpose

**Exemple de recommandation :**
```
Ressource : legacy-app-database
Actuel : Business Critical (8 vCores, 3 650 $/mois)
Recommandé : General Purpose (8 vCores, 1 460 $/mois)

Impact sur les coûts :
- Actuel : 3 650 $/mois
- Projeté : 1 460 $/mois
- Économies : 2 190 $/mois (60%), 26 280 $/an

Analyse de charge de travail :
- Fonctionnalités Business Critical non utilisées :
  × Aucun réplica secondaire lisible utilisé
  × Exigences de faible latence non critiques
  × Performance SSD local non requise
- Exigences de performance satisfaites par General Purpose

Risque : Moyen - Vérifier que l'application tolère la latence du stockage distant
Recommandation : Tester d'abord dans l'environnement de staging
```

#### Consolidation Elastic Pool

**Exemple de recommandation :**
```
Ressource : 12 bases de données Standard S3
Actuel : 12 × S3 (100 DTU chacune, 3 600 $/mois total)
Recommandé : 1 × Elastic Pool (600 eDTU, 900 $/mois)

Impact sur les coûts :
- Actuel : 3 600 $/mois (12 bases de données individuelles)
- Projeté : 900 $/mois (pool élastique avec 12 bases de données)
- Économies : 2 700 $/mois (75%), 32 400 $/an

Analyse d'utilisation :
- Utilisation de pointe des bases de données individuelles : 40-60 DTU
- Utilisation de pointe simultanée : Rare (charges de travail décalées dans le temps)
- Utilisation DTU simultanée totale : ~300 DTU de pointe
- Le pool fournit 600 eDTU avec partage de ressources

Risque : Faible - Capacité de pool suffisante pour l'utilisation simultanée de pointe
Note : Les bases de données partagent les ressources du pool, réduisant le sur-provisionnement
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Optimisation du niveau de service**

```hcl
# Optimisation Azure SQL Database
# Généré par JetScale le 2024-01-15
# ID de recommandation : rec_azuresql_001

resource "azurerm_mssql_database" "app_database_prod" {
  name      = "app-database-prod"
  server_id = azurerm_mssql_server.example.id

  # Précédent : Premium P2 (250 DTU, 930 $/mois)
  # Optimisé : Standard S3 (100 DTU, 300 $/mois)
  # Réduction de coût : 68% (630 $/mois, 7 560 $/an)
  #
  # Justification :
  # - Utilisation DTU moyenne : 35%
  # - Utilisation DTU de pointe : 58%
  # - Fonctionnalités du niveau Premium (latence plus faible, IOPS plus élevés) pas pleinement utilisées
  # - Standard S3 fournit des performances adéquates pour la charge de travail
  sku_name = "S3"

  max_size_gb = 250

  # Configuration existante préservée
  collation               = "SQL_Latin1_General_CP1_CI_AS"
  zone_redundant          = false
  read_scale              = false
  storage_account_type    = "Geo"

  threat_detection_policy {
    state                      = "Enabled"
    email_account_admins       = "Enabled"
    retention_days             = 30
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:recommendation" = "rec_azuresql_001"
      "jetscale:previous_sku"   = "P2"
    }
  )
}
```

**Exemple : Migration Serverless**

```hcl
# Migration Azure SQL Serverless
# Généré par JetScale le 2024-01-15

resource "azurerm_mssql_database" "reporting_database" {
  name      = "reporting-database"
  server_id = azurerm_mssql_server.example.id

  # Précédent : Provisionné (4 vCores, 24/7, 730 $/mois)
  # Optimisé : Serverless (1-4 vCores, ~40% actif, 290 $/mois)
  # Réduction de coût : 60% (440 $/mois, 5 280 $/an)
  #
  # Analyse du modèle d'utilisation :
  # - Actif : Heures ouvrables uniquement (~10 h/jour)
  # - Inactif : Nuits et week-ends (60% du temps)
  # - Cas d'usage : Rapports par lots (démarrage à froid acceptable)
  #
  # Configuration Serverless :
  # - Pause automatique après 1 heure d'inactivité
  # - Min 1 vCore quand actif, max 4 vCores
  # - Temps de reprise d'environ 2 minutes depuis l'état en pause
  sku_name = "GP_S_Gen5_4"

  min_capacity = 1.0
  max_capacity = 4.0

  # Pause automatique après 1 heure (60 minutes) d'inactivité
  auto_pause_delay_in_minutes = 60

  max_size_gb = 100

  collation               = "SQL_Latin1_General_CP1_CI_AS"
  zone_redundant          = false
  read_scale              = false
  storage_account_type    = "Geo"

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:compute_model" = "serverless"
      "jetscale:previous_sku"  = "GP_Gen5_4"
    }
  )
}
```

**Exemple : Elastic Pool**

```hcl
# Consolidation Azure SQL Elastic Pool
# Généré par JetScale le 2024-01-15

resource "azurerm_mssql_elasticpool" "app_pool" {
  name                = "app-elastic-pool"
  resource_group_name = var.resource_group_name
  location            = var.location
  server_name         = azurerm_mssql_server.example.name

  # Précédent : 12 bases de données S3 individuelles (1200 DTU total, 3 600 $/mois)
  # Optimisé : Pool élastique (600 eDTU partagés, 900 $/mois)
  # Réduction de coût : 75% (2 700 $/mois, 32 400 $/an)
  #
  # Analyse :
  # - Les pics des bases de données individuelles coïncident rarement
  # - Utilisation simultanée de pointe : ~300 DTU
  # - Le pool fournit 600 eDTU avec une marge de 2x
  # - Le partage des ressources élimine le sur-provisionnement
  sku {
    name     = "StandardPool"
    tier     = "Standard"
    capacity = 600
  }

  per_database_settings {
    min_capacity = 10
    max_capacity = 100
  }

  max_size_gb = 750

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"         = "true"
      "jetscale:pool_type"         = "consolidation"
      "jetscale:database_count"    = "12"
      "jetscale:previous_total_cost" = "$3600"
    }
  )
}

# Déplacer les bases de données vers le pool élastique
resource "azurerm_mssql_database" "pooled_databases" {
  count = 12

  name           = "app-database-${count.index + 1}"
  server_id      = azurerm_mssql_server.example.id
  elastic_pool_id = azurerm_mssql_elasticpool.app_pool.id

  # Les bases de données partagent maintenant les ressources du pool
  # Aucun sku_name individuel nécessaire

  tags = var.tags
}
```

## Meilleures pratiques

### Surveillance après les changements

Après avoir appliqué les recommandations JetScale :

1. **Premières 24 heures** : Surveiller étroitement les métriques clés
   - Utilisation CPU et DTU/vCore
   - Taux de réussite des connexions
   - Performance des requêtes (temps d'exécution)
   - Utilisation du journal de transactions

2. **Semaine 1** : Valider les performances
   - Comparer les temps d'exécution des requêtes
   - Vérifier les indicateurs de pression sur les ressources
   - Surveiller les échecs de connexion
   - Examiner les taux d'erreur de l'application

3. **Semaines 2-4** : Confirmer les économies de coûts
   - Vérifier que la facturation Azure reflète les économies attendues
   - Vérifier l'absence de frais inattendus
   - Envisager la capacité réservée pour les charges de travail stables

### Stratégie de test

**Tests en pré-production :**
1. Appliquer d'abord les changements à l'environnement dev/staging
2. Exécuter des tests de charge simulant l'utilisation de pointe
3. Surveiller pendant 48-72 heures sous charge réaliste
4. Valider les procédures de sauvegarde/restauration

**Déploiement en production :**
1. Planifier les changements pendant les fenêtres de maintenance
2. Utiliser la géo-réplication Azure SQL pour les déploiements blue/green
3. Avoir un plan de retour arrière prêt
4. Surveiller activement pendant et après le changement

### Capacité réservée

**Quand acheter :**
- Bases de données stables et durables (> 1 an)
- Après le redimensionnement (ne pas réserver des ressources surdimensionnées)
- Jusqu'à 80% d'économies vs paiement à l'utilisation

**Recommandations JetScale :**
- Engagement 1 an ou 3 ans
- Appliqué automatiquement aux ressources correspondantes
- Flexibilité de taille au sein du même niveau de service

## Modèles d'optimisation courants

### Modèle 1 : Base de données Premium sur-provisionnée

**Symptômes :**
- Niveau Premium avec faible utilisation DTU (<40%)
- Fonctionnalités de haute disponibilité non utilisées
- Latence plus faible non critique pour l'application

**Recommandation JetScale :**
- Rétrograder au niveau Standard
- Économies typiques : 60-70%
- Risque : Faible (vérifier les exigences de latence)

### Modèle 2 : Bases de données de développement inactives

**Symptômes :**
- Bases de données utilisées uniquement pendant les heures ouvrables
- Aucune activité la nuit et les week-ends
- Charges de travail de développement ou de rapports

**Recommandation JetScale :**
- Migrer vers le niveau de calcul Serverless
- Économies typiques : 50-80%
- Risque : Très faible (utilisation non continue)

### Modèle 3 : Plusieurs petites bases de données

**Symptômes :**
- Nombreuses bases de données avec des modèles d'utilisation similaires
- Les pics de bases de données individuelles ne coïncident pas
- Sur-provisionnement dû à la capacité de pointe par base de données

**Recommandation JetScale :**
- Consolider dans un Elastic Pool
- Économies typiques : 60-75%
- Risque : Faible (le partage des ressources réduit le gaspillage)

### Modèle 4 : Sur-provisionnement Business Critical

**Symptômes :**
- Niveau Business Critical sans exigences HA
- Réplicas secondaires lisibles non utilisés
- Performance SSD local non requise

**Recommandation JetScale :**
- Rétrograder à General Purpose
- Économies typiques : 60% (différence de prix 2,5x)
- Risque : Moyen (vérifier la tolérance à la latence)

## Dépannage

### Préoccupations concernant les recommandations

**Q : La rétrogradation impactera-t-elle les performances ?**

R : JetScale maintient une marge de 2-3x au-dessus de l'utilisation de pointe. Nous analysons :
- Utilisation DTU/vCore P95 et P99 sur 14 jours
- Métriques de performance des requêtes
- Modèles de connexion et échecs
- Modèles de pics historiques

Les recommandations ne procèdent que si le risque de performance est Faible ou Très Faible.

**Q : Qu'en est-il des délais de démarrage à froid serverless ?**

R : Les bases de données serverless reprennent en environ 2 minutes depuis l'état en pause. Idéal pour :
- Traitement par lots et rapports
- Développement et tests
- Applications tolérantes aux démarrages à froid occasionnels
- À éviter pour : API en temps réel, applications toujours actives

**Q : Puis-je tester avant de m'engager ?**

R : Oui ! Appliquez d'abord les changements à dev/staging :
1. JetScale génère Terraform pour tous les environnements
2. Testez en non-production pendant 48-72 heures
3. Surveillez les métriques de performance
4. Retour arrière si nécessaire (simple changement de niveau)
5. Appliquez en production une fois validé

### Problèmes de performance après optimisation

**Symptôme : Latence accrue des requêtes**

Causes possibles :
- Capacité DTU/vCore insuffisante
- Limitation des E/S de stockage
- Épuisement du pool de connexions

**Résolution :**
1. Vérifier les métriques `cpu_percent` et `dtu_consumption_percent`
2. Examiner les journaux de requêtes lentes
3. Augmenter le niveau/taille d'un cran si nécessaire
4. Envisager Query Performance Insights pour l'optimisation

**Symptôme : Échecs de connexion**

Causes possibles :
- Base de données serverless en état en pause
- Paramètres du pool de connexions nécessitent un ajustement
- Limites de ressources dépassées

**Résolution :**
1. Si serverless : Réduire le délai de pause automatique ou passer en provisionné
2. Vérifier la métrique `connection_failed`
3. Examiner la configuration du pool de connexions
4. Surveiller `sessions_percent` pour la capacité

## Considérations de sécurité

### Chiffrement

Les recommandations JetScale préservent :
- Paramètres Transparent Data Encryption (TDE)
- Configuration Always Encrypted
- Exigences de connexion SSL/TLS
- Chiffrement au repos et en transit

### Conformité

Les changements de configuration maintiennent la conformité :
- Paramètres de géo-réplication préservés
- Politiques de rétention de sauvegarde inchangées
- Paramètres de détection des menaces et d'audit maintenus
- Règles de réseau virtuel et règles de pare-feu préservées

## Limitations

**Non pris en charge actuellement :**
- Recommandations d'achat de capacité réservée
- Optimisation de la réplication inter-régions
- Optimisation de la politique de rétention de sauvegarde
- Recommandations d'optimisation des index et des requêtes

**Feuille de route :**
Ces fonctionnalités sont prévues pour les versions futures. Actuellement, JetScale se concentre sur l'optimisation du niveau de service et du calcul.

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations Azure SQL
GET /api/v1/recommendations?resource_type=azure-sql

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [Documentation API](../api-reference.md) pour la référence complète.

## Support

Besoin d'aide pour l'optimisation Azure SQL ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **GitHub Issues** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation connexe :**
- [Optimisation Azure VM](azure-vm.md)
- [Optimisation RDS](rds.md)
