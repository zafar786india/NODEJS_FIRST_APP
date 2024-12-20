//
import React, { useState } from 'react';

const TodoForm = ({ addTodo }) => {
    const [text, setText] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        if (text.trim()) {
            addTodo(text);
            setText('');
        }
    };

    return (
        <form onSubmit={handleSubmit}>
            <input 
                type="text" 
                value={text} 
                onChange={(e) => setText(e.target.value)} 
                placeholder="Add a todo" 
            />
            <button type="submit">Add</button>
        </form>
    );
};

export default TodoForm;
//
import React from 'react';

const TodoItem = ({ todo, toggleTodo, deleteTodo }) => {
    return (
        <li>
            <input 
                type="checkbox" 
                checked={todo.completed} 
                onChange={() => toggleTodo(todo.id, !todo.completed)} 
            />
            {todo.text}
            <button onClick={() => deleteTodo(todo.id)}>Delete</button>
        </li>
    );
};

export default TodoItem;
//
// TodoList.js
import React from 'react';
import TodoItem from './TodoItem';

const TodoList = ({ todos, toggleTodo, deleteTodo }) => {
    return (
        <ul>
            {todos.map(todo => (
                <TodoItem 
                    key={todo.id} 
                    todo={todo} 
                    toggleTodo={toggleTodo} 
                    deleteTodo={deleteTodo} 
                />
            ))}
        </ul>
    );
};

export default TodoList;
// App.js (React Frontend)
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import TodoList from './TodoList';
import TodoForm from './TodoForm';

const App = () => {
    const [todos, setTodos] = useState([]);

    useEffect(() => {
        axios.get('http://localhost:5000/api/todos')
            .then(response => setTodos(response.data))
            .catch(error => console.error(error));
    }, []);

    const addTodo = (text) => {
        axios.post('http://localhost:5000/api/todos', { text })
            .then(response => setTodos([...todos, response.data]))
            .catch(error => console.error(error));
    };

    const toggleTodo = (id, completed) => {
        axios.put(`http://localhost:5000/api/todos/${id}`, { completed })
            .then(response => {
                setTodos(todos.map(todo => todo.id === id ? response.data : todo));
            })
            .catch(error => console.error(error));
    };

    const deleteTodo = (id) => {
        axios.delete(`http://localhost:5000/api/todos/${id}`)
            .then(() => {
                setTodos(todos.filter(todo => todo.id !== id));
            })
            .catch(error => console.error(error));
    };

    return (
        <div className="app">
            <h1>Todo App</h1>
            <TodoForm addTodo={addTodo} />
            <TodoList todos={todos} toggleTodo={toggleTodo} deleteTodo={deleteTodo} />
        </div>
    );
};

export default App;
//

import React, { useState, useEffect } from 'react';
import axios from 'axios';
import TodoList from './components/TodoList';
import AddTodo from './components/AddTodo';

const API_URL = 'http://localhost:5000/todos';

const App = () => {
    const [todos, setTodos] = useState([]);

    // Fetch todos from backend
    useEffect(() => {
        axios.get(API_URL).then((response) => setTodos(response.data));
    }, []);

    // Add a new todo
    const addTodo = (text) => {
        axios.post(API_URL, { text }).then((response) => {
            setTodos([...todos, response.data]);
        });
    };

    // Edit a todo
    const editTodo = (id, text) => {
        axios.put(`${API_URL}/${id}`, { text }).then((response) => {
            setTodos(todos.map((todo) => (todo.id === id ? response.data : todo)));
        });
    };

    // Toggle completion status
    const toggleComplete = (id) => {
        const todo = todos.find((t) => t.id === id);
        axios.put(`${API_URL}/${id}`, { completed: !todo.completed }).then((response) => {
            setTodos(todos.map((t) => (t.id === id ? response.data : t)));
        });
    };

    // Delete a todo
    const deleteTodo = (id) => {
        axios.delete(`${API_URL}/${id}`).then(() => {
            setTodos(todos.filter((todo) => todo.id !== id));
        });
    };

    return (
        <div style={{ padding: '20px', maxWidth: '500px', margin: 'auto' }}>
            <h1>Todo App</h1>
            <AddTodo addTodo={addTodo} />
            <TodoList
                todos={todos}
                toggleComplete={toggleComplete}
                editTodo={editTodo}
                deleteTodo={deleteTodo}
            />
        </div>
    );
};

export default App;
////



import React from 'react';
import TodoItem from './TodoItem';

const TodoList = ({ todos, toggleComplete, editTodo, deleteTodo }) => {
    return (
        <ul style={{ listStyle: 'none', padding: '0' }}>
            {todos.map((todo) => (
                <TodoItem
                    key={todo.id}
                    todo={todo}
                    toggleComplete={toggleComplete}
                    editTodo={editTodo}
                    deleteTodo={deleteTodo}
                />
            ))}
        </ul>
    );
};

export default TodoList;
///
import React, { useState } from 'react';

const TodoItem = ({ todo, toggleComplete, editTodo, deleteTodo }) => {
    const [isEditing, setIsEditing] = useState(false);
    const [editText, setEditText] = useState(todo.text);

    const handleSave = () => {
        editTodo(todo.id, editText);
        setIsEditing(false);
    };

    return (
        <li
            style={{
                display: 'flex',
                justifyContent: 'space-between',
                padding: '10px',
                borderBottom: '1px solid #ccc',
            }}
        >
            {isEditing ? (
                <div style={{ display: 'flex', flex: 1 }}>
                    <input
                        type="text"
                        value={editText}
                        onChange={(e) => setEditText(e.target.value)}
                        style={{ flex: 1, padding: '8px', marginRight: '10px' }}
                    />
                    <button onClick={handleSave} style={{ marginRight: '5px' }}>
                        Save
                    </button>
                    <button onClick={() => setIsEditing(false)}>Cancel</button>
                </div>
            ) : (
                <>
                    <span
                        onClick={() => toggleComplete(todo.id)}
                        style={{
                            textDecoration: todo.completed ? 'line-through' : 'none',
                            cursor: 'pointer',
                            flex: 1,
                        }}
                    >
                        {todo.text}
                    </span>
                    <div>
                        <button onClick={() => setIsEditing(true)} style={{ marginRight: '10px' }}>
                            Edit
                        </button>
                        <button onClick={() => deleteTodo(todo.id)} style={{ color: 'red' }}>
                            Delete
                        </button>
                    </div>
                </>
            )}
        </li>
    );
};

export default TodoItem;
//
import React, { useState } from 'react';

const AddTodo = ({ addTodo }) => {
    const [text, setText] = useState('');

    const handleSubmit = (e) => {
        e.preventDefault();
        if (!text.trim()) return;
        addTodo(text);
        setText('');
    };

    return (
        <form onSubmit={handleSubmit} style={{ marginBottom: '20px' }}>
            <input
                type="text"
                value={text}
                onChange={(e) => setText(e.target.value)}
                placeholder="Enter a new todo"
                style={{ padding: '8px', width: '75%', marginRight: '10px' }}
            />
            <button type="submit" style={{ padding: '8px' }}>
                Add
            </button>
        </form>
    );
};

export default AddTodo;
