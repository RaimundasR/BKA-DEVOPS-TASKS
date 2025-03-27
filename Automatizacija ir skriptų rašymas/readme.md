## Praktinės užduotys
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