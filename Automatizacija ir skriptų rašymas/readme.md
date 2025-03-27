```markdown
# Ansible paskaitos README (skaidrių komentarai)

## 1 skaidrė: Deklaratyvūs skriptai. Pažintis su Ansible
Tai įvadas į temą. Papasakokite, kad deklaratyvus skriptinimas – tai modernus būdas valdyti infrastruktūrą ir paslaugas, aprašant ne tai, *kaip* viskas turi būti padaryta, bet *kas* turi būti pasiekta. Ansible yra vienas iš tokių įrankių, plačiai naudojamas debesų inžinerijoje, sistemų administravime, DevOps srityje. Jis leidžia lengvai, skaidriai ir saugiai konfigūruoti sistemas, nes yra agentless ir deklaratyvus.

## 2 skaidrė: Temos
Trumpai apibūdinkite:
- **Deklaratyvus ir imperatyvus skriptinimas** – skirtumas tarp aprašymo „ką padaryti“ ir „kaip padaryti“.
- **Infrastruktūra-kaip-kodas (IaC)** – infrastruktūros valdymas naudojant kodą.
 - **Idempotentumo principas** – kodas įvykdomas kelis kartus, bet rezultatas nesikeičia.
- **Konfigūravimo valdymo įrankiai** – pvz., Ansible, Puppet, Chef, jų paskirtis.
- **Ansible įvadas** – kodėl jis populiarus, kuo pasižymi.
- **Inventoriaus kūrimas** – kaip nurodyti, kokius serverius valdome.
- **Serverio paruošimas** – SSH, raktais paremtas prisijungimas.

## 3 skaidrė: Namo statymas vs. aplikacijos statymas
Naudokite metaforą: statant namą reikia plano, medžiagų, meistrų – taip pat ir su aplikacija: reikia architektūros, infrastruktūros, konfigūracijų. Panašumas – abu reikalauja aiškaus aprašymo ir nuoseklumo. Skirtumas – aplikacijos gali būti keičiamos greičiau, lengviau automatizuojamos.

## 4 skaidrė: Netgi namų statybos automatizuojamos
Papasakokite apie šiuolaikinius sprendimus, kaip 3D spausdinimas naudojamas namų statyboje – tai rodo, kad automatizavimas paliečia net fizines statybas.
Pavyzdys IT srityje – Active Directory naudotojo kūrimas naudojant PowerShell:
```powershell
New-ADUser -Name "Cloudenis Pavardenis" -GivenName "Cloudenis" -Surname "Pavardenis" -SamAccountName "tidris" -UserPrincipalName "cloudenis@rebeladmin.com" -Path "OU=Users,OU=Europe,DC=rebeladmin,DC=com" -AccountPassword(Read-Host -AsSecureString "Type Password for User") -Enabled $true
```

## 5–6 skaidrės: Bet automatizuoti galima skirtingai
Paaiškinkite imperatyvų skriptinimą (pvz., Bash) – detalus, paaiškinantis kiekvieną žingsnį.
Deklaratyvus (pvz., Ansible) – jūs tik nurodote, koks turi būti rezultatas.
Palyginkite tą patį tikslą – NGINX diegimą per Bash ir per Ansible.

## 7 skaidrė: Kokie deklaratyvaus skriptinimo privalumai?
Pabrėžkite:
- Paprastesnis ir aiškesnis kodas
- Versijavimas per Git
- Idempotentumas – garantuotas rezultatas
- Infrastruktūra kaip kodas (IaC) – viskas atkuriama
- Skalavimas ir auditas – kas padaryta, kada ir kaip

## 8 skaidrė: Kaip gali atrodyti pilnas stekas (AWS atvejis)
Remkitės paveikslu. Paaiškinkite, kaip skirtingi sluoksniai (resursai, OS, aplikacija) valdomi naudojant skirtingus AWS įrankius:
- **CloudFormation** – deklaratyvus šablonas viskam
- **OpsWorks** – OS ir hosto konfiguravimas
- **CodeDeploy** – aplikacijų diegimas, script'ai, konfigai
Tai rodo, kaip deklaratyvus požiūris taikomas visame steke.

## 9 skaidrė: Idempotentumas
Papasakokite, kas tai yra: veiksmas gali būti kartojamas neribotai, bet rezultatas nesikeičia.
Paaiškinkite paveikslą:
- Kairėje – veiksmas "žvelgti į tortą" nedaro žalos
- Dešinėje – neidempotentiškai kartojant provisioningą, VM padvigubėja
Idempotentiškumas padeda išvengti klaidų, kai konfigūracija taikoma kelis kartus.

## 10 skaidrė: Bet turi būti ir trūkumų, ar ne?
Paminėkite:
- Kodo logika reikalauja modulių, tiekėjų
- Kartais kodas pasensta greičiau nei keičiasi infrastruktūra
- Naujos funkcijos vėluoja atsirasti įrankiuose
- Reikalingas testavimas, mokymasis
Bet tai vis tiek daug geriau nei viską valdyti rankiniu būdu.

## 11 skaidrė: Panaudojimo sritys
Paaiškinkite, kad tokie įrankiai kaip Ansible naudojami:
- Debesų infrastruktūrai kurti (pvz., AWS, Azure)
- Pamatinei infrastruktūrai valdyti (tinklai, saugyklos)
- Linux/Windows serveriams konfigūruoti
- Konfigūracijai palaikyti per „push“ ar „pull“ modelius (pvz., Terraform – pull, Ansible – push)

## 12 skaidrė: Ansible
Apžvelkite, kas yra Ansible:
- Komandų eilutės įrankis
- Naudoja SSH arba WinRM, nėra agento
- Veikia iš vieno valdymo mazgo (Linux)
- Pritaikomas plačiai – OS, tinklai, debesijos paslaugos
- Sudėtingesnėms sistemoms naudojami papildomi įrankiai: AWX, Ansible Tower, Semaphore

## 13 skaidrė: Ansible koncepcijos
Paaškinkite svarbiausias sąvokas:
- **Valdymo mazgas** – kur veikia Ansible
- **Valdomas mazgas** – kokį serverį konfigūruojame
- **Inventorius** – sąrašas serverių
- **Moduliai** – ką galima konfigūruoti (failai, paslaugos, naudotojai)
- **Playbook'ai** – YAML failai su aprašytomis užduotimis


## 14 skaidrė: Praktinės užduotys
Pateikite studentams užduotis:
1. Įdiekite Ansible serveryje (pvz., Ubuntu: `sudo apt install ansible`)
2. Sukurkite `inventory` failą su IP adresu valdomo serverio
3. Sugeneruokite SSH raktą, nukopijuokite jį į valdomą serverį: `ssh-copy-id user@host`
4. Patikrinkite prisijungimą: `ansible all -i inventory -m ping`

### Papildoma praktinė užduotis: NGINX diegimas su Ansible
Toliau pateikiami žingsniai, kaip įdiegti NGINX į nuotolinį serverį naudojant Ansible:

1. **Sukurkite inventoriaus failą (pvz., `inventory.ini`)**:
```ini
[web]
192.168.1.100 ansible_user=ubuntu
```

2. **Sukurkite `nginx-install.yml` playbook'ą:**
```yaml
---
- name: Įdiegti NGINX į serverį
  hosts: web
  become: yes
  tasks:
    - name: Atnaujinti paketų sąrašą
      apt:
        update_cache: yes

    - name: Įdiegti nginx
      apt:
        name: nginx
        state: present

    - name: Įjungti ir paleisti nginx
      service:
        name: nginx
        state: started
        enabled: yes
```

3. **Paleiskite playbook'ą:**
```bash
ansible-playbook -i inventory.ini nginx-install.yml
```

4. **Patikrinkite rezultatą naršyklėje** įvedę serverio IP adresą: `http://192.168.1.100`

<!-- ### Papildoma praktinė užduotis: DigitalOcean dropleto sukūrimas naudojant Ansible

1. **Gaukite DigitalOcean API raktą**:
   - Prisijunkite prie https://cloud.digitalocean.com/account/api/tokens
   - Sukurkite naują asmeninį prieigos raktą (Personal Access Token)

2. **Įdiekite reikalingus priklausomumus:**
```bash
pip install "dopy>=0.3.5" requests python-digitalocean
```

3. **Sukurkite `digitalocean_create.yml` playbook'ą:**
```yaml
---
- name: Sukurti DigitalOcean dropletą
  hosts: localhost
  connection: local
  gather_facts: no
  vars:
    do_token: "{{ lookup('env','DO_TOKEN') }}"
  tasks:
    - name: Kurti naują dropletą
      community.digitalocean.digital_ocean_droplet:
        state: present
        name: ansible-demo-droplet
        api_token: "{{ do_token }}"
        size: s-1vcpu-1gb
        region: fra1
        image: ubuntu-22-04-x64
        ssh_keys: [12345678]  # čia įrašykite savo SSH rakto ID iš DO paskyros
      register: droplet

    - name: Atvaizduoti IP adresą
      debug:
        msg: "Dropleto IP: {{ droplet.data.ip_address }}"
```

4. **Eksportuokite DO API raktą prieš vykdydami:**
```bash
export DO_TOKEN=your_token_here
```

5. **Paleiskite playbook'ą:**
```bash
ansible-playbook digitalocean_create.yml
```

6. **Prisijunkite prie naujo dropleto ir naudokite jį tolesnėms konfigūracijoms.**
```
 -->
