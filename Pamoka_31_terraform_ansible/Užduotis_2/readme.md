# 💪 Užduotis 2 – CRD ir Helm diegimas naudojant FluxCD

## 🎯 Tikslas

> Naudojant FluxCD operatorių, sukurti naują CRD deploy'inimo rolę ir ją integruoti į esamą Ansible projektą. ĮDiegti `podinfo` aplikaciją naudojant `HelmRelease`, su visais reikalingais resursais (CRD, HelmRepository, HelmRelease) ir naudoti `service-podinfo-crds` rolę vietoje ankstesnės `service-podinfo`.

---

## 💠 Reikalavimai

1. **MicroK8s ir FluxCD** jau turi būti įdiegti.
2. Turite turėti `Ansible` projektą su pagrindiniu `playbook.yml` failu.
3. Naudosite FluxCD Helm kontrolerius (Source ir Helm).

---

## 📁 Struktūra

```bash
.
├── playbook.yml
├── roles/
│   └── service-podinfo-crds/
│       ├── tasks/
│       │   └── main.yml
│       └── templates/
│           ├── podinfo-crd.yml.j2
```

---

## 🔧 1. Pakomentuokite seną `service-podinfo` rolę

`playbook.yml`:

```yaml
- hosts: localhost
  tasks:
    # - name: Deploy podinfo using service-podinfo
    #   include_role:
    #     name: service-podinfo

    - name: Deploy podinfo using FluxCD Helm operator
      include_role:
        name: service-podinfo-crds
```

---

## 🧱 2. Naujos rolės kūrimas: `service-podinfo-crds`

### `roles/service-podinfo-crds/tasks/main.yml`

pvz, pradekite nuo

```yaml
- name: Create podinfo namespace using runner name
  become: true
  kubernetes.core.k8s:
    kubeconfig: /home/ubuntu/.kube/config
    state: present
    definition:
      apiVersion: v1
      kind: Namespace
      metadata:
        name: "{{ instance_name }}-podinfo"

```

---

## 📄 3. Šablonai

### `templates/helmrepo.yml.j2`

```yaml
apiVersion: source.toolkit.fluxcd.io/v1
kind: HelmRepository
metadata:
  name: podinfo
  namespace: podinfo
spec:
  url: https://stefanprodan.github.io/podinfo
  interval: 10m
```

---

### `templates/helmrelease.yml.j2`

```yaml
apiVersion: helm.toolkit.fluxcd.io/v2
kind: HelmRelease
metadata:
  name: podinfo
  namespace: podinfo
spec:
  releaseName: podinfo
  chart:
    spec:
      chart: podinfo
      version: "6.3.3"
      sourceRef:
        kind: HelmRepository
        name: podinfo
        namespace: podinfo
  interval: 5m
  install:
    createNamespace: true
  values:
    ingress:
      enabled: true
      className: nginx
      hosts:
        - host: "{{ domain }}"
          paths:
            - path: /
              pathType: Prefix
```

---

## 🚀 Paleidimas

Prieš paleisdami, įsitikinkite, kad `domain` kintamasis perduodamas:

```bash
ansible-playbook playbook.yml -e domain=test.domain.com
```
## Chart versijos keitimas naudojant fluxcd operatoriu

Pakeiskite podinfo versija   version: "6.0.0" ir atnaujinkite diegima.

---

## ✅ Rezultatas

FluxCD automatiškai diegs `podinfo` aplikaciją naudodamas `HelmRelease`, remiantis jūsų aprašytu `HelmRepository`. Nereikia rankiniu būdu taikyti `kubectl` ar `helm` komandų.

Pakeiskite podinfo versija   version: "6.0.0" ir atnaujinkite diegima.


