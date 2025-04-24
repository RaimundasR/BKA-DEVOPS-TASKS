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

# Helm diagrama pvz. (`raimundas0106/todoapp` ) 
- Jus naudokite savo docker image

Šiame repozitoriume pateikiama Helm diagrama, skirta diegti konteinerizuotą `raimundas0106/todoapp` aplikaciją. Ji apima konfigūruojamą diegimą, paslaugą (Service) ir įeinamąjį srautą (Ingress), su galimybe įjungti/išjungti funkcijas per reikšmių failą.

## 📁 Projekto struktūra

```
helm-todoapp/
├── charts/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
├── .github/
│   └── workflows/
│       └── gh-pages.yml
└── README.md
```

---

## 🚀 Naudojimo instrukcija

### 1. Klonuoti ir paruošti diagramą

```bash
git clone https://github.com/<jusu-vartotojas>/helm-todoapp.git
cd helm-todoapp
```

---

### 2. Apibrėžti Helm diagramos metaduomenis (`Chart.yaml`)

```yaml
apiVersion: v2
name: todoapp
description: Helm diagrama Raimundo TODO aplikacijai
type: application
version: 0.1.0
appVersion: "latest"
```

---

### 3. Numatytoji konfigūracija (`values.yaml`)

```yaml
replicaCount: 1

image:
  repository: raimundas0106/todoapp
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
    - host: todoapp.local
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

### 4. Sukurti `templates/deployment.yaml`

```yaml
apiVersion: apps/v1
kind: Deployment
metadata:
  name: {{ include "todoapp.fullname" . }}
spec:
  replicas: {{ .Values.replicaCount }}
  selector:
    matchLabels:
      app: {{ include "todoapp.name" . }}
  template:
    metadata:
      labels:
        app: {{ include "todoapp.name" . }}
    spec:
      containers:
        - name: {{ include "todoapp.name" . }}
          image: "{{ .Values.image.repository }}:{{ .Values.image.tag }}"
          ports:
            - containerPort: 80
```

---

### 5. Sukurti `templates/service.yaml`

```yaml
{{- if .Values.service.enabled }}
apiVersion: v1
kind: Service
metadata:
  name: {{ include "todoapp.fullname" . }}
spec:
  type: {{ .Values.service.type }}
  ports:
    - port: {{ .Values.service.port }}
      targetPort: 80
  selector:
    app: {{ include "todoapp.name" . }}
{{- end }}
```

---

### 6. Sukurti `templates/ingress.yaml`

```yaml
{{- if .Values.ingress.enabled }}
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: {{ include "todoapp.fullname" . }}
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
                name: {{ include "todoapp.fullname" $ }}
                port:
                  number: {{ $.Values.service.port }}
          {{- end }}
    {{- end }}
{{- end }}
```

---

### 7. Paskelbti Helm diagramą per GitHub Pages

#### a. Sukurti GitHub veikseną `.github/workflows/gh-pages.yml`

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
          sed -i "s/^appVersion: .*/appVersion: \"$TAG\"/" charts/todoapp/Chart.yaml

      - name: 📦 Package Helm chart and generate index
        run: |
          mkdir -p .deploy
          helm package charts/todoapp -d .deploy
          helm repo index .deploy --url https://your_username.github.io/helm-todoapp

      - name: 🚫 Disable Jekyll to serve YAML
        run: echo "" > .deploy/.nojekyll

      - name: 🚀 Deploy to GitHub Pages
        uses: peaceiris/actions-gh-pages@v3
        with:
          github_token: ${{ secrets.GITHUB_TOKEN }}
          publish_dir: .deploy
          publish_branch: gh-pages
          force_orphan: true

          publish_dir: .deploy
```

#### b. Įkelti ir paskelbti

```bash
git init
git remote add origin https://github.com/<jusu-vartotojas>/helm-todoapp.git
git add .
git commit -m "Pradinis Helm diagramos commitas"
git push -u origin main
```

Tuomet įjunkite **GitHub Pages** savo repozitorijos nustatymuose, pasirinkdami `gh-pages` šaką arba `/root` katalogą.

---

### 8. Įdiegti diagramą

```bash
helm repo add raimundas https://<jusu-vartotojas>.github.io/helm-todoapp
helm install todoapp raimundas/todoapp
```

---

## 🔧 Diegimo pritaikymas

Įjungti Ingress:

```bash
helm install todoapp raimundas/todoapp --set ingress.enabled=true --set ingress.hosts[0].host=my.todoapp.com
```

Išjungti Service (jei naudojate tik Ingress):

```bash
helm install todoapp raimundas/todoapp --set service.enabled=false
```

---


# Antras užuoties variantas su GitLab (netestuota, galite užtrukti ilgiau)

# Helm diagrama `raimundas0106/todoapp`

Šiame repozitoriume pateikiama Helm diagrama, skirta diegti konteinerizuotą `raimundas0106/todoapp` aplikaciją. Ji apima konfigūruojamą diegimą, paslaugą (Service) ir įeinamąjį srautą (Ingress), su galimybe įjungti/išjungti funkcijas per reikšmių failą.

## 📁 Projekto struktūra

```
helm-todoapp/
├── charts/
├── Chart.yaml
├── values.yaml
├── templates/
│   ├── deployment.yaml
│   ├── service.yaml
│   ├── ingress.yaml
├── .gitlab-ci.yml
└── README.md
```

---

## 🚀 Naudojimo instrukcija

### 1. Klonuoti ir paruošti diagramą

```bash
git clone https://gitlab.com/<jusu-vartotojas>/helm-todoapp.git
cd helm-todoapp
```

---

### 2. Apibrėžti Helm diagramos metaduomenis (`Chart.yaml`)

```yaml
apiVersion: v2
name: todoapp
description: Helm diagrama Raimundo TODO aplikacijai
type: application
version: 0.1.0
appVersion: "latest"
```

---

### 3. Numatytoji konfigūracija (`values.yaml`)

```yaml
replicaCount: 1

image:
  repository: raimundas0106/todoapp
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
    - host: todoapp.local
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

### 4. Sukurti `.gitlab-ci.yml` su automatiniu `appVersion` ir leidimu

```yaml
stages:
  - package
  - pages

variables:
  HELM_EXPERIMENTAL_OCI: 1

before_script:
  - apk add --no-cache curl git bash helm

package_chart:
  stage: package
  script:
    - export TAG=$(git describe --tags --abbrev=0)
    - sed -i "s/appVersion: .*/appVersion: \"$TAG\"/" Chart.yaml
    - mkdir -p public
    - helm package . -d public
    - helm repo index public --url "https://<jusu-vartotojas>.gitlab.io/helm-todoapp"
  artifacts:
    paths:
      - public

pages:
  stage: pages
  script:
    - echo "GitLab Pages leidžia pasiekti Helm repozitoriją."
  artifacts:
    paths:
      - public
  only:
    - main
```

---

### 5. Įjungti GitLab Pages

1. Eikite į **Settings → Pages** ir įsitikinkite, kad puslapiai įjungti.
2. Kai CI baigs veikti, Helm repozitorija bus pasiekiama per:

```
https://<jusu-vartotojas>.gitlab.io/helm-todoapp
```

---

### 6. Įdiegti diagramą

```bash
helm repo add raimundas https://<jusu-vartotojas>.gitlab.io/helm-todoapp
helm repo update
helm install todoapp raimundas/todoapp
```

---

## 🔧 Diegimo pritaikymas

Įjungti Ingress:

```bash
helm install todoapp raimundas/todoapp --set ingress.enabled=true --set ingress.hosts[0].host=my.todoapp.com
```

Išjungti Service (jei naudojate tik Ingress):

```bash
helm install todoapp raimundas/todoapp --set service.enabled=false
```

---
