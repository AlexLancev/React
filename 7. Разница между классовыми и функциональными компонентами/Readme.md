### **Объяснение с примерами**

Разница между классовыми и функциональными компонентами охватывает синтаксис, работу с состоянием, жизненным циклом и производительностью. Вот детальное сравнение:

---

#### **1. Синтаксис и объявление**

**Функциональный компонент:**
```jsx
// Просто функция JavaScript
function Welcome(props) {
  return <h1>Привет, {props.name}</h1>;
}

// Стрелочная функция
const Welcome = (props) => {
  return <h1>Привет, {props.name}</h1>;
};
```

**Классовый компонент:**
```jsx
import React, { Component } from 'react';

class Welcome extends Component {
  render() {
    return <h1>Привет, {this.props.name}</h1>;
  }
}
```

**Ключевое различие:** Функциональные компоненты — это просто функции, классовые — классы ES6, расширяющие `React.Component`.

---

#### **2. Работа с состоянием (State)**

**До появления хуков (React < 16.8):**
```jsx
// Функциональный компонент БЕЗ состояния (только props)
function StatelessComponent(props) {
  return <div>{props.message}</div>;
}

// Классовый компонент СО состоянием
class StatefulComponent extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  render() {
    return <div>Счет: {this.state.count}</div>;
  }
}
```

**После появления хуков (React 16.8+):**
```jsx
// Функциональный компонент СО состоянием (через useState)
function StatefulFunctionalComponent() {
  const [count, setCount] = useState(0);
  
  return (
    <div>
      Счет: {count}
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
    </div>
  );
}

// Классовый компонент (без изменений)
class StatefulClassComponent extends Component {
  state = { count: 0 };
  
  render() {
    return (
      <div>
        Счет: {this.state.count}
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Увеличить
        </button>
      </div>
    );
  }
}
```

**Разница:** В функциональных компонентах состояние — это отдельные переменные через `useState`, в классовых — единый объект `this.state`.

---

#### **3. Методы жизненного цикла vs Хуки**

**Классовый компонент (методы жизненного цикла):**
```jsx
class LifecycleComponent extends Component {
  componentDidMount() {
    // Выполняется после первого рендера
    console.log('Компонент смонтирован');
    this.timer = setInterval(() => {
      this.setState({ time: new Date() });
    }, 1000);
  }
  
  componentDidUpdate(prevProps, prevState) {
    // Выполняется после обновления
    if (this.state.count !== prevState.count) {
      console.log('Count изменился');
    }
  }
  
  componentWillUnmount() {
    // Выполняется перед удалением
    clearInterval(this.timer);
  }
  
  render() { /* ... */ }
}
```

**Функциональный компонент (хуки):**
```jsx
function LifecycleFunctionalComponent() {
  const [time, setTime] = useState(new Date());
  
  useEffect(() => {
    // Аналог componentDidMount
    console.log('Компонент смонтирован');
    const timer = setInterval(() => {
      setTime(new Date());
    }, 1000);
    
    // Возвращаемая функция — аналог componentWillUnmount
    return () => {
      clearInterval(timer);
      console.log('Компонент будет размонтирован');
    };
  }, []); // Пустой массив зависимостей = только при монтировании
  
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    // Аналог componentDidUpdate для count
    console.log('Count изменился:', count);
  }, [count]); // Зависимость от count
  
  return <div>Время: {time.toLocaleTimeString()}</div>;
}
```

**Разница:** Классовые компоненты используют отдельные методы для разных этапов жизненного цикла, функциональные — комбинируют логику в `useEffect`.

---

#### **4. Работа с this и привязка контекста**

**Проблема в классовых компонентах:**
```jsx
class ProblemComponent extends Component {
  state = { count: 0 };
  
  handleClick() {
    // ОШИБКА: this будет undefined в обработчике события
    this.setState({ count: this.state.count + 1 });
  }
  
  // Решение 1: Привязка в конструкторе
  constructor(props) {
    super(props);
    this.handleClick = this.handleClick.bind(this);
  }
  
  // Решение 2: Стрелочная функция как свойство класса
  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  render() {
    return <button onClick={this.handleClick}>Нажать</button>;
  }
}
```

**В функциональных компонентах этой проблемы нет:**
```jsx
function NoProblemComponent() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    // Нет this, контекст не теряется
    setCount(count + 1);
  };
  
  return <button onClick={handleClick}>Нажать</button>;
}
```

---

#### **5. Производительность и оптимизация**

**Классовые компоненты:**
```jsx
// PureComponent для предотвращения лишних рендеров
class OptimizedClassComponent extends React.PureComponent {
  // Автоматически реализует shouldComponentUpdate
  // с поверхностным сравнением props и state
}

// Или вручную
class ManualOptimization extends Component {
  shouldComponentUpdate(nextProps, nextState) {
    // Ручная оптимизация
    return nextProps.value !== this.props.value;
  }
}
```

**Функциональные компоненты:**
```jsx
// React.memo для мемоизации
const OptimizedFunctionalComponent = React.memo(
  function MyComponent(props) {
    return <div>{props.value}</div>;
  },
  // Необязательная функция сравнения
  (prevProps, nextProps) => {
    return prevProps.value === nextProps.value;
  }
);

// useMemo и useCallback для оптимизации внутри компонента
function ComponentWithOptimization({ items }) {
  const computedValue = useMemo(() => {
    // Тяжелые вычисления
    return items.reduce((sum, item) => sum + item.value, 0);
  }, [items]); // Пересчитывается только при изменении items
  
  const handleClick = useCallback(() => {
    console.log('Clicked');
  }, []); // Создается один раз
}
```

---

#### **6. Работа с рефами (Refs)**

**Классовые компоненты:**
```jsx
class RefClassComponent extends Component {
  constructor(props) {
    super(props);
    this.myRef = React.createRef();
  }
  
  focusInput = () => {
    this.myRef.current.focus();
  };
  
  render() {
    return <input ref={this.myRef} />;
  }
}
```

**Функциональные компоненты:**
```jsx
function RefFunctionalComponent() {
  const myRef = useRef(null);
  
  const focusInput = () => {
    myRef.current.focus();
  };
  
  return <input ref={myRef} />;
}
```

---

#### **7. Полное сравнение на примере счетчика**

```jsx
// Классовый компонент
class CounterClass extends Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  componentDidMount() {
    document.title = `Счет: ${this.state.count}`;
  }
  
  componentDidUpdate() {
    document.title = `Счет: ${this.state.count}`;
  }
  
  increment = () => {
    this.setState(prevState => ({
      count: prevState.count + 1
    }));
  };
  
  render() {
    return (
      <div>
        <p>Счет: {this.state.count}</p>
        <button onClick={this.increment}>Увеличить</button>
      </div>
    );
  }
}

// Функциональный компонент
function CounterFunctional() {
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    document.title = `Счет: ${count}`;
  }, [count]); // Зависит от count
  
  const increment = () => {
    setCount(prevCount => prevCount + 1);
  };
  
  return (
    <div>
      <p>Счет: {count}</p>
      <button onClick={increment}>Увеличить</button>
    </div>
  );
}
```

---

### **Резюме**

Основные различия между классовыми и функциональными компонентами лежат в синтаксисе, работе с состоянием, жизненным циклом и подходах к оптимизации.

**Ключевые нюансы:**
1. **Синтаксис и подход:** Классовые компоненты — это классы ES6 с методами жизненного цикла и состоянием в `this.state`. Функциональные компоненты — простые функции, использующие хуки (`useState`, `useEffect`) для тех же возможностей.
2. **Жизненный цикл vs Эффекты:** В классовых компонентах логика жизненного цикла разделена по методам (`componentDidMount`, `componentDidUpdate`, `componentWillUnmount`). В функциональных компонентах вся логика побочных эффектов объединяется в `useEffect`, что позволяет группировать связанный код.
3. **Проблема с `this` и оптимизация:** Классовые компоненты требуют привязки контекста и использования `PureComponent` или `shouldComponentUpdate` для оптимизации. Функциональные компоненты не имеют проблемы с `this` и используют `React.memo`, `useMemo`, `useCallback` для оптимизации рендеров.

**Выжимка по объяснению:**
Классовые компоненты — это унаследованный подход, использующий классы ES6, методы жизненного цикла и состояние через `this.state`. Функциональные компоненты — современный стандарт, представляющий собой функции, которые с помощью хуков (`useState`, `useEffect`) получили все возможности классовых компонентов, но с более чистым и компактным синтаксисом, отсутствием проблем с контекстом `this` и лучшей композиционностью логики.

**Как сказать своими словами кратко:**
>«Раньше для состояния и жизненного цикла нужны были классовые компоненты (это классы с методами вроде componentDidMount). Сейчас есть функциональные компоненты (просто функции) + хуки (useState, useEffect), которые делают то же самое, но код получается чище и без проблем с "this". Сейчас все пишут на функциональных компонентах, классовые встречаются только в старых проектах.»