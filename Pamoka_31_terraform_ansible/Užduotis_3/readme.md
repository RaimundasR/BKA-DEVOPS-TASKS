# 🛠️ Užduotis 3 – CRD Jenkins diegimas naudojant FluxCD (be GitOps)

Šiame vadove parodoma, kaip įdiegti Jenkins naudojant FluxCD CLI be GitOps (t. y. be CRD saugojimo Git'e). Vietoje to, diegimo manifestai (`HelmRepository` ir `HelmRelease`) sugeneruojami bei pritaikomi rankiniu būdu.

---

## 🎯 Reikalavimai

- Veikiantis Kubernetes klasteris
- Įdiegtas FluxCD
- Įdiegtas `flux` CLI
- Įdiegtas `helm`

---

## 1 žingsnis – Pridėti Jenkins Helm repozitoriją

```bash
helm repo add jenkins https://charts.jenkins.io
```

---

## 2 žingsnis – Atsisiųsti ir paredaguoti `values.yaml`

```bash
helm show values jenkins/jenkins > jenkins-values.yaml
```

> Redaguokite šį failą ir pašalinkite nereikalingus parametrus (pvz. persistence, resource limits), taip pat nustatykite administratoriaus slaptažodį ir pan.

---

## 3 žingsnis – Sukurti `HelmRepository` resursą (FluxCD Source Controller)

```bash
flux create source helm jenkins   --url=https://charts.jenkins.io   --interval=10m   --export > jenkins-helmrepo.yaml
```

---

## 4 žingsnis – Sukurti `HelmRelease` resursą (FluxCD Helm Controller)

```bash
flux create helmrelease jenkins   --interval=10m   --release-name=jenkins   --source=HelmRepository/jenkins   --chart=jenkins   --namespace=jenkins   --values=./jenkins-values.yaml   --export > jenkins-helmrelease.yaml
```

---

## 5 žingsnis – Sukurti `jenkins-crd.yaml.j2` failą naudoti su Ansible

Šiame žingsnyje sujungiame abu sugeneruotus CRD (`HelmRepository` ir `HelmRelease`) į vieną Jinja2 šabloną pavadinimu `jenkins-crd.yaml.j2`.

### Komandos:

```bash
cat jenkins-helmrepo.yaml > jenkins-crd.yaml.j2
echo "---" >> jenkins-crd.yaml.j2
cat jenkins-helmrelease.yaml >> jenkins-crd.yaml.j2
```

### Redagavimas:

Atidarykite `jenkins-crd.yaml.j2` ir pataisykite šias reikšmes:

- `namespace`
- `releaseName`
- `chart` versiją (jei reikia)
- `values.yaml` kelią (jei šablonuojama)
- **Svarbiausia – įrašykite tinkamą Ingress host vardą:**

```yaml
controller:
  ingress:
    enabled: true
    hostName: "{{ domain }}"
```

> Pakeiskite `{{ domain }}` į konkretų vardą arba naudokite Jinja2 kintamąjį, jei failas bus šablonuojamas per Ansible., musu atveju mes pasiduodame is runner.sh `domain`

### Failo vieta:

Įkelkite `jenkins-crd.yaml.j2` į Ansible `templates/` katalogą.

---

## 6 žingsnis – Pritaikyti `jenkins-crd.yaml.j2` klasteryje

```bash
kubectl apply -f jenkins-crd.yaml.j2
```

---

## 7 žingsnis – Naudoti su Ansible , tokiu pačiu principu kaip ir podinfo

Naudokite `jenkins-crd.yaml.j2` kaip šabloną `templates/` kataloge, kai automatizuojate diegimą per Ansible.

---

## 8 žingsnis – Patikrinti diegimą (neprivaloma kai naudojame ansible)

```bash
kubectl get helmrelease -n jenkins
kubectl get pods -n jenkins
```

---

Sėkmingai! 🎉 Jenkins dabar turėtų veikti jūsų Kubernetes klasteryje ir būti valdomas FluxCD.
