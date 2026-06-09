### **Объяснение**

**useMemo** кэширует **результат вычислений** (значение).  
**useCallback** кэширует **саму функцию** (ссылку на функцию).

---

### **Простая аналогия**

```javascript
// useMemo — запоминает РЕЗУЛЬТАТ функции
const result = useMemo(() => calculate(10, 20), [10, 20]);
// result = 30 (число)

// useCallback — запоминает САМУ ФУНКЦИЮ
const func = useCallback(() => calculate(10, 20), [10, 20]);
// func = function() { return calculate(10, 20); } (функция)
```

---

### **Наглядный пример**

#### **useMemo (для значений)**
```jsx
function Component({ items }) {
  // ✅ Кэшируется РЕЗУЛЬТАТ сортировки (массив)
  const sortedItems = useMemo(() => {
    console.log('Сортировка...');
    return [...items].sort((a, b) => a.price - b.price);
  }, [items]);
  
  return <ItemList items={sortedItems} />;
}
// sortedItems = [{...}, {...}] // массив объектов
```

#### **useCallback (для функций)**
```jsx
function Component({ onFilter }) {
  // ✅ Кэшируется САМА ФУНКЦИЯ обработки
  const filterHandler = useCallback((item) => {
    return item.price > 100;
  }, []);
  
  return <Filter onFilter={filterHandler} />;
}
// filterHandler = function(item) { ... } // функция
```

---

### **Техническая разница**

```jsx
// ЭТО ЖЕ САМОЕ:
const memoizedValue = useMemo(() => () => doSomething(a), [a]);

// И ЭТО:
const memoizedCallback = useCallback(() => doSomething(a), [a]);
```

**useCallback — это синтаксический сахар для useMemo, когда нужно кэшировать функцию.**

---

### **Когда что использовать**

#### **✅ useMemo — для дорогих вычислений**
```jsx
// Тяжёлые вычисления
const statistics = useMemo(() => ({
  total: data.reduce((sum, x) => sum + x.value, 0),
  average: data.reduce((sum, x) => sum + x.value, 0) / data.length,
}), [data]);

// Преобразования данных
const formattedData = useMemo(() => 
  data.map(item => ({ ...item, date: new Date(item.timestamp) }))
, [data]);
```

#### **✅ useCallback — для функций, передаваемых детям**
```jsx
// Обработчики событий
const handleClick = useCallback(() => {
  console.log('Clicked', id);
}, [id]);

// Функции для дочерних компонентов
const renderItem = useCallback((item) => (
  <ListItem item={item} />
), []);

// Зависимости эффектов
const fetchData = useCallback(async () => {
  const result = await api.get(id);
  setData(result);
}, [id]);

useEffect(() => {
  fetchData();
}, [fetchData]); // Без useCallback — бесконечный цикл
```

---

### **Пример с React.memo**

```jsx
// Дочерний компонент
const ExpensiveChild = React.memo(({ onClick, data }) => {
  console.log('Child renders');
  return <button onClick={onClick}>{data.value}</button>;
});

function Parent() {
  const [count, setCount] = useState(0);
  const [data] = useState({ value: 'test' });
  
  // ❌ Без useCallback — child перерисовывается при каждом увеличении count
  // const handleClick = () => console.log('click');
  
  // ✅ С useCallback — child не перерисовывается
  const handleClick = useCallback(() => console.log('click'), []);
  
  // ✅ useMemo для данных
  const memoizedData = useMemo(() => data, []);
  
  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        Count: {count}
      </button>
      <ExpensiveChild 
        onClick={handleClick}      // useCallback
        data={memoizedData}        // useMemo
      />
    </>
  );
}
// При клике на кнопку Count child не перерисовывается
```

---

### **Что НЕ нужно кэшировать**

```jsx
// ❌ Избыточно (простое вычисление)
const total = useMemo(() => price * quantity, [price, quantity]);
// ✅ Лучше
const total = price * quantity;

// ❌ Избыточно (нет React.memo детей)
const handleClick = useCallback(() => {}, []);
// ✅ Лучше
const handleClick = () => {};
```

---

### **Сравнительная таблица**

| Аспект | **useMemo** | **useCallback** |
|--------|-------------|-----------------|
| **Что кэширует** | Результат вычислений | Саму функцию |
| **Возвращает** | Значение (число, массив, объект) | Функцию |
| **Основное применение** | Тяжёлые вычисления, преобразования данных | Обработчики событий, пропсы для React.memo |
| **Пример зависимости** | `[data, filter]` | `[id, onSuccess]` |
| **Эквивалент** | `useMemo(() => compute())` | `useMemo(() => () => compute())` |

---

### **Резюме**

**useMemo** — «запомни результат этой функции».  
**useCallback** — «запомни саму эту функцию».

**Ключевые нюансы:**
1. **useMemo** возвращает вычисленное значение, **useCallback** возвращает функцию
2. **useMemo** используется для оптимизации дорогих вычислений, **useCallback** — для сохранения ссылочной целостности функций
3. **Оба** требуют указания зависимостей и пересчитываются при их изменении

**Как сказать своими словами:**
«useMemo кэширует то, что функция ВОЗВРАЩАЕТ (например, отфильтрованный список). useCallback кэширует саму функцию (например, обработчик клика). Если нужно, чтобы дочерний компонент с React.memo не перерисовывался — используй useCallback для функций. Если нужно избежать повторных тяжёлых вычислений — используй useMemo.»