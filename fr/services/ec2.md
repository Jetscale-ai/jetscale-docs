# Optimisation EC2

JetScale fournit une optimisation des coûts basée sur l'IA pour les instances Amazon EC2 (Elastic Compute Cloud) autonomes. Nos agents spécialisés analysent vos charges de travail de calcul pour identifier les opportunités de dimensionnement approprié, les migrations de familles d'instances et les alternatives rentables.

## Vue d'ensemble

JetScale optimise les ressources EC2 en analysant :
- **Utilisation des instances** : Modèles d'utilisation CPU et réseau
- **Métriques mémoire** : Lorsque l'agent CloudWatch est installé
- **Analyse des coûts** : Dépenses actuelles vs. configuration optimale
- **Caractéristiques de la charge de travail** : Modèles CPU, besoins d'équilibre des ressources

## Types EC2 pris en charge

### Instances EC2 autonomes

JetScale optimise les instances EC2 autonomes qui ne font pas partie de groupes Auto Scaling.

**Architecture d'instance :**
```
Ec2Instance (Cible d'optimisation, Facturable)
├── EbsVolume - Facturable (optimisé séparément)
└── EbsVolume - Facturable
```

**Ce que nous optimisons :**
- **Dimensionnement approprié du type d'instance** : Changement vers des instances plus petites/plus grandes selon l'utilisation réelle
- **Migration de famille d'instance** : Changement entre familles (T, M, C, R, X, etc.) selon les caractéristiques de la charge de travail
- **Mises à niveau de génération** : Migration vers la dernière génération (M8, C8, R8) pour un meilleur rapport prix/performance
- **Migration Graviton** : Passage aux instances basées sur ARM (suffixe g) pour 10-40% d'économies
- **Suppression des instances inutilisées** : Suppression des instances avec une activité nulle ou proche de zéro (100% d'économies)

**Important :** JetScale optimise actuellement uniquement les instances EC2 autonomes. Les instances gérées par des groupes Auto Scaling, EKS, ECS ou d'autres systèmes d'orchestration ne sont pas encore prises en charge.

## Familles d'instances

JetScale considère toutes les familles d'instances AWS EC2 lors de l'optimisation :

### Usage général (M)
**M8i/M8g/M8a** - Calcul/mémoire/réseau équilibrés
- Utilisation : Serveurs web, serveurs d'applications, petites bases de données, charges de travail mixtes
- Dernières générations : M8 > M7 > M6 > M5

### Optimisées pour le calcul (C)
**C8i/C8g, C7a** - Ratio CPU élevé
- Utilisation : Traitement par lots, HPC, transcodage média, jeux, serveurs web à fort trafic
- Idéal quand : L'utilisation CPU est constamment élevée, les besoins en mémoire sont modérés

### Optimisées pour la mémoire (R)
**R8i/R8g, R7a** - Ratio mémoire élevé
- Utilisation : Bases de données, caches en mémoire, analytique big data
- Idéal quand : L'utilisation mémoire est élevée, le CPU est modéré

### Mémoire extrême (X)
**X8g, X2idn/X2iedn** - Ratios mémoire très élevés
- Utilisation : Grandes bases de données, analytique en mémoire, SAP HANA
- Prime de coût : Significativement plus élevé que la famille R

### Burstables (T)
**T4g, T3, T3a** - Crédits CPU de base + burst
- Utilisation : Dév/test, serveurs web à faible trafic, microservices, charges de travail variables
- Idéal quand : L'utilisation CPU est faible la plupart du temps avec des pics occasionnels
- Attention : Surveillez le solde de crédits CPU pour éviter la limitation

### Optimisées pour le stockage

**Série I (NVMe SSD)** : I8g/I8ge, I7i/I7ie - Stockage local à faible latence
- Utilisation : Bases de données transactionnelles (MySQL, MongoDB, Cassandra)

**Série D (HDD)** : D3/D3en - Stockage HDD haute densité
- Utilisation : Systèmes de fichiers distribués (HDFS), entreposage de données

**Série H (HDD)** : H1 - Débit HDD séquentiel
- Utilisation : Big data, MapReduce, traitement de logs

### Calcul accéléré

**Accélérateurs GPU et ML** - P6, G6, Inf2, Trn2
- Utilisation : Entraînement/inférence ML, rendu graphique
- Non typiquement recommandé pour l'optimisation des coûts (charges de travail spécialisées)

### Types de processeurs

- **Intel** (par défaut) : Le plus commun, large compatibilité
- **AMD** (suffixe a) : Alternative rentable avec bonne performance
- **Graviton** (suffixe g) : Architecture ARM64, meilleur rapport prix/performance (10-40% d'économies)

### Dimensionnement d'instance

Les tailles suivent le modèle : `.large < .xlarge < .2xlarge < .4xlarge < .8xlarge < .12xlarge < .16xlarge < .24xlarge`

Chaque étape double typiquement les vCPU et la mémoire.

## Comment JetScale optimise EC2

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques CloudWatch** (fenêtre glissante de 14 jours) :
- `CPUUtilization` - Utilisation CPU de l'instance (toujours disponible)
- `NetworkIn` / `NetworkOut` - Débit réseau (toujours disponible)
- `mem_used_percent` - Utilisation mémoire (nécessite l'agent CloudWatch)
- `disk_used_percent` - Utilisation disque (nécessite l'agent CloudWatch)

**Données API EC2** :
- Type et état de l'instance
- Heure de lancement
- Zone de disponibilité
- Tags

**Données Cost Explorer** :
- Dépense mensuelle actuelle par instance
- Tendances des coûts historiques

**AWS Compute Optimizer** (si activé) :
- Recommandations de dimensionnement générées par AWS
- Évaluations des risques de performance

### 2. Analyse

Nos agents IA effectuent une analyse approfondie :

**Modèles d'utilisation :**
- Utilisation CPU pic vs. moyenne
- Modèles horaires (identifier les périodes inactives)
- Modèles jour de la semaine (charge weekend vs. semaine)
- Solde de crédits CPU (pour les instances série T)

**Analyse mémoire** (si métriques disponibles) :
- Pourcentage d'utilisation mémoire
- Marge au-dessus de l'utilisation pic
- Mémoire minimale requise avec tampon

**Classification de la charge de travail :**
- **CPU variable** : Considérer série T (burstable)
- **CPU élevé constant** : Considérer série C (optimisé calcul)
- **Équilibré** : Considérer série M (usage général)
- **Intensif en mémoire** : Considérer série R (optimisé mémoire)

**Modélisation des coûts :**
- Coût mensuel actuel
- Coût projeté pour les types d'instances alternatifs
- Pourcentage d'économies et impact annuel

**Évaluation des risques :**
- Probabilité de dégradation des performances
- Calcul de la marge (tampon au-dessus de l'utilisation pic)
- Vérifications de sécurité mémoire

### 3. Recommandations

JetScale génère des recommandations spécifiques et actionnables :

#### Dimensionnement approprié d'instance

**Exemple de recommandation :**
```
Ressource : web-server-prod-1
Actuelle : m5.2xlarge (8 vCPUs, 32 GB RAM)
Recommandée : m5.xlarge (4 vCPUs, 16 GB RAM)

Impact coût :
- Actuel : $280/mois
- Projeté : $140/mois
- Économies : $140/mois (50%), $1,680/an

Analyse de performance :
- CPU moyen : 18%
- CPU pic : 32%
- Mémoire moyenne : 28% (nécessite l'agent CloudWatch pour les métriques mémoire)
- Recommandée fournit 2x de marge au-dessus du pic

Risque : Faible - Marge importante maintenue
```

#### Migration de famille

**Exemple de recommandation :**
```
Ressource : batch-processor-2
Actuelle : m5.xlarge (4 vCPUs, 16 GB RAM)
Recommandée : c6i.xlarge (4 vCPUs, 8 GB RAM)

Impact coût :
- Actuel : $140/mois
- Projeté : $124/mois
- Économies : $16/mois (11%), $192/an

Analyse de charge de travail :
- CPU moyen : 75%
- CPU pic : 92%
- Utilisation mémoire : Faible (25%)
- Type de charge : Intensif en calcul, mémoire sous-utilisée

Recommandation : Migration vers famille C (optimisé calcul)
Risque : Très faible - Charge liée au CPU, marge mémoire adéquate
```

#### Migration Graviton

**Exemple de recommandation :**
```
Ressource : api-server-1
Actuelle : m5.large (2 vCPUs, 8 GB RAM, basée Intel)
Recommandée : m6g.large (2 vCPUs, 8 GB RAM, basée Graviton2)

Impact coût :
- Actuel : $70/mois
- Projeté : $56/mois
- Économies : $14/mois (20%), $168/an

Analyse de performance :
- Graviton2 offre une performance équivalente ou meilleure
- 20% de coût inférieur par instance
- Même configuration vCPU et mémoire
- Nécessite une AMI compatible ARM64

Risque : Très faible - Performance Graviton prouvée
Note : Vérifier la compatibilité de l'application avec l'architecture ARM64
```

#### Mise à niveau de génération

**Exemple de recommandation :**
```
Ressource : database-app-server
Actuelle : m5.xlarge (4 vCPUs, 16 GB RAM)
Recommandée : m7i.xlarge (4 vCPUs, 16 GB RAM)

Impact coût :
- Actuel : $140/mois
- Projeté : $133/mois
- Économies : $7/mois (5%), $84/an

Analyse de performance :
- M7i offre de meilleures performances CPU (Intel Xeon 4e gen)
- Mêmes spécifications, coût inférieur
- Meilleures performances réseau (jusqu'à 12,5 Gbps)
- Efficacité de génération plus récente

Risque : Très faible - Même famille, génération plus récente
```

#### Suppression d'instance inutilisée

**Exemple de recommandation :**
```
Ressource : legacy-test-server
Actuelle : t3.medium (2 vCPUs, 4 GB RAM)
Recommandée : Supprimer/Terminer

Impact coût :
- Actuel : $30/mois
- Projeté : $0/mois
- Économies : $30/mois (100%), $360/an

Analyse d'utilisation :
- CPU moyen : 0,2%
- Activité réseau : Proche de zéro
- Lancée : Il y a 18 mois
- Tags indiquent : environnement test

Risque : Très faible - Apparaît inutilisée
Recommandation : Vérifier avec l'équipe application avant suppression
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Dimensionnement approprié d'instance**

```hcl
# Optimisation instance EC2
# Généré par JetScale le 2024-01-15
# ID recommandation : rec_ec2_001

resource "aws_instance" "web_server_prod_1" {
  # Précédent : m5.2xlarge ($280/mois)
  # Optimisé : m5.xlarge ($140/mois)
  # Réduction coût : 50% ($140/mois, $1,680/an)
  #
  # Justification :
  # - Utilisation CPU moyenne : 18%
  # - Utilisation CPU pic : 32%
  # - Utilisation mémoire moyenne : 28%
  # - Nouvelle instance fournit 2x de marge au-dessus utilisation pic
  instance_type = "m5.xlarge"

  ami           = var.ami_id
  key_name      = var.key_name
  subnet_id     = var.subnet_id

  vpc_security_group_ids = var.security_group_ids

  # Configuration existante préservée
  monitoring    = true
  ebs_optimized = true

  root_block_device {
    volume_type           = "gp3"
    volume_size           = 100
    encrypted             = true
    delete_on_termination = true
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"       = "true"
      "jetscale:recommendation"  = "rec_ec2_001"
      "jetscale:previous_type"   = "m5.2xlarge"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

**Exemple : Migration Graviton**

```hcl
# Migration EC2 Graviton
# Généré par JetScale le 2024-01-15

resource "aws_instance" "api_server_1" {
  # Précédent : m5.large (Intel, $70/mois)
  # Optimisé : m6g.large (Graviton2, $56/mois)
  # Réduction coût : 20% ($14/mois, $168/an)
  #
  # Exigences :
  # - AMI compatible ARM64 requise
  # - Vérifier compatibilité application
  # - Tester en staging avant production
  instance_type = "m6g.large"

  # IMPORTANT : Utiliser AMI ARM64
  # Remplacer par votre ID AMI compatible ARM64
  ami = var.ami_id_arm64

  key_name  = var.key_name
  subnet_id = var.subnet_id

  vpc_security_group_ids = var.security_group_ids

  monitoring    = true
  ebs_optimized = true

  root_block_device {
    volume_type           = "gp3"
    volume_size           = 50
    encrypted             = true
    delete_on_termination = true
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"    = "true"
      "jetscale:architecture" = "arm64"
      "jetscale:processor"    = "graviton2"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

## Gestion des métriques mémoire

### Avec l'agent CloudWatch

Lorsque l'agent CloudWatch est installé et rapporte les métriques mémoire :
- JetScale peut recommander en toute sécurité des instances avec moins de mémoire
- Dimensionnement mémoire basé sur l'utilisation pic + 20% de marge (standard AWS Compute Optimizer)
- Optimisation plus agressive possible

**Exemple :**
```
Actuel : 32 GB RAM, Utilisation pic : 20 GB (62,5%)
RAM sûre minimale : 20 GB × 1,20 = 24 GB
Recommandation : Instance avec 32 GB RAM ou moins (si disponible)
```

### Sans l'agent CloudWatch

Lorsque les métriques mémoire ne sont pas disponibles :
- **Contrainte de sécurité** : Toutes les alternatives doivent avoir EXACTEMENT la même quantité de mémoire
- Impossible de recommander des instances avec moins de mémoire (risque de plantages OOM)
- Optimisation limitée au CPU, réseau et améliorations de coût

**Exemple :**
```
Actuel : 32 GB RAM, pas de métriques mémoire
Contrainte : Toutes les alternatives DOIVENT avoir exactement 32 GB RAM
Optimisation : Famille d'instance, génération, type de processeur uniquement
```

**Pour activer les métriques mémoire :**
1. Installer l'agent CloudWatch sur les instances EC2
2. Configurer l'agent pour collecter la métrique `mem_used_percent`
3. Attendre 14 jours pour des données historiques suffisantes
4. JetScale utilisera automatiquement les métriques mémoire dans les recommandations

## Bonnes pratiques

### Stratégie de test

**Tests pré-production :**
1. Appliquer les changements à l'environnement dév/staging d'abord
2. Exécuter des tests de charge simulant le trafic pic
3. Surveiller pendant 48-72 heures sous charge réaliste
4. Valider la performance de l'application

**Déploiement production :**
1. Utiliser le cycle de vie `create_before_destroy` dans Terraform
2. Planifier les changements pendant les fenêtres de maintenance
3. Avoir un plan de retour arrière prêt (état Terraform précédent)
4. Surveiller activement pendant et après le changement

### Migration Graviton

**Avant de migrer vers Graviton :**
1. Vérifier que l'application est compatible ARM64 (la plupart des apps modernes le sont)
2. Vérifier les dépendances et bibliothèques pour le support ARM
3. Tester avec une AMI ARM64 en staging
4. Consulter la documentation de compatibilité AWS Graviton
5. Compatibles courants : Node.js, Python, Java, Go, .NET, Ruby, PHP

**Charges de travail incompatibles :**
- Binaires ou bibliothèques x86 uniquement
- Applications héritées sans builds ARM
- Logiciels spécifiques nécessitant des instructions Intel/AMD

### Instances réservées & Savings Plans

Bien que JetScale ne génère pas actuellement de recommandations RI/Savings Plan, considérez-les après le dimensionnement approprié :

**Bonne pratique :**
1. Dimensionner correctement les instances avec JetScale d'abord
2. Exécuter la configuration optimisée pendant 30-60 jours
3. Acheter des RI ou Savings Plans pour les charges de travail stables
4. Ne jamais réserver des instances surdimensionnées

## Modèles d'optimisation courants

### Modèle 1 : Instances sur-provisionnées

**Symptômes :**
- Utilisation CPU < 20% en moyenne
- Utilisation mémoire < 30% (si métriques disponibles)
- Instance fonctionnant 24/7 avec utilisation faible constante

**Recommandation JetScale :**
- Réduire de 1-2 tailles d'instance (ex. 2xlarge → xlarge)
- Économies typiques : 40-50%
- Risque : Faible (marge 2-3x maintenue)

### Modèle 2 : Mauvaise famille d'instance

**Symptômes :**
- Utilisation famille M avec CPU constamment élevé (>70%)
- Utilisation famille R avec utilisation mémoire faible (<40%)
- Utilisation famille C avec besoins mémoire élevés

**Recommandation JetScale :**
- Migrer vers famille appropriée (M → C pour CPU, R → M pour équilibré)
- Économies typiques : 10-30%
- Risque : Faible à Moyen (vérifier caractéristiques charge de travail)

### Modèle 3 : Générations héritées

**Symptômes :**
- Utilisation d'instances M4, M5, C4, C5
- Générations plus récentes disponibles (M6, M7, M8)
- Même coût ou inférieur pour meilleure performance

**Recommandation JetScale :**
- Mise à niveau vers dernière génération (M5 → M7, C5 → C7)
- Économies typiques : 5-15% avec meilleure performance
- Risque : Très faible (même famille, matériel plus récent)

### Modèle 4 : Opportunités Graviton manquées

**Symptômes :**
- Utilisation d'instances basées Intel (m5, c5, r5)
- Applications compatibles ARM64
- Aucune exigence x86 spécialisée

**Recommandation JetScale :**
- Migrer vers Graviton (m5 → m6g, c5 → c6g, r5 → r6g)
- Économies typiques : 10-40% avec même ou meilleure performance
- Risque : Faible (vérifier compatibilité ARM64)

## Dépannage

### Préoccupations sur les recommandations

**Q : La réduction de taille de mon instance impactera-t-elle la performance ?**

R : JetScale maintient une marge 2-3x au-dessus de l'utilisation pic. Nous analysons :
- Utilisation CPU au 99e percentile (P99) sur 14 jours
- Utilisation mémoire pic (si disponible)
- Points de saturation réseau
- Modèles de pics historiques

Les recommandations ne procèdent que si le risque de performance est Faible ou Très faible.

**Q : Que faire si je n'ai pas de métriques mémoire ?**

R : JetScale applique une contrainte de sécurité : toutes les alternatives doivent avoir EXACTEMENT la même quantité de mémoire que l'instance actuelle. Cela prévient les risques de sous-provisionnement. Pour débloquer une optimisation mémoire plus agressive, installez l'agent CloudWatch.

**Q : Puis-je tester Graviton sans temps d'arrêt ?**

R : Oui ! Approche recommandée :
1. Lancer une nouvelle instance Graviton avec `create_before_destroy = true`
2. Vérification santé réussie → le trafic bascule vers la nouvelle instance
3. Ancienne instance automatiquement terminée
4. Temps d'arrêt total : typiquement <30 secondes pendant le basculement

### Problèmes de performance après optimisation

**Symptôme : Latence ou temps de réponse augmentés**

Causes possibles :
- CPU insuffisant pour les charges pic
- Pression mémoire (si taille recommandée trop petite)
- Goulot d'étranglement débit réseau

**Résolution :**
1. Vérifier les métriques CPU, mémoire, réseau CloudWatch
2. Comparer performance instance actuelle vs. précédente
3. Si nécessaire, augmenter la taille d'instance d'un cran
4. Retour arrière simple : revenir à l'état Terraform précédent

**Symptôme : Épuisement des crédits CPU série T**

Causes possibles :
- Migration d'une instance à performance fixe vers burstable
- Utilisation CPU supérieure à la baseline série T
- Crédits CPU insuffisants pour le modèle de charge de travail

**Résolution :**
1. Vérifier la métrique `CPUCreditBalance`
2. Si constamment épuisés, migrer vers série M/C
3. Considérer le mode T3 Unlimited (coût supplémentaire pour burst)

## Considérations de sécurité

### Configuration d'instance

Les recommandations JetScale préservent :
- Associations VPC et groupes de sécurité
- Profils d'instance IAM
- Paires de clés
- Paramètres de chiffrement EBS
- Configuration de surveillance

### Compatibilité AMI

Lors de la migration vers Graviton (ARM64) :
- Utiliser une AMI compatible ARM64
- Vérifier que tous les logiciels supportent l'architecture ARM
- Tester minutieusement en non-production d'abord

### Gestion du cycle de vie

Utiliser les règles de cycle de vie Terraform :
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy = var.is_production
}
```

## Limitations

**Non actuellement pris en charge :**
- Optimisation Auto Scaling Group
- Recommandations d'instances spot
- Recommandations d'achat d'instances réservées
- Recommandations Savings Plan
- Automatisation arrêt/démarrage planifié
- Orchestration multi-instances (instances gérées EKS, ECS)

**Feuille de route :**
Ces fonctionnalités sont prévues pour les versions futures. Actuellement, JetScale se concentre sur l'optimisation des instances EC2 autonomes où des économies immédiates peuvent être réalisées grâce au dimensionnement approprié et à la migration.

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations EC2
GET /api/v1/recommendations?resource_type=ec2

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [Documentation API](../api-reference.md) pour une référence complète.

## Support

Besoin d'aide avec l'optimisation EC2 ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **GitHub Issues** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation connexe :**
- [Optimisation EBS](ebs.md)
- [Optimisation RDS](rds.md)
- [Optimisation ElastiCache](elasticache.md)
