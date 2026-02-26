# Optimisation EKS

JetScale fournit une optimisation des coûts pilotée par l'IA pour les clusters Amazon EKS (Elastic Kubernetes Service). Nos agents spécialisés analysent l'utilisation des groupes de nœuds, les types d'instances, les configurations de capacité et la tarification pour identifier les opportunités de dimensionnement optimal, de migration Graviton et d'optimisation Spot.

## Vue d'ensemble

JetScale optimise les clusters EKS en analysant :
- **Utilisation des groupes de nœuds** : CPU, mémoire, réseau et utilisation disque sur tous les nœuds
- **Efficacité du type d'instance** : Si les types d'instances actuels correspondent aux besoins réels des charges de travail
- **Type de capacité** : Opportunités ON_DEMAND vs. SPOT pour les charges de travail tolérantes aux pannes
- **Analyse des coûts** : Coûts du plan de contrôle, coûts des groupes de nœuds et potentiel d'optimisation

## Types EKS pris en charge

### Clusters EKS

JetScale traite chaque groupe de nœuds au sein d'un cluster de manière indépendante, générant des recommandations par groupe de nœuds avec une vue agrégée au niveau du cluster.

**Modèles de calcul pris en charge :**
- **Managed Node Groups** : Gérés par AWS, configuration explicite du type d'instance
- **Karpenter** : Sélection dynamique des instances avec gestion native des interruptions Spot
- **Self-Managed Nodes** : Instances EC2 lancées en dehors des groupes de nœuds gérés EKS

**Ce que nous optimisons :**
- **Dimensionnement optimal du type de nœud** : Faire correspondre les types d'instances aux besoins réels des charges de travail
- **Migration Graviton** : Basculer vers des instances ARM pour 10-40% d'économies
- **Optimisation Spot** : Déplacer les charges de travail tolérantes aux pannes vers la capacité Spot pour 60-90% d'économies
- **Suppression des ressources inutilisées** : Supprimer les clusters ou groupes de nœuds inactifs pour 100% d'économies

### Groupes de nœuds EKS

JetScale optimise également les groupes de nœuds autonomes avec les mêmes types d'analyse et de recommandations que le traitement au niveau du cluster.

## Familles d'instances

### Usage général (série M)
- **M7g** (Graviton3, dernière génération) : Meilleur rapport prix-performance pour les charges de travail générales
- **M6g** (Graviton2) : Option ARM éprouvée
- **M5/M5a** (Intel/AMD) : Compatibilité x86

### Optimisées pour le calcul (série C)
- **C7g** (Graviton3) : Charges de travail conteneurisées intensives en CPU
- **C6g** (Graviton2) : Traitement par lots, CI/CD
- **C5/C5a** (Intel/AMD) : Charges de travail de calcul x86

### Optimisées pour la mémoire (série R)
- **R7g** (Graviton3) : Cache en mémoire, analyses en temps réel
- **R6g** (Graviton2) : Conteneurs gourmands en mémoire
- **R5/R5a** (Intel/AMD) : Charges de travail mémoire x86

### À capacité extensible (série T)
- **T4g** (Graviton2) : Groupes de nœuds de développement/test, services à faible trafic
- **T3/T3a** (Intel/AMD) : Charges de travail variables avec modèle de crédits CPU

## Comment JetScale optimise EKS

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques CloudWatch** (historique d'un an) :
- Utilisation CPU (moyenne, maximum, percentiles) par nœud
- Utilisation mémoire (moyenne, maximum, percentiles) par nœud
- E/S réseau (octets entrants/sortants)
- E/S disque (opérations par seconde)
- Vérifications de l'état de santé des instances

Les métriques sont agrégées sur toutes les instances EC2 d'un groupe de nœuds, avec le nombre d'instances noté (ex. : "[3 instances] moy 25%, max 45%").

**Données de l'API EKS** :
- Version Kubernetes du cluster et type de support
- Types d'instances des groupes de nœuds et configuration de mise à l'échelle (min, max, souhaité)
- Type de capacité (ON_DEMAND ou SPOT)
- Type AMI (AL2_x86_64, AL2_ARM_64, etc.)
- Labels, taints et configuration du modèle de lancement
- Liaison avec l'Auto Scaling Group

**Données de tarification AWS** :
- Tarifs horaires On-Demand par type d'instance
- Tarifs horaires Spot (minimum et moyenne)
- Coût du plan de contrôle EKS :
  - Support Standard : ~0,10 $/heure (~73 $/mois)
  - Support étendu (Kubernetes 1.26 et antérieur) : ~0,60 $/heure (~438 $/mois)

**Liaison des instances :**
- Managed Node Groups → Instances EC2 via le nom de l'Auto Scaling Group
- Nœuds Karpenter → Cluster via le tag `aws:eks:cluster-name`, regroupés par NodePool
- Nœuds auto-gérés → Cluster via le tag du nom du cluster, regroupés par type d'instance

### 2. Analyse

Nos agents IA effectuent une analyse approfondie par groupe de nœuds :

**Modèles d'utilisation :**
- Utilisation CPU et mémoire pic vs. moyenne sur tous les nœuds
- Modèles horaires et journaliers
- Nombre de nœuds vs. consommation réelle des ressources

**Modélisation des coûts :**
- Coût actuel : `tarif_horaire_instance x taille_souhaitée x 730 heures/mois`
- Coût du plan de contrôle inclus dans les totaux au niveau du cluster
- La tarification Spot utilise les tarifs minimums conservateurs (pas les moyennes)

**Classification des charges de travail :**
- Intensives en calcul : Utilisation CPU élevée, mémoire faible
- Intensives en mémoire : Utilisation CPU faible, mémoire élevée
- Équilibrées : Utilisation CPU et mémoire proportionnelle
- Inactives : Utilisation minimale sur toutes les métriques

**Vérifications de sécurité :**
- Les clusters Auto Mode sont ignorés (AWS gère le calcul automatiquement)
- Une réduction de mémoire >25% nécessite des métriques d'utilisation mémoire
- Une réduction de CPU >50% nécessite des métriques d'utilisation CPU
- Les réductions de capacité sans métriques sont rejetées

### 3. Recommandations

JetScale génère des recommandations par groupe de nœuds, puis les agrège au niveau du cluster.

#### Dimensionnement optimal

**Exemple de recommandation :**
```
Ressource : production-cluster / api-node-group
Actuel : 3x m5.xlarge (ON_DEMAND)
  - 4 vCPU, 16 GiB de mémoire par nœud
  - Coût mensuel : $420.48

Analyse CloudWatch (3 nœuds) :
  - CPU : moy. 22%, max 41%
  - Mémoire : moy. 35%, max 52%

Recommandé : 3x m5.large (ON_DEMAND)
  - 2 vCPU, 8 GiB de mémoire par nœud
  - Coût mensuel : $210.24

Impact sur les coûts :
  - Économies : $210.24/mois (50%), $2,522.88/an

Risque : Faible - CPU max à 41% compatible avec 2 vCPU
  Mémoire à 52% de 16 GiB = 8.3 GiB, compatible avec 8 GiB avec marge
Effort d'implémentation : Moyen - Mise à jour progressive du groupe de nœuds requise
Redémarrage nécessaire : Oui (remplacement des nœuds)
```

#### Migration Graviton

**Exemple de recommandation :**
```
Ressource : web-cluster / frontend-nodes
Actuel : 4x m5.large (ON_DEMAND)
  - 2 vCPU, 8 GiB de mémoire par nœud
  - Coût mensuel : $280.32

Recommandé : 4x m6g.large (ON_DEMAND)
  - 2 vCPU, 8 GiB de mémoire par nœud (Graviton2, ARM64)
  - Coût mensuel : $224.26

Impact sur les coûts :
  - Économies : $56.06/mois (20%), $672.72/an

Risque : Faible - Les charges conteneurisées sont généralement compatibles ARM
  Mêmes spécifications vCPU et mémoire
  Meilleur rapport prix-performance sur Graviton
Effort d'implémentation : Moyen - Nécessite des images conteneur compatibles ARM64
Redémarrage nécessaire : Oui (remplacement des nœuds avec nouveau AMI)
```

#### Optimisation Spot

**Exemple de recommandation :**
```
Ressource : batch-cluster / worker-nodes
Actuel : 5x c5.xlarge (ON_DEMAND)
  - 4 vCPU, 8 GiB de mémoire par nœud
  - Coût mensuel : $620.50

Analyse de la charge de travail :
  - Traitement par lots, sans état
  - Tolérant aux pannes (les jobs redémarrent en cas d'échec)
  - Pas de trafic en production

Recommandé : 5x c5.xlarge (SPOT)
  - Mêmes spécifications, capacité Spot
  - Coût mensuel : $186.15 (minimum Spot conservateur)

Impact sur les coûts :
  - Économies : $434.35/mois (70%), $5,212.20/an

Risque : Moyen - Les instances Spot peuvent être récupérées avec un préavis de 2 minutes
  Karpenter gère les interruptions nativement
  Les jobs par lots doivent être idempotents et redémarrables
Effort d'implémentation : Faible (Karpenter) / Moyen (Managed Node Groups)
Redémarrage nécessaire : Oui (remplacement des nœuds)
```

#### Suppression des ressources inutilisées

**Exemple de recommandation :**
```
Ressource : legacy-cluster / test-nodes
Actuel : 2x t3.medium (ON_DEMAND)
  - Coût mensuel : $60.74

Analyse CloudWatch (2 nœuds) :
  - CPU : moy. 0.5%, max 2%
  - Mémoire : moy. 8%, max 12%
  - Aucune activité réseau depuis plus de 30 jours

Recommandé : Supprimer
  - Économies : $60.74/mois (100%), $728.88/an

Risque : Faible - Aucune charge de travail significative détectée
  Le nom ne contient pas "prod" ou "production"
  Vérifier avec l'équipe avant la suppression
```

#### Agrégation au niveau du cluster

Lorsque plusieurs groupes de nœuds sont optimisés :

```
Cluster : production-cluster
Plan de contrôle : $73/mois (support Standard)

Résultats par groupe de nœuds :
1. api-nodes : Dimensionnement m5.xlarge → m5.large (-$210.24/mois)
2. worker-nodes : Migration Graviton m5.large → m6g.large (-$56.06/mois)
3. cache-nodes : Aucune recommandation (déjà optimisé)

Action cluster : Optimisations multiples
Économies totales du cluster : $266.30/mois, $3,195.60/an
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Dimensionnement optimal du groupe de nœuds**

```hcl
# Optimisation du groupe de nœuds EKS : Dimensionnement
# Généré par JetScale le 2024-01-15
# ID de recommandation : rec_eks_001

resource "aws_eks_node_group" "api_nodes" {
  cluster_name    = "production-cluster"
  node_group_name = "api-nodes"
  node_role_arn   = var.node_role_arn
  subnet_ids      = var.subnet_ids

  # Précédent : m5.xlarge ($420.48/mois)
  # Optimisé : m5.large ($210.24/mois)
  # Réduction des coûts : 50% ($210.24/mois, $2,522.88/an)
  #
  # Analyse d'utilisation :
  # - CPU : moy. 22%, max 41% (compatible avec 2 vCPU)
  # - Mémoire : moy. 35%, max 52% de 16 GiB = 8.3 GiB (compatible avec 8 GiB)
  instance_types = ["m5.large"]

  scaling_config {
    desired_size = 3
    max_size     = 5
    min_size     = 2
  }

  update_config {
    max_unavailable = 1
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"        = "true"
      "jetscale:recommendation"   = "rec_eks_001"
      "jetscale:previous_type"    = "m5.xlarge"
    }
  )
}
```

**Exemple : Migration Graviton**

```hcl
# Optimisation du groupe de nœuds EKS : Migration Graviton
# Généré par JetScale le 2024-01-15

resource "aws_eks_node_group" "frontend_nodes" {
  cluster_name    = "web-cluster"
  node_group_name = "frontend-nodes"
  node_role_arn   = var.node_role_arn
  subnet_ids      = var.subnet_ids

  # Précédent : m5.large, x86_64 ($280.32/mois)
  # Optimisé : m6g.large, ARM64 ($224.26/mois)
  # Réduction des coûts : 20% ($56.06/mois, $672.72/an)
  instance_types = ["m6g.large"]

  # AMI ARM64 requise pour les instances Graviton
  ami_type = "AL2_ARM_64"

  scaling_config {
    desired_size = 4
    max_size     = 6
    min_size     = 2
  }

  update_config {
    max_unavailable = 1
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"        = "true"
      "jetscale:previous_type"    = "m5.large"
      "jetscale:migration_type"   = "graviton"
    }
  )
}
```

## Bonnes pratiques

### Stratégie de test

**Tests en pré-production :**
1. Tester les changements de type d'instance sur les groupes de nœuds hors production d'abord
2. Valider la compatibilité ARM64 des images de conteneurs avant la migration Graviton
3. Exécuter des groupes de nœuds Spot en parallèle des groupes ON_DEMAND pour une période d'essai
4. Surveiller la planification des pods et l'alignement des requests/limits de ressources

**Déploiement en production :**
1. Utiliser des mises à jour progressives (`max_unavailable = 1`) pour minimiser les perturbations
2. Surveiller les évictions de pods et la replanification pendant le remplacement des nœuds
3. Valider que les vérifications de santé de l'application réussissent sur les nouveaux types de nœuds
4. Conserver la configuration précédente du groupe de nœuds pour un retour arrière rapide

### Liste de vérification pour la migration Graviton

Avant de migrer vers des instances Graviton (ARM64) :
1. Vérifier que toutes les images de conteneurs ont des builds ARM64 ou multi-architecture
2. Vérifier les dépendances binaires spécifiques à x86 dans les conteneurs
3. Mettre à jour le type AMI à `AL2_ARM_64` dans la configuration du groupe de nœuds
4. Tester avec un petit groupe de nœuds avant de migrer tous les nœuds
5. Surveiller les performances de l'application pendant 48-72 heures après la migration

### Directives pour les instances Spot

**Bons candidats pour Spot :**
- Traitement par lots et pipelines de données
- Workers de build CI/CD
- Microservices stateless avec plusieurs réplicas
- Charges de travail de développement et de test
- Nœuds gérés par Karpenter (gestion native des interruptions)

**Éviter Spot pour :**
- Groupes gérés à nœud unique (pas de redondance lors du retrait)
- Charges de travail stateful (bases de données, files d'attente persistantes)
- Services critiques en production avec des SLAs stricts
- Charges de travail avec des temps d'initialisation longs

### Planification de la capacité

- JetScale utilise une tarification Spot conservatrice (minimum, pas la moyenne) pour les estimations de coûts
- La configuration de mise à l'échelle du groupe de nœuds (min/max/souhaité) est préservée dans les recommandations
- Les coûts du plan de contrôle sont inclus dans les totaux au niveau du cluster
- Les clusters avec support étendu (Kubernetes 1.26 et antérieur) entraînent des coûts de plan de contrôle plus élevés (~438 $/mois vs ~73 $/mois)

## Modèles d'optimisation courants

### Modèle 1 : Groupes de nœuds surprovisionnés

**Symptômes :**
- Utilisation CPU constamment inférieure à 30%
- Utilisation mémoire inférieure à 40%
- Les nœuds ont une capacité inutilisée significative

**Recommandation JetScale :**
- Dimensionner vers des types d'instances plus petits
- Économies typiques : 30-50%
- Risque : Faible (vérifier que l'utilisation pic tient dans la nouvelle capacité)

### Modèle 2 : Opportunité de migration Graviton

**Symptômes :**
- Exécution d'instances x86 (m5, c5, r5)
- Charges de travail conteneurisées (généralement compatibles ARM)
- Pas de dépendances binaires spécifiques à x86

**Recommandation JetScale :**
- Migrer vers les équivalents Graviton (m6g, c7g, r6g, t4g)
- Économies typiques : 10-40%
- Risque : Faible pour la plupart des charges de travail conteneurisées

### Modèle 3 : Workers de traitement par lots ON_DEMAND

**Symptômes :**
- Groupes de nœuds de traitement par lots ou CI/CD
- Charges de travail tolérantes aux pannes, stateless
- Exécution sur capacité ON_DEMAND

**Recommandation JetScale :**
- Basculer vers la capacité Spot
- Économies typiques : 60-90%
- Risque : Moyen (Spot peut être repris, les jobs doivent être redémarrables)

### Modèle 4 : Version Kubernetes héritée

**Symptômes :**
- Version Kubernetes 1.26 ou antérieure
- Frais de support étendu (~438 $/mois pour le plan de contrôle)
- Les clusters en support standard coûtent ~73 $/mois

**Observation JetScale :**
- Le support étendu ajoute ~365 $/mois aux coûts du plan de contrôle
- La mise à niveau de la version Kubernetes réduit les coûts du plan de contrôle d'environ 83%
- JetScale inclut ce coût dans les totaux au niveau du cluster pour la visibilité

## Dépannage

### Préoccupations concernant les recommandations

**Q : Le dimensionnement optimal causera-t-il un temps d'arrêt de l'application ?**

R : Les mises à jour de groupes de nœuds utilisent un remplacement progressif. Avec `max_unavailable = 1`, un nœud est remplacé à la fois. Les pods sont évincés et replanifiés sur les nœuds restants pendant la transition. Assurez-vous que vos déploiements ont des Pod Disruption Budgets (PDBs) configurés pour une gestion gracieuse.

**Q : Comment JetScale gère-t-il les nœuds gérés par Karpenter ?**

R : JetScale découvre les nœuds Karpenter en regroupant les instances EC2 qui partagent le même tag de cluster mais ne font pas partie d'un groupe de nœuds géré. Ceux-ci sont regroupés par tag NodePool et analysés comme des groupes de nœuds synthétiques. La gestion native des interruptions Spot de Karpenter en fait un excellent candidat pour l'optimisation Spot.

**Q : Que se passe-t-il si les métriques mémoire ne sont pas disponibles ?**

R : Si CloudWatch Container Insights n'est pas activé, JetScale n'aura pas de données d'utilisation mémoire. Dans ce cas, les recommandations qui réduisent la capacité mémoire de plus de 25% sont rejetées par mesure de sécurité. L'activation de Container Insights fournit des opportunités d'optimisation plus précises et plus agressives.

**Q : Puis-je annuler un changement de groupe de nœuds ?**

R : Oui. Mettez à jour la configuration du groupe de nœuds pour revenir au type d'instance et à la taille souhaitée précédents. Le processus de mise à jour progressive remplacera les nœuds avec la configuration d'origine.

### Problèmes de performances après optimisation

**Symptôme : Pods en état Pending après le dimensionnement optimal**

Causes possibles :
- Le nouveau type d'instance a une capacité CPU ou mémoire insuffisante pour les resource requests des pods
- La capacité du nœud ne correspond pas aux exigences de planification des pods

**Résolution :**
1. Vérifier les resource requests des pods par rapport à la nouvelle capacité du nœud
2. Vérifier que `kubectl describe node` affiche les ressources allouables
3. Ajuster les requests/limits des pods ou augmenter la taille du nœud si nécessaire
4. Envisager d'ajouter plus de nœuds au lieu de nœuds plus grands

**Symptôme : Les interruptions Spot causent des perturbations de service**

Causes possibles :
- Déploiements à réplica unique sur des nœuds Spot
- Pas de Pod Disruption Budget configuré
- Périodes d'arrêt gracieux longues

**Résolution :**
1. Exécuter plusieurs réplicas sur différents nœuds pour la redondance
2. Configurer des Pod Disruption Budgets
3. Utiliser Karpenter pour la gestion automatique des interruptions Spot
4. Envisager des groupes de nœuds mixtes ON_DEMAND + SPOT pour les services critiques

## Limitations

**Actuellement non pris en charge :**
- Clusters EKS Auto Mode (AWS gère le calcul)
- Optimisation des profils Fargate
- Ajustement de la configuration du Cluster Autoscaler
- Optimisation des ressources au niveau des pods (requests/limits)
- Orchestration multi-cluster
- Recommandations d'instances réservées pour les nœuds EKS

**Contraintes spécifiques à EKS :**
- Tous les nœuds d'un groupe de nœuds géré doivent utiliser le même type d'instance
- La migration Graviton nécessite des images de conteneurs compatibles ARM64
- L'optimisation Spot est rejetée pour les groupes gérés à nœud unique
- CloudWatch Container Insights est nécessaire pour les métriques mémoire

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations EKS
GET /api/v1/recommendations?resource_type=eks

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [Documentation API](../api-reference.md) pour une référence complète.

## Support

Besoin d'aide pour l'optimisation EKS ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Problèmes GitHub** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation associée :**
- [Optimisation EC2](ec2.md)
- [Optimisation RDS](rds.md)
- [Optimisation EBS](ebs.md)
- [Optimisation ElastiCache](elasticache.md)
- [Optimisation S3](s3.md)