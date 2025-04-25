# 💡 CI/CD užduotis: Podinfo ir Jenkins diegimas naudojant GitOps (FluxCD)

## 🔧 Pradinė instrukcija: Flux bootstrap (GitHub)

Prieš pradedant vykdyti užduotį, turi būti paruošta FluxCD aplinka GitHub pagrindu.

### 1. Reikalavimai
- Turite turėti **admin** teises Kubernetes klasteryje.
- Turite turėti prieigą prie **GitHub** paskyros ir/arba organizacijos, kurioje kursite repozitoriją.
- Turite sugeneruoti **GitHub personal access token (PAT)** su teisėmis prie `repo`.

### 2. Nustatykite GitHub token:
```bash
export GITHUB_TOKEN=<jūsų-tokenas>
```
Arba naudokite per `pipe`:
```bash
echo "<jūsų-tokenas>" | flux bootstrap github ...
```

### 3. Paleiskite Flux bootstrap:
```bash
flux bootstrap github \
  --token-auth \
  --owner=<jūsų-github-vartotojas> \
  --repository=<repo-pavadinimas> \
  --branch=main \
  --path=clusters/staging \
  --personal
```

> Jei nurodytas repo neegzistuoja, jis bus sukurtas automatiškai kaip privatus. Naudokite `--private=false`, jei norite viešo repo.

Po sėkmingo `bootstrap`, visi pakeitimai bus daromi Git'e ir Flux juos taikys automatiškai klasteryje.

---

# 📘 Užduotis: CI/CD su Kubernetes ir FluxCD

## 🎯 Tikslas

Susipažinti su CI/CD principais Kubernetes aplinkoje naudojant GitOps metodologiją ir FluxCD valdiklius. Praktikoje įdiegsite dvi aplikacijas:

- **Podinfo** – paprasta testinė aplikacija
- **Jenkins** – CI serveris, kurį taip pat valdysite GitOps principu

## 📁 Struktūra

Git repozitorijoje naudokite šią struktūrą:

```
clusters/
└── staging/
    ├── apps/
    │   └── podinfo.yaml
    ├── common/
    │   └── nginx.yaml
    └── flux-system/
        ├── gotk-components.yaml
        ├── gotk-sync.yaml
        └── kustomization.yaml
```

## 🛠️ Diegimo instrukcijos

### 1. Podinfo
- Helm repo: `https://stefanprodan.github.io/podinfo`
- Versija: `>=6.0.0`
- Namespace: `podinfo-test`

Sukurkite failą `clusters/staging/apps/podinfo.yaml` su šiuo turiniu:

```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: podinfo-test
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: podinfo
  namespace: podinfo-test
spec:
  url: https://stefanprodan.github.io/podinfo
  interval: 10m0s
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: podinfo
  namespace: podinfo-test
spec:
  interval: 5m
  chart:
    spec:
      chart: podinfo
      version: '6.0.0'
      sourceRef:
        kind: HelmRepository
        name: podinfo
        namespace: podinfo-test
  values:
    replicaCount: 1
    ingress:
      enabled: true
      className: "nginx"
      hosts:
        - host: podinfo.devplay.art
          paths:
            - path: /
              pathType: Prefix
```

Minimalus `values` paaiškinimui:
```yaml
values:
  replicaCount: 2
  ingress:
    enabled: true
    className: "nginx"
    hosts:
      - host: podinfo.devplay.art
        paths:
          - path: /
            pathType: Prefix
```
### pridet nginx
```yaml
apiVersion: v1
kind: Namespace
metadata:
  name: nginx-ingress
---
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: nginx
  namespace: nginx-ingress
spec:
  url: https://helm.nginx.com/stable
  interval: 10m
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: nginx-ingress
  namespace: nginx-ingress
spec:
  interval: 5m
  chart:
    spec:
      chart: nginx-ingress
      version: 2.1.0
      sourceRef:
        kind: HelmRepository
        name: nginx
        namespace: nginx-ingress
  values:
    controller:
      name: controller
      kind: deployment
      nginxplus: false
      replicaCount: 1
      ingressClass:
        name: nginx
        create: true
      service:
        create: true
        type: LoadBalancer
        externalTrafficPolicy: Local
      nginxStatus:
        enable: true
        port: 8080
        allowCidrs: "127.0.0.1"
      readyStatus:
        enable: true
        port: 8081
    rbac:
      create: true
      clusterrole:
        create: true
    prometheus:
      create: true
```


### 2. Jenkins
- Helm repo: `https://charts.jenkins.io`
- Versija: `>=4.0.0`
- Namespace: `jenkins`

Konfigūraciją dėkite į `clusters/staging/apps/jenkins.yaml`. Pavyzdys:
```yaml
values:
  controller:
    ingress:
      enabled: true
    adminPassword: "admin"
    serviceType: LoadBalancer
```



---

Sėkmės įvaldant GitOps! 🚀

# 📦 Helm Chart for `yourapp`



---

## 🚀 Create Your Helm Chart

To scaffold a new chart for `yourapp`, run:

```bash
helm create yourapp
```

This generates a boilerplate chart under a `yourapp/` directory. You'll customize this structure in the following steps.

---

## 📄 Define Helm Chart Metadata (`Chart.yaml`)

```yaml
apiVersion: v2
name: yourapp
description: Helm chart for your application
type: application
version: 0.1.0
appVersion: "latest"
```

---

## ⚙️ Default Configuration (`values.yaml`)

```yaml
replicaCount: 1

image:
  repository: yourdockerhub/yourapp
  pullPolicy: IfNotPresent
  tag: "latest"

service:
  enabled: true
  type: ClusterIP
  port: 80

ingress:
  enabled: false
  className: "nginx"
  annotations: {}
  hosts:
    - host: yourapp.local
      paths:
        - path: /
          pathType: Prefix
  tls: []

resources: {}

nodeSelector: {}

tolerations: []

affinity: {}
```

---

## 🏗 Create `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "yourapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "yourapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "yourapp.name" . }}
    spec:
      containers:
        - name: {{ include "yourapp.name" . }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

---

## 🧩 Create `templates/service.yaml`

```yaml
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "yourapp.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ include "yourapp.name" . }}
{{- end }}
```

---

## 🌐 Create `templates/ingress.yaml`

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "yourapp.fullname" . }}
  annotations:
    {{- range $key, $value := .Values.ingress.annotations }}
    {{ $key }}: {{ $value | quote }}
    {{- end }}
spec:
  ingressClassName: {{ .Values.ingress.className }}
  rules:
    {{- range .Values.ingress.hosts }}
    - host: {{ .host }}
      http:
        paths:
          {{- range .paths }}
          - path: {{ .path }}
            pathType: {{ .pathType }}
            backend:
              service:
                name: {{ include "yourapp.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

---

## 📦 Project Structure

```plaintext
yourapp/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── _helpers.tpl
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
│   ├── NOTES.txt
│   └── tests/
│       └── test-connection.yaml
├── .github/
│   └── workflows/
│       └── gh-pages.yml
├── .gitlab-ci.yml
```

---

## 🖋 Define Helpers in `_helpers.tpl`

```gotpl
{{- define "yourapp.name" -}}
{{- default .Chart.Name .Values.nameOverride | trunc 63 | trimSuffix "-" }}
{{- end }}

{{- define "yourapp.fullname" -}}
{{- if .Values.fullnameOverride }}
{{- .Values.fullnameOverride | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- $name := default .Chart.Name .Values.nameOverride }}
{{- if contains $name .Release.Name }}
{{- .Release.Name | trunc 63 | trimSuffix "-" }}
{{- else }}
{{- printf "%s-%s" .Release.Name $name | trunc 63 | trimSuffix "-" }}
{{- end }}
{{- end }}
{{- end }}
```

(remaining content unchanged...)

---

## 🚀 GitHub Actions: Publish Helm Chart to GitHub Pages

Create the following file at `.github/workflows/gh-pages.yml`:

```yaml
name: Publish Helm Chart

on:
  push:
    branches:
      - main

jobs:
  release:
    runs-on: ubuntu-latest

    steps:
      - name: 📥 Checkout repository
        uses: actions/checkout@v3

      - name: 🛠️ Set up Helm
        uses: azure/setup-helm@v3

      - name: 🏷️ Set appVersion from latest Git tag or fallback
        run: |
          TAG=$(git describe --tags --abbrev=0 2>/dev/null || echo "0.1.0")
          sed -i "s/^appVersion: .*/appVersion: \"$TAG\"/" charts/yourapp/Chart.yaml

      - name: 📦 Package Helm chart and generate index
        run: |
          mkdir -p .deploy
          helm package charts/yourapp -d .deploy
          helm repo index .deploy --url https://your-username.github.io/helm-yourapp

      - name: 🚫 Disable Jekyll to serve YAML
        run: echo "" > .deploy/.nojekyll

      - name: 🚀 Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: .deploy
          publish_branch: gh-pages
          force_orphan: true
```

---

## 🦊 GitLab CI: Publish Helm Chart to GitLab Pages

Create the following file `.gitlab-ci.yml` in your repository:

```yaml
stages:
  - package
  - pages

variables:
  HELM_EXPERIMENTAL_OCI: 1

package_chart:
  stage: package
  image: ubuntu:latest
  before_script:
    - apt-get update && apt-get install -y curl git bash gnupg lsb-release
    - curl https://baltocdn.com/helm/signing.asc | gpg --dearmor -o /usr/share/keyrings/helm.gpg
    - echo "deb [signed-by=/usr/share/keyrings/helm.gpg] https://baltocdn.com/helm/stable/debian/ all main" > /etc/apt/sources.list.d/helm-stable-debian.list
    - apt-get update && apt-get install -y helm
  script:
    - >
      TAG=$(git describe --tags --abbrev=0 || echo "0.1.0"); 
      sed -i "s/^appVersion:.*/appVersion: \"$TAG\"/" Chart.yaml; 
      mkdir -p public; 
      helm package . -d public; 
      helm repo index public --url $CI_PAGES_URL
  artifacts:
    paths:
      - public

pages:
  stage: pages
  image: ubuntu:latest
  script:
    - echo "Publishing Helm repo to GitLab Pages"
  artifacts:
    paths:
      - public
  rules:
    - if: '$CI_COMMIT_BRANCH == "main"'
      when: always

```

Enable **GitLab Pages** under **Settings → Pages** and the Helm repo will be available at:

```
https://<your-namespace>.gitlab.io/yourapp
```

You can then install the chart like so:

```bash
helm repo add yourapp https://<your-namespace>.gitlab.io/yourapp
helm repo update
helm install my-yourapp yourapp/yourapp
```

