const express = require('express');
const bodyParser = require('body-parser');
const cors = require('cors');
const fs = require('fs');
const app = express();

const PORT = 5000;
const DATA_FILE = './todos.json';

app.use(cors());
app.use(bodyParser.json());

// Load todos from JSON file
const loadTodos = () => {
    if (!fs.existsSync(DATA_FILE)) {
        fs.writeFileSync(DATA_FILE, JSON.stringify([]));
    }
    const data = fs.readFileSync(DATA_FILE);
    return JSON.parse(data);
};

// Save todos to JSON file
const saveTodos = (todos) => {
    fs.writeFileSync(DATA_FILE, JSON.stringify(todos, null, 2));
};

// Routes
app.get('/todos', (req, res) => {
    const todos = loadTodos();
    res.json(todos);
});

app.post('/todos', (req, res) => {
    const todos = loadTodos();
    const newTodo = { id: Date.now(), text: req.body.text, completed: false };
    todos.push(newTodo);
    saveTodos(todos);
    res.json(newTodo);
});

app.put('/todos/:id', (req, res) => {
    const todos = loadTodos();
    const updatedTodos = todos.map(todo =>
        todo.id === parseInt(req.params.id) ? { ...todo, ...req.body } : todo
    );
    saveTodos(updatedTodos);
    res.json(updatedTodos.find(todo => todo.id === parseInt(req.params.id)));
});

app.delete('/todos/:id', (req, res) => {
    const todos = loadTodos();
    const filteredTodos = todos.filter(todo => todo.id !== parseInt(req.params.id));
    saveTodos(filteredTodos);
    res.status(204).send();
});

// Start the server
app.listen(PORT, () => {
    console.log(`Server running on http://localhost:${PORT}`);
});
