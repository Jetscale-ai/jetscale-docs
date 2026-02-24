# Référence API

> API REST publique pour les intégrations JetScale

## Vue d'ensemble

L'API JetScale vous permet d'accéder de manière programmatique aux recommandations, aux données de coûts et aux fonctionnalités de la plateforme. Utilisez cette API pour créer des intégrations personnalisées, automatiser des workflows ou créer des tableaux de bord internes.

### URL de base

```
Production: https://api.jetscale.ai/v1
```

### Disponible prochainement
L'API publique JetScale est actuellement en version bêta. La documentation complète et l'accès seront disponibles au T2 2025.

Un accès anticipé est disponible pour les clients Entreprise. Contactez [sales@jetscale.ai](mailto:sales@jetscale.ai) pour plus de détails.

---

## Authentification

L'API JetScale utilise des clés API pour l'authentification. Vous pouvez générer des clés API depuis les paramètres de votre compte.

**Incluez votre clé API dans l'en-tête Authorization :**

```bash
curl https://api.jetscale.ai/v1/recommendations \
  -H "Authorization: Bearer YOUR_API_KEY"
```

**Bonnes pratiques de sécurité :**
- Ne jamais exposer les clés API dans le code côté client
- Effectuer une rotation régulière des clés
- Utiliser des clés différentes pour différents environnements
- Révoquer immédiatement les clés inutilisées

---

## Limites de débit

| Forfait | Requêtes/Minute | Requêtes/Heure |
|------|-----------------|---------------|
| Free Trial | 30 | 500 |
| Pro | 100 | 5,000 |
| Enterprise | Personnalisé | Personnalisé |

Les informations de limite de débit sont incluses dans les en-têtes de réponse :

```
X-RateLimit-Limit: 100
X-RateLimit-Remaining: 87
X-RateLimit-Reset: 1706529600
```

---

## Points de terminaison (Disponibles prochainement)

Les points de terminaison suivants seront disponibles dans l'API publique :

### Recommendations

```
GET    /v1/recommendations          List all recommendations
GET    /v1/recommendations/:id      Get recommendation details
POST   /v1/recommendations/:id/approve   Approve recommendation
POST   /v1/recommendations/:id/reject    Reject recommendation
```

### Cloud Accounts

```
GET    /v1/accounts                 List connected cloud accounts
GET    /v1/accounts/:id             Get account details
POST   /v1/accounts                 Connect new cloud account
```

### Cost Data

```
GET    /v1/costs/summary            Get cost summary
GET    /v1/costs/trends             Get cost trends
GET    /v1/savings                  Get realized savings
```

### Resources

```
GET    /v1/resources                List discovered resources
GET    /v1/resources/:id            Get resource details
```

---

## Webhooks (Disponibles prochainement)

Enregistrez des webhooks pour recevoir des notifications en temps réel :

**Événements :**
- `recommendation.created` - Nouvelle recommandation disponible
- `recommendation.approved` - Recommandation approuvée
- `recommendation.deployed` - Modifications déployées
- `savings.milestone` - Jalon d'économies atteint
- `account.connected` - Nouveau compte cloud connecté

**Exemple de charge utile de webhook :**

```json
{
  "event": "recommendation.created",
  "timestamp": "2025-01-29T10:30:00Z",
  "data": {
    "recommendation_id": "rec_abc123",
    "resource_type": "EC2",
    "estimated_savings": 248.16,
    "risk_level": "low"
  }
}
```

---

## Bibliothèques SDK (Feuille de route)

Des bibliothèques SDK officielles sont prévues pour :
- Python
- JavaScript/TypeScript
- Go

**Exemple d'utilisation (Python - Disponible prochainement) :**

```python
from jetscale import JetScaleClient

client = JetScaleClient(api_key="your_api_key")

# Get recommendations
recommendations = client.recommendations.list(
    status="pending",
    min_savings=100
)

# Approve recommendation
client.recommendations.approve("rec_abc123")

# Get cost summary
summary = client.costs.summary(period="last_30_days")
print(f"Total savings: ${summary['total_savings']}")
```

---

## Cas d'usage

### Tableaux de bord personnalisés

Créez des tableaux de bord internes à l'aide de l'API JetScale :
- Extraire les données de recommandations
- Afficher les tendances de coûts
- Afficher les métriques d'économies
- Intégrer les insights JetScale dans les outils existants

### Workflows d'automatisation

Automatisez l'optimisation des coûts :
- Approuver automatiquement les recommandations à faible risque
- Déclencher des déploiements basés sur des règles personnalisées
- Envoyer des notifications Slack pour les nouvelles recommandations
- Générer des rapports de coûts hebdomadaires

### Intégration avec CI/CD

Intégrez l'optimisation des coûts dans votre pipeline de déploiement :
- Vérifier les recommandations avant le déploiement
- Bloquer les déploiements si les seuils de coûts sont dépassés
- Implémenter automatiquement les optimisations approuvées
- Suivre l'impact des déploiements sur les coûts

---

## Premiers pas

**Accès bêta :**

L'API JetScale est actuellement en version bêta privée. Pour demander l'accès :

1. **Contactez les ventes :** [sales@jetscale.ai](mailto:sales@jetscale.ai)
2. **Décrivez votre cas d'usage :** Décrivez vos objectifs d'intégration
3. **Recevez votre clé API :** Obtenez les identifiants d'accès bêta
4. **Rejoignez le programme bêta :** Accédez à la documentation anticipée et au support

**Clients Entreprise :**

Tous les clients avec un forfait Entreprise ont un accès complet à l'API. Contactez votre responsable de la réussite client pour :
- La génération de clés API
- Des limites de débit personnalisées
- Le support d'intégration
- La configuration des webhooks

---

## Support

**Questions sur l'API ?**
- Email : [api-support@jetscale.ai](mailto:api-support@jetscale.ai)
- Problèmes de documentation : [Signaler ici](https://github.com/jetscale-ai/docs/issues)
- Demandes de fonctionnalités : [Demander une fonctionnalité](https://jetscale.ai/feature-requests)

**Documentation connexe :**
- [Comment fonctionne JetScale](how-it-works.md)
- [Intégration GitHub](integrations/github.md)

---

*Dernière mise à jour : 29 janvier 2025*
*Version de l'API : 1.0 (Bêta)*
