### **Объяснение**

**Dependency array** (массив зависимостей) — это второй аргумент хуков `useEffect`, `useMemo`, `useCallback`, который определяет, когда должен выполняться эффект или пересчитываться значение.

```jsx
useEffect(() => {
  // эффект
}, [dep1, dep2]); // ← dependency array
```

---

### **Три варианта массива зависимостей**

#### **1. Пустой массив `[]` — один раз при монтировании**
```jsx
useEffect(() => {
  console.log('Выполнится один раз при создании компонента');
  return () => console.log('Очистка при удалении компонента');
}, []); // ← пустой массив
```

**Когда использовать:**
- Запрос данных при загрузке
- Подписка на события
- Таймеры
- Инициализация сторонних библиотек

---

#### **2. Массив с зависимостями `[dep1, dep2]` — при их изменении**
```jsx
function UserProfile({ userId }) {
  const [user, setUser] = useState(null);
  
  useEffect(() => {
    fetchUser(userId).then(setUser);
  }, [userId]); // ← выполнится при изменении userId
  
  return <div>{user?.name}</div>;
}
```

**Как работает:**
- При первом рендере: выполняется
- При последующих рендерах:
  - Если `userId` изменился → выполняется
  - Если `userId` тот же → пропускается

---

#### **3. Без массива — при каждом рендере**
```jsx
useEffect(() => {
  console.log('Выполняется при каждом рендере');
}); // ← нет массива
```

**⚠️ Опасно:** Может вызвать бесконечные циклы или проблемы с производительностью.

---

### **Правила зависимостей**

#### **Все зависимости должны быть указаны**
```jsx
function Component({ id }) {
  const [count, setCount] = useState(0);
  
  // ❌ Пропущена зависимость count
  useEffect(() => {
    console.log(`ID: ${id}, Count: ${count}`);
  }, [id]); // ← должен быть [id, count]
  
  // ✅ Все зависимости указаны
  useEffect(() => {
    console.log(`ID: ${id}, Count: ${count}`);
  }, [id, count]);
}
```

#### **Лишние зависимости тоже проблема**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  const [text, setText] = useState('');
  
  // ❌ text не используется в эффекте
  useEffect(() => {
    console.log(`Count: ${count}`);
  }, [count, text]); // ← лишняя зависимость text
  
  // ✅ Только нужные зависимости
  useEffect(() => {
    console.log(`Count: ${count}`);
  }, [count]);
}
```

---

### **Особые случаи**

#### **Функции как зависимости**
```jsx
function Parent() {
  const [count, setCount] = useState(0);
  
  // ❌ Без useCallback - бесконечный цикл
  const handleClick = () => {
    console.log(count);
  };
  
  useEffect(() => {
    console.log('Effect runs');
  }, [handleClick]); // ← handleClick новая при каждом рендере
  
  // ✅ С useCallback
  const stableHandleClick = useCallback(() => {
    console.log(count);
  }, [count]);
  
  useEffect(() => {
    console.log('Effect runs only when count changes');
  }, [stableHandleClick]);
}
```

#### **Массивы/объекты как зависимости**
```jsx
function Component() {
  const [items, setItems] = useState([]);
  
  // ❌ items всегда "новая" ссылка, даже если содержимое то же
  useEffect(() => {
    console.log('Items changed');
  }, [items]); // ← срабатывает при каждом рендере
  
  // ✅ Использовать примитивы или мемоизацию
  const itemCount = items.length;
  useEffect(() => {
    console.log('Item count changed');
  }, [itemCount]); // ← примитив работает правильно
}
```

---

### **ESLint правило**

React ESLint плагин автоматически проверяет зависимости:

```jsx
useEffect(() => {
  console.log(id, name);
}, []); // ← ESLint предупредит: "React Hook useEffect has missing dependencies: 'id' and 'name'"
```

**Отключать проверку только в крайних случаях:**
```jsx
useEffect(() => {
  // эффект
}, []); // eslint-disable-line react-hooks/exhaustive-deps
```

---

### **Примеры**

#### **Запрос данных с зависимостями**
```jsx
function UserPosts({ userId, shouldFetch }) {
  const [posts, setPosts] = useState([]);
  
  useEffect(() => {
    if (shouldFetch) {
      fetchPosts(userId).then(setPosts);
    }
  }, [userId, shouldFetch]); // Зависит от userId и shouldFetch
  
  return posts.map(post => <div key={post.id}>{post.title}</div>);
}
```

#### **Подписка/отписка**
```jsx
function Chat({ roomId }) {
  useEffect(() => {
    const connection = createConnection(roomId);
    connection.connect();
    
    return () => {
      connection.disconnect(); // Очистка при изменении roomId или размонтировании
    };
  }, [roomId]); // Эффект перезапускается при смене roomId
}
```

#### **Синхронизация с localStorage**
```jsx
function useLocalStorage(key, initialValue) {
  const [value, setValue] = useState(() => {
    const stored = localStorage.getItem(key);
    return stored ? JSON.parse(stored) : initialValue;
  });
  
  useEffect(() => {
    localStorage.setItem(key, JSON.stringify(value));
  }, [key, value]); // При изменении key или value
  
  return [value, setValue];
}
```

---

### **Резюме**

**Dependency array** — массив значений, от которых зависит выполнение эффекта или пересчёт мемоизированного значения.

**Ключевые нюансы:**
1. **Три режима:** `[]` (один раз), `[deps]` (при изменении deps), без массива (каждый рендер)
2. **Все зависимости:** Должны включать все значения из области видимости, используемые внутри эффекта
3. **Ссылочное равенство:** Для объектов, массивов, функций важно ссылочное равенство, а не глубокое сравнение

**Как сказать своими словами:**
«Массив зависимостей — это список "триггеров" для хуков. В useEffect он говорит: "выполняй этот код, только когда вот эти значения поменялись". В useMemo/useCallback: "пересчитывай значение/функцию, только когда эти зависимости изменились". Если массив пустой — код выполнится один раз при монтировании. Если не указать — будет выполняться при каждом рендере.»