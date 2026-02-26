# Services pris en charge

JetScale fournit une optimisation des coûts basée sur l'IA pour les services cloud les plus critiques sur AWS et Azure.

---

## Services AWS

### Amazon RDS (Relational Database Service)
Optimisez les tailles d'instances de base de données, les configurations de stockage et les achats d'instances réservées.

**Domaines d'optimisation :**
- Dimensionnement approprié des classes d'instance
- Optimisation du type de stockage (gp2 → gp3)
- Révision de la configuration Multi-AZ
- Recommandations d'instances réservées
- Nettoyage des snapshots

[En savoir plus →](services/rds.md)

---

### Amazon EC2 (Elastic Compute Cloud)
Dimensionnez correctement les instances de calcul, identifiez les ressources inactives et optimisez l'achat d'instances.

**Domaines d'optimisation :**
- Recommandations de type et taille d'instance
- Détection des instances inactives
- Opportunités de migration Graviton
- Instances réservées et Savings Plans
- Planification arrêt/démarrage

[En savoir plus →](services/ec2.md)

---

### Amazon EBS (Elastic Block Store)
Optimisez les volumes de stockage avec des recommandations de type et des ajustements de capacité.

**Domaines d'optimisation :**
- Optimisation du type de volume (gp2 → gp3)
- Réglage IOPS et débit
- Gestion du cycle de vie des snapshots
- Détection des volumes inutilisés
- Migration vers stockage froid

[En savoir plus →](services/ebs.md)

---

### Amazon ElastiCache
Optimisez les clusters Redis et Memcached pour le coût et la performance.

**Domaines d'optimisation :**
- Dimensionnement approprié du type de nœud
- Configuration du cluster
- Recommandations de nœuds réservés
- Révision Multi-AZ
- Optimisation de la politique d'éviction

[En savoir plus →](services/elasticache.md)

### Amazon S3 (Simple Storage Service)
Optimisez les coûts de stockage avec le tiering, les politiques de cycle de vie, l'optimisation du chiffrement et l'amélioration du routage réseau.

**Domaines d'optimisation :**
- Activation d'Intelligent-Tiering
- Configuration de règles de cycle de vie (Standard → IA → Glacier → Deep Archive)
- Bucket Key pour réduction des coûts KMS
- S3 Gateway Endpoints pour éliminer les frais de traitement NAT

[En savoir plus →](services/s3.md)

### Amazon EKS (Elastic Kubernetes Service)
Dimensionnez les groupes de nœuds, migrez vers Graviton et optimisez la capacité Spot sur vos clusters Kubernetes.

**Domaines d'optimisation :**
- Dimensionnement du type de nœud par groupe de nœuds
- Migration Graviton (ARM64) pour 10-40% d'économies
- Capacité Spot pour les charges de travail tolérantes aux pannes (60-90% d'économies)
- Détection des clusters et groupes de nœuds inactifs
- Analyse par groupe de nœuds avec agrégation au niveau du cluster

[En savoir plus →](services/eks.md)

## Services Azure

### Azure Virtual Machines
Dimensionnez correctement les VM, optimisez les types d'instance et exploitez la capacité réservée.

**Domaines d'optimisation :**
- Recommandations de taille de VM
- Instances B-series burstables
- Instances réservées Azure VM
- Opportunités d'instances spot
- Planification d'arrêt

[En savoir plus →](services/azure-vm.md)

---

### Azure SQL Database
Optimisez les niveaux de base de données, l'allocation DTU et les configurations de pools élastiques.

**Domaines d'optimisation :**
- Recommandations de niveau de service
- Dimensionnement DTU et vCore
- Optimisation des pools élastiques
- Achats de capacité réservée
- Optimisation du stockage

[En savoir plus →](services/azure-sql.md)

---

## Bientôt disponible

Nous développons activement la prise en charge de services supplémentaires :

### AWS
- **AWS Lambda** - Optimisation de la mémoire et du timeout
- **Amazon DynamoDB** - Recommandations de mode de capacité
- **Amazon ECS** - Dimensionnement approprié des conteneurs
- **Elastic Load Balancing** - Optimisation du type de load balancer
- **Amazon CloudFront** - Optimisation de la distribution

### Azure
- **Azure Blob Storage** - Optimisation du niveau de stockage
- **Azure Functions** - Optimisation du plan et de la mémoire
- **Azure Cosmos DB** - Recommandations de débit et capacité
- **Azure Cache for Redis** - Optimisation du niveau et de la capacité
- **Azure Kubernetes Service** - Optimisation des pools de nœuds
- **Azure App Service** - Recommandations de niveau de plan

---

## Comment fonctionne l'analyse des services

Pour chaque service pris en charge, JetScale :

1. **Découvre les ressources** : Identifie automatiquement toutes les instances sur vos comptes cloud
2. **Collecte les métriques** : Rassemble l'historique de données d'utilisation (CPU, mémoire, disque, réseau)
3. **Analyse IA** : Des agents spécialisés analysent les modèles et identifient les opportunités d'optimisation
4. **Valide la sécurité** : Garantit que les recommandations maintiennent la performance et la disponibilité
5. **Génère du code** : Crée du Terraform prêt pour la production pour l'implémentation

---

## Critères d'optimisation

Chaque recommandation est validée selon :

- **Performance** : Maintient ou améliore les niveaux de performance actuels
- **Disponibilité** : Préserve les configurations haute disponibilité
- **Marge** : Inclut une réserve pour les pics de trafic et la croissance
- **Sécurité** : Retour en arrière facile si nécessaire
- **ROI** : Seuil minimum de réduction des coûts de 20%

---

## Support

Des questions sur les services pris en charge ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Demandes de fonctionnalités** : [Demander un service](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

*© 2025 JetScale, Inc. Tous droits réservés.*
