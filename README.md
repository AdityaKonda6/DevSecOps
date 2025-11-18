# DevSecOps

## Image scanning — Trivy (HTML report)

### Pull Trivy image (optional – you may already have it)

```bash
docker pull aquasec/trivy:latest
```

### Scan local image and produce HTML (PowerShell / Linux examples)

**Download HTML template (official):**

```bash
curl -LO https://raw.githubusercontent.com/aquasecurity/trivy/main/contrib/html.tpl
```

**Scan and create `trivy-report.html`:**

```bash
# Linux / WSL / CloudShell
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$(pwd):/report" \
  aquasec/trivy:latest \
  image --format template --template "@/report/html.tpl" -o /report/trivy-report.html helloworld-web:latest
```

**Fail the build on HIGH/CRITICAL:**

```bash
docker run --rm \
  -v /var/run/docker.sock:/var/run/docker.sock \
  -v "$(pwd):/report" \
  aquasec/trivy:latest \
  image --exit-code 1 --severity HIGH,CRITICAL --format template --template "@/report/html.tpl" -o /report/trivy-report.html helloworld-web:latest
```

Open `trivy-report.html` in your browser or download from CloudShell.

---

## SAST — Bandit (Python static analysis)

1. Install (locally or in CI):

```bash
python -m pip install --upgrade pip
pip install bandit
```

2. Run scan (produce HTML report):

```bash
# from project root (exclude venv and large files)
bandit -r . -f html -o bandit-report.html --exclude venv,__pycache__,node_modules
```

3. Review `bandit-report.html`.

> Add Bandit to your CI pipeline to fail on findings or produce artifacts.

---

## DAST — OWASP ZAP (dynamic scan)

1. Pull ZAP Docker image:

```bash
docker pull zaproxy/zap-stable
```

2. Run ZAP full scan against a running instance (example assumes target accessible at `http://localhost` on same Docker network):

```bash
# If your web container is in network 'my_network' and reachable as 'web:8000'
docker run --rm \
  --network container:nginx-gsquare \
  -v "$(pwd)":/zap/wrk:rw \
  zaproxy/zap-stable \
  zap-full-scan.py -t http://localhost -r /zap/wrk/zap_report.html
```

3. Result: `zap_report.html` in current directory.

> For Docker Desktop local development, run your app and run ZAP against `http://host.docker.internal:8000` if necessary.

---
