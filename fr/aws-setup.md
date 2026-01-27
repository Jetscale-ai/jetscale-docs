# Guide de Configuration AWS

Ce guide vous accompagnera dans la connexion sécurisée de votre compte AWS à JetScale en utilisant un accès basé sur un rôle IAM.

## Prérequis

Avant de commencer, assurez-vous d'avoir:

1. **Activé AWS Cost Optimization Hub**
   Suivez le [Guide de Démarrage AWS Cost Optimization Hub](https://aws.amazon.com/aws-cost-management/cost-optimization-hub/getting-started/)

2. **Activé AWS Compute Optimizer**
   Suivez le [Guide de Démarrage AWS Compute Optimizer](https://aws.amazon.com/compute-optimizer/getting-started/#topic-0)

3. **Accès Administrateur au Compte AWS**
   Vous aurez besoin des permissions pour créer des rôles et des politiques IAM

## Vue d'ensemble

JetScale se connecte à votre compte AWS en utilisant un **rôle IAM inter-comptes** avec des permissions en lecture seule. Cette approche:

- Ne nécessite pas de clés d'accès AWS ou de mots de passe
- Fournit un accès sécurisé et auditable via AWS STS (Security Token Service)
- Utilise un External ID pour une sécurité supplémentaire contre les attaques de type "confused deputy"
- Peut être révoqué instantanément en supprimant le rôle IAM
- Suit les meilleures pratiques de sécurité AWS

## Processus de Configuration

### Étape 1: Générer un External ID

L'External ID est un identifiant unique qui ajoute une couche de sécurité supplémentaire à l'assumption de rôle inter-comptes.

**Vous avez deux options:**

#### Option A: Nous le Générons pour Vous (Recommandé)
JetScale générera un External ID sécurisé et le partagera avec vous lors de l'intégration.

#### Option B: Vous le Générez
Si vous préférez générer votre propre External ID, utilisez l'une de ces méthodes:

```bash
# Générer un UUID (le plus courant)
uuidgen

# Générer une chaîne hexadécimale aléatoire
openssl rand -hex 32

# Générer une chaîne aléatoire encodée en base64
openssl rand -base64 32
```

**Note de Sécurité:** Si vous générez votre propre External ID, veuillez le partager avec nous par email chiffré. Partagez le mot de passe de déchiffrement via un canal de communication séparé.

### Étape 2: Créer le Rôle IAM

1. **Accéder à la Console IAM**
   Allez dans Console AWS → IAM → Roles → Create Role

2. **Sélectionner l'Entité de Confiance**
   - Choisissez "AWS account"
   - Sélectionnez "Another AWS account"
   - Entrez l'**ID de Compte JetScale** (fourni par l'équipe JetScale)

3. **Configurer la Politique de Confiance**

   Ajoutez l'External ID à votre relation de confiance:

   ```json
   {
     "Version": "2012-10-17",
     "Statement": [
       {
         "Effect": "Allow",
         "Principal": {
           "AWS": "arn:aws:iam::[JETSCALE_ACCOUNT_ID]:root"
         },
         "Action": "sts:AssumeRole",
         "Condition": {
           "StringEquals": {
             "sts:ExternalId": "[YOUR_UNIQUE_EXTERNAL_ID]"
           }
         }
       }
     ]
   }
   ```

4. **Nommer Votre Rôle**
   - Nom du rôle: `jetscale-readonly-access` (ou votre nom préféré)
   - Description: "Accès en lecture seule JetScale pour l'optimisation des coûts"

### Étape 3: Attacher les Permissions

JetScale nécessite un accès en lecture seule pour analyser votre infrastructure et vos coûts. La politique de permissions inclut:

**Services Principaux:**
- **Gestion des Coûts**: Cost Explorer, Cost Optimization Hub, Compute Optimizer
- **EC2**: Métadonnées d'instance, volumes, informations de tarification
- **RDS**: Instances de base de données, clusters et configuration
- **S3**: Informations et analyses des buckets
- **ElastiCache**: Informations sur les clusters de cache
- **CloudWatch**: Métriques et données de surveillance

**Considérations de Sécurité:**
- Toutes les permissions sont en lecture seule (actions Describe, Get, List uniquement)
- Aucune permission d'écriture, de modification ou de suppression
- Aucun accès aux données réelles stockées dans les bases de données ou les buckets S3
- Aucun accès aux secrets ou aux informations d'identification
- Limité aux métadonnées et aux informations de configuration

Lors de l'intégration, l'équipe JetScale fournira le document de politique IAM complet et aidera à valider les permissions.

### Étape 4: Partager l'ARN du Rôle

Une fois le rôle créé:

1. Copiez l'**ARN du Rôle** (format: `arn:aws:iam::123456789012:role/jetscale-readonly-access`)
2. Partagez-le avec votre contact JetScale ainsi que:
   - ID du Compte AWS
   - External ID (si vous l'avez généré vous-même)
   - Région AWS (si vous souhaitez limiter la portée)

## Validation

L'équipe JetScale validera la connexion lors de l'intégration en:

1. Tentant d'assumer le rôle
2. Exécutant des requêtes en lecture seule sur les services clés
3. Confirmant l'accès aux données pour l'analyse des coûts

Si des permissions sont manquantes, nous travaillerons avec vous pour les résoudre avant de continuer.

## Meilleures Pratiques de Sécurité

### Révisions Régulières
- Consultez régulièrement les journaux CloudTrail pour auditer les appels API de JetScale
- Configurez des alarmes CloudWatch pour les activités inhabituelles

### Moindre Privilège
- La politique fournie suit les principes de moindre privilège d'AWS
- Demandez des ajustements de permissions si votre politique de sécurité l'exige

### Expiration du Rôle (Optionnel)
Envisagez d'ajouter une durée de session maximale au rôle:
- Recommandé: 12 heures
- Cela limite la validité des informations d'identification assumées

## Dépannage

### Problèmes Courants

**Erreurs "Access Denied" lors de la validation:**
- Vérifiez que l'External ID correspond exactement
- Confirmez que l'ID de Compte JetScale est correct
- Vérifiez que la politique de confiance du rôle autorise `sts:AssumeRole`

**Données de coûts manquantes:**
- Assurez-vous que Cost Explorer est activé (période d'activation de 24 heures)
- Confirmez que Cost Optimization Hub est activé
- Vérifiez que Compute Optimizer a le statut d'opt-in

**Problèmes d'accès régional:**
- JetScale analyse les ressources dans toutes les régions activées
- Assurez-vous que le rôle est global (pas spécifique à une région)

## Support

Besoin d'aide avec la configuration AWS?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Appels de Configuration**: Disponibles pour une assistance pratique
- **Documentation**: Exigences de permissions détaillées fournies lors de l'intégration

Tous les problèmes de permissions seront résolus pendant le processus de validation—nous sommes là pour vous aider!
