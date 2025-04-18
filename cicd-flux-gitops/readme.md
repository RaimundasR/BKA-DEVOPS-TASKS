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

Rekomenduojama slaptažodį saugoti Kubernetes `Secret` objekte.

### 3. Bendras Kustomization

Failas: `clusters/staging/kustomization.yaml`
```yaml
resources:
  - apps/podinfo.yaml
  - apps/jenkins.yaml
  - common/nginx.yaml
```

## 📥 Pristatymas

Iki: _[nurodyta data]_

Pristatykite:
- Nuorodą į savo GitHub repozitoriją (pvz.: https://github.com/RaimundasR/k8s-flux)
- Trumpą aprašymą, kaip patikrinote Jenkins ir Podinfo veikimą

---

Sėkmės įvaldant GitOps! 🚀

