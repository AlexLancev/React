### **Объяснение**

**Кастомный хук** — это JavaScript-функция, которая начинается с `use` и может вызывать другие хуки. Создаётся для переиспользования логики между компонентами.

---

### **Правила кастомных хуков**

1. **Имя начинается с `use`** — `useSomething`
2. **Может вызывать другие хуки** — `useState`, `useEffect` и т.д.
3. **Не должен вызываться условно** — как и все хуки

---

### **1. Простейший кастомный хук**

```jsx
// Хук для отслеживания кликов мыши
function useMousePosition() {
  const [position, setPosition] = useState({ x: 0, y: 0 });
  
  useEffect(() => {
    const handleMouseMove = (e) => {
      setPosition({ x: e.clientX, y: e.clientY });
    };
    
    window.addEventListener('mousemove', handleMouseMove);
    return () => window.removeEventListener('mousemove', handleMouseMove);
  }, []);
  
  return position; // Возвращаем позицию
}

// Использование
function Component() {
  const mousePos = useMousePosition(); // Вызываем как обычный хук
  return <div>Mouse: {mousePos.x}, {mousePos.y}</div>;
}
```

---

### **2. Хук с параметрами**

```jsx
// Хук для подписки на изменение размера окна
function useWindowSize(options = {}) {
  const { debounceDelay = 100 } = options; // Параметры с дефолтами
  const [size, setSize] = useState({
    width: window.innerWidth,
    height: window.innerHeight
  });
  
  useEffect(() => {
    let timeoutId;
    
    const handleResize = () => {
      clearTimeout(timeoutId);
      timeoutId = setTimeout(() => {
        setSize({
          width: window.innerWidth,
          height: window.innerHeight
        });
      }, debounceDelay);
    };
    
    window.addEventListener('resize', handleResize);
    return () => {
      window.removeEventListener('resize', handleResize);
      clearTimeout(timeoutId);
    };
  }, [debounceDelay]); // Зависимость от параметра
  
  return size;
}

// Использование
function Component() {
  const { width, height } = useWindowSize({ debounceDelay: 200 });
  return <div>Window: {width}x{height}</div>;
}
```

---

### **3. Хук, возвращающий несколько значений**

```jsx
// Хук для работы с localStorage
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    try {
      const item = localStorage.getItem(key);
      return item ? JSON.parse(item) : initialValue;
    } catch {
      return initialValue;
    }
  });
  
  const setStoredValue = useCallback((newValue) => {
    try {
      const valueToStore = typeof newValue === 'function' 
        ? newValue(value) 
        : newValue;
      setValue(valueToStore);
      localStorage.setItem(key, JSON.stringify(valueToStore));
    } catch (error) {
      console.error('LocalStorage error:', error);
    }
  }, [key, value]);
  
  const removeValue = useCallback(() => {
    localStorage.removeItem(key);
    setValue(initialValue);
  }, [key, initialValue]);
  
  return [value, setStoredValue, removeValue]; // Возвращаем массив
}

// Использование
function Component() {
  const [theme, setTheme, clearTheme] = useLocalStorage('theme', 'light');
  
  return (
    <div>
      <span>Тема: {theme}</span>
      <button onClick={() => setTheme('dark')}>Тёмная</button>
      <button onClick={() => setTheme('light')}>Светлая</button>
      <button onClick={clearTheme}>Сбросить</button>
    </div>
  );
}
```

---

### **4. Хук с рефом (для DOM операций)**

```jsx
// Хук для определения видимости элемента
function useIsVisible(options = {}) {
  const [isVisible, setIsVisible] = useState(false);
  const ref = useRef(null);
  
  useEffect(() => {
    const observer = new IntersectionObserver(([entry]) => {
      setIsVisible(entry.isIntersecting);
    }, options);
    
    if (ref.current) {
      observer.observe(ref.current);
    }
    
    return () => {
      if (ref.current) {
        observer.unobserve(ref.current);
      }
    };
  }, [options]);
  
  return [ref, isVisible]; // Возвращаем реф и состояние
}

// Использование
function Component() {
  const [ref, isVisible] = useIsVisible({ threshold: 0.5 });
  
  return (
    <div ref={ref}>
      {isVisible ? 'Виден' : 'Не виден'}
    </div>
  );
}
```

---

### **5. Хук для работы с API**

```jsx
// Хук для загрузки данных
function useFetch(url, options = {}) {
  const [data, setData] = useState(null);
  const [loading, setLoading] = useState(true);
  const [error, setError] = useState(null);
  
  useEffect(() => {
    const controller = new AbortController();
    
    const fetchData = async () => {
      try {
        setLoading(true);
        setError(null);
        
        const response = await fetch(url, {
          ...options,
          signal: controller.signal
        });
        
        if (!response.ok) {
          throw new Error(`HTTP ${response.status}`);
        }
        
        const result = await response.json();
        setData(result);
      } catch (err) {
        if (err.name !== 'AbortError') {
          setError(err.message);
        }
      } finally {
        setLoading(false);
      }
    };
    
    fetchData();
    
    return () => controller.abort(); // Отмена при размонтировании
  }, [url, options]);
  
  return { data, loading, error }; // Возвращаем объект
}

// Использование
function UserProfile({ userId }) {
  const { data: user, loading, error } = useFetch(`/api/users/${userId}`);
  
  if (loading) return <div>Загрузка...</div>;
  if (error) return <div>Ошибка: {error}</div>;
  
  return <div>{user.name}</div>;
}
```

---

### **6. Хук для управления состоянием формы**

```jsx
// Хук для работы с формами
function useForm(initialValues = {}, validate = () => ({})) {
  const [values, setValues] = useState(initialValues);
  const [errors, setErrors] = useState({});
  const [touched, setTouched] = useState({});
  
  const handleChange = useCallback((e) => {
    const { name, value, type, checked } = e.target;
    setValues(prev => ({
      ...prev,
      [name]: type === 'checkbox' ? checked : value
    }));
    
    // Валидация при изменении
    if (touched[name]) {
      const validationErrors = validate({ ...values, [name]: value });
      setErrors(prev => ({ ...prev, [name]: validationErrors[name] }));
    }
  }, [values, touched, validate]);
  
  const handleBlur = useCallback((e) => {
    const { name } = e.target;
    setTouched(prev => ({ ...prev, [name]: true }));
    
    // Валидация при потере фокуса
    const validationErrors = validate(values);
    setErrors(prev => ({ ...prev, [name]: validationErrors[name] }));
  }, [values, validate]);
  
  const handleSubmit = useCallback((onSubmit) => (e) => {
    e.preventDefault();
    const validationErrors = validate(values);
    setErrors(validationErrors);
    
    if (Object.keys(validationErrors).length === 0) {
      onSubmit(values);
    }
  }, [values, validate]);
  
  const resetForm = useCallback(() => {
    setValues(initialValues);
    setErrors({});
    setTouched({});
  }, [initialValues]);
  
  return {
    values,
    errors,
    touched,
    handleChange,
    handleBlur,
    handleSubmit,
    resetForm,
    setValues // Для программного изменения
  };
}

// Использование
function LoginForm() {
  const validate = (values) => {
    const errors = {};
    if (!values.email) errors.email = 'Обязательное поле';
    if (!values.password) errors.password = 'Обязательное поле';
    return errors;
  };
  
  const form = useForm({ email: '', password: '' }, validate);
  
  const onSubmit = (data) => {
    console.log('Submit:', data);
  };
  
  return (
    <form onSubmit={form.handleSubmit(onSubmit)}>
      <input
        name="email"
        value={form.values.email}
        onChange={form.handleChange}
        onBlur={form.handleBlur}
      />
      {form.errors.email && <span>{form.errors.email}</span>}
      
      <input
        type="password"
        name="password"
        value={form.values.password}
        onChange={form.handleChange}
        onBlur={form.handleBlur}
      />
      {form.errors.password && <span>{form.errors.password}</span>}
      
      <button type="submit">Войти</button>
    </form>
  );
}
```

---

### **7. Хук-обёртка для контекста**

```jsx
// Создаём контекст
const AuthContext = createContext();

// Провайдер
function AuthProvider({ children }) {
  const [user, setUser] = useState(null);
  const [loading, setLoading] = useState(true);
  
  useEffect(() => {
    // Проверка авторизации при монтировании
    const token = localStorage.getItem('token');
    if (token) {
      // Загрузка пользователя
      fetchUser(token).then(setUser).finally(() => setLoading(false));
    } else {
      setLoading(false);
    }
  }, []);
  
  const login = useCallback(async (credentials) => {
    const response = await loginApi(credentials);
    setUser(response.user);
    localStorage.setItem('token', response.token);
  }, []);
  
  const logout = useCallback(() => {
    setUser(null);
    localStorage.removeItem('token');
  }, []);
  
  const value = useMemo(() => ({
    user,
    loading,
    login,
    logout
  }), [user, loading, login, logout]);
  
  return <AuthContext.Provider value={value}>{children}</AuthContext.Provider>;
}

// Кастомный хук для использования контекста
function useAuth() {
  const context = useContext(AuthContext);
  if (!context) {
    throw new Error('useAuth должен использоваться внутри AuthProvider');
  }
  return context;
}

// Использование
function Component() {
  const { user, login, logout } = useAuth();
  
  if (!user) {
    return <button onClick={() => login({ email: '...', password: '...' })}>Войти</button>;
  }
  
  return (
    <div>
      Привет, {user.name}
      <button onClick={logout}>Выйти</button>
    </div>
  );
}
```

---

### **8. Составные хуки (хуки из хуков)**

```jsx
// Базовый хук для счётчика
function useCounter(initialValue = 0, step = 1) {
  const [count, setCount] = useState(initialValue);
  
  const increment = useCallback(() => {
    setCount(c => c + step);
  }, [step]);
  
  const decrement = useCallback(() => {
    setCount(c => c - step);
  }, [step]);
  
  const reset = useCallback(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return { count, increment, decrement, reset, setCount };
}

// Хук на основе useCounter
function useTimer(initialSeconds = 0) {
  const { count: seconds, increment, reset } = useCounter(initialSeconds, 1);
  const [isRunning, setIsRunning] = useState(false);
  
  useEffect(() => {
    let interval;
    if (isRunning) {
      interval = setInterval(increment, 1000);
    }
    return () => clearInterval(interval);
  }, [isRunning, increment]);
  
  const start = useCallback(() => setIsRunning(true), []);
  const stop = useCallback(() => setIsRunning(false), []);
  const restart = useCallback(() => {
    reset();
    setIsRunning(true);
  }, [reset]);
  
  return { seconds, isRunning, start, stop, restart };
}
```

---

### **Лучшие практики**

#### **✅ Хорошо:**
- Начинать имя с `use`
- Возвращать понятный интерфейс (массив/объект)
- Добавлять очистку эффектов
- Документировать параметры и возвращаемое значение

#### **❌ Плохо:**
- Нарушать правила хуков внутри
- Возвращать JSX (это уже компонент)
- Делать слишком сложные монолитные хуки
- Не обрабатывать ошибки

---

### **Резюме**

**Кастомный хук** — это функция, которая инкапсулирует логику с использованием других хуков.

**Ключевые нюансы:**
1. **Имя с `use`:** Обязательно начинается с `use` для работы ESLint правил
2. **Может использовать хуки:** Внутри можно вызывать `useState`, `useEffect` и т.д.
3. **Переиспользуемая логика:** Основная цель — вынести общую логику из компонентов

**Как сказать своими словами:**
«Кастомный хук — это собственная функция, которая использует стандартные хуки React. Создаёшь функцию, которая начинается с "use", внутри используешь useState, useEffect и другие хуки, а возвращаешь нужные данные или функции. Это как вынести повторяющуюся логику из компонентов в отдельную функцию, которую можно переиспользовать.»