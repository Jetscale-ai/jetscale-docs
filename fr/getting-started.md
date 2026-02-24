# Démarrage avec JetScale

> Commencez à économiser sur vos coûts cloud en 5 minutes

## Vue d'ensemble

JetScale est une plateforme SaaS qui découvre automatiquement les opportunités d'économies de coûts dans votre infrastructure cloud. Ce guide vous accompagnera dans la connexion de votre premier compte cloud et l'examen de vos premières recommandations.

**Délai jusqu'aux premières économies : 5 minutes** ⏱️

---

## Étape 1 : S'inscrire à JetScale

**Créer votre compte :**
1. Visitez [app.jetscale.ai/signup](https://app.jetscale.ai/signup)
2. Entrez votre adresse email et créez un mot de passe
3. Vérifiez votre adresse email
4. Complétez votre profil (nom de l'entreprise, fonction)

**Essai gratuit :**
- 30 jours d'accès complet à la plateforme
- Jusqu'à 10 000 $ d'économies identifiées
- Aucune carte de crédit requise
- Résiliation possible à tout moment

---

## Étape 2 : Connecter votre compte cloud

Choisissez votre fournisseur cloud et suivez le guide de configuration :

### AWS (5 minutes)

**Étapes rapides :**
1. Connectez-vous à votre compte AWS
2. Créez un rôle IAM inter-comptes pour JetScale
3. Copiez l'ARN du rôle et l'External ID
4. Collez-les dans le tableau de bord JetScale
5. Cliquez sur "Connecter" et vérifiez l'accès

[Guide détaillé de configuration AWS →](aws-setup.md)

**Permissions :** Accès en lecture seule à EC2, RDS, EBS, ElastiCache, Cost Explorer

---

### Azure (5 minutes)

**Étapes rapides :**
1. Connectez-vous au portail Azure
2. Créez un Service Principal pour JetScale
3. Attribuez le rôle Lecteur à votre abonnement
4. Copiez l'ID de locataire, l'ID d'abonnement et les informations d'identification
5. Collez-les dans le tableau de bord JetScale
6. Cliquez sur "Connecter" et vérifiez l'accès

[Guide détaillé de configuration Azure →](azure-setup.md)

**Permissions :** Accès en lecture seule aux Machines Virtuelles, SQL, Disques, Cache, Cost Management

---

## Étape 3 : Attendre la découverte (2-5 minutes)

**Ce qui se passe :**
- JetScale analyse toutes les régions de votre compte
- Catalogue les ressources de calcul, bases de données, stockage et cache
- Récupère 30 jours de métriques d'utilisation depuis CloudWatch/Azure Monitor
- L'IA analyse les modèles d'utilisation et identifie les opportunités

**Indicateurs de progression :**
- **Découverte des ressources** → Recherche de toutes les ressources cloud
- **Analyse de l'utilisation** → Collecte de 30 jours de métriques
- **Génération de recommandations** → Validation IA en cours
- **Prêt** → Recommandations disponibles pour révision

---

## Étape 4 : Examiner les recommandations

**Vue d'ensemble du tableau de bord :**

Vous verrez les recommandations organisées par :
- **Économies potentielles** (du plus élevé au plus bas)
- **Niveau de risque** (Faible, Moyen, Élevé)
- **Type de ressource** (EC2, RDS, EBS, etc.)

**Carte de recommandation :**

Chaque carte affiche :
- **Nom de la ressource** et type
- **Économies mensuelles estimées**
- **Niveau de risque** avec code couleur
- **Résumé rapide** de la modification

**En cliquant sur "Voir les détails", vous verrez :**
- Graphiques d'utilisation sur 30 jours (CPU, mémoire, réseau)
- Configuration actuelle vs recommandée
- Calcul détaillé des économies
- Évaluation de l'impact sur les performances
- Étapes de mise en œuvre et instructions de retour en arrière

---

## Étape 5 : Approuver les recommandations

**Sélectionner les recommandations :**
1. Examinez d'abord les recommandations à faible risque (badges verts)
2. Vérifiez les graphiques d'utilisation pour valider l'analyse IA
3. Sélectionnez les recommandations que vous souhaitez mettre en œuvre
4. Cliquez sur le bouton "Approuver la sélection"

**Approbation en masse :**
- Utilisez les filtres pour trouver des types spécifiques (ex : "dimensionnement EC2")
- Sélectionnez plusieurs recommandations à la fois
- Prévisualisez les économies mensuelles combinées
- Approuvez le déploiement par lot

---

## Étape 6 : Déployer les modifications

**Choisissez votre intégration :**

### Option 1 : Pull Request GitHub (Recommandée)

**Configuration (une seule fois) :**
1. Connectez votre compte GitHub dans les paramètres
2. Sélectionnez le dépôt cible pour le code Terraform
3. Configurez le nom de la branche et les réviseurs

**Déploiement :**
1. JetScale crée une nouvelle branche
2. Envoie le code Terraform pour les recommandations approuvées
3. Ouvre une pull request avec documentation complète
4. Votre équipe examine et approuve
5. Fusionnez la PR et appliquez les modifications Terraform

[Guide d'intégration GitHub →](integrations/github.md)

---

### Option 2 : Pull Request Bitbucket

Même flux de travail que GitHub, mais pour les dépôts Bitbucket.

[Guide d'intégration Bitbucket →](integrations/bitbucket.md)

---

### Option 3 : Téléchargement manuel

**Télécharger les fichiers Terraform :**
1. Cliquez sur le bouton "Télécharger Terraform"
2. Recevez un fichier ZIP avec toutes les configurations
3. Extrayez et examinez les fichiers localement
4. Appliquez en utilisant votre flux de travail Terraform existant

```bash
# Exemple de déploiement manuel
unzip jetscale-recommendations.zip
cd jetscale-recommendations/
terraform init
terraform plan
terraform apply
```

---

## Étape 7 : Suivre les économies

**Tableau de bord des économies :**

Après le déploiement, surveillez :
- **Économies réelles** vs économies projetées
- **Économies cumulatives** dans le temps
- **Métriques de performance** après modification
- **Calculs de ROI** pour les rapports de direction

**Découverte continue :**
- JetScale analyse quotidiennement pour de nouvelles opportunités
- Alertes lorsque des recommandations de grande valeur apparaissent
- Suit les tendances d'utilisation pour identifier le gaspillage croissant
- Suggère des optimisations à mesure que votre infrastructure évolue

---

## Intégrations (Optionnel)

### Intégration Jira

**Création automatique de tickets :**
- Crée des tickets Jira pour chaque lot de recommandations
- Suit les économies et l'état de mise en œuvre
- Liens vers les pull requests et le tableau de bord JetScale
- Met à jour l'état lorsque les PR sont fusionnées

[Guide d'intégration Jira →](integrations/jira.md)

---

### Intégration Slack

**Notifications en temps réel :**
- Nouvelles recommandations disponibles
- Jalons d'économies atteints
- Mises à jour de l'état de déploiement
- Rapports de synthèse hebdomadaires

[Guide d'intégration Slack →](integrations/slack.md)

---

## Configuration de l'équipe (Optionnel)

**Inviter des membres de l'équipe :**
1. Allez dans Paramètres → Équipe
2. Cliquez sur "Inviter un membre"
3. Entrez l'email et sélectionnez le rôle :
   - **Admin** : Accès complet, peut approuver les recommandations
   - **Éditeur** : Peut examiner et approuver les recommandations
   - **Lecteur** : Accès en lecture seule au tableau de bord

**Permissions des rôles :**

| Permission | Admin | Éditeur | Lecteur |
|------------|-------|---------|---------|
| Voir les recommandations | ✓ | ✓ | ✓ |
| Approuver les recommandations | ✓ | ✓ | - |
| Configurer les intégrations | ✓ | - | - |
| Gérer les membres de l'équipe | ✓ | - | - |
| Connecter les comptes cloud | ✓ | - | - |

---

## Paramètres et préférences

**Personnaliser JetScale :**

**Paramètres de notification :**
- Fréquence des emails (quotidienne, hebdomadaire, mensuelle)
- Canal Slack pour les notifications
- Seuil d'économies pour les alertes (100 $, 500 $, 1 000 $)

**Tolérance au risque :**
- **Conservateur** : Afficher uniquement les recommandations à faible risque
- **Équilibré** : Afficher les recommandations à risque faible et moyen (par défaut)
- **Agressif** : Afficher toutes les recommandations y compris à haut risque

**Flux d'approbation :**
- **Approbation manuelle** : Examiner chaque recommandation individuellement
- **Approbation automatique à faible risque** : Approuver automatiquement les recommandations < 100 $/mois
- **Règles personnalisées** : Définir les critères d'approbation (ex : "approuver automatiquement le dimensionnement EC2")

**Exclusions :**
- Exclure des ressources spécifiques par tag (ex : "JetScaleIgnore=true")
- Exclure des types de ressources (ex : "ne jamais toucher RDS de production")
- Exclure des comptes ou régions entiers

---

## Meilleures pratiques

### Commencer par les recommandations à faible risque

**Semaine 1 :** Approuver les recommandations à faible risque et économies élevées :
- Optimisation du stockage (migrations gp2→gp3)
- Nettoyage des snapshots et volumes inutilisés
- Dimensionnement des ressources manifestement sur-provisionnées

**Semaine 2-4 :** Étendre aux recommandations à risque moyen :
- Dimensionnement d'instances EC2 avec marge appropriée
- Achats d'instances réservées pour les charges de travail stables
- Optimisation des instances de base de données

**Mois 2+ :** Envisager les optimisations à risque plus élevé :
- Migrations Graviton (nécessite des tests)
- Arrêt programmé des ressources hors production
- Optimisations des groupes Auto Scaling

### Tester d'abord en environnement hors production

**Stratégie de déploiement sûr :**
1. Tester les recommandations d'abord dans les environnements dev/staging
2. Surveiller les performances pendant 1-2 semaines
3. Appliquer les mêmes optimisations en production
4. Étendre progressivement à plus de types de ressources

### Surveiller les performances après déploiement

**Métriques clés à surveiller :**
- Utilisation du CPU et de la mémoire (doit rester en dessous de 80%)
- Temps de réponse et latence (doivent rester stables)
- Taux d'erreur (ne doivent pas augmenter)
- Journaux d'application (surveiller les contraintes de ressources)

**En cas de problèmes :**
- Retour en arrière en utilisant les instructions Terraform fournies
- Contacter le support : [support@jetscale.ai](mailto:support@jetscale.ai)
- JetScale suit les résultats pour améliorer les futures recommandations

---

## Questions fréquentes

### Combien de temps avant de voir des économies ?

**Économies immédiates :**
- Les optimisations de stockage s'appliquent en quelques heures
- Le dimensionnement d'instances prend 5-10 minutes (création avant destruction)

**Économies sur la prochaine facture :**
- La plupart des optimisations réduisent votre prochaine facture mensuelle
- Les instances réservées montrent des économies immédiatement après l'achat

### Que faire si je n'aime pas une recommandation ?

**Vous avez le contrôle :**
- Rejetez toute recommandation (ne sera plus affichée)
- Différez les recommandations pour un examen ultérieur
- Fournissez des commentaires pour améliorer les suggestions futures

### Puis-je annuler les modifications ?

**Oui, facilement :**
- Tout le code Terraform inclut des instructions de retour en arrière
- La plupart des modifications peuvent être annulées sur place
- JetScale fournit des guides de retour en arrière étape par étape
- L'équipe de support est disponible pour vous aider

### Quelle est la précision des estimations d'économies ?

**Très précise :**
- Précision de 98% avec une variance de ±5%
- Basée sur 30 jours de données d'utilisation réelles
- Inclut tous les facteurs de coût (calcul, stockage, transfert de données)
- Tient compte des crédits d'instances réservées

---

## Besoin d'aide ?

**Canaux de support :**
- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Chat en direct** : Disponible dans le tableau de bord (coin inférieur droit)
- **Documentation** : [docs.jetscale.ai](https://docs.jetscale.ai)
- **Programmer une démo** : [jetscale.ai/demo](https://jetscale.ai/demo)

**Temps de réponse :**
- **Essai gratuit** : Réponse sous 24 heures
- **Plan Pro** : Réponse sous 8 heures
- **Entreprise** : Réponse sous 2 heures avec support dédié

---

## Prochaines étapes

**En savoir plus :**
- [Comment fonctionne JetScale](how-it-works.md) - Plongée en profondeur dans la plateforme
- [Analyse IA](ai-analysis.md) - Comment notre IA valide la sécurité
- [Services supportés](services/README.md) - Ce que JetScale optimise
- [FAQ](faq.md) - Réponses aux questions courantes

**Prêt à économiser ?**
- [Commencer l'essai gratuit →](https://app.jetscale.ai/signup)
- [Programmer une démo →](https://jetscale.ai/demo)

---

*Dernière mise à jour : 29 janvier 2025*
