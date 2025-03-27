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

## Paulius task

# Flask Hello World Deployment with Ansible

This project demonstrates how to deploy and run a minimal Flask "Hello World" app on a remote server using Ansible.

## 🐍 Flask App

Minimal Flask app (`hello.py`):

```python
from flask import Flask
app = Flask(__name__)

@app.route('/')
def hello():
    return "Hello, World!"
    
if __name__ == '__main__':
    app.run(host='0.0.0.0')
```

Optional `requirements.txt`:

```
flask
```

## 📦 Requirements

- Python 3
- Ansible installed locally
- SSH access to the remote server
- Remote server should allow connections on port 5000 (Flask default)

## 👤 Inventory File (`hosts.ini`)

```ini
[your_remote_group]
<remote_ip_or_hostname> ansible_user=<your_user> ansible_ssh_private_key_file=~/.ssh/id_rsa
```

Example:

```ini
[your_remote_group]
192.168.1.100 ansible_user=ubuntu ansible_ssh_private_key_file=~/.ssh/id_rsa
```
<!-- 
## 🛠️ Ansible Playbook (`deploy_flask.yml`)

```yaml
---
- name: Deploy Flask Hello World App
  hosts: your_remote_group
  become: true

  vars:
    app_dir: /opt/flask_app
    app_file: hello.py

  tasks:
    - name: Install Python and pip
      apt:
        name:
          - python3
          - python3-pip
        update_cache: yes

    - name: Install Flask
      pip:
        name: flask
        executable: pip3

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory
        owner: "{{ ansible_user }}"
        group: "{{ ansible_user }}"
        mode: '0755'

    - name: Copy Flask app
      copy:
        src: "{{ app_file }}"
        dest: "{{ app_dir }}/{{ app_file }}"
        mode: '0644'

    - name: Start Flask app in background
      shell: |
        nohup python3 {{ app_dir }}/{{ app_file }} > /dev/null 2>&1 &
      args:
        executable: /bin/bash
```

## 🚀 Run the Playbook

```bash
ansible-playbook -i hosts.ini deploy_flask.yml
```

## 🌍 Access the App

Once deployed, open your browser to:

```
http://<remote_ip>:5000
```

You should see:

```
Hello, World!
```

## 🔐 Optional: Open Port 5000 on Remote Server

If needed:

```bash
sudo ufw allow 5000
```

---

## 📝 Notes

- This is a basic setup. For production, consider using `gunicorn`, `systemd`, or Docker.
- The app is started using `nohup`, so it will not survive a reboot. -->


## Rokas task 

## 📦 Project Folder Structure

```
project-root/
├── ansible/
│   ├── playbook.yml
│   └── inventory.ini
├── docker-compose.yml
├── Dockerfile
├── nginx.conf
└── vue-app/
    └── (Vue.js source code)
```

---

## 🐳 Dockerfile (Multi-stage Build)

```Dockerfile
# Stage 1: Build Vue.js app
FROM node:18 AS builder
WORKDIR /app
COPY vue-app/ .
RUN npm install && npm run build

# Stage 2: Nginx serving built files
FROM nginx:alpine
COPY nginx.conf /etc/nginx/nginx.conf
COPY --from=builder /app/dist /usr/share/nginx/html
EXPOSE 80
```

---

## ⚙️ nginx.conf

```nginx
events {}

http {
  include       mime.types;
  default_type  application/octet-stream;

  server {
    listen       80;
    server_name  localhost;

    location / {
      root   /usr/share/nginx/html;
      index  index.html;
      try_files $uri $uri/ /index.html;
    }

    error_page   500 502 503 504  /50x.html;
    location = /50x.html {
      root   /usr/share/nginx/html;
    }
  }
}
```

---

## 🧩 docker-compose.yml

```yaml
version: "3"
services:
  vue-app:
    build: .
    ports:
      - "80:80"
    restart: always
```

---

## 📁 ansible/inventory.ini

```ini
[web]
remote-server-ip ansible_user=ubuntu
```

<!-- ---

## 🚀 ansible/playbook.yml

```yaml
- name: Deploy Vue.js app with Docker Compose
  hosts: web
  become: true
  vars:
    app_dir: /opt/vue-nginx-app

  tasks:
    - name: Ensure docker and docker-compose are installed
      package:
        name: "{{ item }}"
        state: present
      loop:
        - docker.io
        - docker-compose

    - name: Create app directory
      file:
        path: "{{ app_dir }}"
        state: directory

    - name: Copy project files to remote server
      synchronize:
        src: "../"
        dest: "{{ app_dir }}"
        recursive: yes
        delete: yes

    - name: Build and run docker-compose
      shell: |
        docker-compose down || true
        docker-compose build --no-cache
        docker-compose up -d
      args:
        chdir: "{{ app_dir }}"
``` -->

---

## ▶️ To Run the Deployment

From your Ansible control machine:

```bash
cd ansible
ansible-playbook -i inventory.ini playbook.yml
```

---

## ✅ What This Does

- Builds the Vue app inside Docker using multi-stage
- Serves it via Nginx
- Runs everything on the remote server via Docker Compose
- Uses Ansible to automate deployment end-to-end