# sistemadegereciamentodetarefas
package.json
{
  "name": "taskflow",
  "version": "1.0.0",
  "main": "backend/server.js",
  "scripts": {
    "start": "node backend/server.js",
    "dev": "nodemon backend/server.js",
    "test": "jest"
  },
  "dependencies": {
    "cors": "^2.8.5",
    "dotenv": "^16.4.5",
    "express": "^4.21.0",
    "mongoose": "^8.7.1"
  },
  "devDependencies": {
    "jest": "^30.0.0",
    "nodemon": "^3.1.4",
    "supertest": "^7.0.0"
  }
}
.env
PORT=3000
MONGODB_URI=mongodb://localhost:27017/taskflow
backend/config/database.js
const mongoose = require("mongoose");

const connectDB = async () => {
  try {
    await mongoose.connect(process.env.MONGODB_URI);
    console.log("MongoDB conectado");
  } catch (error) {
    console.error(error);
    process.exit(1);
  }
};

module.exports = connectDB;
backend/models/Task.js
const mongoose = require("mongoose");

const taskSchema = new mongoose.Schema({
  title: {
    type: String,
    required: true
  },
  description: {
    type: String,
    required: true
  },
  status: {
    type: String,
    default: "Pendente"
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

module.exports = mongoose.model("Task", taskSchema);
backend/controllers/taskController.js
const Task = require("../models/Task");

exports.createTask = async (req, res) => {
  try {
    const task = await Task.create(req.body);
    res.status(201).json(task);
  } catch (error) {
    res.status(400).json(error);
  }
};

exports.getTasks = async (req, res) => {
  const tasks = await Task.find();
  res.json(tasks);
};

exports.getTaskById = async (req, res) => {
  const task = await Task.findById(req.params.id);

  if (!task) {
    return res.status(404).json({
      message: "Tarefa não encontrada"
    });
  }

  res.json(task);
};

exports.updateTask = async (req, res) => {
  const task = await Task.findByIdAndUpdate(
    req.params.id,
    req.body,
    { new: true }
  );

  res.json(task);
};

exports.deleteTask = async (req, res) => {
  await Task.findByIdAndDelete(req.params.id);

  res.json({
    message: "Tarefa removida com sucesso"
  });
};
backend/routes/taskRoutes.js
const express = require("express");
const router = express.Router();

const {
  createTask,
  getTasks,
  getTaskById,
  updateTask,
  deleteTask
} = require("../controllers/taskController");

router.post("/", createTask);
router.get("/", getTasks);
router.get("/:id", getTaskById);
router.put("/:id", updateTask);
router.delete("/:id", deleteTask);

module.exports = router;
backend/server.js
require("dotenv").config();

const express = require("express");
const cors = require("cors");

const connectDB = require("./config/database");
const taskRoutes = require("./routes/taskRoutes");

const app = express();

connectDB();

app.use(cors());
app.use(express.json());

app.use("/tasks", taskRoutes);

const PORT = process.env.PORT || 3000;

app.listen(PORT, () => {
  console.log(`Servidor rodando na porta ${PORT}`);
});
frontend/index.html
<!DOCTYPE html>
<html lang="pt-BR">
<head>
<meta charset="UTF-8">
<title>TaskFlow</title>
<link rel="stylesheet" href="css/style.css">
</head>
<body>

<h1>TaskFlow</h1>

<form id="taskForm">
  <input type="text" id="title" placeholder="Título" required>
  <input type="text" id="description" placeholder="Descrição" required>
  <button type="submit">Cadastrar</button>
</form>

<ul id="taskList"></ul>

<script src="js/app.js"></script>

</body>
</html>
frontend/js/app.js
const API_URL = "http://localhost:3000/tasks";

const form = document.getElementById("taskForm");
const taskList = document.getElementById("taskList");

async function loadTasks() {
  const response = await fetch(API_URL);
  const tasks = await response.json();

  taskList.innerHTML = "";

  tasks.forEach(task => {
    const li = document.createElement("li");

    li.innerHTML = `
      <strong>${task.title}</strong>
      <br>
      ${task.description}
      <br>
      <button onclick="deleteTask('${task._id}')">
        Excluir
      </button>
    `;

    taskList.appendChild(li);
  });
}

form.addEventListener("submit", async (e) => {
  e.preventDefault();

  await fetch(API_URL, {
    method: "POST",
    headers: {
      "Content-Type": "application/json"
    },
    body: JSON.stringify({
      title: title.value,
      description: description.value
    })
  });

  form.reset();
  loadTasks();
});

async function deleteTask(id) {
  await fetch(`${API_URL}/${id}`, {
    method: "DELETE"
  });

  loadTasks();
}

loadTasks();
frontend/css/style.css
body {
  font-family: Arial, sans-serif;
  max-width: 700px;
  margin: auto;
  padding: 20px;
}

form {
  margin-bottom: 20px;
}

input {
  padding: 10px;
  margin: 5px;
}

button {
  padding: 10px;
  cursor: pointer;
}

li {
  margin: 10px 0;
  padding: 10px;
  border: 1px solid #ccc;
}
Exemplo de Teste Automatizado

tests/task.test.js

const request = require("supertest");

describe("API Tasks", () => {

  test("Deve criar uma tarefa", async () => {

    const response = await request("http://localhost:3000")
      .post("/tasks")
      .send({
        title: "Teste",
        description: "Descrição teste"
      });

    expect(response.statusCode).toBe(201);
  });

});
