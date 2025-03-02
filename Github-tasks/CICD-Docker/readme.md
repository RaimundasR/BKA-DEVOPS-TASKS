# Docker Įdiegimas ir Rootless Vartotojo Sukūrimas Ubuntu 22.04

## 1. Docker Įdiegimas Ubuntu 22.04 (Jammy)

1. **Atnaujinkite esamų paketų sąrašą:**
    ```bash
    sudo apt update
    ```

2. **Įdiekite privalomus paketus, reikalingus apt naudoti HTTPS paketams:**
    ```bash
    sudo apt install apt-transport-https ca-certificates curl software-properties-common
    ```

3. **Pridėkite oficialaus Docker saugyklos GPG raktą:**
    ```bash
    curl -fsSL https://download.docker.com/linux/ubuntu/gpg | sudo gpg --dearmor -o /usr/share/keyrings/docker-archive-keyring.gpg
    ```

4. **Pridėkite Docker saugyklą į APT šaltinius:**
    ```bash
    echo "deb [arch=$(dpkg --print-architecture) signed-by=/usr/share/keyrings/docker-archive-keyring.gpg] https://download.docker.com/linux/ubuntu $(lsb_release -cs) stable" | sudo tee /etc/apt/sources.list.d/docker.list > /dev/null
    ```

5. **Dar kartą atnaujinkite paketų sąrašą:**
    ```bash
    sudo apt update
    ```

6. **Patikrinkite, ar Docker bus diegiamas iš oficialios Docker saugyklos:**
    ```bash
    apt-cache policy docker-ce
    ```
    Rezultato pavyzdys:
    ```
    docker-ce:
      Installed: (none)
      Candidate: 5:20.10.14~3-0~ubuntu-jammy
      Version table:
         5:20.10.14~3-0~ubuntu-jammy 500
            500 https://download.docker.com/linux/ubuntu jammy/stable amd64 Packages
    ```

7. **Įdiekite Docker:**
    ```bash
    sudo apt install docker-ce
    ```

8. **Patikrinkite, ar Docker tarnyba veikia:**
    ```bash
    sudo systemctl status docker
    ```
    Rezultato pavyzdys:
    ```
    ● docker.service - Docker Application Container Engine
         Loaded: loaded (/lib/systemd/system/docker.service; enabled; vendor preset: enabled)
         Active: active (running) since Fri 2022-04-01 21:30:25 UTC; 22s ago
    ```

### Nuoroda į išsamią instrukciją:
Daugiau informacijos galite rasti oficialioje dokumentacijoje: [How to Install and Use Docker on Ubuntu 22.04](https://www.digitalocean.com/community/tutorials/how-to-install-and-use-docker-on-ubuntu-22-04)

---

## 2. Rootless Vartotojo Sukūrimas

1. **Sukurkite rootless vartotoją:**
    ```bash
    sudo useradd -m -s /bin/bash <rootlessuser>
    ```

2. **Išjunkite slaptažodį (Pasirinktinai):**
    Kadangi tai yra praktinė užduotis, galite išjungti slaptažodį:
    ```bash
    sudo passwd -d <rootlessuser>
    ```
3. **Jei norite išvengti sudo naudojimo kiekvieną kartą vykdant docker komandą, pridėkite savo vartotoją į docker grupę:**

```bash
sudo usermod -aG docker <rootlessuser> 
```

6. **Patikrinkite, į kokias grupes priklausote:**

```bash
groups
```

**Rezultatas turėtų atrodyti panašiai kaip pavyzdyje žemiau:**

```bash
Output
sammy sudo docker
```

---

## 3. Leidimas naudoti `sudo` be slaptažodžio (Pasirinktinai)

1. **Redaguokite `sudoers` failą:**
    ```bash
    sudo visudo
    ```

2. **Pridėkite šią eilutę failo pabaigoje:**

    ```bash
    <rootlessuser> ALL=(ALL) NOPASSWD:ALL
    ```
    Tai leis `rootlessuser` naudoti `sudo` komandas be slaptažodžio.
3. **Prisijungiame prie varototjo `rootlessuser`**

```bash
su - rootlessuser
```

4. **Jei esate prisijungę kaip dabartinis vartotojas ir turite sudo teises, galite vykdyti:**

```bash
sudo usermod -aG docker ${USER}
```

---

### Pastaba:
Šie žingsniai skirti praktinėms užduotims atlikti, todėl naudokite atsargiai produkcinėje aplinkoje.

## 4.  Susikurkite Gihub naują repozitoriją

# GitHub Self-Hosted Runner Įdiegimas

### 1. **Eikite į saugyklos pagrindinį puslapį GitHub'e**
- Prisijunkite prie GitHub ir pereikite į norimos saugyklos pagrindinį puslapį.

---

### 2. **Atidarykite nustatymus**
- Po saugyklos pavadinimu spustelėkite **Settings**.
- Jei nematote **Settings** skirtuko, pasirinkite išskleidžiamą meniu ir tada spustelėkite **Settings**.

> **Pastaba:** Jei nematote „Settings“ skirtuko, gali reikėti turėti administratoriaus teises šioje saugykloje.

---

### 3. **Pasirinkite Actions ir Runners**
- Kairėje šoninėje juostoje spustelėkite **Actions**, tada pasirinkite **Runners**.

---

### 4. **Sukurkite naują self-hosted runner**
- Spustelėkite **New self-hosted runner**.

---

### 5. **Pasirinkite operacinę sistemą ir architektūrą**
- Pasirinkite savo self-hosted runner mašinos operacinės sistemos vaizdą ir architektūrą.

---

### 6. **Vykdykite instrukcijas**
- Ekrane pamatysite instrukcijas, kaip atsisiųsti runner programą ir įdiegti ją savo self-hosted runner mašinoje.

> **Pastaba:** Atidžiai vykdykite nurodymus, pateiktus GitHub, kad sėkmingai įdiegtumėte runner programą. Failus sukelkite būtinai į jūsų rootless vartotojo failų sistemą. Tai galite padaryti prieš keliant `su - rootless`

### 7. **GH Runner paslaugos diegimas ir paleidimas kaip nuolatinės paslaugos:**
- Norėdami įdiegti GitHub Runner kaip nuolatinę paslaugą, vykdykite šias komandas:
```bash
 sudo ./svc.sh install
 ```
```bash
 sudo ./svc.sh start
 ```
---

### **Papildoma informacija:**
- Daugiau informacijos apie GitHub self-hosted runner'us galite rasti oficialioje dokumentacijoje: [GitHub Actions Runners](https://docs.github.com/en/actions/hosting-your-own-runners/about-self-hosted-runners)


## 5. Git ir failų kūrimas

### 1. Sukurkite naują šaką nuo `main` repozitorijos

```bash
git checkout -b features/my-first-setup
```
2. Pridėkite `Dockerfile`

```bash
# Use the official Nginx base image
FROM nginx:latest

# Set the working directory
WORKDIR /usr/share/nginx/html

# Remove the default Nginx static files
RUN rm -rf ./*

# Copy the custom HTML file into the Nginx web root directory
COPY index.html .

# Expose port 80 for the web server
EXPOSE 80

# Start Nginx
CMD ["nginx", "-g", "daemon off;"]
```

3. Pridėkite index.html

Galite susikurti savo HTML failą, pvz.:




```html
<!DOCTYPE html>
<html lang="en">
<head>
    <meta charset="UTF-8">
    <meta name="viewport" content="width=device-width, initial-scale=1.0">
    <title>Advanced Nginx Docker Image</title>
    <style>
        body {
            font-family: Arial, sans-serif;
            text-align: center;
            margin-top: 50px;
        }
        h1 {
            color:rgb(106, 108, 200);
        }
        p {
            color: #555;
        }
    </style>
</head>
<body>
    <h1>Welcome to the Modified Nginx Docker Image!</h1>
    <p>This page is served by a custom Nginx container.</p>
</body>
</html>
```

4. Išsaugokite sukurtus failus Git repozitorijoje

pvz:

```bash
git add --all
git commit -m "feat: first initial commit"
git push
```
5. Sukurkite .github/workflows katalogą `.github/workflows`

6. Pridėkite `nginx-ci-cd.yml` failą

Sukurkite `.github/workflow/nginx-ci-cd.yml` failą ir pritaikykite pagal savo GH ACTION nustatymus.

7. Mūsų atveju naudosime `dockerhub`repozitorija.
 
  Pridekite Github secrets:

  `DOCKERHUB_USERNAME`  - įrašykite savo DockerHub vartotojo vardą
  `DOCKERHUB_PASSWORD` - įrašykite savo DockerHub slaptažodį

8. Išsaugokite ir įkelkite į Git repozitoriją


```bash
git add .github/workflow/nginx-ci-cd.yml
git commit -m "ci: add nginx CI/CD workflow"
git push origin features/my-first-setup

```
