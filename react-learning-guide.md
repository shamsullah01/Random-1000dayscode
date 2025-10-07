# React Learning Guide

## Table of Contents
1. [Introduction to React](#introduction-to-react)
2. [Setting Up React](#setting-up-react)
3. [JSX Basics](#jsx-basics)
4. [Components](#components)
5. [Props](#props)
6. [State & useState Hook](#state--usestate-hook)
7. [Event Handling](#event-handling)
8. [Conditional Rendering](#conditional-rendering)
9. [Lists and Keys](#lists-and-keys)
10. [useEffect Hook](#useeffect-hook)
11. [Custom Hooks](#custom-hooks)
12. [Context API](#context-api)
13. [React Router](#react-router)
14. [Advanced Concepts](#advanced-concepts)
15. [Best Practices](#best-practices)

## Introduction to React

React is a JavaScript library for building user interfaces, particularly web applications. It was created by Facebook and is now maintained by Meta and the community.

### Key Concepts:
- **Component-Based**: Build encapsulated components that manage their own state
- **Declarative**: Describe what the UI should look like, not how to achieve it
- **Virtual DOM**: Efficient updating and rendering of components

## Setting Up React

### Create React App (Recommended for beginners)
```bash
npx create-react-app my-app
cd my-app
npm start
```

### Vite (Faster alternative)
```bash
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

## JSX Basics

JSX is a syntax extension for JavaScript that lets you write HTML-like code in your JavaScript files.

```jsx
// Basic JSX
const element = <h1>Hello, World!</h1>;

// JSX with expressions
const name = "React";
const element = <h1>Hello, {name}!</h1>;

// JSX with attributes
const element = <div className="container" id="main">Content</div>;

// JSX with children
const element = (
  <div>
    <h1>Title</h1>
    <p>This is a paragraph</p>
  </div>
);
```

### JSX Rules:
1. Return a single parent element (or use React.Fragment)
2. Close all tags
3. Use `className` instead of `class`
4. Use `htmlFor` instead of `for`

## Components

Components are the building blocks of React applications.

### Function Components (Recommended)
```jsx
// Simple function component
function Welcome() {
  return <h1>Hello, World!</h1>;
}

// Arrow function component
const Welcome = () => {
  return <h1>Hello, World!</h1>;
};

// Component with props
function Welcome(props) {
  return <h1>Hello, {props.name}!</h1>;
}

// Using the component
function App() {
  return (
    <div>
      <Welcome name="Alice" />
      <Welcome name="Bob" />
    </div>
  );
}
```

### Class Components (Legacy)
```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Hello, {this.props.name}!</h1>;
  }
}
```

## Props

Props are how you pass data from parent to child components.

```jsx
// Parent component
function App() {
  const user = {
    name: "John Doe",
    age: 25,
    email: "john@example.com"
  };

  return (
    <div>
      <UserCard user={user} isActive={true} />
    </div>
  );
}

// Child component
function UserCard({ user, isActive }) {
  return (
    <div className={isActive ? "active" : "inactive"}>
      <h2>{user.name}</h2>
      <p>Age: {user.age}</p>
      <p>Email: {user.email}</p>
    </div>
  );
}

// Props with default values
function Button({ children, variant = "primary", onClick }) {
  return (
    <button className={`btn btn-${variant}`} onClick={onClick}>
      {children}
    </button>
  );
}
```

## State & useState Hook

State allows components to have dynamic data that can change over time.

```jsx
import React, { useState } from 'react';

// Basic counter example
function Counter() {
  const [count, setCount] = useState(0);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <button onClick={() => setCount(count - 1)}>Decrement</button>
      <button onClick={() => setCount(0)}>Reset</button>
    </div>
  );
}

// Object state
function UserForm() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });

  const handleInputChange = (field, value) => {
    setUser(prevUser => ({
      ...prevUser,
      [field]: value
    }));
  };

  return (
    <div>
      <input
        type="text"
        placeholder="Name"
        value={user.name}
        onChange={(e) => handleInputChange('name', e.target.value)}
      />
      <input
        type="email"
        placeholder="Email"
        value={user.email}
        onChange={(e) => handleInputChange('email', e.target.value)}
      />
      <input
        type="number"
        placeholder="Age"
        value={user.age}
        onChange={(e) => handleInputChange('age', parseInt(e.target.value))}
      />
    </div>
  );
}

// Array state
function TodoList() {
  const [todos, setTodos] = useState([]);
  const [inputValue, setInputValue] = useState('');

  const addTodo = () => {
    if (inputValue.trim()) {
      setTodos([...todos, { id: Date.now(), text: inputValue, completed: false }]);
      setInputValue('');
    }
  };

  const toggleTodo = (id) => {
    setTodos(todos.map(todo =>
      todo.id === id ? { ...todo, completed: !todo.completed } : todo
    ));
  };

  return (
    <div>
      <input
        type="text"
        value={inputValue}
        onChange={(e) => setInputValue(e.target.value)}
        onKeyPress={(e) => e.key === 'Enter' && addTodo()}
      />
      <button onClick={addTodo}>Add Todo</button>
      <ul>
        {todos.map(todo => (
          <li key={todo.id}>
            <span
              style={{ textDecoration: todo.completed ? 'line-through' : 'none' }}
              onClick={() => toggleTodo(todo.id)}
            >
              {todo.text}
            </span>
          </li>
        ))}
      </ul>
    </div>
  );
}
```

## Event Handling

React handles events through SyntheticEvents, which wrap native DOM events.

```jsx
function EventExamples() {
  const [message, setMessage] = useState('');

  // Click event
  const handleClick = () => {
    alert('Button clicked!');
  };

  // Form submission
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log('Form submitted with message:', message);
  };

  // Input change
  const handleInputChange = (e) => {
    setMessage(e.target.value);
  };

  // Key press
  const handleKeyPress = (e) => {
    if (e.key === 'Enter') {
      console.log('Enter key pressed!');
    }
  };

  // Mouse events
  const handleMouseEnter = () => {
    console.log('Mouse entered!');
  };

  return (
    <div>
      <button onClick={handleClick}>Click Me</button>
      
      <form onSubmit={handleSubmit}>
        <input
          type="text"
          value={message}
          onChange={handleInputChange}
          onKeyPress={handleKeyPress}
          placeholder="Type something..."
        />
        <button type="submit">Submit</button>
      </form>
      
      <div
        onMouseEnter={handleMouseEnter}
        style={{ padding: '20px', backgroundColor: 'lightgray' }}
      >
        Hover over me!
      </div>
    </div>
  );
}
```

## Conditional Rendering

Render different content based on conditions.

```jsx
function ConditionalExamples({ isLoggedIn, user, items }) {
  // Ternary operator
  return (
    <div>
      {isLoggedIn ? (
        <h1>Welcome back, {user.name}!</h1>
      ) : (
        <h1>Please log in</h1>
      )}

      {/* Logical AND operator */}
      {isLoggedIn && <button>Logout</button>}

      {/* Multiple conditions */}
      {isLoggedIn ? (
        user.isAdmin ? (
          <AdminPanel />
        ) : (
          <UserPanel />
        )
      ) : (
        <LoginForm />
      )}

      {/* Conditional rendering with arrays */}
      {items.length > 0 ? (
        <ItemList items={items} />
      ) : (
        <p>No items found</p>
      )}
    </div>
  );
}

// If-else approach
function UserStatus({ user }) {
  if (!user) {
    return <div>Loading...</div>;
  }

  if (user.error) {
    return <div>Error: {user.error}</div>;
  }

  return <div>Hello, {user.name}!</div>;
}
```

## Lists and Keys

Render lists of data efficiently with keys.

```jsx
function ListExamples() {
  const fruits = ['apple', 'banana', 'orange', 'grape'];
  
  const users = [
    { id: 1, name: 'Alice', age: 25 },
    { id: 2, name: 'Bob', age: 30 },
    { id: 3, name: 'Charlie', age: 35 }
  ];

  return (
    <div>
      {/* Simple list */}
      <h3>Fruits:</h3>
      <ul>
        {fruits.map((fruit, index) => (
          <li key={index}>{fruit}</li>
        ))}
      </ul>

      {/* Object list with unique keys */}
      <h3>Users:</h3>
      <div>
        {users.map(user => (
          <UserCard key={user.id} user={user} />
        ))}
      </div>

      {/* List with filtering */}
      <h3>Adult Users (30+):</h3>
      <div>
        {users
          .filter(user => user.age >= 30)
          .map(user => (
            <UserCard key={user.id} user={user} />
          ))}
      </div>
    </div>
  );
}

function UserCard({ user }) {
  return (
    <div style={{ border: '1px solid #ccc', padding: '10px', margin: '5px' }}>
      <h4>{user.name}</h4>
      <p>Age: {user.age}</p>
    </div>
  );
}
```

## useEffect Hook

Handle side effects in functional components.

```jsx
import React, { useState, useEffect } from 'react';

// Basic useEffect
function Timer() {
  const [seconds, setSeconds] = useState(0);

  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(prev => prev + 1);
    }, 1000);

    // Cleanup function
    return () => clearInterval(interval);
  }, []); // Empty dependency array - runs once on mount

  return <div>Seconds: {seconds}</div>;
}

// useEffect with dependencies
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);

  useEffect(() => {
    const fetchUser = async () => {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const userData = await response.json();
        setUser(userData);
      } catch (error) {
        console.error('Error fetching user:', error);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]); // Runs when userId changes

  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;

  return (
    <div>
      <h2>{user.name}</h2>
      <p>{user.email}</p>
    </div>
  );
}

// Multiple useEffects
function Component() {
  const [count, setCount] = useState(0);
  const [name, setName] = useState('');

  // Effect for document title
  useEffect(() => {
    document.title = `Count: ${count}`;
  }, [count]);

  // Effect for localStorage
  useEffect(() => {
    localStorage.setItem('name', name);
  }, [name]);

  // Effect for window resize
  useEffect(() => {
    const handleResize = () => {
      console.log('Window resized');
    };

    window.addEventListener('resize', handleResize);
    return () => window.removeEventListener('resize', handleResize);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Enter your name"
      />
    </div>
  );
}
```

## Custom Hooks

Create reusable stateful logic.

```jsx
import { useState, useEffect } from 'react';

// Custom hook for local storage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      return initialValue;
    }
  });

  const setStoredValue = (value) => {
    try {
      setValue(value);
      window.localStorage.setItem(key, JSON.stringify(value));
    } catch (error) {
      console.error(error);
    }
  };

  return [value, setStoredValue];
}

// Custom hook for API calls
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchData = async () => {
      try {
        setLoading(true);
        const response = await fetch(url);
        if (!response.ok) {
          throw new Error('Network response was not ok');
        }
        const result = await response.json();
        setData(result);
      } catch (error) {
        setError(error.message);
      } finally {
        setLoading(false);
      }
    };

    fetchData();
  }, [url]);

  return { data, loading, error };
}

// Custom hook for form handling
function useForm(initialValues) {
  const [values, setValues] = useState(initialValues);

  const handleChange = (e) => {
    const { name, value } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: value
    }));
  };

  const reset = () => setValues(initialValues);

  return [values, handleChange, reset];
}

// Using custom hooks
function App() {
  const [name, setName] = useLocalStorage('name', '');
  const { data: users, loading, error } = useFetch('/api/users');
  const [formValues, handleInputChange, resetForm] = useForm({
    email: '',
    password: ''
  });

  return (
    <div>
      <input
        type="text"
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Your name (saved to localStorage)"
      />
      
      {loading && <p>Loading users...</p>}
      {error && <p>Error: {error}</p>}
      {users && <UserList users={users} />}
      
      <form>
        <input
          name="email"
          type="email"
          value={formValues.email}
          onChange={handleInputChange}
          placeholder="Email"
        />
        <input
          name="password"
          type="password"
          value={formValues.password}
          onChange={handleInputChange}
          placeholder="Password"
        />
        <button type="button" onClick={resetForm}>Reset</button>
      </form>
    </div>
  );
}
```

## Context API

Share data between components without prop drilling.

```jsx
import React, { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext();
const UserContext = createContext();

// Theme provider
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');

  const toggleTheme = () => {
    setTheme(prevTheme => prevTheme === 'light' ? 'dark' : 'light');
  };

  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// User provider
function UserProvider({ children }) {
  const [user, setUser] = useState(null);

  const login = (userData) => {
    setUser(userData);
  };

  const logout = () => {
    setUser(null);
  };

  return (
    <UserContext.Provider value={{ user, login, logout }}>
      {children}
    </UserContext.Provider>
  );
}

// Custom hooks for contexts
function useTheme() {
  const context = useContext(ThemeContext);
  if (!context) {
    throw new Error('useTheme must be used within a ThemeProvider');
  }
  return context;
}

function useUser() {
  const context = useContext(UserContext);
  if (!context) {
    throw new Error('useUser must be used within a UserProvider');
  }
  return context;
}

// Components using context
function Header() {
  const { theme, toggleTheme } = useTheme();
  const { user, logout } = useUser();

  return (
    <header style={{ 
      backgroundColor: theme === 'light' ? '#fff' : '#333',
      color: theme === 'light' ? '#333' : '#fff'
    }}>
      <h1>My App</h1>
      <button onClick={toggleTheme}>
        Switch to {theme === 'light' ? 'dark' : 'light'} theme
      </button>
      {user ? (
        <div>
          Welcome, {user.name}!
          <button onClick={logout}>Logout</button>
        </div>
      ) : (
        <button>Login</button>
      )}
    </header>
  );
}

// App with providers
function App() {
  return (
    <ThemeProvider>
      <UserProvider>
        <div>
          <Header />
          <main>
            <Content />
          </main>
        </div>
      </UserProvider>
    </ThemeProvider>
  );
}
```

## React Router

Handle navigation in single-page applications.

```jsx
import { BrowserRouter as Router, Routes, Route, Link, useParams, useNavigate } from 'react-router-dom';

// Install: npm install react-router-dom

function App() {
  return (
    <Router>
      <div>
        <nav>
          <Link to="/">Home</Link>
          <Link to="/about">About</Link>
          <Link to="/users">Users</Link>
          <Link to="/contact">Contact</Link>
        </nav>

        <Routes>
          <Route path="/" element={<Home />} />
          <Route path="/about" element={<About />} />
          <Route path="/users" element={<Users />} />
          <Route path="/users/:id" element={<UserDetail />} />
          <Route path="/contact" element={<Contact />} />
          <Route path="*" element={<NotFound />} />
        </Routes>
      </div>
    </Router>
  );
}

function Home() {
  return <h2>Home Page</h2>;
}

function About() {
  return <h2>About Page</h2>;
}

function Users() {
  const users = [
    { id: 1, name: 'Alice' },
    { id: 2, name: 'Bob' },
    { id: 3, name: 'Charlie' }
  ];

  return (
    <div>
      <h2>Users</h2>
      <ul>
        {users.map(user => (
          <li key={user.id}>
            <Link to={`/users/${user.id}`}>{user.name}</Link>
          </li>
        ))}
      </ul>
    </div>
  );
}

function UserDetail() {
  const { id } = useParams();
  const navigate = useNavigate();

  return (
    <div>
      <h2>User Detail - ID: {id}</h2>
      <button onClick={() => navigate('/users')}>Back to Users</button>
      <button onClick={() => navigate(-1)}>Go Back</button>
    </div>
  );
}

function Contact() {
  const navigate = useNavigate();

  const handleSubmit = (e) => {
    e.preventDefault();
    // Handle form submission
    alert('Message sent!');
    navigate('/');
  };

  return (
    <div>
      <h2>Contact</h2>
      <form onSubmit={handleSubmit}>
        <input type="text" placeholder="Name" required />
        <textarea placeholder="Message" required></textarea>
        <button type="submit">Send</button>
      </form>
    </div>
  );
}

function NotFound() {
  return (
    <div>
      <h2>404 - Page Not Found</h2>
      <Link to="/">Go Home</Link>
    </div>
  );
}
```

## Advanced Concepts

### React.memo (Performance Optimization)
```jsx
import React, { memo, useState, useMemo, useCallback } from 'react';

// Memoized component - only re-renders if props change
const ExpensiveComponent = memo(({ data, onItemClick }) => {
  console.log('ExpensiveComponent rendered');
  
  return (
    <div>
      {data.map(item => (
        <div key={item.id} onClick={() => onItemClick(item.id)}>
          {item.name}
        </div>
      ))}
    </div>
  );
});

function ParentComponent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([
    { id: 1, name: 'Item 1' },
    { id: 2, name: 'Item 2' }
  ]);

  // useMemo - memoize expensive calculations
  const expensiveValue = useMemo(() => {
    console.log('Calculating expensive value...');
    return items.reduce((sum, item) => sum + item.id, 0);
  }, [items]);

  // useCallback - memoize functions
  const handleItemClick = useCallback((id) => {
    console.log('Item clicked:', id);
  }, []);

  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <p>Expensive value: {expensiveValue}</p>
      <ExpensiveComponent data={items} onItemClick={handleItemClick} />
    </div>
  );
}
```

### Error Boundaries
```jsx
import React from 'react';

class ErrorBoundary extends React.Component {
  constructor(props) {
    super(props);
    this.state = { hasError: false, error: null };
  }

  static getDerivedStateFromError(error) {
    return { hasError: true, error };
  }

  componentDidCatch(error, errorInfo) {
    console.error('Error caught by boundary:', error, errorInfo);
  }

  render() {
    if (this.state.hasError) {
      return (
        <div>
          <h2>Something went wrong!</h2>
          <details>
            {this.state.error && this.state.error.toString()}
          </details>
        </div>
      );
    }

    return this.props.children;
  }
}

// Usage
function App() {
  return (
    <ErrorBoundary>
      <ComponentThatMightThrow />
    </ErrorBoundary>
  );
}
```

### Refs and useRef
```jsx
import React, { useRef, useEffect } from 'react';

function RefExamples() {
  const inputRef = useRef(null);
  const countRef = useRef(0);

  useEffect(() => {
    // Focus input on mount
    inputRef.current.focus();
  }, []);

  const handleClick = () => {
    countRef.current += 1;
    console.log('Button clicked:', countRef.current, 'times');
    inputRef.current.focus();
  };

  return (
    <div>
      <input ref={inputRef} type="text" placeholder="This will be focused" />
      <button onClick={handleClick}>Click me</button>
    </div>
  );
}
```

## Best Practices

### 1. Component Organization
```jsx
// ✅ Good: Small, focused components
function UserCard({ user }) {
  return (
    <div className="user-card">
      <UserAvatar src={user.avatar} alt={user.name} />
      <UserInfo name={user.name} email={user.email} />
      <UserActions userId={user.id} />
    </div>
  );
}

// ❌ Bad: Large, monolithic component
function UserCard({ user }) {
  return (
    <div className="user-card">
      <div className="avatar">
        <img src={user.avatar} alt={user.name} />
      </div>
      <div className="info">
        <h3>{user.name}</h3>
        <p>{user.email}</p>
      </div>
      <div className="actions">
        <button onClick={() => editUser(user.id)}>Edit</button>
        <button onClick={() => deleteUser(user.id)}>Delete</button>
      </div>
    </div>
  );
}
```

### 2. State Management
```jsx
// ✅ Good: Appropriate state placement
function TodoApp() {
  const [todos, setTodos] = useState([]);
  
  return (
    <div>
      <TodoForm onAddTodo={addTodo} />
      <TodoList todos={todos} onToggleTodo={toggleTodo} />
    </div>
  );
}

// ✅ Good: Use multiple useState for different concerns
function UserProfile() {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(false);
  const [error, setError] = useState(null);
  
  // Component logic...
}

// ❌ Bad: Everything in one state object
function UserProfile() {
  const [state, setState] = useState({
    user: null,
    loading: false,
    error: null,
    formData: {},
    isEditing: false
  });
}
```

### 3. Props and Naming
```jsx
// ✅ Good: Clear, descriptive names
function Button({ children, variant, isLoading, onClick }) {
  return (
    <button 
      className={`btn btn-${variant}`} 
      onClick={onClick}
      disabled={isLoading}
    >
      {isLoading ? 'Loading...' : children}
    </button>
  );
}

// ✅ Good: Destructure props
function UserCard({ user: { name, email, avatar }, onEdit, onDelete }) {
  return (
    <div>
      <img src={avatar} alt={name} />
      <h3>{name}</h3>
      <p>{email}</p>
      <button onClick={onEdit}>Edit</button>
      <button onClick={onDelete}>Delete</button>
    </div>
  );
}
```

### 4. Performance Tips
```jsx
// ✅ Good: Use keys properly
function TodoList({ todos }) {
  return (
    <ul>
      {todos.map(todo => (
        <TodoItem key={todo.id} todo={todo} />
      ))}
    </ul>
  );
}

// ✅ Good: Avoid creating objects/functions in render
function TodoItem({ todo, onToggle, onDelete }) {
  const handleToggle = () => onToggle(todo.id);
  const handleDelete = () => onDelete(todo.id);
  
  return (
    <li>
      <span onClick={handleToggle}>{todo.text}</span>
      <button onClick={handleDelete}>Delete</button>
    </li>
  );
}

// ❌ Bad: Creating functions in render
function TodoItem({ todo, onToggle, onDelete }) {
  return (
    <li>
      <span onClick={() => onToggle(todo.id)}>{todo.text}</span>
      <button onClick={() => onDelete(todo.id)}>Delete</button>
    </li>
  );
}
```

### 5. Error Handling
```jsx
// ✅ Good: Handle errors gracefully
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);

  useEffect(() => {
    const fetchUser = async () => {
      try {
        setLoading(true);
        setError(null);
        const userData = await api.getUser(userId);
        setUser(userData);
      } catch (err) {
        setError(err.message);
      } finally {
        setLoading(false);
      }
    };

    fetchUser();
  }, [userId]);

  if (loading) return <LoadingSpinner />;
  if (error) return <ErrorMessage message={error} />;
  if (!user) return <div>User not found</div>;

  return <UserDetails user={user} />;
}
```

## Common Patterns and Examples

### Form Handling
```jsx
function ContactForm() {
  const [formData, setFormData] = useState({
    name: '',
    email: '',
    message: ''
  });
  const [errors, setErrors] = useState({});
  const [isSubmitting, setIsSubmitting] = useState(false);

  const validate = () => {
    const newErrors = {};
    
    if (!formData.name.trim()) newErrors.name = 'Name is required';
    if (!formData.email.trim()) newErrors.email = 'Email is required';
    else if (!/\S+@\S+\.\S+/.test(formData.email)) {
      newErrors.email = 'Email is invalid';
    }
    if (!formData.message.trim()) newErrors.message = 'Message is required';
    
    return newErrors;
  };

  const handleChange = (e) => {
    const { name, value } = e.target;
    setFormData(prev => ({ ...prev, [name]: value }));
    
    // Clear error when user starts typing
    if (errors[name]) {
      setErrors(prev => ({ ...prev, [name]: '' }));
    }
  };

  const handleSubmit = async (e) => {
    e.preventDefault();
    
    const newErrors = validate();
    if (Object.keys(newErrors).length > 0) {
      setErrors(newErrors);
      return;
    }

    setIsSubmitting(true);
    try {
      await api.submitContactForm(formData);
      alert('Form submitted successfully!');
      setFormData({ name: '', email: '', message: '' });
    } catch (error) {
      alert('Error submitting form: ' + error.message);
    } finally {
      setIsSubmitting(false);
    }
  };

  return (
    <form onSubmit={handleSubmit}>
      <div>
        <input
          type="text"
          name="name"
          value={formData.name}
          onChange={handleChange}
          placeholder="Your Name"
        />
        {errors.name && <span className="error">{errors.name}</span>}
      </div>
      
      <div>
        <input
          type="email"
          name="email"
          value={formData.email}
          onChange={handleChange}
          placeholder="Your Email"
        />
        {errors.email && <span className="error">{errors.email}</span>}
      </div>
      
      <div>
        <textarea
          name="message"
          value={formData.message}
          onChange={handleChange}
          placeholder="Your Message"
        />
        {errors.message && <span className="error">{errors.message}</span>}
      </div>
      
      <button type="submit" disabled={isSubmitting}>
        {isSubmitting ? 'Submitting...' : 'Submit'}
      </button>
    </form>
  );
}
```

## Learning Resources

1. **Official Documentation**: https://react.dev/
2. **React Tutorial**: https://react.dev/learn/tutorial-tic-tac-toe
3. **Create React App**: https://create-react-app.dev/
4. **React Router**: https://reactrouter.com/
5. **React Testing Library**: https://testing-library.com/docs/react-testing-library/intro/

## Next Steps

1. Build a simple todo app using the concepts above
2. Learn about state management libraries (Redux, Zustand)
3. Explore React testing with Jest and React Testing Library
4. Learn about styling solutions (CSS Modules, Styled Components, Tailwind)
5. Study performance optimization techniques
6. Learn about React patterns and architecture
7. Explore React frameworks (Next.js, Gatsby)

Happy learning! 🚀

## Exercises (Practice)

### Beginner
- Build a Todo App (requirements): Add, toggle complete, delete. Persist todos to localStorage. Keep UI simple.
  - Hints: use `useState`, `useEffect` for localStorage, unique `id` with `Date.now()` or `crypto.randomUUID()`.
  - Success criteria: Todos persist across reloads; adding, toggling, deleting works.

- Create a Counter with presets (requirements): Buttons to add +1, -1, reset, and set to preset values (5, 10).
  - Hints: one `useState` for count, small reusable `Button` component.

### Intermediate
- User Search (requirements): A search input that filters a list of users fetched from a public API (e.g., https://jsonplaceholder.typicode.com/users). Debounce the query input (300ms).
  - Hints: `useEffect`, `fetch`, `useRef` or a debounce helper, show loading & error states.
  - Success criteria: Typing filters results after debounce, network requests minimized.

- Form with Validation (requirements): Build a multi-field signup form (name, email, password, confirm password) with inline validation messages and disabled submit until valid.
  - Hints: custom `useForm` or `useState` per field, regex for email validation, password match check.

### Advanced
- Build a small CRUD app with routing (requirements): Create, Read, Update, Delete items (can be local state). Use React Router for separate pages and a context for global state.
  - Hints: structure with `pages/` and `components/`, `useReducer` for complex state, `Context` to share dispatch.

- Performance tuning (requirements): Take a component-heavy page and optimize it with `React.memo`, `useMemo`, and `useCallback`. Measure render counts with console logs.
  - Success criteria: Unnecessary renders are eliminated when interacting with unrelated UI.

## React Cheat Sheet (Quick Reference)

- Creating components
  - Function component: `function MyComp(props) { return <div /> }`
  - Arrow component: `const MyComp = ({ x }) => <div>{x}</div>`

- Common hooks
  - useState: `const [state, setState] = useState(initial)`
  - useEffect: `useEffect(() => { sideEffect(); return cleanup; }, [deps])`
  - useRef: `const ref = useRef(initial)`
  - useContext: `const value = useContext(MyContext)`
  - useMemo: `const memo = useMemo(() => compute(a,b), [a,b])`
  - useCallback: `const fn = useCallback(() => doSomething(), [deps])`

- Performance
  - Memoize pure child components: `export default React.memo(Component)`
  - Avoid inline object/array/function props unless memoized
  - Keys: prefer stable IDs, avoid array index for keys in dynamic lists

- Forms
  - Controlled input: `<input value={value} onChange={e => setValue(e.target.value)} />`
  - Debounce inputs for API calls

- Common patterns
  - Fetch on mount: `useEffect(() => { fetchData() }, [])`
  - Cleanup subscriptions in `useEffect` return

## Quick Setup (Windows PowerShell)

These commands assume you have Node.js and npm installed. If not, install from https://nodejs.org/.

Create a Vite React app (recommended):

```powershell
cd "C:\Users\Win 11\Desktop\random-1000dayscode"
npm create vite@latest my-react-app -- --template react
cd my-react-app
npm install
npm run dev
```

Create using Create React App (slower but familiar):

```powershell
cd "C:\Users\Win 11\Desktop\random-1000dayscode"
npx create-react-app my-app
cd my-app
npm start
```

Useful PowerShell tips
- Run PowerShell as Administrator if you hit permission issues while installing global tools.
- Use double quotes for paths that contain spaces, e.g., `"C:\Users\Win 11\..."`.

## Sample Project Scaffold (Suggested structure)

Project: simple-todos

```
simple-todos/
├─ package.json
├─ README.md
├─ public/
│  └─ index.html
├─ src/
│  ├─ main.jsx
│  ├─ App.jsx
│  ├─ index.css
│  ├─ components/
│  │  ├─ TodoList.jsx
│  │  ├─ TodoItem.jsx
│  │  └─ TodoForm.jsx
│  ├─ hooks/
│  │  └─ useLocalStorage.js
│  └─ pages/
│     └─ About.jsx
└─ .gitignore
```

Minimal `main.jsx` example (Vite):

```jsx
import React from 'react'
import { createRoot } from 'react-dom/client'
import App from './App'
import './index.css'

createRoot(document.getElementById('root')).render(<App />)
```

## Interview Questions (Study)

- What is the Virtual DOM and why does React use it?
- Explain the difference between props and state.
- When and why would you use `useEffect`? Give examples of dependency arrays.
- How does reconciliation work in React? What is a key and why is it important?
- Explain the Context API and when to use it vs prop drilling.
- What are hooks? Can you write a custom hook example?
- How would you optimize performance for a list-heavy React app?
- Describe how to test React components. Which libraries do you use?

## Glossary

- JSX: JavaScript XML — syntax extension that looks like HTML inside JS.
- Component: Reusable UI building block.
- Props: Read-only inputs passed from parent to child.
- State: Internal, mutable data for a component.
- Hook: Special function (useXXX) to access React features in functional components.
- Side effect: Operations like data fetching, subscriptions, timers executed outside of render.
- Context: A way to pass data through the component tree without prop drilling.

---

If you'd like, I can also:
- scaffold a runnable starter project here in the workspace (I can create files and a working `package.json`),
- or generate downloadable ZIP containing the sample project,
- or create a short set of unit tests for one of the exercise solutions.

Let me know which you'd prefer and I will continue.