# Optimisation des machines virtuelles Azure

JetScale offre une optimisation des coûts basée sur l'IA pour les machines virtuelles Azure. Nos agents spécialisés analysent vos charges de travail de calcul pour identifier les opportunités de dimensionnement optimal, les migrations de familles de VM et les alternatives rentables.

## Vue d'ensemble

JetScale optimise les ressources des machines virtuelles Azure en analysant :
- **Utilisation des instances** : modèles d'utilisation du CPU, de la mémoire, du disque et du réseau
- **Analyse des coûts** : dépenses actuelles vs configuration optimale
- **Caractéristiques des charges de travail** : exigences en ressources et modèles de performance
- **Options de tarification** : paiement à l'utilisation, instances réservées, Spot VMs

## Types de VM pris en charge

### Machines virtuelles autonomes

JetScale optimise les VM Azure autonomes qui ne font pas partie de Virtual Machine Scale Sets (VMSS).

**Architecture de VM :**
```
VirtualMachine (Cible d'optimisation, Facturable)
├── OSDisk - Facturable
├── DataDisk - Facturable (optimisé séparément)
└── DataDisk - Facturable
```

**Ce que nous optimisons :**
- **Redimensionnement de la taille de VM** : changement vers des tailles plus petites/plus grandes selon l'utilisation réelle
- **Migration de séries de VM** : basculement entre séries (B, D, E, F, etc.) selon les caractéristiques de la charge de travail
- **Mises à niveau de génération** : migration vers la dernière génération (v5, v6) pour un meilleur rapport prix/performance
- **Suppression des VM inutilisées** : retrait des VM avec une activité nulle ou quasi nulle (100% d'économies)

**Important :** JetScale optimise actuellement uniquement les VM Azure autonomes. Les VM gérées par Virtual Machine Scale Sets, AKS ou d'autres systèmes d'orchestration ne sont pas encore prises en charge.

## Séries et familles de VM

JetScale prend en compte toutes les séries de VM Azure lors de l'optimisation :

### Usage général (B, D)

**Série B** - VM à rafales
- À utiliser pour : dev/test, serveurs web à faible trafic, petites bases de données, microservices
- Idéale quand : l'utilisation du CPU est faible la plupart du temps avec des rafales occasionnelles
- Attention : basée sur des crédits CPU, surveillez le solde de crédits pour éviter la limitation
- Tailles : B1s, B1ms, B2s, B2ms, B4ms, B8ms, B12ms, B16ms, B20ms

**Série D** - Calcul/mémoire/réseau équilibrés
- À utiliser pour : serveurs web, serveurs d'applications, bases de données moyennes, charges de travail mixtes
- Dernières générations : Dv5/Dsv5 > Dv4/Dsv4 > Dv3/Dsv3
- Disponible en : Standard HDD, Standard SSD, Premium SSD
- Tailles : D2-D96 (2-96 vCPU)

### Optimisées pour le calcul (F)

**Série F** - Ratio CPU/mémoire élevé
- À utiliser pour : traitement par lots, serveurs d'applications, serveurs web, analytique, jeux
- Idéale quand : l'utilisation du CPU est constamment élevée, les besoins en mémoire sont modérés
- Dernières générations : Fv2 (Intel), Fasv5/Famsv5 (AMD)
- Tailles : F2-F72 (2-72 vCPU)

### Optimisées pour la mémoire (E, M)

**Série E** - Ratio mémoire/CPU élevé
- À utiliser pour : bases de données relationnelles, analytique en mémoire, applications SAP, mise en cache
- Idéale quand : l'utilisation de la mémoire est élevée, le CPU est modéré
- Dernières générations : Ev5/Esv5 > Ev4/Esv4 > Ev3/Esv3
- Tailles : E2-E96 (2-96 vCPU, jusqu'à 672 Go de RAM)

**Série M** - Mémoire extrême (jusqu'à 4 To de RAM)
- À utiliser pour : large SQL Server, SAP HANA, bases de données en mémoire
- Surcoût : significativement plus cher que la série E
- Charges de travail critiques nécessitant une mémoire massive
- Tailles : M8ms - M416ms (8-416 vCPU)

### Optimisées pour le stockage (L)

**Série L** - Débit disque et E/S élevés
- À utiliser pour : big data, bases de données SQL, bases de données NoSQL, entreposage de données
- Stockage local basé sur NVMe
- Idéale pour : charges de travail intensives en E/S nécessitant une faible latence
- Tailles : L4s-L80s (4-80 vCPU)

### Calcul accéléré (N)

**Série N** - VM avec GPU
- À utiliser pour : formation et inférence IA/ML, rendu graphique, traitement vidéo
- Options : NC (NVIDIA Tesla), NV (NVIDIA Tesla M60), ND (NVIDIA Tesla P40/V100)
- Non recommandée généralement pour l'optimisation des coûts (charges de travail spécialisées)

### Modèles de dimensionnement des VM

Les tailles suivent le modèle : `{Série}{Version}{Taille}`

Exemples :
- **B2s** : série B, 2 vCPU, support disque Standard
- **D4s_v5** : série D v5, 4 vCPU, support Premium SSD
- **E8as_v5** : série E v5 AMD, 8 vCPU, support Premium SSD

**Progression des tailles :**
- Standard : 1, 2, 4, 8, 16, 32, 64, 96+ vCPU
- À rafales : 1, 2, 4, 8, 12, 16, 20 vCPU

Chaque étape double généralement les vCPU et la mémoire.

## Comment JetScale optimise les VM Azure

### 1. Collecte de données

JetScale analyse plusieurs sources de données :

**Métriques Azure Monitor** (fenêtre glissante de 14 jours) :
- `Percentage CPU` - Utilisation CPU de la VM
- `Available Memory Bytes` - Mémoire disponible
- `Network In Total` / `Network Out Total` - Débit réseau
- `Disk Read Bytes` / `Disk Write Bytes` - E/S disque
- `Disk Read Operations/Sec` / `Disk Write Operations/Sec` - IOPS

**Données Azure Resource Manager** :
- Taille et état de la VM
- Type de système d'exploitation (Windows/Linux)
- Date de création
- Emplacement et zone de disponibilité
- Tags et groupes de ressources

**Données Cost Management** :
- Dépense mensuelle actuelle par VM
- Tendances des coûts historiques
- Couverture des instances réservées

**Azure Advisor** (si disponible) :
- Recommandations de dimensionnement générées par Azure
- Évaluations des risques de performance

### 2. Analyse

Nos agents IA effectuent une analyse approfondie :

**Modèles d'utilisation :**
- Utilisation CPU maximale vs moyenne
- Modèles horaires (identification des périodes d'inactivité)
- Modèles hebdomadaires (charge week-end vs jours de semaine)
- Tendances d'utilisation de la mémoire

**Classification des charges de travail :**
- **CPU variable** : envisager série B (à rafales)
- **CPU élevé constant** : envisager série F (optimisée calcul)
- **Équilibrée** : envisager série D (usage général)
- **Intensive en mémoire** : envisager série E (optimisée mémoire)

**Modélisation des coûts :**
- Coût mensuel actuel
- Coût projeté pour les tailles de VM alternatives
- Pourcentage d'économies et impact annuel
- Opportunités d'instances réservées

**Évaluation des risques :**
- Probabilité de dégradation des performances
- Calcul de marge (tampon au-dessus de l'utilisation maximale)
- Vérifications de sécurité mémoire
- Validation de la capacité réseau et stockage

### 3. Recommandations

JetScale génère des recommandations spécifiques et actionnables :

#### Redimensionnement de VM

**Exemple de recommandation :**
```
Ressource : web-server-prod-01
Actuelle : Standard_D16s_v5 (16 vCPU, 64 Go RAM)
Recommandée : Standard_D8s_v5 (8 vCPU, 32 Go RAM)

Impact sur les coûts :
- Actuel : 550 $/mois
- Projeté : 275 $/mois
- Économies : 275 $/mois (50 %), 3 300 $/an

Analyse de performance :
- CPU moyen : 18 %
- CPU maximal : 32 %
- Mémoire moyenne : 35 %
- La recommandée offre une marge de 2x au-dessus du pic

Risque : Faible - Marge amplement maintenue
```

#### Migration de série

**Exemple de recommandation :**
```
Ressource : batch-processor-02
Actuelle : Standard_D8s_v5 (8 vCPU, 32 Go RAM)
Recommandée : Standard_F8s_v2 (8 vCPU, 16 Go RAM)

Impact sur les coûts :
- Actuel : 275 $/mois
- Projeté : 233 $/mois
- Économies : 42 $/mois (15 %), 504 $/an

Analyse de charge de travail :
- CPU moyen : 75 %
- CPU maximal : 92 %
- Utilisation mémoire : Faible (28 %)
- Type de charge : Intensive en calcul, mémoire sous-utilisée

Recommandation : Migrer vers série F (optimisée calcul)
Risque : Très faible - Charge liée au CPU, marge mémoire adéquate
```

#### Mise à niveau de génération

**Exemple de recommandation :**
```
Ressource : api-server-01
Actuelle : Standard_D4s_v3 (4 vCPU, 16 Go RAM)
Recommandée : Standard_D4s_v5 (4 vCPU, 16 Go RAM)

Impact sur les coûts :
- Actuel : 175 $/mois
- Projeté : 165 $/mois
- Économies : 10 $/mois (6 %), 120 $/an

Analyse de performance :
- v5 offre de meilleures performances CPU (Intel Xeon Platinum 8370C)
- Mêmes spécifications, coût inférieur
- Meilleures performances réseau (jusqu'à 12,5 Gbps)
- Efficacité de nouvelle génération

Risque : Très faible - Même série, génération plus récente
```

#### Suppression de VM inutilisée

**Exemple de recommandation :**
```
Ressource : legacy-test-vm
Actuelle : Standard_B2s (2 vCPU, 4 Go RAM)
Recommandée : Supprimer/Désallouer

Impact sur les coûts :
- Actuel : 30 $/mois
- Projeté : 0 $/mois
- Économies : 30 $/mois (100 %), 360 $/an

Analyse d'utilisation :
- CPU moyen : 0,3 %
- Activité réseau : Quasi nulle
- Créée : Il y a 14 mois
- Tags indiquent : environnement de test

Risque : Très faible - Semble inutilisée
Recommandation : Vérifier avec l'équipe applicative avant suppression
```

### 4. Génération Terraform

Pour chaque recommandation, JetScale génère du code Terraform prêt pour la production :

**Exemple : Redimensionnement de VM**

```hcl
# Optimisation de VM Azure
# Généré par JetScale le 2024-01-15
# ID de recommandation : rec_azurevm_001

resource "azurerm_linux_virtual_machine" "web_server_prod_01" {
  name                = "web-server-prod-01"
  resource_group_name = var.resource_group_name
  location            = var.location

  # Précédent : Standard_D16s_v5 (550 $/mois)
  # Optimisé : Standard_D8s_v5 (275 $/mois)
  # Réduction de coût : 50 % (275 $/mois, 3 300 $/an)
  #
  # Justification :
  # - Utilisation CPU moyenne : 18 %
  # - Utilisation CPU maximale : 32 %
  # - Utilisation mémoire moyenne : 35 %
  # - La nouvelle taille offre une marge de 2x au-dessus de l'utilisation maximale
  size = "Standard_D8s_v5"

  admin_username = var.admin_username

  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }

  network_interface_ids = [
    azurerm_network_interface.web_server_nic.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 128
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"      = "true"
      "jetscale:recommendation" = "rec_azurevm_001"
      "jetscale:previous_size"  = "Standard_D16s_v5"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

**Exemple : Migration de série**

```hcl
# Migration de série de VM Azure
# Généré par JetScale le 2024-01-15

resource "azurerm_linux_virtual_machine" "batch_processor_02" {
  name                = "batch-processor-02"
  resource_group_name = var.resource_group_name
  location            = var.location

  # Précédent : Standard_D8s_v5 (Usage général, 275 $/mois)
  # Optimisé : Standard_F8s_v2 (Optimisé calcul, 233 $/mois)
  # Réduction de coût : 15 % (42 $/mois, 504 $/an)
  #
  # Analyse de charge de travail :
  # - CPU moyen : 75 %, CPU maximal : 92 %
  # - Utilisation mémoire : 28 % (sous-utilisée)
  # - Type de charge : Intensive en calcul
  # - Série F offre un ratio CPU/mémoire plus élevé
  size = "Standard_F8s_v2"

  admin_username = var.admin_username

  admin_ssh_key {
    username   = var.admin_username
    public_key = var.ssh_public_key
  }

  network_interface_ids = [
    azurerm_network_interface.batch_processor_nic.id,
  ]

  os_disk {
    caching              = "ReadWrite"
    storage_account_type = "Premium_LRS"
    disk_size_gb         = 256
  }

  source_image_reference {
    publisher = "Canonical"
    offer     = "0001-com-ubuntu-server-focal"
    sku       = "20_04-lts-gen2"
    version   = "latest"
  }

  tags = merge(
    var.tags,
    {
      "jetscale:optimized"     = "true"
      "jetscale:series"        = "compute-optimized"
      "jetscale:previous_size" = "Standard_D8s_v5"
    }
  )

  lifecycle {
    create_before_destroy = true
  }
}
```

## Bonnes pratiques

### Stratégie de test

**Tests en pré-production :**
1. Appliquer d'abord les changements à l'environnement dev/staging
2. Exécuter des tests de charge simulant le trafic de pointe
3. Surveiller pendant 48-72 heures sous charge réaliste
4. Valider les performances de l'application

**Déploiement en production :**
1. Utiliser le lifecycle `create_before_destroy` dans Terraform
2. Planifier les changements pendant les fenêtres de maintenance
3. Avoir un plan de retour arrière prêt (état Terraform précédent)
4. Surveiller activement pendant et après le changement

### Instances réservées et plans d'épargne

Bien que JetScale ne génère pas actuellement de recommandations RI, envisagez-les après le redimensionnement :

**Bonne pratique :**
1. D'abord redimensionner les VM avec JetScale
2. Exécuter la configuration optimisée pendant 30-60 jours
3. Acheter des instances réservées pour les charges stables (1 an ou 3 ans)
4. Ne jamais réserver des VM surdimensionnées

**Instances réservées de VM Azure :**
- Jusqu'à 72 % d'économies vs paiement à l'utilisation
- Engagement d'1 an ou 3 ans
- Flexibilité de taille au sein de la même série de VM
- Options de paiement : Tout initial, Partiel initial, Mensuel

### Spot VMs

Pour les charges de travail non critiques, envisagez les Spot VMs Azure :
- Jusqu'à 90 % d'économies vs paiement à l'utilisation
- Idéales pour : traitement par lots, dev/test, applications sans état
- Risque : Peuvent être évincées avec un préavis de 30 secondes quand Azure a besoin de capacité
- JetScale identifie les candidates à la migration Spot (fonctionnalité en développement)

## Modèles d'optimisation courants

### Modèle 1 : VM sur-provisionnées

**Symptômes :**
- Utilisation CPU < 20 % en moyenne
- Utilisation mémoire < 30 %
- VM fonctionnant 24/7 avec une utilisation constamment faible

**Recommandation JetScale :**
- Réduire de 1-2 tailles de VM (ex. : D16 → D8)
- Économies typiques : 40-50 %
- Risque : Faible (marge de 2-3x maintenue)

### Modèle 2 : Mauvaise série de VM

**Symptômes :**
- Utilisation de série D avec CPU constamment élevé (>70 %)
- Utilisation de série E avec faible utilisation mémoire (<40 %)
- Utilisation de série F avec besoins élevés en mémoire

**Recommandation JetScale :**
- Migrer vers la série appropriée (D → F pour CPU, E → D pour équilibré)
- Économies typiques : 10-30 %
- Risque : Faible à Moyen (vérifier les caractéristiques de charge)

### Modèle 3 : Générations héritées

**Symptômes :**
- Utilisation de VM v3 ou v4
- Générations plus récentes disponibles (v5, v6)
- Même coût ou inférieur pour de meilleures performances

**Recommandation JetScale :**
- Mise à niveau vers la dernière génération (Dv3 → Dv5, Ev4 → Ev5)
- Économies typiques : 5-15 % avec meilleures performances
- Risque : Très faible (même série, matériel plus récent)

### Modèle 4 : VM de développement inactives

**Symptômes :**
- VM dev/test fonctionnant 24/7
- Utilisées uniquement pendant les heures de bureau
- Faible utilisation CPU la nuit et les week-ends

**Recommandation JetScale :**
- Implémenter des planifications d'arrêt automatique (fonctionnalité Azure)
- Réduire vers série B pour charges à rafales
- Économies typiques : 50-70 % avec approche combinée
- Risque : Très faible (environnement hors production)

## Dépannage

### Préoccupations concernant les recommandations

**Q : Le redimensionnement de ma VM impactera-t-il les performances ?**

R : JetScale maintient une marge de 2-3x au-dessus de l'utilisation maximale. Nous analysons :
- Utilisation CPU au 99e percentile (P99) sur 14 jours
- Utilisation mémoire maximale
- Points de saturation réseau
- Modèles de pics historiques

Les recommandations ne procèdent que si le risque de performance est Faible ou Très faible.

**Q : Que faire si je n'ai pas de métriques mémoire ?**

R : Azure Monitor fournit des métriques mémoire par défaut pour toutes les VM. Si les métriques sont manquantes :
1. Vérifier que l'agent Azure Monitor est installé
2. Vérifier que les paramètres de diagnostic sont activés
3. Attendre 14 jours pour des données historiques suffisantes

**Q : Puis-je tester sans interruption de service ?**

R : Oui ! Approche recommandée :
1. Utiliser `create_before_destroy = true` dans Terraform
2. La nouvelle VM se lance avec la taille optimisée
3. Vérification de santé réussie → le trafic bascule vers la nouvelle VM
4. Ancienne VM automatiquement terminée
5. Interruption typique : <30 secondes pendant la bascule

### Problèmes de performance après optimisation

**Symptôme : Augmentation de la latence ou des temps de réponse**

Causes possibles :
- CPU insuffisant pour les charges de pointe
- Pression mémoire
- Goulot d'étranglement du débit réseau

**Résolution :**
1. Vérifier les métriques CPU, mémoire, réseau Azure Monitor
2. Comparer les performances VM actuelles vs précédentes
3. Si nécessaire, augmenter la taille de VM d'un cran
4. Retour arrière simple : revenir à l'état Terraform précédent

**Symptôme : Épuisement des crédits CPU série B**

Causes possibles :
- Migration de VM standard vers à rafales
- Utilisation CPU supérieure à la ligne de base série B
- Crédits CPU insuffisants pour le modèle de charge

**Résolution :**
1. Vérifier la métrique `CPU Credits Remaining`
2. Si constamment épuisée, migrer vers série D/F
3. Envisager série B avec ligne de base plus élevée (ex. : B4ms → B8ms)

## Considérations de sécurité

### Configuration des VM

Les recommandations JetScale préservent :
- Associations de réseau virtuel et sous-réseau
- Groupes de sécurité réseau (NSG)
- Identités gérées
- Références de coffre de clés
- Paramètres de chiffrement de disque
- Configuration des diagnostics de démarrage

### Compatibilité des images

- Utilise la même image/SKU de système d'exploitation que la VM d'origine
- Préserve les images et configurations personnalisées
- Maintient les attachements de disque OS et disques de données
- Aucun changement aux méthodes d'authentification

### Gestion du cycle de vie

Utiliser les règles de lifecycle Terraform :
```hcl
lifecycle {
  create_before_destroy = true
  prevent_destroy       = var.is_production
}
```

## Limitations

**Actuellement non pris en charge :**
- Optimisation des Virtual Machine Scale Sets (VMSS)
- Recommandations Spot VM
- Recommandations d'achat d'instances réservées
- Planification d'arrêt automatique
- Optimisation des pools de nœuds AKS
- Orchestration multi-VM

**Feuille de route :**
Ces fonctionnalités sont prévues pour les versions futures. Actuellement, JetScale se concentre sur l'optimisation des VM Azure autonomes où des économies de coûts immédiates peuvent être réalisées grâce au redimensionnement et à la migration.

## Intégration API

JetScale fournit un accès API pour l'optimisation programmatique :

```bash
# Lister les recommandations de VM Azure
GET /api/v1/recommendations?resource_type=azure-vm

# Obtenir les détails d'une recommandation spécifique
GET /api/v1/recommendations/{recommendation_id}

# Approuver une recommandation (génère le Terraform)
POST /api/v1/recommendations/{recommendation_id}/approve

# Récupérer le Terraform généré
GET /api/v1/recommendations/{recommendation_id}/terraform
```

Consultez notre [documentation API](../api-reference.md) pour la référence complète.

## Support

Besoin d'aide pour l'optimisation des VM Azure ?

- **Email** : [support@jetscale.ai](mailto:support@jetscale.ai)
- **Documentation** : [FAQ](../faq.md)
- **Issues GitHub** : [Signaler un problème](https://github.com/Jetscale-ai/jetscale-docs/issues)

---

**Documentation connexe :**
- [Optimisation Azure SQL](azure-sql.md)
- [Optimisation EC2](ec2.md)
