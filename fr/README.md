# Aperçu

JetScale est une plateforme d'optimisation des coûts cloud alimentée par l'IA qui analyse automatiquement votre infrastructure et génère des recommandations exploitables avec du code infrastructure-as-code prêt pour la production.

---

## Démarrage

<div class="card-grid">

<div class="doc-card">

**[Configuration AWS](aws-setup.md)**

Connectez votre compte AWS en toute sécurité en utilisant des rôles IAM avec des permissions en lecture seule

</div>

<div class="doc-card">

**[Configuration Azure](azure-setup.md)**

Connectez Azure en utilisant l'authentification par service principal avec authentification par certificat ou secret

</div>

<div class="doc-card">

**[Guide de Démarrage Rapide](getting-started.md)**

Soyez opérationnel en quelques minutes avec notre guide d'intégration étape par étape

</div>

</div>

---

## Qu'est-ce que JetScale?

JetScale utilise des agents IA spécialisés pour analyser votre infrastructure cloud, identifier les opportunités d'optimisation des coûts et générer des modifications de code infrastructure-as-code prêtes pour la production, intégrées dans vos workflows existants.

**Capacités Principales:**
- Agents IA formés sur des services cloud spécifiques (RDS, EC2, EBS, ElastiCache, Azure VMs)
- Analyse de données en temps réel à partir de CloudWatch, Cost Explorer et Azure Monitor
- Validation automatisée garantissant que les recommandations maintiennent les performances et les SLA
- Intégration native GitOps avec GitHub, Jira et Bitbucket
- Génération de code Terraform prêt pour la production

---

## Services Supportés

<div class="card-grid">

<div class="doc-card">

<img src="icons/icon-database.png" alt="Base de données" class="card-icon">

**Bases de Données**

AWS RDS - MySQL, PostgreSQL, clusters Aurora
Azure SQL - Azure SQL, MySQL, PostgreSQL
Dimensionnement optimal, sélection de classe d'instance, optimisation de topologie

</div>

<div class="doc-card">

**Calcul**

AWS EC2 - Tous les types et familles d'instances
Azure VMs - Toutes les séries de VM
Recommandations de types d'instances, opportunités de réservation

</div>

<div class="doc-card">

**Stockage**

AWS EBS - volumes gp2, gp3, io1, io2
Azure Storage - Blob, Files, Disques Managés
Optimisation du type de volume, optimisation de niveau, provisionnement IOPS

</div>

<div class="doc-card">

**Mise en Cache**

AWS ElastiCache - Redis, Memcached
Azure Cache - Redis, Redis Enterprise
Optimisation du type de nœud, configuration du cluster

</div>

</div>

> **Prochainement:** S3, Lambda, DynamoDB, Azure Cosmos DB, Azure Functions

---

## Documentation

<div class="card-grid">

<div class="doc-card">

**Concepts de Base**

[Vue d'ensemble de l'Architecture](architecture.md) - Conception du système et workflows
[Agents IA](ai-agents.md) - Comment fonctionnent les agents spécialisés
[Workflow de Recommandation](recommendation-workflow.md) - Processus de bout en bout

</div>

<div class="doc-card">

**Intégrations**

[Intégration GitHub](integrations/github.md) - Création et suivi de PR
[Intégration Jira](integrations/jira.md) - Gestion des problèmes
[Intégration Bitbucket](integrations/bitbucket.md) - Intégration de dépôt

</div>

<div class="doc-card">

**Guides de Services**

[Optimisation RDS](services/rds.md) - Optimisation de base de données
[Optimisation EC2](services/ec2.md) - Optimisation de calcul
[Optimisation EBS](services/ebs.md) - Optimisation de stockage
[Optimisation ElastiCache](services/elasticache.md) - Optimisation de cache

</div>

<div class="doc-card">

**Référence**

[Référence API](api-reference.md) - Documentation de l'API REST
[Configuration](configuration.md) - Configuration de la plateforme
[Guide de Déploiement](deployment.md) - Déploiement auto-hébergé

</div>

</div>

---

## Support

**Besoin d'aide?**
[support@jetscale.ai](mailto:support@jetscale.ai)
[Discussions GitHub](https://github.com/Jetscale-ai/jetscale-docs/discussions)
[Signaler un Problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

**Ressources:**
[Organisation GitHub](https://github.com/Jetscale-ai) · [Journal des Modifications](https://github.com/Jetscale-ai/jetscale-docs/releases)
