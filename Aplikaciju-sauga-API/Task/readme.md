# Užduotis su API

## 1. Susidiegiame Go lang

```bash
sudo snap install go --classic
```

## 2. Sukuriame savo projekto struktūrą

```bash
mkdir my-go-api && cd my-go-api
go mod init my-go-api
```

## 3. Sudiegiame Gin for REST

```bash
go get -u github.com/gin-gonic/gin
```

## 4. Sukuriame paprastą REST API su GraphQL endpoint

Failą sukuriame pavadinimu `main.go`:

```go
package main

import (
    "net/http"
    "strings"

    "github.com/gin-gonic/gin"
)

const expectedToken = "<jusu_sugeneruotas_token>"

func authMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" || !strings.HasPrefix(authHeader, "Bearer ") {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "Authorization header missing or invalid"})
            return
        }

        token := strings.TrimPrefix(authHeader, "Bearer ")
        token = strings.TrimSpace(token)
        if token != expectedToken {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
            return
        }
        c.Next()
    }
}

var items = map[string]map[string]interface{}{
    "1": {"name": "Item One", "price": 100},
}

func main() {
    router := gin.Default()
    router.Use(authMiddleware())

    router.GET("/0/items", func(c *gin.Context) {
        c.JSON(http.StatusOK, items)
    })

    router.GET("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        if item, exists := items[id]; exists {
            c.JSON(http.StatusOK, item)
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.POST("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        var newItem map[string]interface{}
        if err := c.BindJSON(&newItem); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        if _, exists := items[id]; exists {
            c.JSON(http.StatusConflict, gin.H{"error": "Item already exists"})
            return
        }
        items[id] = newItem
        c.JSON(http.StatusCreated, newItem)
    })

    router.POST("/0/items", func(c *gin.Context) {
        var input interface{}
        if err := c.BindJSON(&input); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        switch data := input.(type) {
        case []interface{}:
            for _, item := range data {
                if itemMap, ok := item.(map[string]interface{}); ok {
                    if idVal, exists := itemMap["id"]; exists {
                        id, ok := idVal.(string)
                        if !ok {
                            c.JSON(http.StatusBadRequest, gin.H{"error": "id must be a string"})
                            return
                        }
                        if _, exists := items[id]; !exists {
                            items[id] = itemMap
                        }
                    } else {
                        c.JSON(http.StatusBadRequest, gin.H{"error": "item missing id field"})
                        return
                    }
                } else {
                    c.JSON(http.StatusBadRequest, gin.H{"error": "invalid item format"})
                    return
                }
            }
            c.JSON(http.StatusCreated, gin.H{"message": "Multiple items added", "items": items})
        case map[string]interface{}:
            if idVal, exists := data["id"]; exists {
                id, ok := idVal.(string)
                if !ok {
                    c.JSON(http.StatusBadRequest, gin.H{"error": "id must be a string"})
                    return
                }
                if _, exists := items[id]; exists {
                    c.JSON(http.StatusConflict, gin.H{"error": "Item already exists"})
                    return
                }
                items[id] = data
                c.JSON(http.StatusCreated, data)
            } else {
                c.JSON(http.StatusBadRequest, gin.H{"error": "missing id field in item"})
            }
        default:
            c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid input format"})
        }
    })

    router.PUT("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        var itemUpdate map[string]interface{}
        if err := c.BindJSON(&itemUpdate); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        if item, exists := items[id]; exists {
            for key, value := range itemUpdate {
                item[key] = value
            }
            items[id] = item
            c.JSON(http.StatusOK, item)
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.DELETE("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        if _, exists := items[id]; exists {
            delete(items, id)
            c.JSON(http.StatusOK, gin.H{"message": "Deleted item " + id})
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.DELETE("/0/items", func(c *gin.Context) {
        items = make(map[string]map[string]interface{})
        c.JSON(http.StatusOK, gin.H{"message": "All items deleted"})
    })

    router.Run(":8081")
}
```

## 5. Konfigūruojame kaip servisą (Ubuntu 20)

```bash
sudo nano /etc/systemd/system/my-go-api.service
```

```ini
[Unit]
Description=My Go API Server
After=network.target

[Service]
ExecStart=/root/my-go-api/my-go-api
WorkingDirectory=/root/my-go-api
Restart=always
User=root

[Install]
WantedBy=multi-user.target
```

Paleidžiame servisą:

```bash
systemctl daemon-reload
systemctl enable my-go-api.service
systemctl start my-go-api.service
```

Patikriname statusą:

```bash
systemctl status my-go-api.service
```

## Testavimas su `curl`

**Gauti visus elementus:**

```bash
curl -H "Authorization: Bearer <jusu_sugeneruotas_token>" http://localhost:8081/0/items
```

**Pridėti vieną elementą:**

```bash
curl -X POST -H "Authorization: Bearer <jusu_sugeneruotas_token>" \
     -H "Content-Type: application/json" \
     -d '{"name": "Item Two", "price":200}' \
     http://localhost:8081/0/items/2
```

**Pridėti kelis elementus:**

```bash
curl -X POST -H "Authorization: Bearer <jusu_sugeneruotas_token>" \
     -H "Content-Type: application/json" \
     -d '[{"id": "3", "name": "Item Three", "price":300}, {"id": "4", "name": "Item Four", "price":400}]' \
     http://localhost:8081/0/items
```

**Atnaujinti elementą:**

```bash
curl -X PUT -H "Authorization: Bearer <jusu_sugeneruotas_token>" \
     -H "Content-Type: application/json" \
     -d '{"price":250}' \
     http://localhost:8081/0/items/3
```

**Ištrinti vieną elementą:**

```bash
curl -X DELETE -H "Authorization: Bearer <jusu_sugeneruotas_token>" \
     http://localhost:8081/0/items/2
```

**Ištrinti visus elementus:**

```bash
curl -X DELETE -H "Authorization: Bearer <jusu_sugeneruotas_token>" \
     http://localhost:8081/0/items
```

> ⚠️ **Pastaba:** Nepamirškite į kiekvieną `curl` užklausą įtraukti `Authorization` antraštės — tik taip jūsų API bus apsaugota.



# Užduotis su API 2: Interaktyvi


## Tikslas

Šios užduoties tikslas — sukurti paprastą REST API naudojant Go kalbą ir Gin biblioteką. API leidžia atlikti CRUD (sukurti, nuskaityti, atnaujinti, ištrinti) operacijas su `items` kolekcija. API yra apsaugota naudojant `Bearer` tipo autentifikaciją.

Taip pat šią API galima paleisti kaip sisteminį `service`'ą Ubuntu sistemoje.

---

## Testavimas su Postman

Norint testuoti šią API naudojant **Postman**, reikės:

1. Atidaryti Postman.
2. Kurti naują `Request`.
3. Pasirinkti metodą (pvz., `GET`, `POST`, `PUT`, `DELETE`).
4. Nurodyti URL, pakeitus `localhost` į tavo serverio viešą IP (pvz., `http://<tavo_serverio_ip>:8081/0/items`).
5. Pridėti `Authorization` antraštę:

   - Pasirinkti tab'ą **Headers**
   - Pridėti `Key`: `Authorization`
   - Pridėti `Value`: `Bearer <jusu_sugeneruotas_token>`
   - Pridėti `Accept`: `application/json`
   - Pridėti `Content-Type`: `application/json`

### Pavyzdžiai:

#### Gauti visus elementus (`GET`):

- **URL:** `http://<tavo_serverio_ip>:8081/0/items`
- **Method:** `GET`
- **Headers:**  
  `Authorization: Bearer <jusu_sugeneruotas_token>`

#### Pridėti vieną elementą (`POST`):

- **URL:** `http://<tavo_serverio_ip>:8081/0/items/2`
- **Method:** `POST`
- **Headers:**  
  `Authorization: Bearer ...`  
  `Content-Type: application/json`
- **Body (raw JSON):**
```json
{
  "name": "Item Two",
  "price": 200
}
```

#### Atnaujinti elementą (`PUT`):

- **URL:** `http://<tavo_serverio_ip>:8081/0/items/2`
- **Method:** `PUT`
- **Headers:**  
  `Authorization: Bearer ...`  
  `Content-Type: application/json`
- **Body (raw JSON):**
```json
{
  "price": 250
}
```

#### Ištrinti elementą (`DELETE`):

- **URL:** `http://<tavo_serverio_ip>:8081/0/items/2`
- **Method:** `DELETE`
- **Headers:**  
  `Authorization: Bearer ...`

---

> 🛡️ **Svarbu:** Autorizacija būtina kiekvienai užklausai. Be tinkamo `Bearer` tokeno užklausos bus atmestos (401 klaida).

---

# Užduotis su API 3: Token išoriniuose nustatymuose

## Tikslas

Sukurti saugesnę REST API versiją, kur `Bearer` token yra saugomas išoriniame faile (`env.config.json`) ir nebekoduojamas tiesiogiai `main.go` faile. Ši versija taip pat paleidžiama kaip `systemd` servisas.

---

## 1. Sukurk `env.config.json` failą

```json
{
  "TOKEN": "<jusu_token_>"
}
```

> Šis failas turėtų būti tame pačiame kataloge kaip `main.go`.

---

## 2. Pilnas `main.go` failas su token nuskaitymu iš JSON

```go
package main

import (
    "encoding/json"
    "fmt"
    "io/ioutil"
    "net/http"
    "os"
    "strings"

    "github.com/gin-gonic/gin"
)

var expectedToken string

func loadTokenFromFile(path string) (string, error) {
    file, err := ioutil.ReadFile(path)
    if err != nil {
        return "", err
    }

    var config map[string]string
    if err := json.Unmarshal(file, &config); err != nil {
        return "", err
    }

    token, exists := config["TOKEN"]
    if !exists {
        return "", fmt.Errorf("TOKEN reikšmė nerasta JSON faile")
    }

    return token, nil
}

func authMiddleware() gin.HandlerFunc {
    return func(c *gin.Context) {
        authHeader := c.GetHeader("Authorization")
        if authHeader == "" || !strings.HasPrefix(authHeader, "Bearer ") {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "Authorization header missing or invalid"})
            return
        }

        token := strings.TrimPrefix(authHeader, "Bearer ")
        token = strings.TrimSpace(token)
        if token != expectedToken {
            c.AbortWithStatusJSON(http.StatusUnauthorized, gin.H{"error": "Invalid token"})
            return
        }
        c.Next()
    }
}

var items = map[string]map[string]interface{}{
    "1": {"name": "Item One", "price": 100},
}

func main() {
    var err error
    expectedToken, err = loadTokenFromFile("env.config.json")
    if err != nil {
        panic("Nepavyko nuskaityti tokeno: " + err.Error())
    }

    router := gin.Default()
    router.Use(authMiddleware())

    router.GET("/0/items", func(c *gin.Context) {
        c.JSON(http.StatusOK, items)
    })

    router.GET("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        if item, exists := items[id]; exists {
            c.JSON(http.StatusOK, item)
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.POST("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        var newItem map[string]interface{}
        if err := c.BindJSON(&newItem); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        if _, exists := items[id]; exists {
            c.JSON(http.StatusConflict, gin.H{"error": "Item already exists"})
            return
        }
        items[id] = newItem
        c.JSON(http.StatusCreated, newItem)
    })

    router.POST("/0/items", func(c *gin.Context) {
        var input interface{}
        if err := c.BindJSON(&input); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        switch data := input.(type) {
        case []interface{}:
            for _, item := range data {
                if itemMap, ok := item.(map[string]interface{}); ok {
                    if idVal, exists := itemMap["id"]; exists {
                        id, ok := idVal.(string)
                        if !ok {
                            c.JSON(http.StatusBadRequest, gin.H{"error": "id must be a string"})
                            return
                        }
                        if _, exists := items[id]; !exists {
                            items[id] = itemMap
                        }
                    } else {
                        c.JSON(http.StatusBadRequest, gin.H{"error": "item missing id field"})
                        return
                    }
                } else {
                    c.JSON(http.StatusBadRequest, gin.H{"error": "invalid item format"})
                    return
                }
            }
            c.JSON(http.StatusCreated, gin.H{"message": "Multiple items added", "items": items})
        case map[string]interface{}:
            if idVal, exists := data["id"]; exists {
                id, ok := idVal.(string)
                if !ok {
                    c.JSON(http.StatusBadRequest, gin.H{"error": "id must be a string"})
                    return
                }
                if _, exists := items[id]; exists {
                    c.JSON(http.StatusConflict, gin.H{"error": "Item already exists"})
                    return
                }
                items[id] = data
                c.JSON(http.StatusCreated, data)
            } else {
                c.JSON(http.StatusBadRequest, gin.H{"error": "missing id field in item"})
            }
        default:
            c.JSON(http.StatusBadRequest, gin.H{"error": "Invalid input format"})
        }
    })

    router.PUT("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        var itemUpdate map[string]interface{}
        if err := c.BindJSON(&itemUpdate); err != nil {
            c.JSON(http.StatusBadRequest, gin.H{"error": err.Error()})
            return
        }
        if item, exists := items[id]; exists {
            for key, value := range itemUpdate {
                item[key] = value
            }
            items[id] = item
            c.JSON(http.StatusOK, item)
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.DELETE("/0/items/:item_id", func(c *gin.Context) {
        id := c.Param("item_id")
        if _, exists := items[id]; exists {
            delete(items, id)
            c.JSON(http.StatusOK, gin.H{"message": "Deleted item " + id})
        } else {
            c.JSON(http.StatusNotFound, gin.H{"error": "Item not found"})
        }
    })

    router.DELETE("/0/items", func(c *gin.Context) {
        items = make(map[string]map[string]interface{})
        c.JSON(http.StatusOK, gin.H{"message": "All items deleted"})
    })

    router.Run(":8081")
}
```

---

## 3. Perkompiliuoti Go aplikaciją

Po pakeitimų:

```bash
go build -o my-go-api main.go
```

Tai sugeneruos vykdomąjį failą `my-go-api`.

---

## 4. Perkrauti systemd servisą (jei paleista kaip servisas)

```bash
sudo systemctl daemon-reload
sudo systemctl restart my-go-api.service
```

Patikrink statusą:

```bash
sudo systemctl status my-go-api.service
```

---

## 5. Testavimas per Postman

**URL šablonas:**

```
http://<tavo-serverio-viesas-ip>:8081/0/items
```

**Headers:**

- `Authorization: Bearer <TOKEN>`  
  Pvz.: `Bearer <jusu_token_>`

**Pavyzdžiai:**

- `GET`: gauti visus elementus
- `POST`: pridėti elementą
- `PUT`: atnaujinti elementą
- `DELETE`: pašalinti vieną arba visus

> 🛡️ **Svarbu:** `env.config.json` nereikėtų kelti į Git (`.gitignore`).


