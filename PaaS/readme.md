
---

## Praktinė užduotis AWS aplinkoje – PaaS tema

### 🎯 Tikslas
Susipažinti su AWS PaaS paslaugomis: valdomomis duomenų bazėmis (Amazon RDS), objektine saugykla (Amazon S3), bei serverless automatizacija naudojant Lambda ir SNS.

---

### 🧪 Užduotis

1. **Sukurkite saugos grupę (Security Group)**, kuri leistų prieigą prie MSSQL ar MySQL duomenų bazės per TCP portą (1433 arba 3306) iš visų IP adresų (`0.0.0.0/0`).

2. **Sukurkite naują RDS instanciją** naudodami Amazon RDS paslaugą:
   - Pasirinkite MSSQL arba MySQL.
   - Nurodykite instancijos pavadinimą, naudotoją ir slaptažodį.
   - Įjunkite "Public access: Yes".
   - Pasirinkite mažiausius resursus (pvz., db.t3.micro, jei taikoma free tier).

3. **Prisijunkite prie duomenų bazės** naudodami SQL Server Management Studio (MSSQL) arba MySQL Workbench, nurodę duomenų bazės adresą, vartotoją ir slaptažodį.

4. **Sukurkite S3 bucket'ą** (pvz., `studento-vardas-paas-task`) Amazon S3 paslaugoje:
   - Įkelkite bet kokį failą (pvz., `test.txt`).

5. **Sukurkite SNS (Simple Notification Service) temą**:
   - Sukurkite temą (pvz., `S3FileUploadNotification`).
   - Prenumeruokite savo el. pašto adresą.
   - Patvirtinkite prenumeratą (patvirtinimo nuoroda ateis į el. paštą).

6. **Sukurkite Lambda funkciją**:
   - Programavimo kalba: Python arba Node.js.
   - Funkcija turi būti iškviečiama kai failas įkeliamas į jūsų S3 bucket.
   - Funkcija turi išsiųsti pranešimą į SNS temą.

7. **Sukurkite S3 įvykio trigerį**:
   - Konfigūruokite S3 bucket'ą taip, kad įvykdytų Lambda funkciją, kai įkeltas naujas objektas.

---

### ✅ Patikrinimo žingsniai

- Įkelkite naują failą į S3 bucket.
- Patikrinkite, ar Lambda funkcija buvo paleista.
- Patikrinkite, ar el. paštu gavote pranešimą per SNS.
- Patikrinkite, ar galite prisijungti prie RDS duomenų bazės.

---

Ši užduotis padės įsitikinti, kad išmanote pagrindinius AWS PaaS komponentus ir mokate juos sujungti į veikiančią mini sistemą – nuo duomenų bazės iki automatizuotų el. laiškų siuntimo.

---

