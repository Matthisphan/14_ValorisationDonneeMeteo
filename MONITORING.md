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

Métriques exposées par **django-prometheus** sur l'endpoint `/metrics` :

```promql
# Status du service Django
up{job="django"}

# Requêtes HTTP totales
rate(django_http_requests_total[5m])

# Latence des requêtes (95e centile)
django_http_request_duration_seconds_bucket{le="0.5"}

# Exceptions non capturées
django_http_requests_unknown_latency_total

# Requêtes par vue
rate(django_http_requests_total{job="django", view="*"}[5m])

# Database queries
rate(django_db_execute_total[5m])

# Model saves
rate(django_model_saves_total[5m])
```

**Exemple de requête Prometheus :**
Accède à http://localhost:9090/graph et tape :
```
rate(django_http_requests_total[5m])
```

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
