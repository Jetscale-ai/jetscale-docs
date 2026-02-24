# Optimisation RDS

JetScale fournit une optimisation des coûts alimentée par l'IA pour Amazon RDS (Relational Database Service), incluant les instances autonomes et les clusters Aurora. Nos agents spécialisés analysent vos charges de travail de base de données pour identifier les opportunités de dimensionnement approprié et les améliorations de configuration.

## Vue d'ensemble

JetScale optimise les ressources RDS en analysant :
- **Utilisation des instances** : modèles de CPU, mémoire, réseau et connexions
- **Analyse des coûts** : dépenses actuelles vs. configuration optimale
- **Métriques de performance** : performance des requêtes, latence de réplication, taux de succès du cache tampon
- **Haute disponibilité** : configurations Multi-AZ pour les instances autonomes

## Types RDS pris en charge

### Clusters Aurora

Les clusters Aurora sont notre **cible d'optimisation principale**. Nous analysons le cluster dans son ensemble plutôt que les instances individuelles.

**Architecture du cluster :**
```
RdsDbCluster (Cible d'optimisation)
├── RdsDbInstance (Writer) - Facturable
├── RdsDbInstance (Reader) - Facturable
├── RdsDbInstance (Reader) - Facturable
└── AuroraDbClusterStorage - Facturable (I/O + Stockage)
```

**Ce que nous optimisons :**
- **Classe d'instance** : dimensionner correctement tous les nœuds du cluster pour correspondre à la charge de travail réelle (tous les nœuds utilisent la même classe d'instance)
- **Migration Graviton** : basculer vers des instances ARM (r6g, r7g, m6g, m7g) pour 10-40% d'économies
- **Configuration du stockage** : Aurora Standard vs. I/O-Optimized selon les modèles d'I/O

**Configurations de stockage :**
- **Aurora Standard** (`aurora`) : coût de calcul inférieur, frais d'I/O de 0,20 $/million
- **Aurora I/O-Optimized** (`aurora-iopt1`) : coût de calcul supérieur (+20%), frais d'I/O nuls

**Important :** Toutes les instances d'un cluster Aurora doivent utiliser la même classe d'instance. JetScale dimensionne pour le nœud avec la plus forte utilisation de ressources afin d'assurer des performances adéquates pour tous les membres du cluster.

### Instances RDS autonomes

Les instances RDS traditionnelles (MySQL, PostgreSQL, MariaDB, SQL Server, Oracle) sont optimisées individuellement.

**Architecture d'instance :**
```
RdsDbInstance (Cible d'optimisation, Facturable)
└── RdsDbInstanceStorage - Facturable
```

**Ce que nous optimisons :**
- **Classe d'instance** : dimensionner correctement vers la famille appropriée (t3, m5, r5, r6g, etc.)
- **Migration Graviton** : basculer vers des instances ARM (r6g, r7g, m6g, m7g) pour 10-40% d'économies
- **Configuration Multi-AZ** : désactiver Multi-AZ pour ~50% d'économies lorsque la haute disponibilité n'est pas critique (réduit la disponibilité)

**Modèle de facturation :**
- Facturation à la seconde avec un minimum de 10 minutes
- Reserved Instances : jusqu'à 66% d'économies (engagement 1 an ou 3 ans)
- Database Savings Plans : flexibilité supplémentaire entre les familles d'instances

## Comment JetScale optimise RDS

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques CloudWatch** (fenêtre glissante de 14 jours) :
- `CPUUtilization` - Utilisation CPU de l'instance
- `FreeableMemory` - Mémoire disponible
- `DatabaseConnections` - Connexions actives
- `ReadLatency` / `WriteLatency` - Performance du stockage
- `ReadThroughput` / `WriteThroughput` - I/O disque
- `NetworkReceiveThroughput` / `NetworkTransmitThroughput`
- `AuroraBinlogReplicaLag` (Aurora uniquement)

**Données API RDS** :
- Classe d'instance et version du moteur
- Type de stockage (Aurora Standard vs I/O-Optimized)
- Statut Multi-AZ (instances autonomes uniquement)
- Membres du cluster et rôles (Aurora uniquement)

**Données Cost Explorer** :
- Dépenses mensuelles actuelles par instance/cluster
- Tendances de coûts historiques
- Utilisation des Reserved Instances

**AWS Compute Optimizer** (si activé) :
- Recommandations de dimensionnement générées par AWS
- Évaluations des risques de performance

### 2. Analyse

Nos agents IA effectuent une analyse approfondie :

**Modèles d'utilisation :**
- Utilisation maximale vs. moyenne sur toutes les métriques
- Modèles horaires (identifier les périodes d'inactivité)
- Modèles hebdomadaires (charge week-end vs. semaine)
- Tendances de croissance au fil du temps

**Évaluation de la performance :**
- Taux de succès du cache tampon (indique l'adéquation de la mémoire)
- Débit de stockage vs. IOPS provisionnées
- Débit réseau vs. limites de l'instance
- Modèles de latence de réplication

**Modélisation des coûts :**
- Répartition des coûts actuels (calcul, stockage, I/O, sauvegarde)
- Coût projeté pour les configurations alternatives
- Opportunités de Reserved Instance / Savings Plan
- Analyse de la prime Multi-AZ

**Évaluation des risques :**
- Calcul de la marge (tampon au-dessus de l'utilisation maximale)
- Probabilité de dégradation des performances
- Évaluation de l'impact sur la disponibilité

### 3. Recommandations

JetScale génère des recommandations spécifiques et actionnables :

#### Dimensionnement approprié des instances

**Exemple de recommandation :**
```
Ressource : production-postgres-db
Actuel : db.r5.2xlarge (8 vCPUs, 64 GB RAM)
Recommandé : db.r5.xlarge (4 vCPUs, 32 GB RAM)

Impact sur les coûts :
- Actuel : 730 $/mois
- Projeté : 365 $/mois
- Économies : 365 $/mois (50%), 4 380 $/an

Analyse de performance :
- CPU moyen : 15-25%
- CPU maximal : 35%
- Mémoire moyenne : 30-40%
- La recommandation fournit une marge de 2x au-dessus du pic

Risque : Faible - Marge amplement maintenue
```

#### Optimisation Multi-AZ

**Exemple de recommandation :**
```
Ressource : staging-postgres-db
Actuel : db.m5.large avec Multi-AZ activé
Recommandé : db.m5.large avec Multi-AZ désactivé

Impact sur les coûts :
- Actuel : 280 $/mois (prime Multi-AZ incluse)
- Projeté : 140 $/mois (Single-AZ)
- Économies : 140 $/mois (50%), 1 680 $/an

Analyse de disponibilité :
- Environnement : Staging/Non-Production
- SLA actuel : 99,95% (Multi-AZ)
- SLA projeté : 99,9% (Single-AZ)
- RPO/RTO : Acceptable pour la charge de travail de staging

Risque : Faible - Environnement de non-production
Note : Multi-AZ fournit un basculement automatique en ~2 minutes
```

#### Aurora I/O-Optimized

**Exemple de recommandation :**
```
Ressource : customer-aurora-cluster
Actuel : Configuration Standard
Recommandé : Configuration I/O-Optimized

Impact sur les coûts :
- Actuel : 584 $/mois (instances) + 375 $/mois (I/O) = 959 $/mois
- Projeté : 701 $/mois (instances) + 0 $ (I/O) = 701 $/mois
- Économies : 258 $/mois (27%), 3 096 $/an

Analyse I/O :
- Requêtes I/O mensuelles : 1,9 milliard
- Coût I/O à 0,20 $/million : 375 $/mois
- Prime I/O-Optimized : augmentation de 20% du coût d'instance (117 $/mois)
- Économies nettes : 258 $/mois

Recommandation : Basculer vers I/O-Optimized
Seuil : Bénéfique lorsque les coûts I/O > 15% des coûts d'instance
```

#### Migration Graviton

**Exemple de recommandation :**
```
Ressource : api-database-cluster
Actuel : 3x db.r5.xlarge (basé Intel)
Recommandé : 3x db.r6g.xlarge (basé Graviton2)

Impact sur les coûts :
- Actuel : 1 095 $/mois (3 instances @ 365 $/mois chacune)
- Projeté : 876 $/mois (3 instances @ 292 $/mois chacune)
- Économies : 219 $/mois (20%), 2 628 $/an

Analyse de performance :
- Graviton2 offre des performances équivalentes ou supérieures
- Coût inférieur de 20% par instance
- Moteur : aurora-postgresql 14.7 (compatible Graviton)
- Même configuration vCPU et mémoire

Risque : Très faible - Performance Graviton éprouvée
Note : Nécessite une vérification de compatibilité de version du moteur
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Dimensionnement approprié d'instance**

```hcl
# Optimisation d'instance RDS
# Généré par JetScale le 2024-01-15
# ID de recommandation : rec_rds_001

resource "aws_db_instance" "production_postgres" {
  identifier = "production-postgres-db"

  # Précédent : db.r5.2xlarge (730 $/mois)
  # Optimisé : db.r5.xlarge (365 $/mois)
  # Réduction des coûts : 50% (365 $/mois, 4 380 $/an)
  #
  # Justification :
  # - Utilisation CPU moyenne : 15-25%
  # - Utilisation CPU maximale : 35%
  # - Utilisation mémoire moyenne : 30-40%
  # - La nouvelle instance fournit une marge de 2x au-dessus de l'utilisation maximale
  # - La surveillance des performances ne montre aucune pression mémoire
  instance_class = "db.r5.xlarge"

  engine         = "postgres"
  engine_version = "14.7"

  allocated_storage     = 500
  storage_type          = "gp3"
  iops                  = 3000
  storage_encrypted     = true

  multi_az = true

  # Configuration existante préservée
  db_name  = var.db_name
  username = var.db_username
  password = var.db_password

  vpc_security_group_ids = var.security_group_ids
  db_subnet_group_name   = var.db_subnet_group_name

  backup_retention_period = 7
  backup_window          = "03:00-04:00"
  maintenance_window     = "mon:04:00-mon:05:00"

  skip_final_snapshot = false
  final_snapshot_identifier = "${var.identifier}-final-snapshot"

  tags = merge(
    var.tags,
    {
      "jetscale:optimized" = "true"
      "jetscale:recommendation" = "rec_rds_001"
    }
  )
}
```

**Exemple : Aurora I/O-Optimized**

```hcl
# Optimisation I/O du cluster Aurora
# Généré par JetScale le 2024-01-15

resource "aws_rds_cluster" "customer_aurora" {
  cluster_identifier = "customer-aurora-cluster"

  engine         = "aurora-postgresql"
  engine_version = "15.4"
  engine_mode    = "provisioned"

  # Basculer vers la configuration I/O-Optimized
  # Précédent : Standard (Coûts I/O élevés : 375 $/mois)
  # Optimisé : I/O-Optimized (prime d'instance de 20%, coûts I/O de 0 $)
  # Économies nettes : 258 $/mois (27%), 3 096 $/an
  #
  # Analyse :
  # - I/O mensuelles : 1,9 milliard de requêtes
  # - Coût I/O (Standard) : 375 $/mois
  # - Prime d'instance (I/O-Optimized) : 117 $/mois
  # - Bénéfice net : 258 $/mois
  storage_type = "aurora-iopt1"

  master_username = var.master_username
  master_password = var.master_password

  database_name = var.database_name

  vpc_security_group_ids = var.vpc_security_group_ids
  db_subnet_group_name   = var.db_subnet_group_name

  backup_retention_period = 7
  preferred_backup_window = "03:00-04:00"
  preferred_maintenance_window = "mon:04:00-mon:05:00"

  enabled_cloudwatch_logs_exports = ["postgresql"]

  tags = merge(
    var.tags,
    {
      "jetscale:optimized" = "true"
      "jetscale:storage-mode" = "io-optimized"
    }
  )
}

resource "aws_rds_cluster_instance" "customer_aurora_instances" {
  count = 3

  identifier         = "customer-aurora-${count.index + 1}"
  cluster_identifier = aws_rds_cluster.customer_aurora.id

  instance_class = "db.r6g.xlarge"
  engine         = aws_rds_cluster.customer_aurora.engine
  engine_version = aws_rds_cluster.customer_aurora.engine_version

  publicly_accessible = false

  tags = var.tags
}
```

## Bonnes pratiques

### Surveillance après les modifications

Après avoir appliqué les recommandations JetScale :

1. **Premières 24 heures** : Surveiller de près les métriques clés
   - Utilisation CPU et mémoire
   - Connexions à la base de données
   - Latence des requêtes (p50, p95, p99)
   - IOPS et débit du stockage

2. **Semaine 1** : Valider la performance
   - Comparer les temps d'exécution des requêtes
   - Vérifier les indicateurs de pression mémoire
   - Surveiller la latence de réplication (le cas échéant)
   - Examiner les taux d'erreur de l'application

3. **Semaines 2-4** : Confirmer les économies de coûts
   - Vérifier que la facturation AWS reflète les économies attendues
   - S'assurer qu'il n'y a pas de frais inattendus
   - Vérifier les recommandations de Reserved Instance

### Stratégie de test

**Test en pré-production :**
1. Appliquer les modifications d'abord à l'environnement dev/staging
2. Exécuter des tests de charge simulant le trafic de pointe
3. Surveiller pendant 48-72 heures sous charge réaliste
4. Valider les procédures de sauvegarde/restauration

**Déploiement en production :**
1. Planifier les modifications pendant les fenêtres de maintenance
2. Utiliser des déploiements blue/green pour Aurora (temps d'arrêt nul)
3. Avoir un plan de retour arrière prêt
4. Surveiller activement pendant et après la modification

### Reserved Instances & Savings Plans

**Quand acheter :**
- Bases de données stables et de longue durée (> 1 an)
- Après le dimensionnement approprié (ne pas réserver d'instances surdimensionnées)
- Lorsque les économies d'engagement > coût d'opportunité

**Recommandations JetScale :**
- Nous analysons l'utilisation des RI et suggérons des achats optimaux
- Envisager 1 an plutôt que 3 ans pour la flexibilité
- Les RI flexibles en taille permettent des changements de famille d'instances
- Combiner avec les Compute Savings Plans pour une flexibilité maximale

## Modèles d'optimisation courants

### Modèle 1 : Base de données OLTP sur-provisionnée

**Symptômes :**
- Utilisation CPU < 20% en moyenne
- Utilisation mémoire < 40%
- Pics de connexion peu fréquents

**Recommandation JetScale :**
- Réduire de 1-2 tailles d'instance
- Économies typiques : 50-66%
- Risque : Faible (marge de 2-3x maintenue)

### Modèle 2 : Coûts I/O élevés sur Aurora

**Symptômes :**
- Coûts I/O > 15% de la facture RDS totale
- Requêtes fréquentes en lecture intensive
- Analyses de grandes tables

**Recommandation JetScale :**
- Basculer vers la configuration I/O-Optimized
- Économies typiques : 20-40% sur le coût total du cluster
- Avantage supplémentaire : coûts prévisibles

### Modèle 3 : Multi-AZ non-production

**Symptômes :**
- Environnements dev/staging/test avec Multi-AZ activé
- Haute disponibilité non requise pour la non-production
- Tolérance acceptable aux temps d'arrêt

**Recommandation JetScale :**
- Désactiver Multi-AZ pour les charges de travail non-production
- Économies typiques : 50% sur les coûts d'instance
- Compromis : basculement manuel requis vs. automatique
- Idéal pour : développement, staging, test, analytique

### Modèle 4 : Opportunités Graviton manquées

**Symptômes :**
- Utilisation d'instances Intel (r5, m5, r6i)
- Versions de moteur compatibles disponibles
- Pas de dépendances spécifiques ARM

**Recommandation JetScale :**
- Migrer vers des instances Graviton (r6g, r7g, m6g, m7g)
- Économies typiques : 10-40% avec des performances identiques ou meilleures
- Exigences : vérifier la compatibilité de version du moteur
- Idéal pour : la plupart des moteurs RDS modernes (PostgreSQL 12+, MySQL 8+, MariaDB 10.4+)

## Dépannage

### Préoccupations concernant les recommandations

**Q : La réduction de mon instance impactera-t-elle les performances ?**

R : JetScale maintient une marge de 2-3x au-dessus de l'utilisation maximale. Nous analysons :
- Utilisation CPU/mémoire P99 sur 14 jours
- Taux de succès du cache tampon
- Points de saturation réseau et stockage
- Tendances de croissance

Les recommandations ne sont émises que si le risque de performance est Faible ou Très Faible.

**Q : Qu'en est-il des pics de trafic soudains ?**

R : Notre analyse inclut :
- Métriques P99 (99e percentile), pas seulement les moyennes
- Tampon pour les pics inattendus (2-3x au-dessus du pic)
- Modèles de pics historiques
- Validation des contrôles de santé au niveau application

**Q : Puis-je tester avant de m'engager ?**

R : Oui ! Appliquez d'abord les modifications à dev/staging :
1. JetScale génère Terraform pour tous les environnements
2. Testez en non-production pendant 48-72 heures
3. Surveillez les métriques de performance
4. Retour arrière si nécessaire (simple revert Terraform)
5. Appliquez à la production une fois validé

### Problèmes de performance après optimisation

**Symptôme : Latence accrue des requêtes**

Causes possibles :
- Taille insuffisante du pool de connexions pour une instance plus petite
- Pression mémoire causant une augmentation des I/O disque
- Les paramètres du groupe de paramètres nécessitent un ajustement

**Résolution :**
1. Vérifier la métrique `FreeableMemory`
2. Examiner les journaux de requêtes lentes
3. Augmenter la taille de l'instance d'un niveau si nécessaire
4. Ajuster `shared_buffers` ou le paramètre équivalent

**Symptôme : Coûts I/O élevés sur Aurora**

Causes possibles :
- Utilisation d'Aurora Standard avec une charge de travail I/O élevée
- Requêtes fréquentes en lecture intensive
- Analyses de grandes tables

**Résolution :**
1. Vérifier le pourcentage de coût I/O de la facture RDS totale
2. Si les coûts I/O > 15% des coûts d'instance, envisager I/O-Optimized
3. Surveiller les métriques `VolumeReadIOPs` et `VolumeWriteIOPs`
4. I/O-Optimized ajoute 20% au coût d'instance mais élimine tous les frais I/O

## Considérations de sécurité

### Chiffrement

Les recommandations JetScale préservent les paramètres de chiffrement existants :
- État du chiffrement du stockage maintenu
- Clés KMS inchangées
- Exigences de connexion TLS/SSL préservées

### Conformité

Les modifications d'instance maintiennent la conformité :
- Configuration Multi-AZ modifiée uniquement lorsque explicitement recommandé
- Moteur et version du moteur jamais modifiés
- Associations VPC et groupes de sécurité préservés
- Groupes de paramètres existants maintenus

### Identifiants

- Terraform utilise `var.db_password` (externalisé)
- Mots de passe maîtres non exposés dans le code généré
- Les secrets doivent utiliser AWS Secrets Manager
- Politiques de rotation inchangées

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations RDS
GET /api/v1/recommendations?resource_type=rds

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [Documentation API](../api-reference.md) pour une référence complète.

## Support

Besoin d'aide avec l'optimisation RDS ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **GitHub Issues** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation connexe :**
- [Optimisation ElastiCache](elasticache.md)
- [Optimisation EC2](ec2.md)
- [Optimisation EBS](ebs.md)
