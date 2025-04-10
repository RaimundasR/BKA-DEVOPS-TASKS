# Studentų užduotis: MicroK8s, FluxCD ir Podinfo diegimas per Terraform + Ansible + Helm

## Tikslas

Naudojant scenarijų `runner.sh`, automatizuoti pilnos infrastruktūros sukūrimą:

- Sukurti EC2 instanciją AWS debesijoje su Terraform.
- Naudojant Ansible, įdiegti MicroK8s, FluxCD ir podinfo.
- Jei nurodytas domenas, sukurti DNS A įrašą Cloudflare'e.

---

## Projekto struktūra

```
aws-provisioner/
├── runner.sh                      <- pagrindinis paleidimo scenarijus
├── config/
│   └── infra.config.json          <- konfigūracijos failas
├── terraform/
│   ├── aws/                       <- EC2 provisioningas
│   └── cloudflare/                <- Cloudflare DNS A įrašo valdymas
├── ansible/
│   ├── playbook/
│   │   └── playbook.yml          <- pagrindinis playbook
│   └── roles/
│       ├── service-microk8s/     <- MicroK8s diegimas
│       ├── service-fluxcd/       <- FluxCD diegimas
│       └── service-podinfo/     <- Podinfo diegimas
```

---

## Veikimo principas

### EC2 diegimas:

- EC2 instancija kuriama naudojant Terraform.
- Instancijai suteikiamas viešas IP.
- SSH prisijungimo informacija įrašoma į `inventory.ini` failą.

### Ansible veiksmai:

1. Diegiamas MicroK8s ir reikalingi plugin'ai (dns, ingress).
2. Sukuriamas `~/.kube/config` failas ir nurodomas `KUBECONFIG` kintamasis.
3. Diegiamas Helm.
4. Diegiamas FluxCD (įskaitant helm ir source kontrolerius).
5. Diegiamas Podinfo, naudojant nurodytą domeną ir `nginx` ingress klasę.

### DNS

- Jei naudotojas nurodo domeną per `--domain`, script'as:
  - naudoja Terraform modulį `cloudflare` su API tokenu ir Zone ID
  - sukuria atitinkamą DNS A įrašą į Cloudflare

---

## Paleidimas

### 1. Sukūrimas

```bash
./runner.sh -a create --name mano-instas --domain mano.domenas.lt
```

- Sukurs EC2, įdiegs MicroK8s, FluxCD, Podinfo
- Sukurs DNS įrašą Cloudflare'e

### 2. Naikinimas

```bash
./runner.sh -a delete --name mano-instas --domain mano.domenas.lt
```

- Pašalins EC2
- Pašalins DNS A įrašą Cloudflare'e

> **Pastaba:** `--name` parametras yra privalomas.

---

## Konfigūracijos pavyzdys `config/infra.config.json`

```json
{
  "region": "eu-north-1",
  "instance_type": "t3.micro",
  "ami": "ami-1234567890abcdef0",
  "key_name": "ec2-key",
  "private_key_path": "~/.ssh/ec2-key.pem",
  "ssh_user": "ubuntu",
  "cloudflare_api_token": "api-token-goes-here",
  "cloudflare_zone_id": "zone-id-goes-here"
}
```

---

## Reikalavimai

- Terraform 1.0+
- Ansible
- Helm
- AWS prieiga su `ec2_micro_provisioner` profiliu
- Cloudflare API tokenas su DNS rašymo teisėmis

---

## Užduoties eiga ir rezultatai

1. Peržiūrėkite projekto struktūra.
2. Supildykite `infra.config.json`.
3. Paleiskite `runner.sh` su parinktimis:
   - Sukūrimui: `-a create --name testas --domain testas.tavo-domenas.lt`
   - Naikinimui: `-a delete --name testas --domain testas.tavo-domenas.lt`
4. Patikrinkite, ar:
   - EC2 instancija sukurta
   - veikia MicroK8s, FluxCD, podinfo
   - DNS A įrašas sukurtas Cloudflare
5. Papildomai: prisijunkite prie EC2 ir patikrinkite `kubectl get pods --all-namespaces`

---


