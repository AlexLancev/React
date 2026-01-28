### **Объяснение**

**useCallback** — хук для мемоизации (кэширования) функций. Возвращает ту же самую функцию между рендерами, пока не изменятся зависимости.

```jsx
const memoizedCallback = useCallback(() => {
  doSomething(a, b);
}, [a, b]);
```

---

### **Зачем нужен**

#### **Проблема: новая функция при каждом рендере**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Создаётся новая функция handleClick при каждом рендере
  const handleClick = () => {
    console.log('Clicked:', count);
  };
  
  // Child перерендеривается каждый раз, даже с React.memo
  return <Child onClick={handleClick} />;
}

const Child = React.memo(({ onClick }) => {
  console.log('Child render');
  return <button onClick={onClick}>Click</button>;
});
```

#### **Решение: useCallback**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // ✅ Та же функция, пока count не изменится
  const handleClick = useCallback(() => {
    console.log('Clicked:', count);
  }, [count]);
  
  // Child не перерендеривается при изменении других состояний
  return <Child onClick={handleClick} />;
}
```

---

### **Как работает**

```jsx
// Эквивалент useCallback через useMemo:
const memoizedCallback = useMemo(
  () => () => doSomething(a, b),
  [a, b]
);

// То же самое с useCallback:
const memoizedCallback = useCallback(
  () => doSomething(a, b),
  [a, b]
);
```

---

### **Примеры использования**

#### **1. Оптимизация дочерних компонентов**
```jsx
function Parent({ items }) {
  const [filter, setFilter] = useState('');
  
  // Фильтрация кэшируется
  const filterItems = useCallback((item) => {
    return item.name.includes(filter);
  }, [filter]); // Обновляется только при изменении filter
  
  return <ItemList items={items} filterFn={filterItems} />;
}

const ItemList = React.memo(({ items, filterFn }) => {
  const filtered = items.filter(filterFn);
  return filtered.map(item => <div key={item.id}>{item.name}</div>);
});
```

#### **2. Обработчики событий**
```jsx
function Form() {
  const [text, setText] = useState('');
  
  // submitHandler не меняется при вводе текста
  const submitHandler = useCallback((e) => {
    e.preventDefault();
    console.log('Submit:', text);
  }, [text]); // Зависит от text
  
  return (
    <form onSubmit={submitHandler}>
      <input value={text} onChange={e => setText(e.target.value)} />
      <SubmitButton onSubmit={submitHandler} />
    </form>
  );
}

const SubmitButton = React.memo(({ onSubmit }) => {
  console.log('Button render'); // Рендерится только при изменении onSubmit
  return <button type="submit">Submit</button>;
});
```

#### **3. Функции для useEffect**
```jsx
function Timer() {
  const [count, setCount] = useState(0);
  
  // tick не меняется, useEffect не перезапускается
  const tick = useCallback(() => {
    setCount(c => c + 1);
  }, []); // Пустой массив - функция создаётся один раз
  
  useEffect(() => {
    const interval = setInterval(tick, 1000);
    return () => clearInterval(interval);
  }, [tick]); // Зависит от tick
  
  return <div>{count}</div>;
}
```

---

### **Распространённые ошибки**

#### **❌ Пустые зависимости для функции, которая использует стейт**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  
  // ❌ Всегда показывает count = 0 (замыкание на первоначальное значение)
  const badCallback = useCallback(() => {
    console.log(count);
  }, []); // Нет count в зависимостях
  
  // ✅ Правильно
  const goodCallback = useCallback(() => {
    console.log(count);
  }, [count]); // count в зависимостях
}
```

#### **❌ Использование там, где не нужно**
```jsx
// Избыточно, если нет React.memo дочерних компонентов
const handleClick = useCallback(() => {}, []); // ❌

// Лучше обычная функция, если нет проблем с производительностью
const handleClick = () => {}; // ✅
```

---

### **Когда использовать**

#### **✅ С React.memo дочерними компонентами**
```jsx
const ExpensiveComponent = React.memo(({ onClick }) => {
  // Тяжёлый рендер
});

function Parent() {
  const onClick = useCallback(() => {}, []);
  return <ExpensiveComponent onClick={onClick} />;
}
```

#### **✅ Зависимости useEffect/useMemo/useCallback**
```jsx
const fetchData = useCallback(async () => {
  // запрос данных
}, [query]);

useEffect(() => {
  fetchData();
}, [fetchData]); // Без useCallback был бы бесконечный цикл
```

#### **✅ Передача функций контекстом**
```jsx
const AppContext = createContext();

function App() {
  const handleLogin = useCallback((user) => {
    // логика входа
  }, []);
  
  return (
    <AppContext.Provider value={{ handleLogin }}>
      <Child />
    </AppContext.Provider>
  );
}
```

---

### **Отличие от useMemo**

| **useCallback** | **useMemo** |
|-----------------|-------------|
| Кэширует функцию | Кэширует значение |
| `() => doSomething()` | `doSomething()` |
| Возвращает функцию | Возвращает результат функции |

---

### **Резюме**

**useCallback** — хук для сохранения ссылки на функцию между рендерами.

**Ключевые нюансы:**
1. **Ссылочное равенство:** Возвращает ту же функцию, пока зависимости не изменились
2. **Для оптимизации:** Используется с `React.memo`, `useEffect`, контекстом
3. **Зависимости:** Важно правильно указать все зависимости, которые функция использует

**Как сказать своими словами:**
«useCallback — это как useMemo, но для функций. Он запоминает функцию и возвращает ту же самую, пока не поменялись зависимости. Нужен в основном для оптимизации: чтобы дочерние компоненты с React.memo не перерисовывались из-за новой функции, или чтобы useEffect не перезапускался.»