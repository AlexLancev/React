### **Объяснение**

**useRef** создаёт изменяемый объект с свойством `.current`, который сохраняется между рендерами. Изменение `.current` **не вызывает ререндер**.

---

### **1. Основное: доступ к DOM (знают все)**
```jsx
const inputRef = useRef(null);
<input ref={inputRef} />
inputRef.current.focus(); // Фокус на инпут
```

---

### **2. Что ещё умеет useRef (менее известное)**

#### **А. Хранение изменяемых значений между рендерами**

```jsx
function Timer() {
  const [seconds, setSeconds] = useState(0);
  const intervalRef = useRef(); // Храним ID интервала
  
  useEffect(() => {
    intervalRef.current = setInterval(() => {
      setSeconds(s => s + 1);
    }, 1000);
    
    return () => clearInterval(intervalRef.current);
  }, []);
  
  const stopTimer = () => {
    clearInterval(intervalRef.current); // Останавливаем по ref
  };
  
  return (
    <div>
      Seconds: {seconds}
      <button onClick={stopTimer}>Stop</button>
    </div>
  );
}
```

#### **Б. Сохранение предыдущих значений**
```jsx
function Component() {
  const [count, setCount] = useState(0);
  const prevCountRef = useRef();
  
  useEffect(() => {
    prevCountRef.current = count; // Сохраняем после каждого рендера
  });
  
  return (
    <div>
      <p>Now: {count}, Before: {prevCountRef.current}</p>
      <button onClick={() => setCount(c => c + 1)}>+</button>
    </div>
  );
}
```

#### **В. Хранение флагов/состояний без ререндера**
```jsx
function Form() {
  const [value, setValue] = useState('');
  const isMountedRef = useRef(false); // Флаг монтирования
  
  useEffect(() => {
    isMountedRef.current = true; // Устанавливаем после монтирования
    return () => {
      isMountedRef.current = false; // Сбрасываем при размонтировании
    };
  }, []);
  
  const submit = async () => {
    await fetch('/api');
    if (isMountedRef.current) { // Проверяем, не размонтирован ли компонент
      setValue('Saved');
    }
  };
  
  return <input value={value} onChange={e => setValue(e.target.value)} />;
}
```

#### **Г. Кэширование тяжёлых вычислений (альтернатива useMemo)**
```jsx
function Component({ data }) {
  const cacheRef = useRef(new Map()); // Кэш в ref
  
  const getProcessedData = (id) => {
    if (cacheRef.current.has(id)) {
      return cacheRef.current.get(id); // Возвращаем из кэша
    }
    
    const result = expensiveCalculation(data, id); // Тяжёлое вычисление
    cacheRef.current.set(id, result); // Сохраняем в кэш
    return result;
  };
  
  // Без ререндера при обновлении кэша!
}
```

#### **Д. Измерение интервалов времени**
```jsx
function Stopwatch() {
  const [time, setTime] = useState(0);
  const startTimeRef = useRef();
  
  const start = () => {
    startTimeRef.current = Date.now(); // Запоминаем время старта
    setInterval(() => {
      const elapsed = Date.now() - startTimeRef.current;
      setTime(elapsed);
    }, 10);
  };
  
  return (
    <div>
      Time: {(time / 1000).toFixed(2)}s
      <button onClick={start}>Start</button>
    </div>
  );
}
```

#### **Е. Подсчёт рендеров**
```jsx
function RenderCounter() {
  const renderCount = useRef(0);
  renderCount.current += 1; // Не вызывает ререндер!
  
  return <div>Рендеров: {renderCount.current}</div>;
}
```

#### **Ж. Хранение предыдущих props/state (шаблон)**
```jsx
function usePrevious(value) {
  const ref = useRef();
  
  useEffect(() => {
    ref.current = value;
  });
  
  return ref.current; // Возвращаем предыдущее значение
}

function Component({ value }) {
  const prevValue = usePrevious(value); // Получаем предыдущее значение
  
  return (
    <div>
      Current: {value}
      Previous: {prevValue}
    </div>
  );
}
```

#### **З. Управление анимациями/requestAnimationFrame**
```jsx
function MovingBox() {
  const boxRef = useRef();
  const animationIdRef = useRef();
  
  const animate = () => {
    // Логика анимации
    boxRef.current.style.left = `${Math.random() * 100}px`;
    animationIdRef.current = requestAnimationFrame(animate);
  };
  
  useEffect(() => {
    animationIdRef.current = requestAnimationFrame(animate);
    return () => cancelAnimationFrame(animationIdRef.current);
  }, []);
  
  return <div ref={boxRef} className="box" />;
}
```

#### **И. Интеграция с не-React кодом**
```jsx
function ChartComponent({ data }) {
  const chartRef = useRef();
  const chartInstanceRef = useRef(); // Храним экземпляр чарта
  
  useEffect(() => {
    // Инициализация сторонней библиотеки
    chartInstanceRef.current = new ThirdPartyChart(chartRef.current, {
      data: data,
      // ... опции
    });
    
    return () => {
      // Очистка
      chartInstanceRef.current.destroy();
    };
  }, [data]);
  
  return <div ref={chartRef} />;
}
```

---

### **Ключевые отличия от useState**

| **useRef** | **useState** |
|------------|--------------|
| `ref.current` можно менять напрямую | Только через `setState()` |
| Изменение **не вызывает ререндер** | Изменение **вызывает ререндер** |
| Можно изменять в обработчиках и эффектах | Асинхронные обновления |
| Значение доступно сразу после изменения | Значение доступно на следующем рендере |

```jsx
function Example() {
  const [state, setState] = useState(0); // Вызывает ререндер
  const ref = useRef(0); // Не вызывает ререндер
  
  const handleClick = () => {
    setState(n => n + 1); // ✅ Рендер
    ref.current += 1;     // ❌ Без рендера
  };
}
```

---

### **Когда использовать useRef вместо useState**

1. **Таймеры/интервалы** — храним ID
2. **Ссылки на DOM элементы** — доступ к элементам
3. **Флаги/состояния** которые не должны вызывать ререндер
4. **Кэширование значений** между рендерами
5. **Предыдущие значения** для сравнения
6. **Интеграция со сторонними библиотеками**

---

### **Важное предупреждение**

```jsx
function DangerousComponent() {
  const [state, setState] = useState(0);
  const ref = useRef(0);
  
  useEffect(() => {
    // ❌ ОПАСНО: effect зависит от ref, но ref не в зависимостях
    console.log(ref.current);
  }, []); // ESLint не предупредит!
  
  // Лучше: использовать callback ref
  const callbackRef = useCallback((node) => {
    if (node) {
      console.log('DOM node mounted');
    }
  }, []);
  
  return <div ref={callbackRef} />;
}
```

---

### **Резюме**

**useRef** — это коробка с изменяемым значением `.current`, которое сохраняется между рендерами.

**Ключевые нюансы:**
1. **Без ререндера:** Изменение `.current` не вызывает перерисовку компонента
2. **Ссылочная целостность:** Объект ref один и тот же на всех рендерах
3. **Две основные задачи:** Доступ к DOM и хранение мутабельных значений

**Как сказать своими словами:**
«useRef — это как карман у компонента, куда можно положить любую штуку, и она будет там лежать между перерисовками. Чаще всего туда кладут ссылки на DOM-элементы (чтобы фокусироваться, измерять размеры). Но можно положить и что угодно: ID таймера, предыдущее значение состояния, флаг "компонент жив", кэш вычислений. Главное — изменение содержимого ref не заставляет компонент перерисовываться.»