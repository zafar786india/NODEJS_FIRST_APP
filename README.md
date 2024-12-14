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
/// react
import React, { useState, useEffect } from 'react';
import axios from 'axios';

const API_URL = 'http://localhost:5000/todos';

const App = () => {
    const [todos, setTodos] = useState([]);
    const [newTodo, setNewTodo] = useState('');
    const [editTodoId, setEditTodoId] = useState(null);
    const [editTodoText, setEditTodoText] = useState('');

   
    useEffect(() => {
        axios.get(API_URL).then((response) => setTodos(response.data));
    }, []);

   
    const addTodo = () => {
        if (!newTodo.trim()) return;
        axios.post(API_URL, { text: newTodo }).then((response) => {
            setTodos([...todos, response.data]);
            setNewTodo('');
        });
    };

   
    const startEdit = (id, text) => {
        setEditTodoId(id);
        setEditTodoText(text);
    };

    const saveEdit = () => {
        axios.put(`${API_URL}/${editTodoId}`, { text: editTodoText }).then((response) => {
            setTodos(todos.map((todo) => (todo.id === editTodoId ? response.data : todo)));
            setEditTodoId(null);
            setEditTodoText('');
        });
    };

   
    const cancelEdit = () => {
        setEditTodoId(null);
        setEditTodoText('');
    };

   
    const toggleComplete = (id) => {
        const todo = todos.find((t) => t.id === id);
        axios.put(`${API_URL}/${id}`, { completed: !todo.completed }).then((response) => {
            setTodos(todos.map((t) => (t.id === id ? response.data : t)));
        });
    };

    
    const deleteTodo = (id) => {
        axios.delete(`${API_URL}/${id}`).then(() => {
            setTodos(todos.filter((t) => t.id !== id));
        });
    };

    return (
        <div style={{ padding: '20px', maxWidth: '500px', margin: 'auto' }}>
            <h1>Todo App</h1>
            <div>
                <input
                    type="text"
                    value={newTodo}
                    onChange={(e) => setNewTodo(e.target.value)}
                    placeholder="Enter a new todo"
                    style={{ padding: '8px', width: '75%', marginRight: '10px' }}
                />
                <button onClick={addTodo} style={{ padding: '8px' }}>
                    Add
                </button>
            </div>
            <ul style={{ listStyle: 'none', padding: '0', marginTop: '20px' }}>
                {todos.map((todo) => (
                    <li
                        key={todo.id}
                        style={{
                            display: 'flex',
                            justifyContent: 'space-between',
                            padding: '10px',
                            borderBottom: '1px solid #ccc',
                        }}
                    >
                        {editTodoId === todo.id ? (
                            <div style={{ display: 'flex', flex: 1 }}>
                                <input
                                    type="text"
                                    value={editTodoText}
                                    onChange={(e) => setEditTodoText(e.target.value)}
                                    style={{ flex: 1, padding: '8px', marginRight: '10px' }}
                                />
                                <button onClick={saveEdit} style={{ marginRight: '5px' }}>
                                    Save
                                </button>
                                <button onClick={cancelEdit}>Cancel</button>
                            </div>
                        ) : (
                            <>
                                <span
                                    onClick={() => toggleComplete(todo.id)}
                                    style={{
                                        textDecoration: todo.completed ? 'line-through' : 'none',
                                        cursor: 'pointer',
                                    }}
                                >
                                    {todo.text}
                                </span>
                                <div>
                                    <button
                                        onClick={() => startEdit(todo.id, todo.text)}
                                        style={{ marginRight: '10px' }}
                                    >
                                        Edit Itams
                                    </button>
                                    <button onClick={() => deleteTodo(todo.id)} style={{ color: 'red' }}>
                                        Delete Items
                                    </button>
                                </div>
                            </>
                        )}
                    </li>
                ))}
            </ul>
        </div>
    );
};

export default App;
