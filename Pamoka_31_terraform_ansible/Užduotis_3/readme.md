# 🛠️ Užduotis – Jenkins diegimas naudojant `runner.sh` (su FluxCD, be GitOps)

Šiame vadove parodoma, kaip automatiškai įdiegti Jenkins į MicroK8s klasterį EC2 serveryje naudojant `runner.sh` skriptą. Įdiegimas atliekamas naudojant FluxCD, tačiau **be GitOps** – Helm CRD (HelmRepository ir HelmRelease) yra šablonuojami per Ansible.

---

## 🎯 Reikalavimai

- Turite prieigą prie AWS (naudojamas EC2)
- Turite Cloudflare domeną DNS įrašų kūrimui
- Veikiantis Kubernetes klasteris
- Įdiegtas FluxCD
- Įdiegtas `flux` CLI
- Įdiegtas `helm`
- Įrašyti: `terraform`, `ansible`, `kubectl`
- Failas `config/infra.config.json` su AWS ir SSH informacija

> `jq` ir `awscli` nėra būtini, jei EC2 paleidžiate tik per `runner.sh`, bet `jq` naudojamas konfigūracijos reikšmėms nuskaityti iš JSON.

> Jenkins CRD (`HelmRepository` ir `HelmRelease`) šablonus galima generuoti naudojant `flux` CLI ir `helm show values`, o tada pritaikyti su Ansible. Analogiškai kaip ir su Podinfo, naudokite `--export` vėliavėlę, kad sugeneruoti failai būtų integruojami į jūsų infrastruktūrą:

```bash
helm show values jenkins/jenkins > jenkins-values.yaml

flux create source helm jenkins \
  --url=https://charts.jenkins.io \
  --interval=10m \
  --export > jenkins-helmrepo.yaml

flux create helmrelease jenkins \
  --interval=10m \
  --release-name=jenkins \
  --source=HelmRepository/jenkins \
  --chart=jenkins \
  --namespace=jenkins \
  --values=./jenkins-values.yaml \
  --export > jenkins-helmrelease.yaml
```

---

### 📁 Projekto struktūra:

```
.
├── runner.sh                       # Main CLI for managing infrastructure
├── terraform/
│   ├── aws/                        # EC2 provisioning logic
│   └── cloudflare/                # DNS record logic
├── config/
│   └── infra.config.json          # AWS key, region, user config
├── ansible/
│   ├── playbook/
│   │   ├── playbook.yml           # Base installer: MicroK8s + FluxCD
│   │   ├── install_tool.yml       # Tool installer (runs role service-<tool>)
│   │   └── uninstall_tool.yml     # Tool remover
│   └── roles/
│       ├── service-<tool>         # Tool-specific logic
│       ├── common-*               # Shared modules like microk8s, fluxcd
│       └── service-tool-installer # Template-based tool installation
└── setup-deps/
    └── install_deps.sh            # Optional: installs Ansible/Terraform locally
```

---

## 1 žingsnis – Paleisti Jenkins diegimą

```bash
./runner.sh -a create --name jenkins-instance --domain jenkins.devplay.art --tool jenkins
```

Ši komanda:
- Provisionuoja EC2 instanciją AWS'e
- Įdiegia MicroK8s ir FluxCD
- Įdiegia Jenkins naudojant Ansible (`install_tool.yml`)
- Sukuria Cloudflare DNS įrašą

---

## 2 žingsnis – Jenkins CRD šablonas

Failas `jenkins-crd.yaml.j2` turi būti patalpintas į:

```
ansible/roles/service-tool-installer/templates/jenkins-crd.yaml.j2
```

Jis turi būti Jinja2 šablonas su bent tokiais laukais:

```yaml
apiVersion: source.toolkit.fluxcd.io/v1beta1
kind: HelmRepository
metadata:
  name: jenkins
  namespace: jenkins
spec:
  interval: 10m
  url: https://charts.jenkins.io
---
apiVersion: helm.toolkit.fluxcd.io/v2beta1
kind: HelmRelease
metadata:
  name: jenkins
  namespace: jenkins
spec:
  interval: 10m
  releaseName: jenkins
  chart:
    spec:
.......
  values:
.......
        enabled: true
        hostName: "{{ domain }}"
```

---

## 3 žingsnis – Patikrinti, ar Jenkins įdiegtas

```bash
kubectl get pods -n jenkins
kubectl get helmrelease -n jenkins
```

---

## 4 žingsnis – Pašalinti tik Jenkins

```bash
./runner.sh --name jenkins-instance --domain jenkins.devplay.art --tool jenkins --destroy
```

Tai:
- Ištrins HelmRelease ir susijusius resursus
- Pašalins DNS įrašą
- EC2 paliks nepaliestą

---

## 5 žingsnis – Visiškai pašalinti infrastruktūrą

```bash
./runner.sh -a delete --name jenkins-instance --domain jenkins.devplay.art
```

Tai pašalins:
- Jenkins (jei nurodytas `--tool`)
- EC2 instanciją
- Cloudflare DNS įrašus

---

Sėkmingai! 🎉 Jenkins turėtų būti automatiškai įdiegtas jūsų klasteryje ir valdomas FluxCD be GitOps.

