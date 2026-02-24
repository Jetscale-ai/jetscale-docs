# Intégration Slack

JetScale s'intègre avec Slack pour diffuser des notifications d'optimisation des coûts en temps réel directement dans vos canaux d'équipe, tenant chacun informé des nouvelles recommandations, du statut des déploiements et des opportunités d'économies.

## Aperçu

L'intégration Slack permet à JetScale de :
- Publier des notifications pour les nouvelles recommandations d'optimisation des coûts
- Alerter les équipes des opportunités d'économies significatives
- Notifier des mises à jour de statut de déploiement
- Partager les réalisations en matière d'économies
- Fournir des résumés interactifs de recommandations

## Prérequis

Avant de connecter Slack, vous aurez besoin de :

1. **Espace de travail Slack** avec des permissions d'administrateur ou d'installation d'application
2. **Accès aux canaux** pour recevoir les notifications JetScale
3. **Permissions d'espace de travail** pour installer des applications

### Types d'espaces de travail pris en charge

- Slack Free Plan
- Slack Pro
- Slack Business+
- Slack Enterprise Grid

## Guide d'installation

### Étape 1 : Connecter Slack dans JetScale

![Écran de connexion Slack](#)
*Emplacement de capture d'écran : page d'intégration Slack de JetScale*

1. Naviguez vers **Paramètres** → **Intégrations** dans JetScale
2. Cliquez sur **Connecter Slack**
3. Vous serez redirigé vers la page d'autorisation de Slack
4. Sélectionnez votre espace de travail dans le menu déroulant
5. Cliquez sur **Autoriser** pour accorder les permissions

JetScale demandera les permissions suivantes :
- **Envoyer des messages** : Publier des notifications dans les canaux
- **Voir les informations des canaux** : Lister les canaux disponibles
- **Télécharger des fichiers** : Partager des rapports détaillés (optionnel)

### Étape 2 : Sélectionner le canal de notification

![Écran de sélection de canal](#)
*Emplacement de capture d'écran : configuration du canal Slack*

1. Après l'autorisation, vous reviendrez sur JetScale
2. Sélectionnez votre **canal de notification par défaut** dans le menu déroulant
3. Cliquez sur **Enregistrer le canal**

Vous pouvez modifier le canal à tout moment ou configurer plusieurs canaux pour différents types de notifications.

### Étape 3 : Configurer les préférences de notification

![Préférences de notification](#)
*Emplacement de capture d'écran : paramètres de notification*

Personnalisez ce qui est envoyé à Slack :

- **Nouvelles recommandations** (recommandé)
- **Opportunités de haute valeur** (économies > 500 $/mois)
- **Mises à jour du statut de déploiement**
- **Résumé hebdomadaire des économies**
- **Rapports de coûts mensuels**

## Fonctionnement

### Types de notifications

#### Alertes de nouvelles recommandations

Lorsque JetScale identifie une opportunité d'optimisation des coûts :

```
💰 Nouvelle recommandation d'optimisation des coûts

Ressource : production-postgres-db
Type : Dimensionnement d'instance RDS
Actuel : db.r5.2xlarge (730 $/mois)
Recommandé : db.r5.xlarge (365 $/mois)

Économies mensuelles : 365 $ (50 %)
Économies annuelles : 4 380 $

Impact sur les performances : Faible risque
✅ Marge de 2x au-dessus de l'utilisation maximale maintenue

[Voir dans JetScale] [Voir Terraform]
```

#### Opportunités de haute valeur

Pour les recommandations dépassant votre seuil d'économies configuré :

```
🎯 Optimisation de haute valeur détectée !

Ressource : customer-aurora-cluster
Type : Migration Aurora I/O-Optimized
Économies mensuelles : 1 250 $ (35 %)
Économies annuelles : 15 000 $

Cela représente 8 % de vos coûts cloud totaux.

Statistiques rapides :
• Niveau de risque : Très faible
• Temps d'implémentation : ~30 minutes
• Temps d'arrêt requis : Aucun

[Examiner la recommandation] [Planifier l'implémentation]
```

#### Mises à jour du statut de déploiement

Lorsque vous implémentez une recommandation :

```
✅ Optimisation déployée avec succès

Ressource : api-server-prod
Optimisation : Migration EC2 Graviton (m5.xlarge → m6g.xlarge)

Déploiement terminé à 14h45
Métriques de performance : Toutes au vert ✓
Surveillance active pendant 48 heures

Économies mensuelles attendues : 140 $
Économies annuelles attendues : 1 680 $

[Voir le tableau de bord des métriques]
```

#### Résumé hebdomadaire des économies

Chaque lundi matin (configurable) :

```
📊 Résumé hebdomadaire d'optimisation des coûts

Économies totales identifiées : 2 450 $/mois
Recommandations implémentées : 3
Nouvelles recommandations : 7

🏆 Principales opportunités :
1. Dimensionnement d'instance RDS : 850 $/mois
2. Migration EC2 Graviton : 420 $/mois
3. Optimisation de volume EBS : 380 $/mois

📈 Mois en cours :
• Économies implémentées : 3 200 $/mois
• Impact annuel projeté : 38 400 $

[Voir le tableau de bord complet] [Examiner toutes les recommandations]
```

#### Rapports de coûts mensuels

Premier jour de chaque mois :

```
📊 Rapport mensuel des coûts cloud - Janvier 2024

Dépenses cloud totales : 28 450 $
Économies optimisées : 4 200 $ (réduction de 14,8 %)
Recommandations implémentées : 12

💰 Répartition des économies :
• Optimisation RDS : 1 850 $/mois
• Dimensionnement EC2 : 1 420 $/mois
• Réduction de volume EBS : 930 $/mois

🎯 Opportunités restantes : 3 800 $/mois

Économies cumulées de l'année : 12 600 $

[Voir le rapport détaillé] [Télécharger CSV]
```

## Paramètres de notification

### Configuration de la fréquence

Contrôlez la fréquence de réception des notifications :

| Type de notification | Options de fréquence |
|-------------------|-------------------|
| Nouvelles recommandations | Temps réel, Résumé quotidien, Résumé hebdomadaire |
| Opportunités de haute valeur | Temps réel (recommandé) |
| Statut de déploiement | Temps réel |
| Résumé des économies | Hebdomadaire, Bimensuel |
| Rapports de coûts | Mensuel, Trimestriel |

### Configuration des seuils

Définissez des seuils d'économies minimales pour réduire le bruit :

- **Économies mensuelles minimales** : 50 $, 100 $, 250 $, 500 $, 1000 $
- **Économies annuelles minimales** : 1000 $, 2500 $, 5000 $, 10000 $
- **Pourcentage d'économies minimal** : 10 %, 20 %, 30 %, 50 %

### Routage des canaux

Routez différents types de notifications vers différents canaux :

**Exemple de configuration :**
```
#cloud-costs → Toutes les recommandations et rapports
#engineering-alerts → Opportunités de haute valeur uniquement
#devops → Mises à jour du statut de déploiement
#leadership → Rapports de coûts mensuels uniquement
```

## Commandes slash (optionnel)

JetScale fournit des commandes slash optionnelles pour les requêtes interactives :

| Commande | Description | Exemple |
|---------|-------------|---------|
| `/jetscale summary` | Résumé des économies du mois en cours | `/jetscale summary` |
| `/jetscale recommendations` | Lister les 5 meilleures recommandations | `/jetscale recommendations` |
| `/jetscale report` | Générer un rapport de coûts | `/jetscale report last-month` |
| `/jetscale help` | Afficher les commandes disponibles | `/jetscale help` |

**Remarque** : Les commandes slash nécessitent des permissions supplémentaires lors de l'installation. Elles sont entièrement optionnelles et peuvent être activées/désactivées indépendamment des notifications.

## Gestion de l'intégration

### Voir le statut de l'intégration

Depuis **Paramètres** → **Intégrations** → **Slack**, vous pouvez voir :
- Nom de l'espace de travail connecté
- Canaux de notification actifs
- Notifications récentes envoyées
- Heure de la dernière vérification de connexion
- Préférences de notification

### Tester la connexion

Pour vérifier que votre intégration Slack fonctionne :

1. Naviguez vers **Paramètres** → **Intégrations** → **Slack**
2. Cliquez sur **Envoyer un message de test**
3. Vérifiez votre canal configuré pour une notification de test

### Changer le canal de notification

Pour router les notifications vers un canal différent :

1. Naviguez vers **Paramètres** → **Intégrations** → **Slack**
2. Cliquez sur **Changer de canal**
3. Sélectionnez un nouveau canal dans le menu déroulant
4. Cliquez sur **Mettre à jour**

**Important** : Vous devez inviter le bot `@JetScale` dans les canaux privés avant qu'ils n'apparaissent dans le menu déroulant de sélection.

### Mettre à jour les préférences

Pour modifier les paramètres de notification :

1. Naviguez vers **Paramètres** → **Intégrations** → **Slack**
2. Cliquez sur **Configurer les notifications**
3. Activez/désactivez les types de notifications
4. Ajustez les paramètres de fréquence
5. Définissez les seuils d'économies
6. Cliquez sur **Enregistrer les modifications**

### Déconnecter Slack

Pour supprimer l'intégration Slack :

1. Naviguez vers **Paramètres** → **Intégrations** → **Slack**
2. Cliquez sur **Déconnecter**
3. Confirmez la déconnexion

**Important** : La déconnexion va :
- Arrêter toutes les notifications vers Slack
- Supprimer le bot JetScale de votre espace de travail
- Conserver l'historique des notifications dans le tableau de bord JetScale
- Désactiver les commandes slash (si activées)

Vous pouvez vous reconnecter à tout moment en répétant le processus d'installation.

## Dépannage

### Échec de connexion

**Problème** : Erreur "Impossible de se connecter à Slack"

**Solutions** :
- Vérifiez que vous avez les permissions d'installer des applications dans votre espace de travail
- Vérifiez si votre organisation nécessite une approbation de l'administrateur pour les nouvelles applications
- Assurez-vous d'avoir sélectionné le bon espace de travail lors de l'autorisation
- Essayez de déconnecter et de reconnecter

### Réception des notifications impossible

**Problème** : Les notifications n'apparaissent pas dans le canal Slack

**Solutions** :
- Vérifiez que le bot JetScale est membre du canal (invitez `@JetScale`)
- Vérifiez que les préférences de notification sont activées
- Confirmez que les seuils d'économies ne filtrent pas toutes les recommandations
- Envoyez un message de test pour vérifier la connexion
- Vérifiez le paramètre "Autoriser les applications à envoyer des messages" de Slack

### Bot absent de la liste des canaux

**Problème** : Le canal cible n'apparaît pas dans le menu déroulant

**Solutions** :
- Les canaux publics apparaissent automatiquement
- Les canaux privés nécessitent que vous invitiez `@JetScale` d'abord
- Tapez `/invite @JetScale` dans le canal cible
- Actualisez la page d'intégration JetScale après l'invitation

### Notifications en double

**Problème** : Réception de la même notification plusieurs fois

**Solutions** :
- Vérifiez si plusieurs membres de l'équipe ont connecté des intégrations séparées
- Examinez les préférences de notification pour les paramètres en double
- Vérifiez que vous n'avez pas configuré plusieurs canaux de notification pour le même type
- Contactez le support si le problème persiste

### Commandes slash non fonctionnelles

**Problème** : Les commandes `/jetscale` retournent une erreur

**Solutions** :
- Vérifiez que les commandes slash sont activées dans les paramètres d'intégration
- Réautorisez l'intégration Slack pour accorder les permissions de commande
- Vérifiez la syntaxe de la commande (utilisez `/jetscale help` pour référence)
- Assurez-vous que le bot dispose des permissions nécessaires dans le canal

## Bonnes pratiques

### Organisation des canaux

**Structure de canaux recommandée :**
- **#cloud-costs** : Canal principal pour toutes les notifications liées aux coûts
- **#engineering-alerts** : Recommandations prioritaires nécessitant une action immédiate
- **#devops-deploy** : Statut de déploiement et suivi de l'implémentation
- **#leadership-reports** : Résumés mensuels et métriques de haut niveau

### Stratégie de notification

**Pour les petites équipes (< 20 personnes) :**
- Canal unique pour toutes les notifications
- Alertes en temps réel pour les opportunités de haute valeur
- Résumé hebdomadaire pour les recommandations de routine

**Pour les équipes moyennes (20-100 personnes) :**
- Canaux séparés par type de notification
- Temps réel pour les déploiements et haute valeur
- Résumé quotidien pour les nouvelles recommandations

**Pour les grandes équipes (100+ personnes) :**
- Routage des canaux basé sur les rôles
- Filtrage agressif des seuils
- Résumés hebdomadaires préférés
- Canal de résumé exécutif pour la direction

### Intégration aux flux de travail

**Flux de travail d'approbation des recommandations :**
1. JetScale publie une recommandation dans `#cloud-costs`
2. L'équipe discute dans le fil de discussion
3. DevOps utilise la réaction 👍 pour indiquer l'approbation
4. Un ingénieur implémente via le tableau de bord JetScale
5. Mise à jour du statut publiée automatiquement

**Utilisez les réactions Slack pour le suivi :**
- 👀 = En cours d'examen
- 👍 = Approuvé pour l'implémentation
- 🚀 = Déployé en staging
- ✅ = Déployé en production
- ❌ = Rejeté (documenter la raison dans le fil)

### Seuils d'alerte

**Approche conservatrice** (moins de bruit) :
- Économies minimales : 500 $/mois
- Seuil de haute valeur : 1000 $/mois
- Résumés hebdomadaires pour les recommandations de routine

**Approche agressive** (tout capturer) :
- Économies minimales : 50 $/mois
- Seuil de haute valeur : 500 $/mois
- Notifications en temps réel pour toutes les recommandations

**Point de départ recommandé :**
- Économies minimales : 100 $/mois (récurrent mensuel)
- Seuil de haute valeur : 500 $/mois
- Résumé quotidien pour les nouvelles recommandations
- Temps réel pour haute valeur et déploiements

## Considérations de sécurité

### Permissions

JetScale demande uniquement les permissions Slack minimales requises :
- **Publier des messages** : Requis pour toutes les notifications
- **Lire les informations de canal** : Requis pour lister les canaux disponibles
- **Télécharger des fichiers** : Optionnel, pour les rapports détaillés

JetScale ne peut pas :
- Lire les messages de vos canaux
- Accéder aux messages privés ou aux DM
- Modifier les paramètres de canal
- Inviter ou supprimer des utilisateurs
- Accéder aux paramètres de l'espace de travail

### Confidentialité des données

- Les notifications Slack contiennent uniquement des informations récapitulatives
- Les informations d'identification sensibles ne sont jamais incluses dans les messages
- Les données détaillées sont disponibles uniquement via des liens authentifiés du tableau de bord JetScale
- Tous les liens nécessitent une session JetScale active

### Piste d'audit

Chaque notification envoyée est enregistrée :
- Horodatage et canal destinataire
- Type et contenu de la notification
- Utilisateur ayant configuré l'intégration
- Journaux d'audit disponibles dans votre tableau de bord JetScale

## Référence API

Pour un accès programmatique à l'intégration Slack, consultez notre [Documentation API](../api-reference.md#slack-integration).

Points de terminaison clés :
- `POST /api/v2/integrations/slack/me/action/connect` - Initier le flux OAuth
- `GET /api/v2/integrations/slack/channels` - Lister les canaux disponibles
- `POST /api/v2/integrations/slack/me/action/configure` - Mettre à jour les préférences
- `POST /api/v2/integrations/slack/notifications/action/send-test` - Envoyer un message de test
- `POST /api/v2/integrations/slack/me/action/disconnect` - Supprimer l'intégration

## Support

Besoin d'aide avec l'intégration Slack ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **GitHub Issues** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)
