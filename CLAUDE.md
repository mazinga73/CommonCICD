# CLAUDE.md — Template per nuovi progetti

Questo file è un template da copiare in ogni nuovo progetto che desideri utilizzare i template CI/CD da CommonCICD.

---

## Template da usare per [PROGETTO_NAME]

### Repository template
**CommonCICD Repository:** `https://github.com/[ORG]/CommonCICD`

Consulta `INSTRUCTIONS.md` in quel repository per la documentazione completa dei template.

---

## Configurazione per questo progetto

### Info progetto
- **Nome progetto:** `[PROGETTO_NAME]`
- **Repository GitHub:** `[GITHUB_REPOSITORY]` (formato: `owner/repo`)
- **Owner GitHub:** `[GITHUB_OWNER]`

### Servizi da creare pipeline per
Quando Claude crea pipeline CI/CD per questo progetto, utilizza i template CommonCICD per:

| Servizio | Tipo | Path | Note |
|---|---|---|---|
| `[SERVICE_NAME_1]` | Kubernetes | `src/[SERVICE_NAME_1]` | Microservizio containerizzato |
| `[SERVICE_NAME_2]` | Kubernetes | `src/[SERVICE_NAME_2]` | Microservizio containerizzato |
| `[LIBRARY_NAME]` | NuGet | `src/[LIBRARY_NAME]` | Libreria .NET condivisa |

> Adatta la tabella ai servizi/librerie effettivi del tuo progetto

---

## Istruzioni per Claude

Quando ti viene chiesto di creare una pipeline CI/CD per questo progetto:

### 1. Per pipeline Kubernetes (microservizi)

**Riferimento:** CommonCICD `github-actions/Kubernetes_Service_CI.yaml`

1. Copia il template in `.github/workflows/[SERVICE_NAME]_CI.yaml`
2. Sostituisci questi placeholder:
   - `[SERVICE_NAME]` → nome servizio (es. `OrderService`)
   - `[PATH_SERVICENAME]` e `[PATH_SERVICE_NAME]` → path cartella (es. `src/OrderService`)
   - `[GITHUB_REPOSITORY]` → `[GITHUB_REPOSITORY]`
   - `[GITHUB_OWNER]` → `[GITHUB_OWNER]`
   - `[HELM_CHART_NAME]` → nome kebab-case (es. `order-service`)
   - `[HELM_CHART_PATH]` → `helm/charts`

3. Copia il chart Helm template in `helm/charts/[HELM_CHART_NAME]/`:
   - `Chart.yaml`
   - `values.yaml`
   - `templates/deployment.yaml`

4. Adatta `Chart.yaml` e `values.yaml` per il nuovo servizio

5. **Prerequisiti da verificare:**
   - Secret GitHub Actions `TOKEN` (PAT con scope `packages:write`)
   - Runner self-hosted registrato nel cluster Kubernetes
   - Secret Kubernetes `ghcr-secret` nel cluster (per pull immagini)

### 2. Per pipeline NuGet (librerie)

**Riferimento:** CommonCICD `github-actions/NugetPackage_CI.yaml`

1. Copia il template in `.github/workflows/[LIBRARY_NAME]_CI.yaml`
2. Sostituisci:
   - `[LIBRARYNAME]` → nome libreria (es. `CommonUtils`)
   - `[PATH_LIBRARYNAME]` → path cartella (es. `src/CommonUtils`)

3. **Prerequisiti da verificare:**
   - Secret GitHub Actions `TOKEN` (PAT con scope `packages:write`)
   - .NET SDK 10.0.x (adattare la versione se necessario)

---

## Consulta la documentazione

Per dettagli completi su:
- Tutti i placeholder
- Prerequisiti infrastrutturali
- Esempi di applicazione
- Note di design
- Troubleshooting

Leggi **CommonCICD/INSTRUCTIONS.md** all'indirizzo:
`https://github.com/[ORG]/CommonCICD/blob/main/INSTRUCTIONS.md`

---

## Customizzazione locale

Se necessiti di modifiche ai template per questo progetto specifico:

1. **Variazioni locali:** Modifica il workflow/chart nel repository locale senza toccare CommonCICD
2. **Contributi al template:** Se la modifica è utile e generalizzabile, proposte come PR a CommonCICD

---

## Secrets da configurare

Prima di usare le pipeline, configura questi secrets in GitHub:

### Per tutti i workflow
- `TOKEN` — Personal Access Token con scope:
  - `packages:write` (push immagini ghcr.io)
  - `packages:write` (publish pacchetti NuGet)

### Per Kubernetes deployment
- Nel cluster, creare il secret:
  ```bash
  kubectl create secret docker-registry ghcr-secret \
    --docker-server=ghcr.io \
    --docker-username=[GITHUB_OWNER] \
    --docker-password=[TOKEN] \
    --docker-email=[EMAIL]
  ```

---

## Flusso di lavoro consigliato

1. **Nuova pipeline:** Chiedi a Claude di generarla usando i template CommonCICD
2. **Modifica template:** Usa INSTRUCTIONS.md come riferimento per placeholder e configurazioni
3. **Deploy:** Le pipeline triggheranno automaticamente su push ai path configurati

---

## Template di commit message per pipeline

Quando Claude crea/modifica pipeline, usa message type `ci`:

```
ci: [Kubernetes|NuGet] pipeline for [SERVICE_NAME]

- Aggiunto workflow GitHub Actions [SERVICE_NAME]_CI.yaml
- Aggiunto Helm chart [HELM_CHART_NAME]
- Basato su template CommonCICD
```

---

## Link utili

- **CommonCICD Repository:** https://github.com/[ORG]/CommonCICD
- **INSTRUCTIONS.md:** https://github.com/[ORG]/CommonCICD/blob/main/INSTRUCTIONS.md
- **GitHub Actions Docs:** https://docs.github.com/en/actions
- **Helm Docs:** https://helm.sh/docs/
- **GitVersion:** https://gitversion.net/

---

## Manutenzione

Questo file CLAUDE.md deve essere aggiornato quando:
- Cambia la struttura del progetto
- Vengono aggiunti/rimossi servizi
- Cambia il repository GitHub
- Cambia il processo di deploy

Vedi il template di questo file su CommonCICD per nuovi progetti.
