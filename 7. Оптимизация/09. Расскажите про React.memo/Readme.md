### **Объяснение**

**React.memo** — это компонент высшего порядка (HOC), который мемоизирует результат рендера компонента. Если пропсы не изменились, компонент не перерисовывается.

### **Резюме**

**React.memo** — оптимизация для предотвращения лишних рендеров при неизменных пропсах.

**Ключевые нюансы:**
1. **Мемоизация компонента:** Запоминает результат рендера и повторно использует его при одинаковых пропсах
2. **Поверхностное сравнение:** По умолчанию сравнивает пропсы через `Object.is` (аналог `===`)
3. **Работает с useCallback/useMemo:** Для объектов/функций нужна мемоизация пропсов

**Как сказать своими словами:**
«React.memo — это как кэш для компонента. Он запоминает, как компонент выглядел с определёнными пропсами, и если при следующем рендере пропсы те же самые — не перерисовывает его. Полезно для "тяжёлых" компонентов или когда родитель часто обновляется, а дети — нет. Но работает только с пропсами, и для объектов/функций нужно дополнительно использовать useMemo/useCallback.»

```jsx
const MemoizedComponent = React.memo(MyComponent);
```

---

### **Как работает**

React.memo сравнивает пропсы текущего и предыдущего рендера:

```jsx
// Без React.memo
function Parent() {
  const [count, setCount] = useState(0);
  
  // Child перерисовывается при каждом изменении count
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Child text="Static text" /> {/* Все равно перерисовывается! */}
    </>
  );
}

// С React.memo
const Child = React.memo(function Child({ text }) {
  console.log('Child renders'); // 👈 Только при изменении text
  return <div>{text}</div>;
});
// Теперь Child не перерисовывается при изменении count
```

---

### **Поверхностное сравнение (по умолчанию)**

React.memo использует поверхностное сравнение (shallow compare):

```jsx
const User = React.memo(({ user }) => {
  return <div>{user.name}</div>;
});

// ❌ Проблема: объект user создаётся заново при каждом рендере
function Parent() {
  const [count, setCount] = useState(0);
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <User user={{ name: 'John' }} /> {/* Новый объект каждый раз! */}
    </>
  );
}

// ✅ Решение: useMemo или вынести за пределы
function Parent() {
  const [count, setCount] = useState(0);
  const user = useMemo(() => ({ name: 'John' }), []);
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>+</button>
      <User user={user} />
    </>
  );
}
```

---

### **Кастомная функция сравнения**

Можно передать вторым аргументом функцию сравнения:

```jsx
const User = React.memo(
  function User({ user }) {
    return <div>{user.name}</div>;
  },
  // Кастомное сравнение
  (prevProps, nextProps) => {
    // Возвращаем true, если пропсы РАВНЫ (рендер не нужен)
    return prevProps.user.id === nextProps.user.id;
  }
);

// Теперь компонент обновляется только при изменении user.id
// Даже если изменились другие поля в user
```

---

### **Когда использовать**

#### **✅ Хорошие случаи:**

1. **Чистые компоненты отображения**
```jsx
const UserCard = React.memo(({ user }) => (
  <div className="card">
    <h3>{user.name}</h3>
    <p>{user.email}</p>
  </div>
));
```

2. **Часто перерисовываемые родители**
```jsx
function Game() {
  const [score, setScore] = useState(0);
  
  // Часто обновляется
  useEffect(() => {
    const interval = setInterval(() => {
      setScore(s => s + 1);
    }, 16); // 60 FPS
  }, []);
  
  // Статичный UI не перерисовывается
  const Controls = React.memo(() => (
    <div className="controls">
      <button>Pause</button>
      <button>Restart</button>
    </div>
  ));
  
  return (
    <>
      <Score value={score} />
      <Controls /> {/* Не перерисовывается */}
    </>
  );
}
```

3. **Большие списки**
```jsx
const ListItem = React.memo(({ item, onSelect }) => {
  return (
    <li onClick={() => onSelect(item.id)}>
      {item.name}
    </li>
  );
});

function List({ items }) {
  const handleSelect = useCallback((id) => {
    console.log('Selected:', id);
  }, []);
  
  return (
    <ul>
      {items.map(item => (
        <ListItem 
          key={item.id} 
          item={item}
          onSelect={handleSelect} // useCallback обязательно!
        />
      ))}
    </ul>
  );
}
```

---

### **Когда НЕ использовать**

#### **❌ Плохие случаи:**

1. **Компоненты, которые часто меняют пропсы**
```jsx
// ❌ Избыточно
const Counter = React.memo(({ count }) => {
  return <div>{count}</div>;
});

function Parent() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    setInterval(() => setCount(c => c + 1), 1000);
  }, []);
  
  return <Counter count={count} />; // count меняется каждую секунду
}
// React.memo бесполезен, сравнение всегда false
```

2. **Компоненты с объектами/функциями без мемоизации**
```jsx
const Child = React.memo(({ onClick }) => (
  <button onClick={onClick}>Click</button>
));

function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ onClick создаётся заново каждый раз
  const handleClick = () => console.log('click');
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Child onClick={handleClick} /> {/* Всегда перерисовывается! */}
    </>
  );
}
```

3. **Очень простые компоненты**
```jsx
// ❌ Нагрузка от сравнения больше выгоды
const SimpleText = React.memo(({ text }) => <span>{text}</span>);
```

---

### **React.memo + useCallback**

Часто используются вместе:

```jsx
const Button = React.memo(({ onClick, children }) => {
  console.log('Button render');
  return <button onClick={onClick}>{children}</button>;
});

function Dashboard() {
  const [count, setCount] = useState(0);
  
  // ✅ С useCallback
  const handleSave = useCallback(() => {
    console.log('Saving...');
  }, []);
  
  // ❌ Без useCallback (перерисовывает Button)
  // const handleSave = () => console.log('Saving...');
  
  return (
    <div>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <Button onClick={handleSave}>Save</Button>
    </div>
  );
}
// Button не перерисовывается при изменении count
```

---

### **Производительность**

```jsx
// Замер производительности
const ExpensiveComponent = React.memo(
  ({ data }) => {
    // Тяжёлые вычисления
    const result = expensiveCalculation(data);
    return <div>{result}</div>;
  },
  (prev, next) => {
    // Кастомное сравнение только по нужным полям
    return prev.data.id === next.data.id;
  }
);
```

---

### **Ограничения**

1. **Только для пропсов:** Не предотвращает рендер при изменении состояния или контекста внутри компонента
2. **Сравнение пропсов:** Может быть дороже, чем сам рендер для простых компонентов
3. **Не глубокое сравнение:** Для объектов/массивов нужно кастомное сравнение или мемоизация пропсов

---