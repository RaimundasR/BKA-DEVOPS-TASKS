# **Docker-Based JavaScript Application with MongoDB Backend**

## **Objective**

Create a JavaScript-based web application running in Docker containers that allows users to enter personal data (First Name, Last Name, Age, City) and save it in a MongoDB NoSQL database. After submitting data, a file should be generated displaying all stored entries.

## **Project Structure**

- **Frontend (Node.js & Express + Basic HTML Form)** - Allows users to input and check stored data.
- **Backend (MongoDB)** - Stores the submitted data.
- **Docker Compose** - Manages both containers.

## **Installation & Deployment**

### **1. Install Docker & Docker Compose**

Ensure Docker and Docker Compose are installed on your system.

```sh
sudo apt update
sudo apt install -y docker.io docker-compose
sudo systemctl enable --now docker
```

### **2. Create the Project Directory**

```sh
mkdir docker-js-mongo-app && cd docker-js-mongo-app
```

### **3. Create ****`docker-compose.yml`**

In the project root, create the `docker-compose.yml` file:

```yaml
version: '3.8'

services:
  mongo:
    image: mongo:latest
    container_name: mongo_db
    restart: always
    ports:
      - "27017:27017"
    volumes:
      - mongo_data:/data/db

  frontend:
    build: ./frontend
    container_name: js_app
    restart: always
    ports:
      - "3000:3000"
    environment:
      - SERVER_HOST=0.0.0.0
    depends_on:
      - mongo

volumes:
  mongo_data:
```

### **4. Create Frontend Application**

#### **4.1 Create ****`frontend/Dockerfile`**
```bash
FROM node:18
WORKDIR /app
COPY package*.json ./
RUN npm install
COPY . .
CMD ["node", "server.js"]
EXPOSE 3000
```

#### **4.2 Create ****`frontend/package.json `**

```json
{
  "name": "frontend",
  "version": "1.0.0",
  "main": "server.js",
  "dependencies": {
    "express": "^4.18.2",
    "mongoose": "^7.2.1",
    "cors": "^2.8.5"
  }
}
```

#### **4.3 Create ****`frontend/server.js`**

```js
const express = require('express');
const mongoose = require('mongoose');
const fs = require('fs');
const cors = require('cors');
const path = require('path');

const app = express();
app.use(express.json());
app.use(cors());
app.use(express.static(path.join(__dirname, 'public')));

mongoose.connect('mongodb://mongo:27017/mydatabase', {
    useNewUrlParser: true,
    useUnifiedTopology: true
});

const UserSchema = new mongoose.Schema({
    firstName: String,
    lastName: String,
    age: Number,
    city: String
});
const User = mongoose.model('User', UserSchema);

app.post('/submit', async (req, res) => {
    try {
        console.log("Received data:", req.body);
        const newUser = new User(req.body);
        await newUser.save();
        res.json({ message: 'User data saved successfully!' });
    } catch (error) {
        console.error("Error saving data:", error);
        res.status(500).json({ message: 'Error saving data', error });
    }
});

app.get('/data', async (req, res) => {
    try {
        const users = await User.find();
        res.json(users);
    } catch (error) {
        res.status(500).json({ message: 'Error retrieving data', error });
    }
});

app.get('/generate-file', async (req, res) => {
    try {
        console.log("Generating user file...");
        const users = await User.find();
        const filePath = path.join(__dirname, 'public', 'users.json');
        fs.writeFileSync(filePath, JSON.stringify(users, null, 2));
        console.log("File saved at:", filePath);
        res.download(filePath);
    } catch (error) {
        console.error("Error generating file:", error);
        res.status(500).json({ message: 'Error generating file', error });
    }
});

app.get('/', (req, res) => {
    res.sendFile(path.join(__dirname, 'public', 'index.html'));
});

const PORT = 3000;
const HOST = '0.0.0.0';
app.listen(PORT, HOST, () => console.log(`Frontend running on http://${HOST}:${PORT}`));
```

#### **4.4 Create ****`frontend/public/index.html`**

```html
<!DOCTYPE html>
<html>
<head>
    <title>User Form</title>
    <script>
        const apiUrl = window.location.origin;

        async function submitData() {
            const data = {
                firstName: document.getElementById('firstName').value,
                lastName: document.getElementById('lastName').value,
                age: parseInt(document.getElementById('age').value),
                city: document.getElementById('city').value
            };
            await fetch(`${apiUrl}/submit`, {
                method: 'POST',
                headers: { 'Content-Type': 'application/json' },
                body: JSON.stringify(data)
            });
            loadData();
        }

        async function loadData() {
            const response = await fetch(`${apiUrl}/data`);
            const data = await response.json();
            const tableBody = document.getElementById('dataTableBody');
            tableBody.innerHTML = '';
            data.forEach(user => {
                const row = `<tr>
                    <td>${user.firstName}</td>
                    <td>${user.lastName}</td>
                    <td>${user.age}</td>
                    <td>${user.city}</td>
                </tr>`;
                tableBody.innerHTML += row;
            });
        }

        function checkData() {
            window.open(`${apiUrl}/generate-file`, '_blank');
        }
    </script>
</head>
<body>
    <h2>Enter User Information</h2>
    <input type="text" id="firstName" placeholder="First Name">
    <input type="text" id="lastName" placeholder="Last Name">
    <input type="number" id="age" placeholder="Age">
    <input type="text" id="city" placeholder="City">
    <button onclick="submitData()">Submit</button>
    
    <h3>Stored Data</h3>
    <table border="1">
        <thead>
            <tr>
                <th>First Name</th>
                <th>Last Name</th>
                <th>Age</th>
                <th>City</th>
            </tr>
        </thead>
        <tbody id="dataTableBody"></tbody>
    </table>
    
    <h3>Download Data</h3>
    <button onclick="checkData()">Download JSON File</button>
    
    <script>
        loadData();
    </script>
</body>
</html>
```

### **5. Restart the Application**

```sh
docker-compose down && cd
```

### **6. Expected Outcome**

- The front-end form collects user data and stores it in MongoDB.
- Users can view stored data in a table format below the form.
- Users can download a JSON file of the stored data.
- The app runs in Docker containers.
- API requests now dynamically use the correct server IP, preventing `localhost` issues.

Try again and let me know if you need more debugging! 🚀

