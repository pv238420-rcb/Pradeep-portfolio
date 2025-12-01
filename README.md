# LPU Campus Helper

A simple web app for LPU students to manage notes, timetables, and Q&A, with clean UI and full authentication.

## Features
- User Registration & Login (JWT Auth)
- Notes Upload System (coming soon)
- Timetable Manager (coming soon)
- Q&A Section (coming soon)
- Responsive UI

---

## Project Structure
client/ # React frontend
server/ # Node.js backend (Express + MongoDB)
.github/ci # GitHub Actions pipeline
docker-compose # Easy development setup


---

## Getting Started

### Backend Setup


cd server
npm install
npm run dev


### Frontend Setup


cd client
npm install
npm start


---

## Environment Variables
Create `server/.env`:



MONGO_URI=your_mongo_database_url
JWT_SECRET=your_secret_key
PORT=5000


---

## Scripts
### Client
- `npm start` – start React

### Server
- `npm run dev` – start Express with nodemon

---

## License
MIT License

📄 .gitignore
node_modules/
.env
dist/
build/
.DS_Store

📄 docker-compose.yml
version: "3"
services:
  server:
    build: ./server
    ports:
      - "5000:5000"
    environment:
      - MONGO_URI=${MONGO_URI}
      - JWT_SECRET=${JWT_SECRET}

  client:
    build: ./client
    ports:
      - "3000:3000"

📁 .github/workflows/ci.yml
name: Node.js CI

on:
  push:
    branches: [ "main" ]
  pull_request:
    branches: [ "main" ]

jobs:
  build:
    runs-on: ubuntu-latest

    steps:
      - uses: actions/checkout@v3
      - name: Use Node.js 18
        uses: actions/setup-node@v3
        with:
          node-version: 18

      - name: Install Dependencies
        run: |
          cd server && npm install
          cd ../client && npm install

      - name: Run Tests (placeholder)
        run: echo "No tests yet"

📁 SERVER SIDE (Node.js + Express + MongoDB)
server/package.json
{
  "name": "lpu-campus-helper-server",
  "version": "1.0.0",
  "main": "src/index.js",
  "scripts": {
    "dev": "nodemon src/index.js"
  },
  "dependencies": {
    "bcryptjs": "^2.4.3",
    "cors": "^2.8.5",
    "dotenv": "^16.0.3",
    "express": "^4.18.2",
    "jsonwebtoken": "^9.0.0",
    "mongoose": "^7.0.0"
  },
  "devDependencies": {
    "nodemon": "^2.0.22"
  }
}

server/src/config.js
import dotenv from "dotenv";
dotenv.config();

export default {
  mongoURI: process.env.MONGO_URI,
  jwtSecret: process.env.JWT_SECRET,
  port: process.env.PORT || 5000,
};

server/src/models/User.js
import mongoose from "mongoose";

const UserSchema = new mongoose.Schema({
  name: { type: String, required: true },
  email: { type: String, required: true, unique: true },
  password: { type: String, required: true }
});

export default mongoose.model("User", UserSchema);

server/src/routes/auth.js
import express from "express";
import bcrypt from "bcryptjs";
import jwt from "jsonwebtoken";
import User from "../models/User.js";
import config from "../config.js";

const router = express.Router();

// Register
router.post("/register", async (req, res) => {
  try {
    const { name, email, password } = req.body;

    const existing = await User.findOne({ email });
    if (existing) return res.status(400).json({ message: "Email already exists" });

    const hashed = await bcrypt.hash(password, 10);

    const user = await User.create({
      name,
      email,
      password: hashed
    });

    res.json({ message: "User registered", user });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

// Login
router.post("/login", async (req, res) => {
  try {
    const { email, password } = req.body;

    const user = await User.findOne({ email });
    if (!user) return res.status(404).json({ message: "User not found" });

    const match = await bcrypt.compare(password, user.password);
    if (!match) return res.status(400).json({ message: "Wrong password" });

    const token = jwt.sign({ id: user._id }, config.jwtSecret, {
      expiresIn: "7d",
    });

    res.json({ message: "Login success", token });
  } catch (err) {
    res.status(500).json({ error: err.message });
  }
});

export default router;

server/src/index.js
import express from "express";
import mongoose from "mongoose";
import cors from "cors";
import config from "./config.js";
import authRoutes from "./routes/auth.js";

const app = express();
app.use(cors());
app.use(express.json());

// Routes
app.use("/api/auth", authRoutes);

// MongoDB connect
mongoose
  .connect(config.mongoURI)
  .then(() => console.log("MongoDB Connected"))
  .catch((err) => console.log(err));

app.listen(config.port, () =>
  console.log(`Server running on port ${config.port}`)
);

📁 CLIENT SIDE (React)
client/package.json
{
  "name": "lpu-campus-helper-client",
  "version": "1.0.0",
  "dependencies": {
    "axios": "^1.3.0",
    "react": "^18.0.0",
    "react-dom": "^18.0.0",
    "react-router-dom": "^6.14.0"
  },
  "scripts": {
    "start": "react-scripts start"
  }
}

client/src/index.js
import React from "react";
import ReactDOM from "react-dom/client";
import App from "./App";

const root = ReactDOM.createRoot(document.getElementById("root"));
root.render(<App />);

client/src/App.js
import { BrowserRouter, Routes, Route } from "react-router-dom";
import Login from "./pages/Login";
import Register from "./pages/Register";

export default function App() {
  return (
    <BrowserRouter>
      <Routes>
        <Route path="/" element={<Login />} />
        <Route path="/register" element={<Register />} />
      </Routes>
    </BrowserRouter>
  );
}

client/src/pages/Login.js
import axios from "axios";
import { useState } from "react";

export default function Login() {
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  async function submit() {
    const res = await axios.post("http://localhost:5000/api/auth/login", {
      email,
      password,
    });
    alert(res.data.message);
  }

  return (
    <div>
      <h1>Login</h1>
      <input placeholder="Email" onChange={(e) => setEmail(e.target.value)} />
      <input placeholder="Password" type="password"
        onChange={(e) => setPassword(e.target.value)} />
      <button onClick={submit}>Login</button>
    </div>
  );
}

client/src/pages/Register.js
import axios from "axios";
import { useState } from "react";

export default function Register() {
  const [name, setName] = useState("");
  const [email, setEmail] = useState("");
  const [password, setPassword] = useState("");

  async function submit() {
    const res = await axios.post("http://localhost:5000/api/auth/register", {
      name,
      email,
      password,
    });
    alert(res.data.message);
  }

  return (
    <div>
      <h1>Register</h1>
      <input placeholder="Name" onChange={(e) => setName(e.target.value)} />
      <input placeholder="Email" onChange={(e) => setEmail(e.target.value)} />
      <input placeholder="Password" type="password"
        onChange={(e) => setPassword(e.target.value)} />
      <button onClick={submit}>Register</button>
    </div>
  );
}

