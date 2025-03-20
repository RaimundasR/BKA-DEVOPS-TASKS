# Praktinė užduotis 1 : Helm 

## Užduotis 1.1
- Decodinkite config faila
- Eikite į [Artifact Hub](https://artifacthub.io/)
- Paieškoje įveskite **nginx-controller**
- Sudiegikite helm cli ingress-controller **nginx-controller** 

```bash
helm install [RELEASE_NAME] ingress-nginx/ingress-nginx --namespace [NAMESPACE] --create-namespace

```

## Užduotis 1.2

### 1. Rasti aplikaciją `podinfo` Helm Chart kataloge
- Eikite į [Artifact Hub](https://artifacthub.io/)
- Paieškoje įveskite **podinfo**
- Suraskite repozitorijos savininką **stefanprodan**

### 2. Atsidaryti `podinfo` repozitoriją
- `podinfo` Helm Chart puslapyje dešiniajam kampe suraskite `Source` nuorodą
- Paspauskite ją – ji atsidarys `GitHub` repozitorijoje

### 3. Atsisiųsti `podinfo` repozitoriją į savo katalogą
- Grįžkite į terminalą ir eikite į savo katalogą:
  ```sh
  cd ONAV-K8S/day2/<vardaspavarde>
  ```
- Nuklonuokite `podinfo` repozitoriją:
  ```sh
  git clone https://github.com/stefanprodan/podinfo.git
  ```
- Pakeiskite resursus reikalingus įsidiegti ir pasiekti `podinfo` aplikaciją naudojant subdomeną **`stud<nr>.devplay.art`**
  `podinfo/podinfo/charts/podinfo/values.yaml`

  ```sh
  helm install my-podinfo podinfo/podinfo --version 6.7.1 --values=./values.yaml --namespace <pavadinimas>-podinfo --create-namespace
  ```

  istrinti 
  
  `helm uninstall my-podinfo -n stud9-podinfo`

  ###  Jeigu reikia nuorodti `config`failą :

  ```bash
  helm install <release-name> <chart-name> --kubeconfig <path-to-kubeconfig>
   ```

  ```bash
  helm install my-release my-chart \
  --kubeconfig ~/.kube/custom-config \
  --namespace my-namespace \
  ```

### 6. Patikrinkite Service, Pod ir Ingress
```sh
kubectl get all -n podinfo
```



--- 

# Praktinė užduotis 2 : flux controllers

# Podinfo diegimas su FluxCD Helm Controller

## Tikslas

Įdiegti **FluxCD Helm Controller** ir **Source Controller** savo Kubernetes klasteryje naudojant FluxCD. Programėlė turi būti pasiekiama šiuo URL formatu: `https://stud<Nr>.devplay.art`

Pakeiskite `<Nr>` į jums priskirtą studento numerį.

---

## Reikalavimai

1. **Kubernetes klasteris** - užtikrinkite prieigą prie klasterio.
2. **kubectl** - įdiegta ir sukonfigūruota klasteriui.
3. **FluxCD CLI** - [Įdiegimo instrukcija](https://fluxcd.io/flux/get-started/#install-the-flux-cli)

---

## 1 žingsnis: FluxCD Helm Controller ir Source Controller diegimas

Vadovaukitės oficialiomis diegimo instrukcijomis:
- [Helm Controller](https://fluxcd.io/flux/components/helm/helmreleases/#helm-controller)
- [Source Controller](https://fluxcd.io/flux/components/source/)

- Diegiame fluxcli  >> https://fluxcd.io/flux/installation/

Diegimas:
```bash
flux install \
  --components=source-controller,helm-controller
```

Patikrinkite ar viskas veikia:

```bash
kubectl get pods -n flux-system
```

---

## 2 žingsnis: Sukurkite HelmRepository ir HelmRelease

Sukurkite failą `podinfo-helmrelease.yaml` su šiuo turiniu savo sukurtame kataloge ir pakeiskite reikšmes pagal reikiamus reikalavimus:

>**Svarbu!** Pakeiskite `<Nr>` į savo studento numerį. Pvz. `stud1.devplay.art`

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: podinfo
  namespace: stud<Nr>-podinfo
spec:
  interval: 10m0s
  url: https://stefanprodan.github.io/podinfo
---
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: podinfo
  namespace: stud<Nr>-podinfo
spec:
  chart:
    spec:
      chart: podinfo
      sourceRef:
        kind: HelmRepository
        name: podinfo
      version: '6.0.0'
  interval: 10m0s
  values:
    replicaCount: 2
    ingress:
      enabled: true
      className: "nginx"
      hosts:
        - host: stud<Nr>.devplay.art
          paths:
            - path: /
              pathType: Prefix

```

---

## 3 žingsnis: Taikykite konfigūraciją

Paleiskite programėlę naudodami FluxCD:
```bash
kubectl apply -f podinfo-helmrelease.yaml
```

Patikrinkite diegimo būseną:
```bash
kubectl get helmrelease -A
```

---

## 4 žingsnis: Patikrinkite diegimą

Aplankykite programėlę naršyklėje:
```
https://stud<Nr>.devplay.art
```
Įsitikinkite, kad DNS nukreipia į jūsų Kubernetes klasterį.

---

## 5 žingsnis: Išvalykite diegimą

Norėdami pašalinti diegimą:
```bash
kubectl delete -f podinfo-helmrelease.yaml
```

---

**Sėkmės diegiant!**
