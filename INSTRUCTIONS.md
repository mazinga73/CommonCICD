# CommonCICD Repository — Istruzioni

Questo repository contiene **template riutilizzabili per pipeline CI/CD** e chart Helm per microservizi .NET containerizzati. I template sono progettati per essere copiati e adattati a nuovi servizi tramite sostituzione di placeholder.

---

## Indice

1. [Struttura del repository](#struttura-del-repository)
2. [Template disponibili](#template-disponibili)
3. [Placeholder e sostituzioni](#placeholder-e-sostituzioni)
4. [Prerequisiti infrastrutturali](#prerequisiti-infrastrutturali)
5. [Istruzioni per Claude](#istruzioni-per-claude)
6. [Esempi di applicazione](#esempi-di-applicazione)

---

## Struttura del repository

```
CommonCICD/
├── github-actions/
│   ├── Kubernetes_Service_CI.yaml      # Template per servizi Kubernetes
│   └── NugetPackage_CI.yaml            # Template per librerie NuGet
└── helm/
    └── charts/
        ├── Chart.yaml                  # Metadati del chart
        ├── values.yaml                 # Valori di default
        └── templates/
            └── deployment.yaml         # Template Deployment Kubernetes
```

---

## Template disponibili

### 1. `github-actions/Kubernetes_Service_CI.yaml`

**Uso:** Build e deploy di microservizi containerizzati su Kubernetes.

**Cosa fa:**
- Ascolta push su path specifici della cartella servizio (path filtering per monorepo)
- Costruisce un'immagine Docker con tag doppio: `run_id` (immutabile) e `latest`
- Esegue login a GitHub Container Registry (ghcr.io)
- Spinge le immagini a ghcr.io
- Installa Helm
- Installa/reinstalla il release Helm (strategia: uninstall + install)

**Stack:** Docker, Helm 3+, Kubernetes, GitHub Container Registry (ghcr.io)

**Runner:** `self-hosted` (il runner deve essere nel cluster Kubernetes o avere kubeconfig)

**Placeholder da sostituire:**
- `[SERVICE_NAME]` — nome del microservizio
- `[PATH_SERVICENAME]` — path relativo della cartella servizio nel monorepo (es. `src/MyService`)
- `[PATH_SERVICE_NAME]` — idem con underscore (nel Dockerfile path)
- `[GITHUB_REPOSITORY]` — nome del repository GitHub (formato: `owner/repo`)
- `[HELM_CHART_NAME]` — nome del release Helm (es. `my-service`)
- `[HELM_CHART_PATH]` — path ai chart Helm nel runner (es. `helm/charts`)

**Secret richiesto:**
- `secrets.TOKEN` — Personal Access Token o token con scope `packages:write` per push a ghcr.io

---

### 2. `github-actions/NugetPackage_CI.yaml`

**Uso:** Build, versionamento e pubblicazione di librerie .NET su GitHub Packages.

**Cosa fa:**
- Ascolta push su path specifici della cartella libreria
- Calcola versione SemVer automaticamente con GitVersion (da commit history)
- Esegue `dotnet build` e `dotnet pack` con versione SemVer
- Carica il pacchetto `.nupkg` come artefatto GitHub
- Job separato `release`: scarica artefatto e pubblica su GitHub Packages (solo su branch main)

**Stack:** .NET, GitVersion, GitHub Packages, GitHub Actions (artifacts)

**Runner:** `ubuntu-latest` (cloud-hosted)

**Placeholder da sostituire:**
- `[LIBRARYNAME]` — nome della libreria (es. `CommonUtils`)
- `[PATH_LIBRARYNAME]` — path relativo della cartella libreria nel monorepo (es. `src/CommonUtils`)

**Secret richiesto:**
- `secrets.TOKEN` — Personal Access Token con scope `packages:write`

---

### 3. `helm/charts/` (Helm Chart template)

**Uso:** Deployment di microservizi su Kubernetes.

**Componenti:**

#### `Chart.yaml`
Metadati del chart. Placeholder:
- `[HELM_CHART_NAME]` — nome del chart e del release
- `[SERVICE_NAME]` — nome del servizio

#### `values.yaml`
Valori di default. Configura:
- Repository immagine Docker: `ghcr.io/[GITHUB_REPOSITORY]/[SERVICE_NAME]`
- Tag immagine: `latest` (coerente con il workflow CI)
- Pull policy: `Always` (garantisce immagine aggiornata con tag `latest`)
- Replica count: `1` (modificabile per scalare)
- Resource limits e requests: CPU/Memory

#### `templates/deployment.yaml`
Template Kubernetes per il Deployment. Caratteristiche:
- Usa `imagePullSecrets: ghcr-secret` (il cluster deve avere questo Secret pre-configurato)
- `rollme` annotation: timestamp Unix per forzare rolling update anche con tag `latest`
- Resource limits/requests da `values.yaml`
- Label e naming coerente da `Chart.Name`

---

## Placeholder e sostituzioni

Tabella completa di tutti i placeholder e come sostituirli:

| Placeholder | Significato | Esempio |
|---|---|---|
| `[SERVICE_NAME]` | Nome del microservizio | `OrderService` |
| `[PATH_SERVICENAME]` | Path relativo nel monorepo | `src/OrderService` |
| `[PATH_SERVICE_NAME]` | Idem con underscore | `src/OrderService` |
| `[GITHUB_REPOSITORY]` | Owner/repo GitHub | `myorg/myrepo` |
| `[HELM_CHART_NAME]` | Nome del release Helm | `order-service` |
| `[HELM_CHART_PATH]` | Path ai chart nel runner | `helm/charts` |
| `[LIBRARYNAME]` | Nome della libreria .NET | `CommonUtils` |
| `[PATH_LIBRARYNAME]` | Path relativo della libreria | `src/CommonUtils` |

---

## Prerequisiti infrastrutturali

### Per il workflow `Kubernetes_Service_CI.yaml`:

1. **Runner self-hosted con accesso al cluster:**
   - Il runner GitHub Actions registrato deve essere nel cluster Kubernetes
   - oppure avere `kubeconfig` configurato con accesso al cluster
   - Docker installato nel runner

2. **Credenziali ghcr.io:**
   - Secret GitHub Actions: `TOKEN` (PAT con scope `packages:write`)

3. **Secret Kubernetes nel cluster:**
   - Creare un Secret di tipo `docker-registry` con nome `ghcr-secret`:
     ```bash
     kubectl create secret docker-registry ghcr-secret \
       --docker-server=ghcr.io \
       --docker-username=<github_username> \
       --docker-password=<PAT_token> \
       --docker-email=<email>
     ```

4. **Helm installato nel runner**
   - Il workflow esegue `curl ... | bash` per installarlo

### Per il workflow `NugetPackage_CI.yaml`:

1. **Secret GitHub Actions:**
   - `TOKEN` (PAT con scope `packages:write`)

2. **.NET SDK:**
   - Workflow usa `setup-dotnet@v1` per .NET 10.0.x
   - Adattare la versione a seconda del progetto

---

## Istruzioni per Claude

Quando applicare questi template a un nuovo repository:

### Per creare una pipeline CI/CD Kubernetes per un servizio:

1. **Copia il file** `github-actions/Kubernetes_Service_CI.yaml` in `.github/workflows/[SERVICE_NAME]_CI.yaml`

2. **Sostituisci i placeholder:**
   - `[SERVICE_NAME]` → nome del servizio (es. `OrderService`)
   - `[PATH_SERVICENAME]` → path della cartella (es. `src/OrderService`)
   - `[PATH_SERVICE_NAME]` → idem
   - `[GITHUB_REPOSITORY]` → owner/repo (es. `myorg/myrepo`)
   - `[HELM_CHART_NAME]` → nome Helm (es. `order-service`)
   - `[HELM_CHART_PATH]` → path chart (es. `helm/charts`)

3. **Copia la cartella Helm:**
   - Copia `helm/charts/` in `helm/charts/` del target repository (se non esiste)
   - Adatta i nomi secondo i placeholder

4. **Verifica di avere i prerequisiti** (runner self-hosted, secrets, etc.)

### Per creare una pipeline NuGet:

1. **Copia il file** `github-actions/NugetPackage_CI.yaml` in `.github/workflows/[LIBRARYNAME]_CI.yaml`

2. **Sostituisci i placeholder:**
   - `[LIBRARYNAME]` → nome della libreria (es. `CommonUtils`)
   - `[PATH_LIBRARYNAME]` → path della libreria (es. `src/CommonUtils`)

3. **Verifica di avere:**
   - Secret `TOKEN` con scope `packages:write`
   - .NET SDK nella versione corretta (adattare `dotnet-version` nel workflow se necessario)

---

## Esempi di applicazione

### Esempio 1: Servizio "OrderService" in monorepo

**Repository target:** `myorg/myrepo`
**Servizio:** `OrderService` in cartella `src/OrderService`

#### Step 1: Crea workflow Kubernetes

Copia `Kubernetes_Service_CI.yaml` e rinomina in `.github/workflows/OrderService_CI.yaml`

Sostituisci:
```yaml
# PRIMA
on:
  push:
    paths:
      - '[PATH_SERVICENAME]/**'
      - '.github/workflows/[SERVICE_NAME]_CI.yaml'

# DOPO
on:
  push:
    paths:
      - 'src/OrderService/**'
      - '.github/workflows/OrderService_CI.yaml'
```

```yaml
# PRIMA
docker build -t ghcr.io/${{ github.repository_owner }}/[GITHUB_REPOSITORY]/[SERVICE_NAME]:...

# DOPO
docker build -t ghcr.io/${{ github.repository_owner }}/myorg/myrepo/OrderService:...
```

```yaml
# PRIMA
helm install [HELM_CHART_NAME] [HELM_CHART_PATH]/[HELM_CHART_NAME]

# DOPO
helm install order-service helm/charts/order-service
```

#### Step 2: Crea chart Helm

Copia `helm/charts/` in target repo come `helm/charts/order-service/`

Modifica `helm/charts/order-service/Chart.yaml`:
```yaml
name: order-service
description: Helm chart for OrderService application
```

Modifica `helm/charts/order-service/values.yaml`:
```yaml
dockerImage:
  repository: ghcr.io/myorg/myrepo/OrderService
  tag: latest
  pullPolicy: Always
```

### Esempio 2: Libreria "CommonUtils"

**Repository target:** `myorg/mylibs`
**Libreria:** `CommonUtils` in cartella `src/CommonUtils`

#### Step: Crea workflow NuGet

Copia `NugetPackage_CI.yaml` e rinomina in `.github/workflows/CommonUtils_CI.yaml`

Sostituisci:
```yaml
# PRIMA
on:
  push:
    paths:
      - '[PATH_LIBRARYNAME]/**'
      - '.github/workflows/[LIBRARYNAME]_CI.yaml'

# DOPO
on:
  push:
    paths:
      - 'src/CommonUtils/**'
      - '.github/workflows/CommonUtils_CI.yaml'
```

```yaml
# PRIMA
run: dotnet build [PATH_LIBRARYNAME]/[LIBRARYNAME].csproj ...

# DOPO
run: dotnet build src/CommonUtils/CommonUtils.csproj ...
```

---

## Note di design

- **Monorepo-friendly:** Path filtering nel `on.push.paths` permette multiple servizi nello stesso repo senza trigger incrociati
- **Immutabilità:** Tag `run_id` garantisce tracciabilità di quale build ha generato quale immagine
- **Versionamento SemVer:** GitVersion calcola automaticamente versioni da commit history, non richiede tag manuali
- **Strategia Helm:** Uninstall+install è semplice ma non idempotente come `helm upgrade --install`; modificare se necessario
- **Artefatti intermedi:** NugetPackage_CI usa GitHub Actions artifacts per passare il `.nupkg` tra job
- **Runner self-hosted:** Kubernetes_Service_CI richiede runner nel cluster; NugetPackage_CI usa cloud
- **Pull policy Always:** Coerente con tag `latest`; modificare in `IfNotPresent` se si usano tag immutabili

---

## Troubleshooting

**Immagine Docker non aggiornata dopo deploy:**
- Verificare che `imagePullPolicy: Always` sia in `values.yaml`
- Verificare che la `rollme` annotation sia presente in `deployment.yaml`

**Push a ghcr.io fallisce:**
- Verificare che il secret `TOKEN` sia configurato in GitHub
- Verificare che il token abbia scope `packages:write`
- Verificare che `ghcr-secret` Kubernetes esista nel cluster

**GitVersion non calcola versione corretta:**
- Verificare `fetch-depth: 0` in checkout (necessario per leggere tutta la storia)
- Verificare che il repository abbia tag Git o commit per stabilire la baseline di versioning

---

## Contatti e contributi

Per aggiornamenti ai template o nuovi template, aggiornare questo repository e comunicare al team.
