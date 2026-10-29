# ci-template

Workflow réutilisable des microservices GoodFood : tests, build de l'image,
scan de vulnérabilités, publication sur `registry.c-mbk.fr` avec un tag
`vX.X.X` incrémenté, puis mise à jour de l'image dans `k3s-gitops`.

## Utilisation

```yaml
name: CI Docker

on:
  push:
    branches: [main]

jobs:
  build:
    uses: GoodFoodCorp/ci-template/.github/workflows/build-deploy.yml@main
    with:
      app_name: "order-service"
      test_image: "golang:1.26"
      test_command: "go vet ./... && go test ./..."
    secrets: inherit
```

## Entrées

| Entrée | Défaut | Rôle |
|---|---|---|
| `app_name` | — | Nom de l'image et du dossier `apps/back/<app_name>` du GitOps |
| `dockerfile_path` | `./Dockerfile` | Dockerfile à construire |
| `test_image` | `""` | Image dans laquelle les tests tournent |
| `test_command` | `""` | Commande de tests ; vide = pas de job de tests |
| `fail_on_vulnerabilities` | `true` | Bloque la publication si Trivy trouve une faille CRITICAL corrigeable |

Exemples de `test_command` :

| Service | `test_image` | `test_command` |
|---|---|---|
| Go | `golang:1.26` | `go vet ./... && go test ./...` |
| NestJS / React | `node:20` | `npm ci && npm test` |
| .NET | `mcr.microsoft.com/dotnet/sdk:8.0` | `dotnet test` |
| Python | `python:3.12` | `pip install -r requirements.txt && pytest` |

## Déroulé

1. **test** : tests du service (sauté si `test_command` est vide).
2. **build-and-push** : uniquement si les tests passent. L'image est construite
   localement, scannée par Trivy, puis poussée avec le nouveau tag `vX.X.X` et
   `latest`.
3. **update-gitops** : remplace l'image dans
   `apps/back/<app_name>/deployment.yaml` par le tag immuable `vX.X.X`.
