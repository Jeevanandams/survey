Certainly! Here's the full code for a basic todo application using the MERN stack:

**Frontend (React)**:

```jsx
// App.js
import React, { useState, useEffect } from 'react';
import axios from 'axios';
import TodoList from './components/TodoList';
import AddTodoForm from './components/AddTodoForm';

function App() {
  const [todos, setTodos] = useState([]);

  useEffect(() => {
    fetchTodos();
  }, []);

  const fetchTodos = async () => {
    try {
      const response = await axios.get('/api/todos');
      setTodos(response.data);
    } catch (error) {
      console.error('Error fetching todos:', error);
    }
  };

  const addTodo = async (todoText) => {
    try {
      const response = await axios.post('/api/todos', { text: todoText });
      setTodos([...todos, response.data]);
    } catch (error) {
      console.error('Error adding todo:', error);
    }
  };

  const deleteTodo = async (id) => {
    try {
      await axios.delete(`/api/todos/${id}`);
      setTodos(todos.filter(todo => todo._id !== id));
    } catch (error) {
      console.error('Error deleting todo:', error);
    }
  };

  return (
    <div className="App">
      <h1>Todo App</h1>
      <AddTodoForm addTodo={addTodo} />
      <TodoList todos={todos} deleteTodo={deleteTodo} />
    </div>
  );
}

export default App;
```

```jsx
// TodoList.js
import React from 'react';
import TodoItem from './TodoItem';

function TodoList({ todos, deleteTodo }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo._id} todo={todo} deleteTodo={deleteTodo} />
      ))}
    </ul>
  );
}

export default TodoList;
```

```jsx
// TodoItem.js
import React from 'react';

function TodoItem({ todo, deleteTodo }) {
  return (
    <li>
      {todo.text}
      <button onClick={() => deleteTodo(todo._id)}>Delete</button>
    </li>
  );
}

export default TodoItem;
```

```jsx
// AddTodoForm.js
import React, { useState } from 'react';

function AddTodoForm({ addTodo }) {
  const [text, setText] = useState('');

  const handleSubmit = (e) => {
    e.preventDefault();
    addTodo(text);
    setText('');
  };

  return (
    <form onSubmit={handleSubmit}>
      <input type="text" value={text} onChange={(e) => setText(e.target.value)} />
      <button type="submit">Add Todo</button>
    </form>
  );
}

export default AddTodoForm;
```

**Backend (Node.js & Express)**:

```javascript
// server.js
const express = require('express');
const mongoose = require('mongoose');
const todoRoutes = require('./routes/todoRoutes');

const app = express();
const PORT = process.env.PORT || 5000;

// Middleware
app.use(express.json());

// Connect to MongoDB
mongoose.connect('mongodb://localhost:27017/todoapp', { useNewUrlParser: true, useUnifiedTopology: true })
  .then(() => console.log('Connected to MongoDB'))
  .catch(err => console.error('Error connecting to MongoDB:', err));

// Routes
app.use('/api/todos', todoRoutes);

// Start server
app.listen(PORT, () => console.log(`Server running on port ${PORT}`));
```

```javascript
// models/Todo.js
const mongoose = require('mongoose');

const todoSchema = new mongoose.Schema({
  text: {
    type: String,
    required: true
  },
  createdAt: {
    type: Date,
    default: Date.now
  }
});

const Todo = mongoose.model('Todo', todoSchema);

module.exports = Todo;
```

```javascript
// routes/todoRoutes.js
const express = require('express');
const router = express.Router();
const Todo = require('../models/Todo');

router.get('/', async (req, res) => {
  try {
    const todos = await Todo.find().sort({ createdAt: -1 });
    res.json(todos);
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

router.post('/', async (req, res) => {
  const todo = new Todo({
    text: req.body.text
  });

  try {
    const newTodo = await todo.save();
    res.status(201).json(newTodo);
  } catch (error) {
    res.status(400).json({ message: error.message });
  }
});

router.delete('/:id', async (req, res) => {
  try {
    await Todo.findByIdAndDelete(req.params.id);
    res.json({ message: 'Todo deleted' });
  } catch (error) {
    res.status(500).json({ message: error.message });
  }
});

module.exports = router;
```
