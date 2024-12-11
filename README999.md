const express = require('express');
const fs = require('fs');
const path = require('path');

const app = express();
const port = 3000;

// Middleware to parse JSON requests
app.use(express.json());

// Serve static files (like index.html) from the root directory
app.use(express.static(path.join(__dirname)));

// Path to the todos.json file
const todosFilePath = path.join(__dirname, 'todos.json');

// Function to read data from the todos.json file
const readTodosFromFile = () => {
    const data = fs.readFileSync(todosFilePath, 'utf8');
    return JSON.parse(data);
};

// Function to write data to the todos.json file
const writeTodosToFile = (todos) => {
    fs.writeFileSync(todosFilePath, JSON.stringify(todos, null, 2), 'utf8');
};


// 1. Get all todos
app.get('/todos', (req, res) => {
    const todos = readTodosFromFile();
    res.json(todos);
});


// 2. Add a new todo (completed = false by default)
app.post('/todos', (req, res) => {
    const { task } = req.body;

    if (!task) {
        return res.status(400).json({ error: 'Task is required' });
    }
    const todos = readTodosFromFile();
    const newTodo = {
        id: Date.now(),
        task,
        completed: false // New task starts as pending
    };

    todos.push(newTodo);
    writeTodosToFile(todos);

    res.status(201).json(newTodo);
});

// 3. Edit an existing todo (update task or status)
app.put('/todos/:id', (req, res) => {
    const { id } = req.params;
    const { task, completed } = req.body;

    const todos = readTodosFromFile();
    const todoIndex = todos.findIndex((todo) => todo.id === parseInt(id));

    if (todoIndex === -1) {
        return res.status(404).json({ error: 'Todo not found' });
    }

    if (task) todos[todoIndex].task = task;
    if (completed !== undefined) todos[todoIndex].completed = completed;

    writeTodosToFile(todos);

    res.json(todos[todoIndex]);
});


// 4. Delete a todo
app.delete('/todos/:id', (req, res) => {
    const { id } = req.params;
  
    const todos = readTodosFromFile();
    const todoIndex = todos.findIndex((todo) => todo.id === parseInt(id));
  
    if (todoIndex === -1) {
      return res.status(404).json({ error: 'Todo not found' });
    }
  
    todos.splice(todoIndex, 1);
    writeTodosToFile(todos);
  
    res.status(204).send();
  });
  

// Start the server
app.listen(port, () => {
    console.log(`To-Do app is running on http://localhost:${port}`);
});
