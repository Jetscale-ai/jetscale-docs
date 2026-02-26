# FAQ

## Général

<details>
<summary><strong>Qu'est-ce que JetScale?</strong></summary>

JetScale est une plateforme d'optimisation des coûts cloud alimentée par l'IA qui analyse automatiquement votre infrastructure et génère des recommandations exploitables avec du code infrastructure-as-code prêt pour la production. Nos agents IA spécialisés analysent vos ressources AWS et Azure pour identifier les opportunités d'optimisation des coûts tout en maintenant les performances et les SLA.

</details>

<details>
<summary><strong>Comment fonctionne JetScale?</strong></summary>

JetScale utilise des agents IA spécialisés formés sur des services cloud spécifiques (RDS, EC2, EBS, ElastiCache, S3, EKS, Azure VMs). Ces agents analysent les données en temps réel de CloudWatch, Cost Explorer et Azure Monitor pour identifier les opportunités d'optimisation. Lorsque des opportunités sont trouvées, JetScale génère du code Terraform prêt pour la production et s'intègre directement dans vos workflows GitOps existants via GitHub, Jira ou Bitbucket.

</details>

<details>
<summary><strong>Quels fournisseurs cloud JetScale supporte-t-il?</strong></summary>

JetScale prend actuellement en charge AWS et Azure. Nous analysons les services suivants:
- **AWS**: RDS, EC2, EBS, ElastiCache, S3, EKS
- **Azure**: Azure SQL, Machines Virtuelles, Disques Managés, Azure Cache for Redis

Des services supplémentaires (Lambda, DynamoDB, ECS, Azure Cosmos DB, Azure Functions) arrivent bientôt.

</details>

<details>
<summary><strong>Combien puis-je économiser avec JetScale?</strong></summary>

Les économies varient en fonction de votre infrastructure, mais nos clients constatent généralement des réductions de 20 à 40% des coûts cloud. Les économies réelles dépendent de facteurs tels que l'utilisation actuelle des ressources, les types d'instances et l'architecture de l'infrastructure. JetScale fournit une analyse détaillée de l'impact sur les coûts pour chaque recommandation avant que vous ne l'implémentiez.

</details>

## Sécurité et Accès

<details>
<summary><strong>Quelles permissions JetScale nécessite-t-il?</strong></summary>

JetScale nécessite un **accès en lecture seule** à vos comptes cloud:
- **AWS**: Rôle IAM inter-comptes avec permissions de lecture pour Cost Explorer, Compute Optimizer, CloudWatch et métadonnées de service
- **Azure**: Service principal avec les rôles Lecteur, Lecteur Cost Management et Lecteur de Monitoring

Nous ne nécessitons jamais de permissions d'écriture, de modification ou de suppression. Nous ne pouvons pas accéder aux données réelles stockées dans vos bases de données, comptes de stockage ou buckets S3. Nous n'accédons qu'aux métadonnées et aux informations de configuration.

</details>

<details>
<summary><strong>Comment JetScale protège-t-il mes données?</strong></summary>

La sécurité est notre priorité absolue:
- Tous les accès sont en lecture seule sans permissions d'écriture
- Nous utilisons AWS STS avec External ID pour un accès inter-comptes sécurisé
- L'authentification Azure utilise des service principals par certificat ou par secret
- Toutes les transmissions de données sont chiffrées en transit et au repos
- Nous suivons les normes de conformité SOC 2 Type II
- L'accès peut être révoqué instantanément en supprimant le rôle IAM ou le service principal

</details>

<details>
<summary><strong>JetScale peut-il accéder aux données de mon application?</strong></summary>

Non. JetScale n'a accès qu'aux métadonnées d'infrastructure, à la configuration, aux métriques et aux données de coûts. Nous ne pouvons pas accéder:
- Aux données stockées dans les bases de données (RDS, Azure SQL, etc.)
- Aux fichiers dans les buckets S3 ou Azure Storage
- Au code d'application ou aux secrets
- Aux informations d'identification utilisateur ou aux informations sensibles

Nous analysons uniquement les configurations de ressources, les métriques d'utilisation et les informations de coûts.

</details>

<details>
<summary><strong>Comment révoquer l'accès de JetScale?</strong></summary>

L'accès peut être révoqué immédiatement:
- **AWS**: Supprimez le rôle IAM créé pour JetScale
- **Azure**: Supprimez les attributions de rôles du service principal ou supprimez l'enregistrement d'application

Une fois révoqué, JetScale perd immédiatement tout accès à votre compte cloud.

</details>

## Recommandations

<details>
<summary><strong>Comment les recommandations sont-elles générées?</strong></summary>

Nos agents IA analysent plusieurs sources de données:
1. **Métriques de performance**: CPU, mémoire, IOPS, utilisation réseau depuis CloudWatch/Azure Monitor
2. **Données de coûts**: Dépenses actuelles depuis Cost Explorer/Cost Management
3. **Configuration**: Types d'instances, classes de stockage, topologie de cluster
4. **Meilleures pratiques AWS/Azure**: Recommandations Compute Optimizer et Azure Advisor

Les recommandations sont validées pour s'assurer qu'elles maintiennent les exigences de performance et les SLA de votre application avant d'être présentées.

</details>

<details>
<summary><strong>Dans quel format les recommandations sont-elles fournies?</strong></summary>

JetScale génère du **code Terraform** prêt pour la production pour chaque recommandation. Ce code:
- Suit la structure de vos modules Terraform existants et vos conventions de nommage
- Inclut des commentaires détaillés expliquant les modifications
- Peut être révisé et testé avant application
- S'intègre dans votre workflow GitOps via des pull requests

</details>

<details>
<summary><strong>Puis-je personnaliser les critères de recommandation?</strong></summary>

Oui. Vous pouvez configurer:
- **Seuils de coûts**: Montant minimum d'économies pour générer des recommandations
- **Tolérance au risque**: Stratégies d'optimisation conservatrices, équilibrées ou agressives
- **Portée des services**: Activer/désactiver des services ou types de ressources spécifiques
- **Exigences de performance**: Définir des seuils personnalisés de CPU, mémoire ou IOPS
- **Heures d'activité**: Définir les périodes de pointe/hors pointe pour l'analyse de dimensionnement

</details>

<details>
<summary><strong>À quelle fréquence les recommandations sont-elles mises à jour?</strong></summary>

Les recommandations sont continuellement mises à jour en fonction de vos changements d'infrastructure et de vos modèles d'utilisation. De nouvelles recommandations apparaissent lorsque:
- Des changements d'infrastructure sont détectés
- Les modèles d'utilisation évoluent avec le temps
- De nouvelles opportunités d'optimisation des coûts émergent
- AWS/Azure introduisent de nouveaux types d'instances ou modèles de tarification

</details>

<details>
<summary><strong>Que se passe-t-il si j'ignore une recommandation?</strong></summary>

Vous avez toujours le contrôle. Si vous ignorez une recommandation:
- Elle sera marquée comme rejetée dans le tableau de bord
- Vous pouvez fournir des commentaires sur pourquoi elle n'était pas appropriée
- Des recommandations similaires peuvent être supprimées en fonction de vos commentaires
- Vous pouvez revoir les recommandations rejetées à tout moment

</details>

## Intégration

<details>
<summary><strong>Comment JetScale s'intègre-t-il dans mon workflow?</strong></summary>

JetScale s'intègre nativement avec vos outils existants:
- **GitHub**: Crée des pull requests avec les modifications de code Terraform
- **Jira**: Crée des tickets pour les workflows de révision et d'approbation
- **Bitbucket**: S'intègre avec les pipelines Bitbucket et les pull requests
- **Terraform**: Génère du code compatible avec vos modules existants

Les recommandations suivent vos processus standard de révision de code et de déploiement.

</details>

<details>
<summary><strong>JetScale applique-t-il automatiquement les recommandations?</strong></summary>

Non. JetScale **n'applique jamais** automatiquement de modifications à votre infrastructure. Chaque recommandation:
1. Est présentée pour révision humaine
2. Nécessite une approbation explicite
3. Passe par votre processus PR/CI standard
4. Peut être testée dans des environnements hors production d'abord

Vous conservez le contrôle complet sur les modifications implémentées et le moment de leur implémentation.

</details>

<details>
<summary><strong>JetScale peut-il fonctionner avec mon code Terraform existant?</strong></summary>

Oui. JetScale analyse votre état Terraform existant et la structure de vos modules pour:
- Générer du code qui correspond à votre style et vos conventions
- Respecter vos modèles de nommage et l'organisation des ressources
- S'intégrer avec vos modules et variables existants
- Maintenir la compatibilité avec votre version de Terraform

</details>

## Tarification et Support

<details>
<summary><strong>Comment JetScale est-il tarifé?</strong></summary>

La tarification de JetScale est basée sur vos dépenses cloud mensuelles sous gestion. Contactez [support@jetscale.ai](mailto:support@jetscale.ai) pour obtenir des informations de tarification détaillées adaptées à la taille de votre infrastructure et à vos besoins.

</details>

<details>
<summary><strong>Y a-t-il un essai gratuit?</strong></summary>

Oui. Nous offrons un essai gratuit de 14 jours avec accès complet à la plateforme. Pendant l'essai, vous pouvez:
- Connecter vos comptes AWS et/ou Azure
- Recevoir des recommandations d'optimisation des coûts
- Examiner le code Terraform généré
- Tester les intégrations GitHub/Jira/Bitbucket

Aucune carte de crédit requise pour commencer votre essai.

</details>

<details>
<summary><strong>Quelles options de support sont disponibles?</strong></summary>

Nous fournissons:
- **Support par email**: [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation**: Guides complets et référence API
- **Assistance à la configuration**: Aide pratique pour la configuration initiale
- **Support dédié**: Disponible pour les clients entreprise
- **Communauté**: GitHub Discussions pour questions et retours

Les clients entreprise bénéficient d'un support prioritaire avec des canaux Slack dédiés et des garanties SLA.

</details>

<details>
<summary><strong>JetScale peut-il aider avec les environnements multi-cloud?</strong></summary>

Oui. JetScale prend en charge AWS et Azure dans une seule plateforme. Vous pouvez:
- Gérer plusieurs comptes AWS et abonnements Azure
- Obtenir des recommandations d'optimisation des coûts unifiées entre les clouds
- Comparer les coûts entre AWS et Azure pour des services similaires
- Optimiser les ressources sur les deux fournisseurs depuis un seul tableau de bord

</details>
