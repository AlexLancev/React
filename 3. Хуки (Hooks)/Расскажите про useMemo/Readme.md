### **Объяснение**

**useMemo** — хук для мемоизации (кэширования) вычисляемых значений. Оптимизирует производительность, предотвращая повторные тяжёлые вычисления при каждом рендере.

```jsx
const memoizedValue = useMemo(() => computeExpensiveValue(a, b), [a, b]);
```

---

### **Как работает**

#### **Без useMemo (плохо)**
```jsx
function Component({ items }) {
  // ❌ Вычисляется при каждом рендере, даже если items не изменились
  const sortedItems = items.sort((a, b) => a.price - b.price);
  
  return <ItemList items={sortedItems} />;
}
```

#### **С useMemo (хорошо)**
```jsx
function Component({ items }) {
  // ✅ Вычисляется только при изменении items
  const sortedItems = useMemo(() => {
    return items.sort((a, b) => a.price - b.price);
  }, [items]); // Зависимость: items
  
  return <ItemList items={sortedItems} />;
}
```

---

### **Когда использовать**

#### **✅ Тяжёлые вычисления**
```jsx
function ProductList({ products, filter }) {
  const filteredProducts = useMemo(() => {
    console.log('Фильтрация...'); // Выполняется только при изменении products или filter
    return products.filter(p => p.category === filter);
  }, [products, filter]);
  
  return filteredProducts.map(p => <Product key={p.id} product={p} />);
}
```

#### **✅ Сложные преобразования данных**
```jsx
function Analytics({ data }) {
  const statistics = useMemo(() => {
    return {
      total: data.reduce((sum, item) => sum + item.value, 0),
      average: data.reduce((sum, item) => sum + item.value, 0) / data.length,
      max: Math.max(...data.map(item => item.value))
    };
  }, [data]);
  
  return <StatsDisplay stats={statistics} />;
}
```

#### **✅ Оптимизация дочерних компонентов**
```jsx
function Parent({ user }) {
  const config = useMemo(() => ({
    theme: user.preferredTheme,
    language: user.language,
    currency: user.currency
  }), [user.preferredTheme, user.language, user.currency]);
  
  // ChildComponent не перерендерится, пока config не изменится
  return <ChildComponent config={config} />;
}
```

---

### **Когда НЕ использовать**

#### **❌ Простые вычисления**
```jsx
// Избыточно
const total = useMemo(() => a + b, [a, b]); // ❌ Сложение дешёвое

// Лучше
const total = a + b; // ✅
```

#### **❌ Неправильные зависимости**
```jsx
const value = useMemo(() => {
  return compute(a);
}, [b]); // ❌ Зависит от a, но указан b - кэш не работает
```

---

### **Отличие от useCallback**

```jsx
// useMemo — для значений
const memoizedValue = useMemo(() => compute(a), [a]);

// useCallback — для функций
const memoizedFunction = useCallback(() => doSomething(a), [a]);

// Эквивалентно:
const memoizedFunction = useMemo(() => () => doSomething(a), [a]);
```

---

### **Пример с рекурсивными вычислениями**

```jsx
function Fibonacci({ n }) {
  const fib = useMemo(() => {
    function calculateFib(num) {
      if (num <= 1) return num;
      return calculateFib(num - 1) + calculateFib(num - 2);
    }
    return calculateFib(n);
  }, [n]); // Вычисляется только при изменении n
  
  return <div>Fibonacci({n}) = {fib}</div>;
}
```

---

### **Резюме**

**useMemo** — хук для кэширования результатов дорогостоящих вычислений.

**Ключевые нюансы:**
1. **Кэширование значений:** Возвращает мемоизированное значение, а не функцию
2. **Зависимости:** Пересчитывается только при изменении указанных зависимостей
3. **Не гарантия:** React может сбросить кэш для освобождения памяти

**Как сказать своими словами:**
«useMemo — это кэш для вычислений. Если есть тяжёлая операция (сортировка, фильтрация, сложные математические вычисления), которая зависит от каких-то данных, useMemo запомнит результат и будет возвращать его, пока зависимости не изменятся. Используй, только когда вычисления действительно дорогие, иначе накладные расходы на мемоизацию могут быть больше выгоды.»