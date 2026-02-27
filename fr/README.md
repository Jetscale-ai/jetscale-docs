# Documentation JetScale

Transformez les coûts cloud en avantage concurrentiel avec une optimisation alimentée par l'IA.

---

## Qu'est-ce que JetScale?

JetScale découvre automatiquement les opportunités d'économies dans votre infrastructure cloud et les livre sous forme de modifications prêtes à déployer. Plus d'analyse manuelle, plus de conjectures, seulement des économies prouvées que vous pouvez mettre en œuvre immédiatement.

**Votre facture cloud n'a pas à être un mystère.** JetScale vous offre:
- **Visibilité instantanée** sur les gaspillages d'argent
- **Recommandations générées par IA** qui maintiennent les performances
- **Modifications prêtes pour la production** intégrées dans votre flux de travail
- **ROI mesurable** dès le premier jour

---

## Démarrage Rapide (3 Étapes Simples)

### 1. Connectez Votre Compte Cloud
Accordez à JetScale un accès en lecture seule à votre environnement AWS ou Azure. Nous découvrirons automatiquement toutes vos ressources : instances EC2, bases de données RDS, volumes de stockage, et plus encore.

### 2. Examinez les Recommandations
Notre IA analyse l'historique des modèles d'utilisation et identifie les opportunités d'optimisation. Chaque recommandation inclut:
- Économies mensuelles estimées
- Évaluation de l'impact sur les performances
- Comparaison de configuration avant/après
- Niveau de risque d'implémentation

### 3. Déployez les Modifications
Sélectionnez les recommandations à implémenter. JetScale génère du code Terraform et crée une pull request dans votre dépôt. Examinez, approuvez et déployez en toute confiance.

**Les clients moyens économisent 30 à 40% sur les dépenses cloud dans les 90 premiers jours.**

---

## Pourquoi JetScale?

### IA Qui Comprend Votre Infrastructure
Contrairement aux outils de coûts génériques, JetScale utilise des agents IA spécialisés formés sur des services cloud spécifiques. Nos agents comprennent les nuances des déploiements RDS multi-AZ, des instances EC2 bursteables et des exigences IOPS EBS.

### Approche Axée sur la Sécurité
Chaque recommandation est validée pour garantir:
- Les SLA de performance sont maintenus
- Les configurations de haute disponibilité sont préservées
- Marge suffisante pour les pics de trafic
- Retour en arrière facile si nécessaire

### Intégré Dans Votre Flux de Travail
JetScale fonctionne avec les outils que vous utilisez déjà:
- **GitHub/Bitbucket**: Pull requests automatisées avec documentation détaillée
- **Jira**: Création automatique de tickets avec suivi des économies
- **Terraform**: Code d'infrastructure prêt pour la production
- **Slack**: Notifications en temps réel pour les nouvelles opportunités

---

## Ce Que JetScale Optimise

### Services AWS
- **Cost Explorer**: Analyse des Instances Réservées et des Plans d'Économies
- **EBS**: Optimisation du type de volume (gp2→gp3), nettoyage des snapshots
- **EC2**: Dimensionnement optimal des instances, identification des ressources inactives, migration Graviton
- **EKS**: Dimensionnement des groupes de nœuds, migration Graviton, optimisation Spot
- **ElastiCache**: Optimisation du type de nœud, configuration du cluster
- **RDS**: Dimensionnement des instances, optimisation du stockage, recommandations d'Instances Réservées
- **S3**: Intelligent-Tiering, politiques de cycle de vie, Bucket Key, Gateway Endpoints

### Services Azure
- **Azure Cache for Redis**: Recommandations de niveau et de capacité
- **Azure SQL**: Recommandations de niveau, optimisation des pools élastiques
- **Disques Managés**: Optimisation Premium vs Standard
- **Machines Virtuelles**: Dimensionnement optimal, optimisation de la série B, capacité réservée

### Prochainement
Lambda/Functions, DynamoDB/Cosmos DB, ECS/AKS, Blob Storage, Load Balancers

---

## Sécurité & Confiance

**Vos données, votre contrôle:**
- Accès en lecture seule aux comptes cloud (aucune permission d'écriture)
- Rôles inter-comptes avec vérification d'ID externe
- Infrastructure conforme SOC 2 Type II
- Aucun stockage d'identifiants. Tout utilise des jetons temporaires
- Votre code reste dans vos dépôts

**Sécurité de niveau entreprise:**
- Données chiffrées au repos et en transit (TLS 1.3)
- Audits de sécurité réguliers par des tiers
- Conformité RGPD et CCPA
- Politiques de rétention des données configurables

---

## Documentation

### Démarrage
- [Guide de Démarrage](/fr/getting-started) - Configuration complète en 5 minutes
- [Configuration du Compte AWS](/fr/aws-setup) - Connecter le compte cloud AWS
- [Configuration du Compte Azure](/fr/azure-setup) - Connecter le compte cloud Azure

### Comment Ça Marche
- [Comment Fonctionne JetScale](/fr/how-it-works) - Aperçu de la plateforme et flux de travail
- [Analyse IA](/fr/ai-analysis) - Comment notre IA génère les recommandations
- [Guide d'Intégration](/fr/integrations/) - Configuration GitHub, Jira, Slack

### Référence
- [Services Supportés](/fr/services/) - Liste complète des ressources optimisées
- [Documentation API](/fr/api-reference) - API publique pour les intégrations (Beta)
- [FAQ](/fr/faq) - Questions fréquemment posées

---

## Support

Email: [support@jetscale.ai](mailto:support@jetscale.ai)

---

## À Propos de JetScale

JetScale a été fondé par des vétérans de l'infrastructure cloud frustrés par la complexité de l'optimisation des coûts. Nous croyons que chaque équipe d'ingénierie devrait avoir accès à une intelligence de coûts de niveau entreprise sans avoir besoin de spécialistes FinOps dédiés.

Notre mission: Rendre l'optimisation des coûts cloud automatique, précise et exploitable.

[En Savoir Plus Sur Nous](https://jetscale.ai/about)

---

*© 2025 JetScale, Inc. Tous droits réservés.*
