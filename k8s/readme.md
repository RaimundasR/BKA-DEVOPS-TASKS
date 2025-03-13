# Kubernetes Nginx Serviso Diegimas su Ingress

## 1. Aprašymas
Šiame dokumente aprašoma, kaip sukurti Nginx podą Kubernetes klasteryje, prijungti jį per `Service` ir `Ingress`, bei įkelti pasirinktinį `index.html` failą.

## 2. Konfigūracijos

### **1. Sukurkite `nginx.yaml` podui**

```yaml
apiVersion: v1
kind: Pod
metadata:
  name: my-nginx
  namespace: tavo-namespace
  labels:
    app: my-nginx
spec:
  containers:
    - name: nginx
      image: nginx:latest
      ports:
        - containerPort: 80
```

---

### **2. Sukurkite `service.yaml`**

```yaml
apiVersion: v1
kind: Service
metadata:
  name: my-nginx-service
  namespace: tavo-namespace
spec:
  selector:
    app: my-nginx
  ports:
    - protocol: TCP
      port: 80
      targetPort: 80
  type: ClusterIP
```

---

### **3. Sukurkite `ingress.yaml`**

```yaml
apiVersion: networking.k8s.io/v1
kind: Ingress
metadata:
  name: my-nginx-ingress
  namespace: tavo-namespace
  annotations:
    nginx.ingress.kubernetes.io/rewrite-target: /
    kubernetes.io/ingress.class: "nginx"
spec:
  ingressClassName: nginx 
  rules:
    - host: op.devplay.art
      http:
        paths:
          - path: /
            pathType: Prefix
            backend:
              service:
                name: my-nginx-service
                port:
                  number: 80
```


📌 **Pastaba:** Pakeisk `stud<num>.devplay.art` į realų hostname, jei reikia.

---

## 3. Testavimas

### **Patikrinkite, ar Nginx veikia per DNS**
- Pridėkite `/etc/hosts` (jei lokaliai testuojate jei ne ignoruokite etc/hosts):
  ```
  127.0.0.1 nginx.tavo-dns.local
  ```
- Atidarykite naršyklėje: `http://<jusu_dns>`

---

## 4. `index.html` failo įkėlimas į podą

### **1. Sukurkite `index.html` failą**

```html
<h1>Labas, tai mano nginx servisas, Vardas Pavardė</h1>
```

Išsaugokite šį failą savo kompiuteryje.

### **2. Įkelkite `index.html` į Nginx podą**

1. Nukopijuokite `index.html` į podą:
   ```sh
   kubectl cp index.html tavo-namespace/my-nginx:/usr/share/nginx/html/index.html
   ```

2. Patikrinkite, ar failas persikėlė:
   ```sh
   kubectl exec -n tavo-namespace my-nginx -- cat /usr/share/nginx/html/index.html
   ```

---

## 5. Galutinis testas
Atidarykite `http://nginx.tavo-dns.local` naršyklėje. Turėtumėte matyti:
```
Labas, tai mano nginx servisas, Vardas Pavardė
```

🔹 **Sėkmės!** 🚀
