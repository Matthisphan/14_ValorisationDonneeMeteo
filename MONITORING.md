# Monitoring & Observability

Ce projet inclut une stack complète de monitoring avec **Prometheus** et **Grafana**.

## Architecture

```
┌─────────────┐
│  Backend    │ (expose /metrics)
├─────────────┤
       ↓
┌─────────────┐
│ Prometheus  │ (scrape toutes les 5s)
├─────────────┤
       ↓
┌─────────────┐
│   Grafana   │ (visualize les métriques)
└─────────────┘
```

## Démarrage de la stack

```bash
# Lancer tous les services
docker compose -f docker-compose.dev.yml up -d

# Vérifier le statut
docker compose -f docker-compose.dev.yml ps
```

## Accès aux services

| Service | URL | Identifiants |
|---------|-----|--------------|
| **Prometheus** | http://localhost:9090 | - |
| **Grafana** | http://localhost:3000 | admin / admin |
| **Backend** | http://localhost:8000 | - |

## Prometheus

### Vérifier les métriques

1. Ouvre http://localhost:9090
2. Onglet "Graph"
3. Tape une métrique (ex: `up{job="django"}`)
4. Clique "Execute"

### Targets configurées

Le fichier `prometheus.yml` scrape les endpoints suivants :

```yaml
scrape_configs:
  - job_name: "django"
    static_configs:
      - targets: ["backend:8000"]
```

**Note** : Le backend doit exposer un endpoint `/metrics` pour que Prometheus puisse collecter les métriques.

## Grafana

### Accès initial

- **URL** : http://localhost:3000
- **Username** : admin
- **Password** : admin

### Datasource Prometheus

La datasource Prometheus est auto-provisionnée au démarrage. Pour vérifier :

1. Onglet "Configuration" > "Data Sources"
2. Tu devrais voir "Prometheus" avec l'URL `http://prometheus:9090`

### Dashboard "Application Metrics"

Un dashboard pré-configuré est fourni avec :

- **Backend Service Status** : statut du service (up/down)
- **Backend CPU Usage** : utilisation CPU en %
- **Backend Memory Usage** : utilisation mémoire en MB

Le dashboard se met à jour toutes les 10 secondes.

### Créer un dashboard personnalisé

1. Clique "+" > "Dashboard"
2. Clique "Add a new panel"
3. Configure la métrique (ex: `up{job="django"}`)
4. Sauvegarde le dashboard

## Métriques à monitorer

Métriques réellement exposées par **django-prometheus** sur l'endpoint `/metrics` :

```promql
# Status du service Django
up{job="django"}

# Requêtes HTTP par méthode
sum(rate(django_http_requests_total_by_method_total{job="django"}[5m])) by (method)

# Latence p95 par vue et méthode
histogram_quantile(0.95, sum(rate(django_http_requests_latency_seconds_by_view_method_bucket{job="django"}[5m])) by (le, view, method))

# Requêtes HTTP par vue
sum(rate(django_http_requests_total_by_view_transport_method_total{job="django"}[5m])) by (view)

# Réponses HTTP par statut
sum(rate(django_http_responses_total_by_status_total{job="django"}[5m])) by (status)

# Exceptions par vue
sum(rate(django_http_exceptions_total_by_view_total{job="django"}[5m])) by (view)
```

### Dashboard Grafana fourni

Le dashboard auto-provisionné **Django Backend Metrics** contient :

- Backend Service Status
- HTTP Requests by Method
- Request Latency p95
- HTTP Requests by View
- HTTP Responses by Status
- Exceptions by View

**Astuce** : si les panels sont vides, vérifie d'abord dans Prometheus que la cible est `UP`, puis génère quelques requêtes vers le backend avant de rafraîchir Grafana.

## Dépannage

### Prometheus ne scrape pas le backend

1. Vérifie que le backend est accessible : `curl http://localhost:8000/metrics`
2. Vérifie que l'endpoint `/metrics` retourne du contenu Prometheus
3. Redémarre Prometheus : `docker compose -f docker-compose.dev.yml restart prometheus`

### Grafana n'affiche pas les données

1. Vérifie la datasource Prometheus (Configuration > Data Sources)
2. Teste la connexion avec un bouton "Test" dans la configuration
3. Vérifie que les métriques existent dans Prometheus (http://localhost:9090/graph)

### Réinitialiser Grafana

```bash
docker compose -f docker-compose.dev.yml down
docker volume rm <project>_grafana-storage
docker compose -f docker-compose.dev.yml up -d
```

## Production

Pour la production, consulte la [documentation de Prometheus](https://prometheus.io/docs/prometheus/latest/configuration/configuration/) et [Grafana](https://grafana.com/docs/grafana/latest/).

Les images de base de production doivent utiliser les **Docker Hardened Images (DHI)** si disponibles.
