# Guide de Configuration Azure

Ce guide vous accompagnera dans la connexion sécurisée de votre abonnement Azure à JetScale en utilisant l'enregistrement d'application Azure avec authentification par service principal.

## Prérequis

Avant de commencer, assurez-vous d'avoir:

1. **Accès à l'Abonnement Azure**
   Vous aurez besoin du rôle Propriétaire ou Administrateur d'Accès Utilisateur sur l'abonnement

2. **Services Azure Activés**
   - Azure Cost Management
   - Azure Advisor
   - Azure Monitor

3. **Accès au Portail Azure ou à Azure CLI**
   Soit le Portail Azure, soit Azure CLI installé pour la configuration

## Vue d'ensemble

JetScale se connecte à votre abonnement Azure en utilisant un **Enregistrement d'Application (Service Principal)** avec des permissions RBAC en lecture seule. Cette approche:

- Ne nécessite pas d'informations d'identification utilisateur
- Fournit un accès sécurisé et auditable via Azure Active Directory
- Prend en charge l'authentification par certificat (la plus sécurisée) ou par secrets clients
- Peut être révoquée instantanément via Azure IAM
- Suit les meilleures pratiques de sécurité Microsoft

## Processus de Configuration

### Étape 1: Créer un Enregistrement d'Application

1. **Accéder à Microsoft Entra ID** (anciennement Azure Active Directory)
   Portail Azure → Microsoft Entra ID → Enregistrements d'applications

2. **Créer un Nouvel Enregistrement**
   - Cliquez sur "Nouvel enregistrement"
   - **Nom**: `jetscale-readonly-access`
   - **Types de comptes pris en charge**: Locataire unique
   - **URI de redirection**: Laisser vide
   - Cliquez sur "Inscrire"

3. **Enregistrer les Valeurs Importantes**

   Après la création, notez ces identifiants (vous les partagerez avec JetScale):
   - **ID d'application (client)**
   - **ID d'annuaire (locataire)**
   - **ID d'abonnement** (depuis la page Abonnements)

### Étape 2: Choisir la Méthode d'Authentification

JetScale prend en charge deux méthodes d'authentification. Choisissez celle qui correspond à votre politique de sécurité:

#### Option A: Authentification par Certificat (Recommandé)

L'authentification par certificat est l'option la plus sécurisée et recommandée pour les environnements de production.

**Générer un Certificat:**

```bash
# Générer un certificat auto-signé (valide 2 ans)
openssl req -x509 -newkey rsa:4096 -keyout jetscale-cert-key.pem \
  -out jetscale-cert-public.pem -days 730 -nodes \
  -subj "/CN=jetscale-read-access/O=VotreOrganisation/C=FR"

# Extraire le certificat public pour le téléchargement sur le Portail Azure
openssl x509 -in jetscale-cert-public.pem -out jetscale-cert.cer -outform DER
```

**Télécharger sur l'Enregistrement d'Application:**

1. Accédez à votre Enregistrement d'Application → Certificats et secrets → Certificats
2. Cliquez sur "Télécharger le certificat"
3. Sélectionnez `jetscale-cert.cer`
4. Ajoutez une description: "Certificat d'Accès JetScale"
5. Cliquez sur "Ajouter"
6. **Enregistrez l'Empreinte** affichée

#### Option B: Authentification par Secret Client

Si votre organisation préfère les secrets aux certificats:

1. Accédez à Enregistrement d'Application → Certificats et secrets
2. Cliquez sur "Nouveau secret client"
3. **Description**: `Accès JetScale`
4. **Expire**: Choisissez la durée (recommandé 24 mois ou moins selon votre politique de sécurité)
5. Cliquez sur "Ajouterh6. **Important**: Copiez la **Valeur** du secret immédiatement. Elle n'est affichée qu'une seule fois

### Étape 3: Attribuer les Permissions RBAC

JetScale nécessite un accès en lecture seule au niveau de l'abonnement. Vous pouvez choisir entre des rôles intégrés ou un rôle personnalisé:

#### Option A: Utiliser les Rôles Intégrés (Le Plus Simple)

Attribuez ces trois rôles intégrés à votre Enregistrement d'Application:

1. Accédez à **Abonnements** → Sélectionnez votre abonnement → **Contrôle d'accès (IAM)**
2. Cliquez sur **Ajouter** → **Ajouter une attribution de rôle**
3. Attribuez chacun des rôles suivants:

| Rôle | Portée | Objectif |
|------|--------|----------|
| **Lecteur** | Abonnement | Accès en lecture seule à toutes les ressources |
| **Lecteur Cost Management** | Abonnement | Accès aux données de coûts et d'utilisation |
| **Lecteur de Monitoring** | Abonnement | Accès aux métriques Azure Monitor |

Pour chaque rôle:
- Recherchez votre Enregistrement d'Application par nom (`jetscale-readonly-access`)
- Sélectionnez-le et cliquez sur "Enregistrer"

#### Option B: Créer un Rôle Personnalisé (Avancé)

Pour un contrôle granulaire, créez un rôle personnalisé avec des permissions spécifiques.

**Ce que le Rôle Personnalisé Inclut:**
- **Calcul**: Métadonnées VM, informations de dimensionnement, métriques de performance
- **Gestion des Coûts**: Analyse des coûts, données d'utilisation, informations de facturation
- **Stockage**: Métadonnées et analyses des comptes de stockage
- **Bases de Données**: Métadonnées Azure SQL, MySQL, PostgreSQL, MariaDB, CosmosDB
- **Mise en Cache**: Configuration Redis et Redis Enterprise
- **Réseau**: Réseaux virtuels, équilibreurs de charge, passerelles d'application
- **Surveillance**: Métriques et insights Azure Monitor
- **Advisor**: Recommandations Azure Advisor

**Garanties de Sécurité:**
- Toutes les actions sont en lecture seule
- Exclut explicitement l'accès aux clés de compte de stockage (`listkeys`, `regeneratekey`)
- Aucune permission d'écriture, de modification ou de suppression
- Aucun accès aux données réelles stockées dans les bases de données ou les comptes de stockage

L'équipe JetScale fournira la définition complète du rôle personnalisé lors de l'intégration.

### Étape 4: Partager les Informations d'Identification en Toute Sécurité

Partagez les informations suivantes avec JetScale:

**Informations Requises:**
- **ID de locataire** (ID d'annuaire)
- **ID d'abonnement**
- **ID d'application (client)**
- **Méthode d'authentification choisie** (certificat ou secret)
- **Empreinte du certificat** (si vous utilisez des certificats) OU **Secret client** (si vous utilisez des secrets)

**Meilleures Pratiques de Sécurité pour le Partage:**

#### Pour l'Authentification par Certificat:

1. Créez un fichier de métadonnées d'identification (ne contient pas la clé privée):
   ```json
   {
     "tenant_id": "[VOTRE_ID_LOCATAIRE]",
     "subscription_id": "[VOTRE_ID_ABONNEMENT]",
     "client_id": "[VOTRE_ID_CLIENT]",
     "auth_method": "certificate",
     "certificate_thumbprint": "[EMPREINTE_CERTIFICAT]"
   }
   ```

2. Chiffrez le certificat et le fichier de métadonnées:
   ```bash
   gpg --symmetric --cipher-algo AES256 jetscale-cert-key.pem
   gpg --symmetric --cipher-algo AES256 jetscale-azure-creds.json
   ```

3. Envoyez les fichiers chiffrés par email à votre contact JetScale
4. Partagez le mot de passe de déchiffrement via un canal séparé (par ex., Slack, téléphone)

#### Pour l'Authentification par Secret:

1. Créez un fichier d'identification chiffré:
   ```bash
   # Créer le fichier d'identification
   cat > jetscale-azure-creds.json <<EOF
   {
     "tenant_id": "[VOTRE_ID_LOCATAIRE]",
     "subscription_id": "[VOTRE_ID_ABONNEMENT]",
     "client_id": "[VOTRE_ID_CLIENT]",
     "auth_method": "secret",
     "client_secret": "[VOTRE_SECRET_CLIENT]"
   }
   EOF

   # Chiffrer avec GPG
   gpg --symmetric --cipher-algo AES256 jetscale-azure-creds.json
   ```

2. Envoyez le fichier chiffré par email à votre contact JetScale
3. Partagez le mot de passe de déchiffrement via un canal séparé

## Validation

Vous pouvez tester les informations d'identification vous-même avant de les partager:

**Pour l'Authentification par Certificat:**
```bash
az login --service-principal \
  -u [CLIENT_ID] \
  --certificate jetscale-cert-key.pem \
  --tenant [TENANT_ID]

az account show
```

**Pour l'Authentification par Secret:**
```bash
az login --service-principal \
  -u [CLIENT_ID] \
  -p [CLIENT_SECRET] \
  --tenant [TENANT_ID]

az account show
```

L'équipe JetScale validera également l'accès lors de l'intégration en:
1. S'authentifiant avec les informations d'identification fournies
2. Exécutant des requêtes en lecture seule sur les services Azure clés
3. Confirmant l'accès aux données pour l'analyse des coûts

## Meilleures Pratiques de Sécurité

### Rotation Régulière des Informations d'Identification
- **Certificats**: Rotation avant l'expiration de 2 ans
- **Secrets**: Rotation tous les 12-24 mois selon votre politique de sécurité

### Surveillance des Activités
- Consultez les journaux d'activité Azure pour l'activité du service principal
- Configurez des alertes pour les modèles d'API inhabituels

### Moindre Privilège
- Commencez avec des rôles intégrés; passez aux rôles personnalisés si nécessaire
- Révisez régulièrement les permissions attribuées

### Accès Conditionnel (Optionnel)
Envisagez d'ajouter des politiques d'accès conditionnel:
- Restrictions IP pour le service principal
- Politiques d'accès basées sur le temps

## Dépannage

### Problèmes Courants

**Erreurs "Authentification échouée":**
- Vérifiez que l'ID de locataire et l'ID de client sont corrects
- Confirmez que l'empreinte du certificat correspond (si vous utilisez des certificats)
- Vérifiez que le secret n'a pas expiré (si vous utilisez des secrets)

**Erreurs "Permissions insuffisantes":**
- Assurez-vous que les trois rôles intégrés sont attribués (Lecteur, Lecteur Cost Management, Lecteur de Monitoring)
- Vérifiez que les attributions de rôles sont au niveau de l'abonnement, pas au niveau du groupe de ressources

**Données de coûts manquantes:**
- Les données de coûts peuvent prendre 24-48 heures pour apparaître dans Cost Management
- Assurez-vous que Cost Management est activé pour votre abonnement

**Accès multi-abonnement:**
- JetScale peut analyser plusieurs abonnements
- Attribuez le même Enregistrement d'Application à des abonnements supplémentaires avec les mêmes rôles RBAC

## Support

Besoin d'aide avec la configuration Azure?

- **Email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Appels de Configuration**: Disponibles pour une assistance pratique
- **Documentation**: Exigences de permissions détaillées fournies lors de l'intégration

Tous les problèmes de permissions seront résolus pendant le processus de validation. Nous sommes là pour vous aider!
