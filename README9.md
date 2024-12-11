<!DOCTYPE html>
<html lang="en">
<head>
  <meta charset="UTF-8">
  <meta name="viewport" content="width=device-width, initial-scale=1.0">
  <title>To-Do Application</title>
  <style>
    body {
      font-family: Arial, sans-serif;
      margin: 0;
      padding: 20px;
      background-color: #f4f4f4;
    }
    h1 {
      text-align: center;
      color: #333;
    }
    .todo-container {
      max-width: 600px;
      margin: 0 auto;
    }
    input[type="text"] {
      width: calc(100% - 100px);
      padding: 10px;
      font-size: 16px;
    }
    button {
      padding: 10px 20px;
      font-size: 16px;
      cursor: pointer;
    }
    table {
      width: 100%;
      margin-top: 20px;
      border-collapse: collapse;
    }
    table, th, td {
      border: 1px solid #ddd;
    }
    th, td {
      padding: 10px;
      text-align: left;
    }
    td button {
      padding: 5px 10px;
      margin-right: 10px;
    }
    td input[type="text"] {
      width: 80%;
      padding: 5px;
    }
    .completed {
      background-color: #c8e6c9; /* light green */
    }
    .pending {
      background-color: #f1f1f1; /* light gray */
    }
  </style>
</head>
<body>
  <div class="todo-container">
    <h1>To-Do Application</h1>
    <div>
      <input type="text" id="todoInput" placeholder="Enter a new task">
      <button onclick="addTodo()">Add Task</button>
    </div>

    <table id="todoTable">
      <thead>
        <tr>
          <th>Task</th>
          <th>Status</th>
          <th>Actions</th>
        </tr>
      </thead>
      <tbody>
        <!-- To-Do Items will be populated here -->
      </tbody>
    </table>
  </div>

  <script>
    // Fetch and display the current todos
    async function fetchTodos() {
      const response = await fetch('http://localhost:3000/todos');
      const todos = await response.json();
      const tableBody = document.getElementById('todoTable').getElementsByTagName('tbody')[0];

      // Clear existing rows
      tableBody.innerHTML = '';

      // Populate the table with todos
      todos.forEach(todo => {
        const row = tableBody.insertRow();
        const statusClass = todo.completed ? 'completed' : 'pending';
        row.classList.add(statusClass);
        row.innerHTML = `
          <td>
            <input type="text" value="${todo.task}" onchange="updateTask(${todo.id}, this.value)">
          </td>
          <td>
            <button onclick="toggleStatus(${todo.id}, ${todo.completed})">
              ${todo.completed ? 'Mark Pending' : 'Mark Completed'}
            </button>
          </td>
          <td>
            <button onclick="deleteTodo(${todo.id})">Delete</button>
          </td>
        `;
      });
    }

    // Add a new todo item
    async function addTodo() {
      const todoInput = document.getElementById('todoInput');
      const task = todoInput.value.trim();

      if (!task) {
        alert('Please enter a task.');
        return;
      }

      const response = await fetch('http://localhost:3000/todos', {
        method: 'POST',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ task })
      });

      if (response.ok) {
        todoInput.value = ''; // Clear input field
        fetchTodos(); // Refresh the list
      } else {
        alert('Failed to add task');
      }
    }

    // Edit an existing todo item
    async function updateTask(id, newTask) {
      const response = await fetch(`http://localhost:3000/todos/${id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ task: newTask })
      });

      if (!response.ok) {
        alert('Failed to update task');
      }
    }

    // Toggle the status of a task (completed / pending)
    async function toggleStatus(id, currentStatus) {
      const newStatus = !currentStatus;
      const response = await fetch(`http://localhost:3000/todos/${id}`, {
        method: 'PUT',
        headers: {
          'Content-Type': 'application/json'
        },
        body: JSON.stringify({ completed: newStatus })
      });

      if (response.ok) {
        fetchTodos(); // Refresh the list
      } else {
        alert('Failed to update task status');
      }
    }

    // Delete a todo item
    async function deleteTodo(id) {
      const confirmation = confirm('Are you sure you want to delete this task?');
      if (!confirmation) return;

      const response = await fetch(`http://localhost:3000/todos/${id}`, {
        method: 'DELETE'
      });

      if (response.ok) {
        fetchTodos(); // Refresh the list
      } else {
        alert('Failed to delete task');
      }
    }

    // Load the todo list when the page is loaded
    window.onload = fetchTodos;
  </script>
</body>
</html>
