# Optimisation EBS

JetScale fournit une optimisation des coûts pilotée par l'IA pour les volumes Amazon EBS (Elastic Block Store). Nos agents spécialisés analysent vos charges de travail de stockage pour identifier les opportunités d'optimisation du type de volume, de réglage des IOPS/débit et de nettoyage des volumes inutilisés.

## Vue d'ensemble

JetScale optimise les volumes EBS en analysant :
- **Utilisation des volumes** : Modèles d'utilisation des IOPS et du débit
- **Efficacité du type de volume** : Rentabilité du type de volume actuel
- **État de rattachement** : Identifier les volumes inutilisés, non rattachés
- **Analyse des coûts** : Dépenses actuelles vs. configuration optimale

## Types EBS pris en charge

### Volumes EBS

JetScale optimise les volumes EBS rattachés aux instances EC2 ou les volumes non rattachés dans votre compte.

**Ce que nous optimisons :**
- **Migration du type de volume** : gp2 → gp3 (20% d'économies), io1 → io2 (meilleure durabilité)
- **Optimisation des IOPS** : Dimensionnement optimal des IOPS provisionnées en fonction de l'utilisation réelle
- **Optimisation du débit** : Ajustement du débit gp3 pour correspondre aux besoins réels
- **Suppression des volumes non rattachés** : Suppression des volumes inutilisés pour 100% d'économies

**Types de volumes pris en charge :**
- **gp3** (SSD usage général) - Dernière génération, meilleur rapport qualité-prix
- **gp2** (SSD usage général) - Génération précédente
- **io2** (SSD IOPS provisionnées) - Hautes performances, 99,999% de durabilité
- **io1** (SSD IOPS provisionnées) - Ancienne génération
- **st1** (HDD optimisé pour le débit) - Faible coût, charges intensives en débit
- **sc1** (HDD froid) - Coût le plus bas, accès peu fréquents

## Types de volumes EBS

### SSD usage général (gp3)

**Idéal pour :** La plupart des charges de travail
- **Performances de base** : 3 000 IOPS, débit de 125 MB/s
- **Évolutif** : Jusqu'à 16 000 IOPS, débit de 1 000 MB/s
- **Tarification** : Paiement pour le stockage + IOPS/débit supplémentaires optionnels
- **Avantage de coût** : ~20% moins cher que gp2 pour des performances équivalentes

**Cas d'utilisation :**
- Volumes de démarrage
- Postes de travail virtuels
- Environnements de développement et de test
- Applications interactives à faible latence

### SSD usage général (gp2)

**Type de volume hérité**
- **Performances** : 3 IOPS par Go (minimum 100 IOPS, maximum 16 000 IOPS)
- **Rafale** : Peut atteindre 3 000 IOPS en utilisant des crédits de rafale
- **Migration recommandée** : Presque toujours préférable de migrer vers gp3

**Pourquoi migrer vers gp3 :**
- 20% d'économies sur le stockage
- Personnalisation des IOPS indépendamment de la taille
- Pas de gestion des crédits de rafale nécessaire

### SSD IOPS provisionnées (io2)

**Idéal pour :** Charges de travail critiques, intensives en E/S
- **Performances** : Jusqu'à 64 000 IOPS (256 000 avec Block Express)
- **Durabilité** : 99,999% (vs. 99,8-99,9% pour gp3)
- **Latence** : Latence inférieure à la milliseconde
- **Tarification** : Paiement pour le stockage + IOPS provisionnées

**Cas d'utilisation :**
- Grandes bases de données relationnelles (Oracle, SAP HANA)
- Applications critiques nécessitant la plus haute durabilité
- Applications nécessitant des performances IOPS soutenues

### SSD IOPS provisionnées (io1)

**Volume hautes performances hérité**
- **Performances** : Jusqu'à 64 000 IOPS
- **Durabilité** : 99,8-99,9% (inférieure à io2)
- **Migration recommandée** : Migrer vers io2 pour une meilleure durabilité au même coût

### HDD optimisé pour le débit (st1)

**Idéal pour :** Charges de travail intensives en débit
- **Performances** : Débit max de 500 MB/s, 500 IOPS max
- **Tarification** : Coût inférieur aux options SSD
- **Ne peut pas être un volume de démarrage**

**Cas d'utilisation :**
- Traitement de big data
- Entrepôts de données
- Traitement de journaux
- Charges de travail d'E/S séquentielles importantes

### HDD froid (sc1)

**Idéal pour :** Données rarement consultées
- **Performances** : Débit max de 250 MB/s, 250 IOPS max
- **Tarification** : Coût le plus bas par Go
- **Ne peut pas être un volume de démarrage**

**Cas d'utilisation :**
- Stockage d'archives
- Données rarement consultées
- Stockage optimisé en coût pour de grands ensembles de données

## Comment JetScale optimise EBS

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques CloudWatch** (période historique) :
- `VolumeReadOps` / `VolumeWriteOps` - Utilisation des IOPS
- `VolumeReadBytes` / `VolumeWriteBytes` - Utilisation du débit
- `VolumeThroughputPercentage` - Pourcentage du débit provisionné utilisé
- `VolumeConsumedReadWriteOps` - Total des IOPS consommées (gp2/gp3)
- `BurstBalance` - Solde de crédits de rafale (gp2 uniquement)

**Données de l'API EBS** :
- Type de volume (gp3, gp2, io2, io1, st1, sc1)
- Taille en Go
- IOPS provisionnées
- Débit provisionné (gp3 uniquement)
- État de rattachement (rattaché vs. non rattaché)
- ID de l'instance EC2 rattachée
- État du volume (available, in-use, creating, deleting)

**Données de Cost Explorer** :
- Dépenses mensuelles actuelles par volume
- Tendances des coûts historiques

**AWS Compute Optimizer** (si activé) :
- Recommandations EBS générées par AWS
- Évaluations des risques de performances

### 2. Analyse

Nos agents IA effectuent une analyse approfondie :

**Modèles d'utilisation :**
- Utilisation des IOPS pic vs. moyenne
- Utilisation du débit pic vs. moyenne
- Modèles horaires
- Tendances du solde de crédits de rafale (gp2)

**Modélisation des coûts :**
- Répartition des coûts actuels (stockage + IOPS + débit)
- Coût projeté pour les types de volumes alternatifs
- Économies grâce à la réduction des IOPS/débit
- Coût des volumes non rattachés

**Vérifications de sécurité :**
- Vérifier que le volume n'est pas rattaché avant de recommander la suppression
- S'assurer que les réductions d'IOPS/débit n'impactent pas les performances
- Valider les chemins de migration (gp2→gp3, io1→io2)

### 3. Recommandations

JetScale génère des recommandations spécifiques et actionnables :

#### Migration gp2 vers gp3

**Exemple de recommandation :**
```
Ressource : vol-0abc123 (volume racine du serveur web)
Actuel : gp2, 100 Go
Recommandé : gp3, 100 Go, 3 000 IOPS, 125 MB/s

Impact sur les coûts :
- Actuel : 10 $/mois
- Projeté : 8 $/mois
- Économies : 2 $/mois (20%), 24 $/an

Analyse des performances :
- IOPS actuelles : 300 de base (3 IOPS/Go)
- Utilisation des IOPS pic : 250
- Base gp3 : 3 000 IOPS (dépasse largement les besoins actuels)

Risque : Très faible - Amélioration des performances avec économies
Note : Peut être effectué en ligne sans détacher le volume
```

#### Dimensionnement optimal des IOPS (io2)

**Exemple de recommandation :**
```
Ressource : vol-0def456 (volume de base de données)
Actuel : io2, 500 Go, 10 000 IOPS
Recommandé : io2, 500 Go, 5 000 IOPS

Impact sur les coûts :
- Actuel : 40 $/mois (stockage) + 650 $/mois (IOPS) = 690 $/mois
- Projeté : 40 $/mois (stockage) + 325 $/mois (IOPS) = 365 $/mois
- Économies : 325 $/mois (47%), 3 900 $/an

Analyse des performances :
- Provisionnées : 10 000 IOPS
- Utilisation pic : 3 200 IOPS
- Utilisation moyenne : 1 800 IOPS
- Recommandé : 5 000 IOPS (56% de marge au-dessus du pic)

Risque : Faible - Marge significative au-dessus de l'utilisation pic maintenue
```

#### Optimisation du débit gp3

**Exemple de recommandation :**
```
Ressource : vol-0ghi789 (volume de stockage des journaux)
Actuel : gp3, 1 000 Go, 3 000 IOPS, 500 MB/s
Recommandé : gp3, 1 000 Go, 3 000 IOPS, 250 MB/s

Impact sur les coûts :
- Actuel : 80 $/mois (stockage) + 40 $/mois (débit supplémentaire) = 120 $/mois
- Projeté : 80 $/mois (stockage) + 10 $/mois (débit supplémentaire) = 90 $/mois
- Économies : 30 $/mois (25%), 360 $/an

Analyse des performances :
- Provisionné : 500 MB/s
- Utilisation pic : 180 MB/s
- Utilisation moyenne : 95 MB/s
- Recommandé : 250 MB/s (39% de marge au-dessus du pic)

Risque : Faible - Marge adéquate au-dessus de l'utilisation pic
Note : Le débit peut être ajusté en ligne
```

#### Migration io1 vers io2

**Exemple de recommandation :**
```
Ressource : vol-0jkl012 (base de données de production)
Actuel : io1, 1 000 Go, 20 000 IOPS
Recommandé : io2, 1 000 Go, 20 000 IOPS

Impact sur les coûts :
- Actuel : 125 $/mois (stockage) + 1 300 $/mois (IOPS) = 1 425 $/mois
- Projeté : 125 $/mois (stockage) + 1 300 $/mois (IOPS) = 1 425 $/mois
- Économies : 0 $/mois (même coût, meilleure durabilité)

Analyse des performances :
- Mêmes performances IOPS (20 000 IOPS)
- Mêmes caractéristiques de débit
- Durabilité : 99,8-99,9% (io1) → 99,999% (io2)
- Mieux adapté aux charges de travail critiques

Risque : Très faible - Même coût, meilleure durabilité
Bénéfice : Taux de défaillance annuel 10 à 100 fois inférieur
```

#### Suppression d'un volume non rattaché

**Exemple de recommandation :**
```
Ressource : vol-0mno345 (legacy-backup-volume)
Actuel : gp2, 500 Go, non rattaché
Recommandé : Supprimer

Impact sur les coûts :
- Actuel : 50 $/mois
- Projeté : 0 $/mois
- Économies : 50 $/mois (100%), 600 $/an

Analyse du volume :
- État : Disponible (non rattaché à aucune instance)
- Dernier rattachement : >180 jours
- Créé : Il y a 2 ans
- Tags : Aucun tag de production trouvé

Risque : Faible - Le volume est non rattaché et inutilisé
Recommandation : Vérifier avec l'équipe avant suppression, créer un snapshot si nécessaire
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Migration gp2 vers gp3**

```hcl
# Optimisation de volume EBS : gp2 → gp3
# Généré par JetScale le 2024-01-15
# ID de recommandation : rec_ebs_001

resource "aws_ebs_volume" "web_server_root" {
  availability_zone = "us-east-1a"

  # Précédent : gp2, 100 Go (10 $/mois)
  # Optimisé : gp3, 100 Go (8 $/mois)
  # Réduction des coûts : 20% (2 $/mois, 24 $/an)
  #
  # Amélioration des performances :
  # - gp2 : 300 IOPS de base (3 IOPS/Go)
  # - gp3 : 3 000 IOPS de base (amélioration x10)
  # - Utilisation pic : 250 IOPS (bien dans la base gp3)
  type = "gp3"
  size = 100

  # Performances de base gp3 (sans coût supplémentaire)
  iops       = 3000
  throughput = 125

  encrypted  = true
  kms_key_id = var.kms_key_id

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:recommendation" = "rec_ebs_001"
      "jetscale:previous_type"  = "gp2"
    }
  )
}

# Rattachement préservé
resource "aws_volume_attachment" "web_server_root" {
  device_name = "/dev/sda1"
  volume_id   = aws_ebs_volume.web_server_root.id
  instance_id = var.instance_id
}
```

**Exemple : Dimensionnement optimal des IOPS**

```hcl
# Optimisation IOPS du volume EBS
# Généré par JetScale le 2024-01-15

resource "aws_ebs_volume" "database_volume" {
  availability_zone = "us-east-1b"

  type = "io2"
  size = 500

  # Précédent : 10 000 IOPS (650 $/mois pour les IOPS)
  # Optimisé : 5 000 IOPS (325 $/mois pour les IOPS)
  # Réduction des coûts : 50% sur le coût des IOPS (325 $/mois, 3 900 $/an)
  #
  # Analyse d'utilisation :
  # - IOPS pic : 3 200
  # - IOPS moyenne : 1 800
  # - Recommandé : 5 000 IOPS (56% de marge au-dessus du pic)
  iops = 5000

  encrypted  = true
  kms_key_id = var.kms_key_id

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:previous_iops"  = "10000"
    }
  )
}
```

## Bonnes pratiques

### Stratégie de test

**Tests en pré-production :**
1. Tester les modifications de volumes en dev/staging d'abord
2. Surveiller les performances pendant 48-72 heures
3. Valider les modèles d'E/S de l'application
4. Vérifier les performances des requêtes de la base de données (le cas échéant)

**Déploiement en production :**
1. Planifier les changements pendant les fenêtres de maintenance
2. Les modifications de volumes peuvent être effectuées en ligne (sans temps d'arrêt)
3. Certaines modifications peuvent impacter les performances pendant la migration
4. Surveiller activement les métriques CloudWatch pendant le changement

### Migration du type de volume

**Migration gp2 → gp3 :**
- Peut être effectuée en ligne sans détachement
- Peut subir un impact de performance bref pendant la migration
- Presque toujours recommandée (20% d'économies, meilleures performances)

**Migration io1 → io2 :**
- Peut être effectuée en ligne
- Même coût, meilleure durabilité (99,999% vs. 99,8-99,9%)
- Fortement recommandée pour les charges de travail critiques

### Nettoyage des volumes non rattachés

**Avant de supprimer les volumes non rattachés :**
1. Vérifier que le volume est vraiment inutilisé
2. Vérifier les tags pour les informations de propriété
3. Créer un snapshot si les données peuvent être nécessaires
4. Confirmer avec l'équipe applicative
5. Attendre une période de rétention appropriée après la création du snapshot avant suppression

**Stratégie de snapshot :**
```bash
# Créer un snapshot avant suppression
aws ec2 create-snapshot \
  --volume-id vol-0abc123 \
  --description "Sauvegarde avant suppression - Volume de sauvegarde hérité"

# Attendre la fin du snapshot
aws ec2 wait snapshot-completed \
  --snapshot-ids snap-0xyz789

# Supprimer le volume après une période de rétention appropriée
aws ec2 delete-volume --volume-id vol-0abc123
```

## Modèles d'optimisation courants

### Modèle 1 : Volumes gp2 hérités

**Symptômes :**
- Utilisation de volumes gp2
- gp3 disponible dans la région
- Aucune exigence de rafale spéciale

**Recommandation JetScale :**
- Migrer tous les gp2 → gp3
- Économies typiques : 20% sur les coûts de stockage
- Amélioration des performances : IOPS de base plus élevées (3 000 vs. 3 IOPS/Go)
- Risque : Très faible (peut être effectué en ligne)

### Modèle 2 : IOPS surprovisionnées

**Symptômes :**
- Utilisation de volumes io2 ou io1
- Utilisation des IOPS pic < 50% des provisionnées
- Coûts IOPS élevés par rapport aux coûts de stockage

**Recommandation JetScale :**
- Réduire les IOPS provisionnées à 2x l'utilisation pic
- Économies typiques : 30-50% sur les coûts IOPS
- Risque : Faible (maintien d'une marge de 2x)

### Modèle 3 : Accumulation de volumes non rattachés

**Symptômes :**
- Plusieurs volumes non rattachés dans le compte
- Volumes provenant d'instances terminées
- Anciens volumes de sauvegarde plus nécessaires

**Recommandation JetScale :**
- Créer des snapshots si nécessaire
- Supprimer les volumes non rattachés
- Économies typiques : 100% sur les volumes supprimés
- Risque : Faible si correctement vérifié

### Modèle 4 : IOPS/débit gp3 inutiles

**Symptômes :**
- Volumes gp3 avec >3 000 IOPS ou >125 MB/s de débit
- Utilisation réelle en dessous des performances de base
- Paiement pour des performances inutilisées

**Recommandation JetScale :**
- Réduire à la base (3 000 IOPS, 125 MB/s)
- Économies typiques : 10-30% sur les coûts de volume
- Risque : Très faible si l'utilisation est bien en dessous de la base

## Dépannage

### Préoccupations concernant les recommandations

**Q : La migration de gp2 vers gp3 causera-t-elle un temps d'arrêt ?**

R : Non. Les modifications de volumes EBS sont effectuées en ligne sans détacher le volume. Le volume reste disponible pendant la modification, bien qu'il puisse y avoir de brefs impacts sur les performances pendant le processus de migration.

**Q : Combien de temps prend une modification de volume ?**

R : Les modifications de volumes se terminent généralement en quelques minutes à quelques heures selon :
- La taille du volume
- La charge d'E/S actuelle
- Le type de modification (changement de type, changement d'IOPS, changement de taille)

Vous pouvez surveiller la progression via la Console AWS ou la CLI.

**Q : Puis-je annuler une modification de volume ?**

R : Vous ne pouvez pas annuler directement, mais vous pouvez modifier à nouveau le volume une fois la modification actuelle terminée. Il y a une période de refroidissement (généralement 6 heures) entre les modifications.

**Q : Que se passe-t-il si je réduis trop les IOPS ?**

R : Vous pouvez augmenter à nouveau les IOPS après la fin de la modification. Surveillez les métriques CloudWatch pour :
- `VolumeQueueLength` (doit rester faible)
- `VolumeThroughputPercentage` (ne doit pas atteindre 100%)
- Métriques de latence de l'application

### Problèmes de performances après optimisation

**Symptôme : Latence accrue de l'application après réduction des IOPS**

Causes possibles :
- IOPS réduites en dessous des besoins pic réels
- `VolumeQueueLength` en augmentation
- Application subissant des attentes d'E/S

**Résolution :**
1. Vérifier CloudWatch `VolumeReadOps` et `VolumeWriteOps`
2. Vérifier la métrique `VolumeQueueLength`
3. Augmenter les IOPS si la longueur de la file est systématiquement > 0
4. Autoriser une période de refroidissement de 6 heures entre les modifications

**Symptôme : Épuisement des crédits de rafale gp2 après migration**

Causes possibles :
- Migration vers gp3 mais la charge de travail nécessite encore une capacité de rafale
- IOPS de base insuffisantes pour la charge de travail

**Résolution :**
- gp3 n'utilise pas de crédits de rafale - les performances sont constantes
- En cas de problèmes de performances, augmenter les IOPS gp3 au-dessus de la base de 3 000
- Coût : 0,065 $ par IOPS provisionnée par mois

## Considérations de sécurité

### Chiffrement

Les recommandations JetScale préservent :
- Le statut de chiffrement EBS
- Les associations de clés KMS
- Les paramètres de chiffrement en transit

### Rattachements de volumes

- Volumes rattachés : Modifications uniquement de type/IOPS/débit
- Volumes non rattachés : Recommandations de suppression (avec vérifications de sécurité)
- Rattachements de volumes préservés dans Terraform

### Snapshots

Avant de supprimer des volumes :
- Toujours créer un snapshot si les données peuvent être nécessaires
- Taguer les snapshots de manière appropriée
- Définir des politiques de cycle de vie des snapshots
- Envisager des copies de snapshots vers d'autres régions pour la reprise après sinistre

## Limitations

**Actuellement non pris en charge :**
- Optimisation des snapshots (politiques de cycle de vie, copies inter-régions)
- Volumes multi-attachement
- Réduction de la taille des volumes (limitation AWS)
- Optimisation de la restauration snapshot-vers-volume

**Limites de modification de volumes :**
- Période de refroidissement de 6 heures entre les modifications
- Impossible de modifier les volumes de <6 heures
- Impossible de diminuer la taille du volume (limitation AWS)

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations EBS
GET /api/v1/recommendations?resource_type=ebs

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [Documentation API](../api-reference.md) pour une référence complète.

## Support

Besoin d'aide pour l'optimisation EBS ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Problèmes GitHub** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation associée :**
- [Optimisation EC2](ec2.md)
- [Optimisation RDS](rds.md)
- [Optimisation ElastiCache](elasticache.md)
