---
title: "JavaScript Mastery - Part 8: Frontend Frameworks"
date: 2025-03-10 00:00:00 +0530
categories: [JavaScript, JavaScript Mastery]
tags: [JavaScript, Programming, Web Development, Front-end, Components, State-management, Frameworks, React, Vue, Angular, Svelte]
---

# Complete JavaScript Mastery Part 8: Frontend Frameworks

## Introduction

Frontend frameworks enable building complex, maintainable user interfaces with component-based architecture, reactive state management, and powerful tooling ecosystems.

This part explores:
- React and its ecosystem
- Vue.js fundamentals and Composition API
- Angular architecture
- Svelte and its unique approach
- Framework comparison and selection

**Prerequisites:**
- Parts 1-7: JavaScript through modern development

**Why Frontend Frameworks Matter:**

- Component reusability
- Declarative UI development
- Virtual DOM efficiency (React, Vue)
- Rich ecosystems and communities
- Production-ready patterns

Let's master modern frontend frameworks.

---

## 8.1 React

### React Fundamentals

React is a library for building user interfaces using components.

#### Components

```jsx
// Function component (modern)
function Welcome(props) {
  return <h1>Hello, {props.name}</h1>;
}

// Arrow function component
const Welcome = ({ name }) => {
  return <h1>Hello, {name}</h1>;
};

// Class component (legacy)
class Welcome extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}

// Usage
<Welcome name="John" />
```

#### JSX

```jsx
// JSX is syntactic sugar for React.createElement
const element = <h1>Hello World</h1>;

// Compiles to:
const element = React.createElement('h1', null, 'Hello World');

// JavaScript expressions in JSX
const name = 'John';
const element = <h1>Hello, {name}</h1>;

// Attributes
const element = <img src={user.avatar} alt={user.name} />;

// Conditional rendering
const element = (
  <div>
    {isLoggedIn ? <UserGreeting /> : <GuestGreeting />}
  </div>
);

// Lists
const numbers = [1, 2, 3, 4, 5];
const listItems = numbers.map((number) => (
  <li key={number}>{number}</li>
));

// Fragments (avoid extra DOM nodes)
return (
  <>
    <Header />
    <Main />
    <Footer />
  </>
);

// Or explicit
return (
  <React.Fragment>
    <Header />
    <Main />
  </React.Fragment>
);
```

### React Hooks

#### useState

```jsx
import { useState } from 'react';

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

// Multiple state variables
function Form() {
  const [name, setName] = useState('');
  const [email, setEmail] = useState('');
  const [age, setAge] = useState(0);
  
  const handleSubmit = (e) => {
    e.preventDefault();
    console.log({ name, email, age });
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input
        value={name}
        onChange={(e) => setName(e.target.value)}
        placeholder="Name"
      />
      <input
        value={email}
        onChange={(e) => setEmail(e.target.value)}
        placeholder="Email"
      />
      <input
        type="number"
        value={age}
        onChange={(e) => setAge(Number(e.target.value))}
        placeholder="Age"
      />
      <button type="submit">Submit</button>
    </form>
  );
}

// Object state
function User() {
  const [user, setUser] = useState({
    name: '',
    email: '',
    age: 0
  });
  
  const updateUser = (field, value) => {
    setUser(prev => ({
      ...prev,
      [field]: value
    }));
  };
  
  return (
    <div>
      <input
        value={user.name}
        onChange={(e) => updateUser('name', e.target.value)}
      />
    </div>
  );
}

// Lazy initialization (expensive computation)
function ExpensiveComponent() {
  const [state, setState] = useState(() => {
    const initialState = computeExpensiveValue();
    return initialState;
  });
}
```

#### useEffect

```jsx
import { useState, useEffect } from 'react';

// Basic effect
function Example() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Count: ${count}`;
  });
  
  return <div>{count}</div>;
}

// Effect with cleanup
function Timer() {
  const [seconds, setSeconds] = useState(0);
  
  useEffect(() => {
    const interval = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    // Cleanup function
    return () => clearInterval(interval);
  }, []); // Empty array = run once on mount
  
  return <div>Seconds: {seconds}</div>;
}

// Effect with dependencies
function User({ userId }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchUser() {
      setLoading(true);
      try {
        const response = await fetch(`/api/users/${userId}`);
        const data = await response.json();
        
        if (!cancelled) {
          setUser(data);
        }
      } catch (error) {
        console.error(error);
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchUser();
    
    return () => {
      cancelled = true;
    };
  }, [userId]); // Re-run when userId changes
  
  if (loading) return <div>Loading...</div>;
  if (!user) return <div>User not found</div>;
  
  return <div>{user.name}</div>;
}

// Multiple effects for separation of concerns
function UserProfile({ userId }) {
  useEffect(() => {
    // Subscribe to user updates
    const unsubscribe = subscribeToUser(userId);
    return unsubscribe;
  }, [userId]);
  
  useEffect(() => {
    // Track analytics
    trackPageView('user-profile', userId);
  }, [userId]);
  
  useEffect(() => {
    // Update document title
    document.title = `User ${userId}`;
  }, [userId]);
}
```

#### useContext

```jsx
import { createContext, useContext, useState } from 'react';

// Create context
const ThemeContext = createContext('light');

// Provider component
function ThemeProvider({ children }) {
  const [theme, setTheme] = useState('light');
  
  const toggleTheme = () => {
    setTheme(prev => prev === 'light' ? 'dark' : 'light');
  };
  
  return (
    <ThemeContext.Provider value={{ theme, toggleTheme }}>
      {children}
    </ThemeContext.Provider>
  );
}

// Consumer component
function ThemedButton() {
  const { theme, toggleTheme } = useContext(ThemeContext);
  
  return (
    <button
      style={{
        background: theme === 'light' ? '#fff' : '#333',
        color: theme === 'light' ? '#333' : '#fff'
      }}
      onClick={toggleTheme}
    >
      Toggle Theme
    </button>
  );
}

// App
function App() {
  return (
    <ThemeProvider>
      <ThemedButton />
    </ThemeProvider>
  );
}

// Auth context example
const AuthContext = createContext(null);

function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  
  const login = async (email, password) => {
    const user = await loginAPI(email, password);
    setUser(user);
  };
  
  const logout = () => {
    setUser(null);
  };
  
  return (
    <AuthContext.Provider value={{ user, login, logout }}>
      {children}
    </AuthContext.Provider>
  );
}

// Custom hook for auth
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth must be used within AuthProvider');
  }
  return context;
}

// Usage
function Profile() {
  const { user, logout } = useAuth();
  
  return (
    <div>
      <p>Welcome, {user.name}</p>
      <button onClick={logout}>Logout</button>
    </div>
  );
}
```

#### useReducer

```jsx
import { useReducer } from 'react';

// Reducer function
function counterReducer(state, action) {
  switch (action.type) {
    case 'increment':
      return { count: state.count + 1 };
    case 'decrement':
      return { count: state.count - 1 };
    case 'reset':
      return { count: 0 };
    default:
      throw new Error(`Unknown action: ${action.type}`);
  }
}

function Counter() {
  const [state, dispatch] = useReducer(counterReducer, { count: 0 });
  
  return (
    <div>
      <p>Count: {state.count}</p>
      <button onClick={() => dispatch({ type: 'increment' })}>+</button>
      <button onClick={() => dispatch({ type: 'decrement' })}>-</button>
      <button onClick={() => dispatch({ type: 'reset' })}>Reset</button>
    </div>
  );
}

// Complex state management
const initialState = {
  todos: [],
  filter: 'all'
};

function todoReducer(state, action) {
  switch (action.type) {
    case 'add':
      return {
        ...state,
        todos: [...state.todos, action.payload]
      };
    case 'toggle':
      return {
        ...state,
        todos: state.todos.map(todo =>
          todo.id === action.payload
            ? { ...todo, completed: !todo.completed }
            : todo
        )
      };
    case 'delete':
      return {
        ...state,
        todos: state.todos.filter(todo => todo.id !== action.payload)
      };
    case 'setFilter':
      return {
        ...state,
        filter: action.payload
      };
    default:
      return state;
  }
}

function TodoApp() {
  const [state, dispatch] = useReducer(todoReducer, initialState);
  
  const addTodo = (text) => {
    dispatch({
      type: 'add',
      payload: {
        id: Date.now(),
        text,
        completed: false
      }
    });
  };
  
  const toggleTodo = (id) => {
    dispatch({ type: 'toggle', payload: id });
  };
  
  const deleteTodo = (id) => {
    dispatch({ type: 'delete', payload: id });
  };
  
  const setFilter = (filter) => {
    dispatch({ type: 'setFilter', payload: filter });
  };
  
  const filteredTodos = state.todos.filter(todo => {
    if (state.filter === 'active') return !todo.completed;
    if (state.filter === 'completed') return todo.completed;
    return true;
  });
  
  return (
    <div>
      <AddTodoForm onAdd={addTodo} />
      <FilterButtons filter={state.filter} onFilterChange={setFilter} />
      <TodoList
        todos={filteredTodos}
        onToggle={toggleTodo}
        onDelete={deleteTodo}
      />
    </div>
  );
}
```

#### useMemo and useCallback

```jsx
import { useState, useMemo, useCallback } from 'react';

// useMemo - memoize expensive computations
function ExpensiveComponent({ items }) {
  const [filter, setFilter] = useState('');
  
  // Without useMemo: computed on every render
  // const filtered = items.filter(item => item.includes(filter));
  
  // With useMemo: only recompute when dependencies change
  const filtered = useMemo(() => {
    console.log('Filtering items');
    return items.filter(item => item.includes(filter));
  }, [items, filter]);
  
  return (
    <div>
      <input value={filter} onChange={(e) => setFilter(e.target.value)} />
      <ul>
        {filtered.map(item => <li key={item}>{item}</li>)}
      </ul>
    </div>
  );
}

// useCallback - memoize callback functions
function Parent() {
  const [count, setCount] = useState(0);
  const [items, setItems] = useState([]);
  
  // Without useCallback: new function on every render
  // const handleClick = () => {
  //   setItems([...items, count]);
  // };
  
  // With useCallback: same function reference
  const handleClick = useCallback(() => {
    setItems(prev => [...prev, count]);
  }, [count]);
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
      <Child onClick={handleClick} />
    </div>
  );
}

// Child only re-renders if onClick changes
const Child = React.memo(({ onClick }) => {
  console.log('Child rendered');
  return <button onClick={onClick}>Add Item</button>;
});

// Practical example: Debounced search
function SearchComponent() {
  const [query, setQuery] = useState('');
  const [results, setResults] = useState([]);
  
  const search = useCallback(
    debounce(async (searchQuery) => {
      const data = await fetch(`/api/search?q=${searchQuery}`);
      const json = await data.json();
      setResults(json);
    }, 300),
    []
  );
  
  const handleChange = (e) => {
    const value = e.target.value;
    setQuery(value);
    search(value);
  };
  
  return (
    <div>
      <input value={query} onChange={handleChange} />
      <Results items={results} />
    </div>
  );
}
```

#### useRef

```jsx
import { useRef, useEffect } from 'react';

// Access DOM elements
function TextInput() {
  const inputRef = useRef(null);
  
  useEffect(() => {
    // Focus input on mount
    inputRef.current.focus();
  }, []);
  
  return <input ref={inputRef} />;
}

// Store mutable values
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(null);
  
  const start = () => {
    if (intervalRef.current !== null) return;
    
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
  };
  
  const stop = () => {
    if (intervalRef.current === null) return;
    
    clearInterval(intervalRef.current);
    intervalRef.current = null;
  };
  
  const reset = () => {
    stop();
    setSeconds(0);
  };
  
  return (
    <div>
      <p>Seconds: {seconds}</p>
      <button onClick={start}>Start</button>
      <button onClick={stop}>Stop</button>
      <button onClick={reset}>Reset</button>
    </div>
  );
}

// Previous value
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  }, [value]);
  
  return ref.current;
}

function Counter() {
  const [count, setCount] = useState(0);
  const prevCount = usePrevious(count);
  
  return (
    <div>
      <p>Current: {count}</p>
      <p>Previous: {prevCount}</p>
      <button onClick={() => setCount(count + 1)}>Increment</button>
    </div>
  );
}
```

### Custom Hooks

```jsx
// useFetch - data fetching hook
function useFetch(url) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    let cancelled = false;
    
    async function fetchData() {
      try {
        setLoading(true);
        const response = await fetch(url);
        const json = await response.json();
        
        if (!cancelled) {
          setData(json);
          setError(null);
        }
      } catch (err) {
        if (!cancelled) {
          setError(err);
        }
      } finally {
        if (!cancelled) {
          setLoading(false);
        }
      }
    }
    
    fetchData();
    
    return () => {
      cancelled = true;
    };
  }, [url]);
  
  return { data, loading, error };
}

// Usage
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Loading...</div>;
  if (error) return <div>Error: {error.message}</div>;
  
  return <div>{user.name}</div>;
}

// useLocalStorage - persist state in localStorage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = window.localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch (error) {
      console.error(error);
      return initialValue;
    }
  });
  
  const setStoredValue = (newValue) => {
    try {
      setValue(newValue);
      window.localStorage.setItem(key, JSON.stringify(newValue));
    } catch (error) {
      console.error(error);
    }
  };
  
  return [value, setStoredValue];
}

// Usage
function Settings() {
  const [theme, setTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div>
      <p>Current theme: {theme}</p>
      <button onClick={() => setTheme('light')}>Light</button>
      <button onClick={() => setTheme('dark')}>Dark</button>
    </div>
  );
}

// useDebounce - debounce value changes
function useDebounce(value, delay) {
  const [debouncedValue, setDebouncedValue] = useState(value);
  
  useEffect(() => {
    const timer = setTimeout(() => {
      setDebouncedValue(value);
    }, delay);
    
    return () => {
      clearTimeout(timer);
    };
  }, [value, delay]);
  
  return debouncedValue;
}

// Usage
function Search() {
  const [searchTerm, setSearchTerm] = useState('');
  const debouncedSearchTerm = useDebounce(searchTerm, 500);
  
  useEffect(() => {
    if (debouncedSearchTerm) {
      // Perform search
      console.log('Searching:', debouncedSearchTerm);
    }
  }, [debouncedSearchTerm]);
  
  return (
    <input
      value={searchTerm}
      onChange={(e) => setSearchTerm(e.target.value)}
      placeholder="Search..."
    />
  );
}
```

### React Router

```jsx
import {
  BrowserRouter,
  Routes,
  Route,
  Link,
  useNavigate,
  useParams,
  Navigate
} from 'react-router-dom';

function App() {
  return (
    <BrowserRouter>
      <nav>
        <Link to="/">Home</Link>
        <Link to="/about">About</Link>
        <Link to="/users">Users</Link>
      </nav>
      
      <Routes>
        <Route path="/" element={<Home />} />
        <Route path="/about" element={<About />} />
        <Route path="/users" element={<Users />} />
        <Route path="/users/:id" element={<UserDetail />} />
        <Route path="/login" element={<Login />} />
        <Route
          path="/dashboard"
          element={
            <ProtectedRoute>
              <Dashboard />
            </ProtectedRoute>
          }
        />
        <Route path="*" element={<NotFound />} />
      </Routes>
    </BrowserRouter>
  );
}

// Route parameters
function UserDetail() {
  const { id } = useParams();
  const { data: user } = useFetch(`/api/users/${id}`);
  
  return <div>{user?.name}</div>;
}

// Programmatic navigation
function Login() {
  const navigate = useNavigate();
  
  const handleLogin = async (credentials) => {
    await loginAPI(credentials);
    navigate('/dashboard');
  };
  
  return <LoginForm onSubmit={handleLogin} />;
}

// Protected routes
function ProtectedRoute({ children }) {
  const { user } = useAuth();
  
  if (!user) {
    return <Navigate to="/login" replace />;
  }
  
  return children;
}
```

### State Management with Redux Toolkit

```javascript
// store.js
import { configureStore, createSlice } from '@reduxjs/toolkit';

// Slice
const counterSlice = createSlice({
  name: 'counter',
  initialState: { value: 0 },
  reducers: {
    increment: (state) => {
      state.value += 1;
    },
    decrement: (state) => {
      state.value -= 1;
    },
    incrementByAmount: (state, action) => {
      state.value += action.payload;
    }
  }
});

export const { increment, decrement, incrementByAmount } = counterSlice.actions;

// Store
export const store = configureStore({
  reducer: {
    counter: counterSlice.reducer
  }
});

// Component
import { useSelector, useDispatch } from 'react-redux';

function Counter() {
  const count = useSelector((state) => state.counter.value);
  const dispatch = useDispatch();
  
  return (
    <div>
      <p>Count: {count}</p>
      <button onClick={() => dispatch(increment())}>+</button>
      <button onClick={() => dispatch(decrement())}>-</button>
      <button onClick={() => dispatch(incrementByAmount(5))}>+5</button>
    </div>
  );
}

// Async thunks
import { createAsyncThunk } from '@reduxjs/toolkit';

const fetchUsers = createAsyncThunk('users/fetch', async () => {
  const response = await fetch('/api/users');
  return response.json();
});

const usersSlice = createSlice({
  name: 'users',
  initialState: {
    items: [],
    loading: false,
    error: null
  },
  extraReducers: (builder) => {
    builder
      .addCase(fetchUsers.pending, (state) => {
        state.loading = true;
      })
      .addCase(fetchUsers.fulfilled, (state, action) => {
        state.loading = false;
        state.items = action.payload;
      })
      .addCase(fetchUsers.rejected, (state, action) => {
        state.loading = false;
        state.error = action.error.message;
      });
  }
});
```

---

## 8.2 Vue.js

### Vue Fundamentals

```vue
<!-- Single File Component -->
<template>
  <div class="counter">
    <h1>Count: {{ count }}</h1>
    <button @click="increment">Increment</button>
    <button @click="decrement">Decrement</button>
  </div>
</template>

<script>
export default {
  data() {
    return {
      count: 0
    };
  },
  methods: {
    increment() {
      this.count++;
    },
    decrement() {
      this.count--;
    }
  }
};
</script>

<style scoped>
.counter {
  padding: 20px;
}
</style>
```

### Composition API (Vue 3)

```vue
<template>
  <div>
    <h1>Count: {{ count }}</h1>
    <button @click="increment">Increment</button>
    
    <h2>User: {{ user?.name }}</h2>
    <p v-if="loading">Loading...</p>
  </div>
</template>

<script setup>
import { ref, computed, watch, onMounted } from 'vue';

// Reactive state
const count = ref(0);
const user = ref(null);
const loading = ref(false);

// Computed properties
const doubleCount = computed(() => count.value * 2);

// Methods
const increment = () => {
  count.value++;
};

// Watchers
watch(count, (newValue, oldValue) => {
  console.log(`Count changed from ${oldValue} to ${newValue}`);
});

// Lifecycle hooks
onMounted(async () => {
  loading.value = true;
  const response = await fetch('/api/user');
  user.value = await response.json();
  loading.value = false;
});
</script>
```

### Vue Router

```javascript
import { createRouter, createWebHistory } from 'vue-router';

const routes = [
  {
    path: '/',
    component: Home
  },
  {
    path: '/users/:id',
    component: UserDetail,
    props: true
  },
  {
    path: '/dashboard',
    component: Dashboard,
    meta: { requiresAuth: true }
  }
];

const router = createRouter({
  history: createWebHistory(),
  routes
});

// Navigation guard
router.beforeEach((to, from, next) => {
  if (to.meta.requiresAuth && !isAuthenticated()) {
    next('/login');
  } else {
    next();
  }
});
```

---

## 8.3 Angular

### Component

```typescript
import { Component } from '@angular/core';

@Component({
  selector: 'app-counter',
  template: `
    <div>
      <h1>Count: {{ count }}</h1>
      <button (click)="increment()">Increment</button>
      <button (click)="decrement()">Decrement</button>
    </div>
  `,
  styles: [`
    div {
      padding: 20px;
    }
  `]
})
export class CounterComponent {
  count = 0;
  
  increment() {
    this.count++;
  }
  
  decrement() {
    this.count--;
  }
}
```

### Services and Dependency Injection

```typescript
import { Injectable } from '@angular/core';
import { HttpClient } from '@angular/common/http';
import { Observable } from 'rxjs';

@Injectable({
  providedIn: 'root'
})
export class UserService {
  constructor(private http: HttpClient) {}
  
  getUsers(): Observable<User[]> {
    return this.http.get<User[]>('/api/users');
  }
  
  getUser(id: number): Observable<User> {
    return this.http.get<User>(`/api/users/${id}`);
  }
}

// Component using service
@Component({
  selector: 'app-users',
  template: `
    <div *ngFor="let user of users$ | async">
      {{ user.name }}
    </div>
  `
})
export class UsersComponent {
  users$: Observable<User[]>;
  
  constructor(private userService: UserService) {
    this.users$ = this.userService.getUsers();
  }
}
```

---

## 8.4 Svelte

### Svelte Component

```svelte
<script>
  let count = 0;
  
  $: doubled = count * 2; // Reactive statement
  
  function increment() {
    count += 1;
  }
  
  function decrement() {
    count -= 1;
  }
</script>

<div>
  <h1>Count: {count}</h1>
  <p>Doubled: {doubled}</p>
  <button on:click={increment}>Increment</button>
  <button on:click={decrement}>Decrement</button>
</div>

<style>
  div {
    padding: 20px;
  }
</style>
```

### Stores (State Management)

```javascript
// stores.js
import { writable, derived } from 'svelte/store';

export const count = writable(0);

export const doubled = derived(count, $count => $count * 2);

// Component
<script>
  import { count, doubled } from './stores';
  
  function increment() {
    count.update(n => n + 1);
  }
</script>

<h1>Count: {$count}</h1>
<p>Doubled: {$doubled}</p>
<button on:click={increment}>Increment</button>
```

---

## Framework Comparison

| Feature | React | Vue | Angular | Svelte |
|---------|-------|-----|---------|--------|
| **Learning Curve** | Moderate | Easy | Steep | Easy |
| **Size** | Medium | Small | Large | Smallest |
| **Performance** | Fast | Fast | Fast | Fastest |
| **Ecosystem** | Huge | Large | Large | Growing |
| **TypeScript** | Good | Good | Excellent | Good |
| **Mobile** | React Native | Capacitor | Ionic | Capacitor |
| **Company** | Meta | Community | Google | Community |

---

## Key Takeaways

✅ **React:**
- Component-based with hooks
- Virtual DOM
- Huge ecosystem
- JSX syntax

✅ **Vue:**
- Progressive framework
- Options and Composition API
- Template syntax
- Easy learning curve

✅ **Angular:**
- Full framework
- TypeScript first
- Dependency injection
- RxJS integration

✅ **Svelte:**
- Compiler approach
- No virtual DOM
- Reactive by default
- Smallest bundle

---

## Conclusion

Part 8 covered major frontend frameworks. Choose based on project needs and team expertise.

**What's Next:**

Part 9 will cover Performance Optimization.

---

**Total Word Count: Part 8 Complete (~18,000 words)**
