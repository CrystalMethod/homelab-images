# Homelab-Images

Zentrales Repository für selbst gebaute Container-Images des Homelabs.

Dieses Repository ist strikt von dem GitOps-Repository
[`CrystalMethod/homelab`](https://github.com/CrystalMethod/homelab) getrennt:
Hier entstehen und werden Images veröffentlicht, dort wird nur noch auf die
fertigen, gepinnten Images referenziert. Dadurch trennt sich die
Image-Herstellung (dieses Repo, eigene Build-Pipeline) von der Bereitstellung
(GitOps-Repo, `image:`-Referenz).

## Derzeitige Images

### `jupyterhub`

JupyterHub-Laufzeitimage im Verzeichnis [`jupyterhub/`](jupyterhub/). Es enthält
die Notebook-Basis sowie JupyterHub, OAuthenticator, DockerSpawner und
JupyterLab-Git.

**Konfiguration ist bewusst nicht im Image:** Die `jupyterhub_config.py` wird
vom GitOps-Repository verwaltet und zur Laufzeit in den Container eingebunden.
Dadurch erfordert eine Konfigurationsänderung keinen Image-Neubau.

### `stirling-pdf`

Stirling-PDF-Image im Verzeichnis [`stirling-pdf/`](stirling-pdf/). Es basiert
auf dem gepinnten Upstream-Image `stirlingtools/stirling-pdf:<version>-fat` und
entfernt nur den mitgelieferten Brotli-Decoder `org.brotli:dec`.

**Grund:** Apache HttpClient 5 bietet `Accept-Encoding: br` an, sobald der
Decoder im Klassenpfad liegt. Der Decoder verliert bei Antworten bis 16 KiB
Daten, wenn Spring vorab ein Byte liest
([google/brotli#1612](https://github.com/google/brotli/issues/1612)). Cloudflare
liefert den Authentik-Token-Endpunkt mit Brotli aus; Stirling-PDF scheitert
dann beim OIDC-Login mit `invalid_token_response` und
`Unexpected end-of-input`. Ohne Decoder verhandelt HttpClient gzip.

Konfiguration, Volumes und Entrypoint bleiben unverändert. Das Image wird
entfernt, sobald Stirling-PDF oder die Brotli-Bibliothek den Fehler behebt.

## Build und Release

CI (`.github/workflows/build-jupyterhub.yaml`) baut Multi-Arch-Images
(`linux/amd64`, `linux/arm64`) und pusht sie nach
`ghcr.io/crystalmethod/jupyterhub` mit den Tags:

- `latest` – bei Änderungen am Standard-Branch `main`
- `<sha>` – Git-SHA des Commits
- semantische Version beim Tag `jupyterhub-<version>` (z.B. `jupyterhub-6.1.1`)

Einen Release erzeugen (baut und pusht zusätzlich versionierte Tags):

```bash
git tag "jupyterhub-6.1.1"
git push origin "jupyterhub-6.1.1"
```

Das Stirling-PDF-Image hat denselben Ablauf: Änderungen unter `stirling-pdf/`
bauen `ghcr.io/crystalmethod/stirling-pdf` mit den Tags `latest`, `<sha>` und
beim Git-Tag `stirling-pdf-<version>` (z.B. `stirling-pdf-3.4.0-fat`) die
versionierte Variante.

Das GitOps-Repository referenziert das fertige Image Digest-gepinnt, z.B.:

```yaml
image: ghcr.io/crystalmethod/jupyterhub:6.1.1@sha256:<digest>
```
