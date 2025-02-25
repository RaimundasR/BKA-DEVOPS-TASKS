# Užduotys: Linux komandinė eilutė ir pagrindinės komandos

### 1. **Naršymas kataloguose**  
- Naudodamiesi `cd` komanda pereikite į savo namų katalogą, o tada į `/tmp` katalogą.  
- Patikrinkite savo buvimo vietą naudodami `pwd` komandą.  

**Tikslas:** Išmokti naviguoti tarp skirtingų katalogų komandinėje eilutėje.  

---

### 2. **Failų ir katalogų kūrimas**  
- Sukurkite naują katalogą `projektas` savo namų kataloge naudodami `mkdir`.  
- Tame kataloge sukurkite tuščią failą `skaityk.txt` su `touch` komanda.  

**Tikslas:** Suprasti katalogų kūrimo ir tuščių failų inicializavimo procesą.  

---

### 3. **Failų kopijavimas ir perkėlimas**  
- Nukopijuokite failą `skaityk.txt` į tą patį katalogą su nauju pavadinimu `kopija.txt` naudodami `cp`.  
- Perkelkite `kopija.txt` į katalogą `/tmp` su `mv` komanda.  

**Tikslas:** Išmokti kopijuoti ir perkelti failus Linux sistemoje.  

---

### 4. **Failų peržiūra ir redagavimas**  
- Atidarykite `skaityk.txt` su tekstiniu redaktoriumi **nano**.  
- Parašykite kelias eilutes teksto ir išsaugokite pakeitimus (`Ctrl+O` ir `Ctrl+X`).  
- Peržiūrėkite failo turinį naudodami `cat` komandą.  

**Tikslas:** Suprasti, kaip peržiūrėti ir redaguoti tekstinius failus naudojant komandų eilutę.  

---

### 5. **Failų ištrynimas**  
- Ištrinkite failą `skaityk.txt` naudodami `rm` komandą.  
- Patikrinkite, ar failas buvo sėkmingai ištrintas su `ls`.  

**Tikslas:** Išmokti saugiai ištrinti failus Linux sistemoje.  

---

### 6. **Informacijos apie failus ir katalogus gavimas**  
- Naudodami `ls -la` komandą peržiūrėkite `projektas` katalogo turinį su visomis detalėmis.  
- Patikrinkite, kas yra dabartinis vartotojas su `whoami`.  
- Peržiūrėkite sistemos datą ir laiką su `date`.  

**Tikslas:** Išmokti gauti pagrindinę informaciją apie failus, vartotoją ir sistemą.  

---

### 7. **Failų paieška**  
- Naudokite `find` komandą, kad surastumėte visus `.txt` failus savo namų kataloge.  
- Raskite failus, kurie buvo pakeisti per pastarąsias 7 dienas.  

**Tikslas:** Suprasti, kaip efektyviai ieškoti failų naudojant skirtingus kriterijus.  

---

### 8. **Failų turinio paieška**  
- Naudokite `grep` komandą, kad surastumėte visus failus kataloge `/var/log`, kuriuose yra žodis „klaida“.  
- Naudokite `grep -i`, kad ieškotumėte žodžio nepriklausomai nuo raidžių dydžio.  

**Tikslas:** Suprasti, kaip ieškoti specifinio teksto failų turinyje.  

---

### 9. **Procesų stebėjimas ir valdymas**  
- Patikrinkite visus veikiančius procesus su `ps aux`.  
- Suraskite savo paleistą procesą naudodami `grep`.  
- Priverstinai uždarykite procesą su `kill`.  

**Tikslas:** Išmokti stebėti ir valdyti veikiančius procesus.  

---

### 10. **Prisijungimas prie nuotolinio serverio ir failų kopijavimas**  
- Prisijunkite prie nuotolinio serverio naudodami SSH komandą:  
    ```bash
    ssh vartotojas@serveris -p 22
    ```
- Naudokite `scp`, kad nukopijuotumėte failą į nuotolinį serverį.  
    ```bash
    scp failas.txt vartotojas@serveris:/kelias/
    ```  
---
### 11. **Sed texto pakeitimas**



Sukurkite faila gyvunai.txt ir įrašykite su vim ar nano tesktą **suo**, pakeiskite faile suo teksta į kate komanda su sed:

Paaiškinimas:
**-i** – nurodo, kad pakeitimai turi būti įrašyti tiesiai į failą,
**'s/suo/kate/g'** **– s** reiškia „substituciją“, g reiškia „globalų pakeitimą“, kuris pakeičia visus suo pasikartojimus kiekvienoje eilutėje,
**gyvunai.txt** – failo pavadinimas, kuriame atliekami pakeitimai.


### 12. Parašykite skriptą ir sukonfiguruokite jį kaip servisą --> žingsniai kaipo tai padaryti https://www.squash.io/executing-bash-script-at-startup-in-ubuntu-linux/


**Tikslas:** Suprasti, kaip saugiai prisijungti prie nuotolinių serverių ir perkelti failus.  
