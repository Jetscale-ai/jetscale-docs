# Intégrations

Connectez JetScale à votre flux de travail de développement existant pour automatiser l'implémentation de l'optimisation des coûts.

---

## Intégrations Disponibles

### GitHub
Créez automatiquement des pull requests avec du code Terraform prêt pour la production pour vos recommandations d'optimisation des coûts.

**Fonctionnalités :**
- Création automatique de PR avec analyse détaillée
- Gestion des branches et versioning
- Intégration du workflow de revue de code
- Piste d'audit complète

[En savoir plus →](github.md)

---

### Jira
Suivez les recommandations d'optimisation des coûts sous forme de tickets Jira avec des métriques d'économies et le statut d'implémentation.

**Fonctionnalités :**
- Création automatique de tickets
- Suivi des économies
- Synchronisation du statut
- Mappage de champs personnalisés

[En savoir plus →](jira.md)

---

### Slack
Recevez des notifications en temps réel pour les nouvelles opportunités d'optimisation et les mises à jour d'implémentation.

**Fonctionnalités :**
- Alertes en temps réel
- Résumés des économies
- Collaboration d'équipe
- Notifications personnalisables

[En savoir plus →](slack.md)

---

### Bitbucket
Créez des pull requests dans des dépôts Bitbucket avec du code d'optimisation des coûts et de la documentation.

**Fonctionnalités :**
- Création automatique de PR
- Gestion des branches
- Intégration de revue de code
- Permissions de dépôt

[En savoir plus →](bitbucket.md)

---

## Comment Fonctionnent les Intégrations

Les intégrations JetScale suivent un modèle cohérent :

1. **Connexion** : Liez votre compte en utilisant une authentification sécurisée (OAuth ou tokens API)
2. **Configuration** : Sélectionnez les dépôts, projets ou canaux à utiliser
3. **Automatisation** : JetScale crée automatiquement des PR, tickets ou notifications
4. **Suivi** : Surveillez le statut d'implémentation et les économies à travers les outils

---

## Sécurité et Permissions

Toutes les intégrations JetScale :
- Utilisent une authentification standard de l'industrie (OAuth 2.0, tokens API)
- Nécessitent des permissions minimales (lecture seule lorsque possible)
- Chiffrent les identifiants au repos en utilisant AES-256
- N'exposent jamais les tokens dans les journaux ou les réponses API
- Supportent la révocation immédiate

---

## Premiers Pas

1. Naviguez vers **Paramètres** → **Intégrations** dans votre tableau de bord JetScale
2. Sélectionnez l'intégration que vous souhaitez connecter
3. Suivez le guide de configuration pour votre outil choisi
4. Examinez et approuvez la première recommandation pour tester l'intégration

---

## Support

Besoin d'aide pour configurer les intégrations ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Problèmes** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

*© 2025 JetScale, Inc. Tous droits réservés.*
