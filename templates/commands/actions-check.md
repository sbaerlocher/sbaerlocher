---
description: Prüfe GitHub Actions auf Best Practices
---

Analysiere alle GitHub Actions Workflows in `.github/workflows/` auf Best Practices.

**Prüfe folgende Aspekte:**

## 1. Action Pinning (Sicherheit) - KRITISCH

- [ ] Alle Actions mit SHA gepinnt (nicht nur `@v4`)
- [ ] Korrekte Version im Kommentar (`# v4.2.2`)
- [ ] SHA stimmt mit Kommentar überein

```yaml
# ✅ Korrekt
uses: actions/checkout@b4ffde65f46336ab88eb53be808477a3936bae11 # v4.1.1

# ❌ Falsch - Supply Chain Risiko
uses: actions/checkout@v4
```

**Warum SHA-Pinning?**

- Tags können nachträglich verschoben werden (Supply-Chain-Attack)
- SHA ist unveränderlich
- Renovate aktualisiert automatisch mit `pinDigests: true`

## 2. Permissions (Least Privilege)

- [ ] `permissions:` explizit gesetzt (Job- oder Workflow-Level)
- [ ] Nur benötigte Permissions (`contents: read`)
- [ ] Keine `write-all` oder fehlende Permissions

```yaml
# ✅ Korrekt
permissions:
  contents: read
  pull-requests: write

# ❌ Falsch - zu permissiv
permissions: write-all
```

## 3. Step Namen

- [ ] Jeder Step hat einen `name:`
- [ ] Namen sind beschreibend
- [ ] Workflow/Job hat aussagekräftigen `name:`

## 4. Timeouts

- [ ] `timeout-minutes:` bei allen Jobs gesetzt
- [ ] Sinnvolle Werte für die jeweiligen Tasks

```yaml
jobs:
  build:
    timeout-minutes: 15  # Verhindert hängende Jobs
```

## 5. Caching

- [ ] Cache vorhanden für Dependencies (node_modules, .terraform, etc.)
- [ ] `restore-keys:` mit Fallback-Kette

## 6. Secrets Management

- [ ] Keine Secrets in Logs (`echo` ohne Wert)
- [ ] Secrets nur über `${{ secrets.NAME }}`
- [ ] Keine hardcodierten Credentials

## 7. Concurrency

- [ ] `concurrency:` bei Deploy-Workflows (verhindert parallele Deployments)
- [ ] `cancel-in-progress: false` für Deployments
- [ ] `cancel-in-progress: true` für CI (spart Ressourcen)

```yaml
concurrency:
  group: ${{ github.workflow }}-${{ github.ref }}
  cancel-in-progress: true
```

## 8. Renovate Integration

- [ ] Tool-Versionen mit Renovate-Kommentar

```yaml
env:
  # renovate: datasource=github-releases depName=helm/helm
  HELM_VERSION: "3.14.0"
```

## 9. Workflow Namen

- [ ] `name:` Feld vorhanden und aussagekräftig
- [ ] Volle Namen, keine Abkürzungen (`Continuous Integration` nicht `CI`)

## 10. Dateiname-Konvention

- [ ] Lowercase
- [ ] Hyphens statt Underscores (`ci.yml` nicht `CI.yml`)
- [ ] Beschreibend (`deploy.yml`, `security.yml`)

---

**Ausgabe:**

Für jeden Punkt:

- ✅ Erfüllt
- ⚠️ Teilweise erfüllt (mit Erklärung)
- ❌ Fehlt (mit Empfehlung)

**Am Ende:**

```text
GitHub Actions Best Practices Score: X/10 Kategorien erfüllt

Kritische Probleme:
1. ...

Empfohlene Verbesserungen:
1. ...

Soll ich die Änderungen anwenden? (Ja/Nein)
```
