### **Объяснение с примерами**

**Жизненный цикл компонента** — это последовательность методов, которые вызываются React в процессе создания, обновления и удаления компонента. Эти методы позволяют выполнять код в определённые моменты "жизни" компонента.

---

#### **1. Три основные фазы жизненного цикла**

```jsx
class LifecycleDemo extends React.Component {
  // Фаза 1: Монтирование (создание и вставка в DOM)
  // Фаза 2: Обновление (изменение props или state)
  // Фаза 3: Размонтирование (удаление из DOM)
}
```

---

#### **2. Фаза монтирования (Mounting)**

Компонент создаётся и вставляется в DOM впервые.

```jsx
class MountingPhase extends React.Component {
  // 1. Конструктор (вызывается первым)
  constructor(props) {
    super(props);
    console.log('1. constructor()');
    this.state = { count: 0 };
    this.timer = null;
  }

  // 2. static getDerivedStateFromProps() (редко используется)
  static getDerivedStateFromProps(nextProps, prevState) {
    console.log('2. getDerivedStateFromProps()');
    // Используется для обновления state на основе props
    if (nextProps.initialCount !== prevState.count) {
      return { count: nextProps.initialCount };
    }
    return null;
  }

  // 3. render() (обязательный метод)
  render() {
    console.log('3. render()');
    return (
      <div>
        <h1>Счет: {this.state.count}</h1>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Увеличить
        </button>
      </div>
    );
  }

  // 4. componentDidMount() (вызывается после вставки в DOM)
  componentDidMount() {
    console.log('4. componentDidMount()');
    // Идеальное место для:
    // - Запросов к API
    // - Подписки на события
    // - Работы с DOM
    // - Таймеров
    
    this.timer = setInterval(() => {
      console.log('Таймер работает...');
    }, 1000);
    
    // Пример запроса данных
    fetch('/api/data')
      .then(response => response.json())
      .then(data => this.setState({ data }));
  }
}
```

**Последовательность при монтировании:**
```
constructor() → getDerivedStateFromProps() → render() → componentDidMount()
```

---

#### **3. Фаза обновления (Updating)**

Компонент обновляется при изменении props или state.

```jsx
class UpdatingPhase extends React.Component {
  state = { count: 0, data: null };
  
  // 1. static getDerivedStateFromProps() (при обновлении тоже)
  static getDerivedStateFromProps(nextProps, prevState) {
    console.log('getDerivedStateFromProps() при обновлении');
    return null;
  }
  
  // 2. shouldComponentUpdate() (оптимизация производительности)
  shouldComponentUpdate(nextProps, nextState) {
    console.log('shouldComponentUpdate()');
    // Решаем, нужно ли перерисовывать компонент
    // По умолчанию: true (всегда перерисовывать)
    
    // Пример оптимизации: перерисовывать только если count изменился
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
    
    // Или используйте PureComponent для поверхностного сравнения
  }
  
  // 3. render() (перерисовка)
  render() {
    console.log('render() при обновлении');
    return (
      <div>
        <h1>Счет: {this.state.count}</h1>
        <button onClick={this.handleClick}>Увеличить</button>
      </div>
    );
  }
  
  handleClick = () => {
    this.setState({ count: this.state.count + 1 });
  };
  
  // 4. getSnapshotBeforeUpdate() (редко используется)
  getSnapshotBeforeUpdate(prevProps, prevState) {
    console.log('getSnapshotBeforeUpdate()');
    // Позволяет захватить информацию из DOM перед обновлением
    // Например, позицию скролла
    
    const list = document.getElementById('messages');
    if (list) {
      return list.scrollHeight;
    }
    return null;
  }
  
  // 5. componentDidUpdate() (после обновления DOM)
  componentDidUpdate(prevProps, prevState, snapshot) {
    console.log('componentDidUpdate()');
    
    // Идеальное место для:
    // - Дополнительных запросов к API (при изменении props)
    // - Работы с DOM после обновления
    // - Сравнения текущих и предыдущих props/state
    
    // Пример: запрос данных при изменении userId в props
    if (this.props.userId !== prevProps.userId) {
      this.fetchUserData(this.props.userId);
    }
    
    // Пример использования snapshot
    if (snapshot !== null) {
      const list = document.getElementById('messages');
      list.scrollTop = list.scrollHeight - snapshot;
    }
  }
  
  fetchUserData = (userId) => {
    // Запрос данных пользователя
  };
}
```

**Последовательность при обновлении:**
```
getDerivedStateFromProps() → shouldComponentUpdate() → render() → 
getSnapshotBeforeUpdate() → componentDidUpdate()
```

---

#### **4. Фаза размонтирования (Unmounting)**

Компонент удаляется из DOM.

```jsx
class UnmountingPhase extends React.Component {
  state = { isVisible: true };
  
  componentDidMount() {
    this.timer = setInterval(() => {
      console.log('Таймер тикает...');
    }, 1000);
    
    this.subscription = eventBus.subscribe('event', this.handleEvent);
  }
  
  // 1. componentWillUnmount() (очистка ресурсов)
  componentWillUnmount() {
    console.log('componentWillUnmount()');
    
    // Обязательно очищаем:
    // - Таймеры
    // - Подписки на события
    // - Запросы к API
    // - Любые сторонние библиотеки
    
    clearInterval(this.timer);
    this.subscription.unsubscribe();
    
    // Примеры очистки:
    window.removeEventListener('resize', this.handleResize);
    document.removeEventListener('click', this.handleClick);
    this.websocket.close();
  }
  
  toggleVisibility = () => {
    this.setState(prevState => ({
      isVisible: !prevState.isVisible
    }));
  };
  
  render() {
    return (
      <div>
        <button onClick={this.toggleVisibility}>
          {this.state.isVisible ? 'Скрыть' : 'Показать'}
        </button>
        {this.state.isVisible && <ChildComponent />}
      </div>
    );
  }
}

class ChildComponent extends React.Component {
  componentWillUnmount() {
    console.log('ChildComponent будет удален');
    // Очистка ресурсов этого компонента
  }
  
  render() {
    return <div>Дочерний компонент</div>;
  }
}
```

---

#### **5. Обработка ошибок (Error Boundaries)**

```jsx
class ErrorBoundary extends React.Component {
  state = { hasError: false, error: null };
  
  // 1. static getDerivedStateFromError() (при ошибке в дочерних)
  static getDerivedStateFromError(error) {
    console.log('getDerivedStateFromError()');
    // Обновляем state для отображения запасного UI
    return { hasError: true, error };
  }
  
  // 2. componentDidCatch() (логирование ошибок)
  componentDidCatch(error, errorInfo) {
    console.log('componentDidCatch()');
    // Логируем ошибку в сервис (Sentry, LogRocket и т.д.)
    logErrorToService(error, errorInfo);
  }
  
  render() {
    if (this.state.hasError) {
      // Запасной UI
      return (
        <div>
          <h1>Что-то пошло не так</h1>
          <p>{this.state.error.message}</p>
          <button onClick={() => this.setState({ hasError: false })}>
            Попробовать снова
          </button>
        </div>
      );
    }
    
    return this.props.children;
  }
}

// Использование
function App() {
  return (
    <ErrorBoundary>
      <BuggyComponent />
    </ErrorBoundary>
  );
}

class BuggyComponent extends React.Component {
  render() {
    // Искусственная ошибка
    throw new Error('Тестовая ошибка!');
    return <div>Этот компонент сломан</div>;
  }
}
```

---

#### **6. Полная диаграмма жизненного цикла**

```
МОНТИРОВАНИЕ:
constructor()
↓
getDerivedStateFromProps()
↓
render()
↓
componentDidMount()

ОБНОВЛЕНИЕ (при изменении props/state):
getDerivedStateFromProps()
↓
shouldComponentUpdate()
↓
render()
↓
getSnapshotBeforeUpdate()
↓
componentDidUpdate()

ОБНОВЛЕНИЕ (при изменении только props):
getDerivedStateFromProps()
↓
shouldComponentUpdate()
↓
render()
↓
getSnapshotBeforeUpdate()
↓
componentDidUpdate()

РАЗМОНТИРОВАНИЕ:
componentWillUnmount()

ОБРАБОТКА ОШИБОК:
getDerivedStateFromError()
↓
componentDidCatch()
```

---

#### **7. Сравнение с функциональными компонентами и хуками**

```jsx
// Классовый компонент
class ClassComponent extends React.Component {
  componentDidMount() {
    console.log('Компонент смонтирован');
  }
  
  componentDidUpdate() {
    console.log('Компонент обновился');
  }
  
  componentWillUnmount() {
    console.log('Компонент будет размонтирован');
  }
  
  render() { return <div>Класс</div>; }
}

// Функциональный компонент с хуками
function FunctionalComponent() {
  // useEffect заменяет все три метода жизненного цикла
  
  // Аналог componentDidMount
  useEffect(() => {
    console.log('Компонент смонтирован');
    
    // Аналог componentWillUnmount (функция очистки)
    return () => {
      console.log('Компонент будет размонтирован');
    };
  }, []); // Пустой массив зависимостей = только при монтировании
  
  // Аналог componentDidUpdate для конкретных значений
  const [count, setCount] = useState(0);
  
  useEffect(() => {
    console.log('Count изменился:', count);
    // Выполняется при каждом изменении count
  }, [count]); // Зависит от count
  
  // Аналог componentDidUpdate (без зависимостей)
  useEffect(() => {
    console.log('Компонент обновился (любое изменение)');
    // Выполняется при КАЖДОМ рендере
  }); // Без массива зависимостей
  
  return <div>Функциональный</div>;
}
```

**Сравнение методов жизненного цикла и хуков:**
- `componentDidMount` → `useEffect(() => {}, [])`
- `componentDidUpdate` → `useEffect(() => {}, [dependencies])`
- `componentWillUnmount` → `return () => {}` в `useEffect`
- `shouldComponentUpdate` → `React.memo()` + `useMemo`/`useCallback`

---

#### **8. Практический пример: компонент с данными**

```jsx
class UserProfile extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      user: null,
      loading: true,
      error: null
    };
  }
  
  componentDidMount() {
    this.fetchUserData(this.props.userId);
  }
  
  componentDidUpdate(prevProps) {
    // Если userId изменился, загружаем новые данные
    if (this.props.userId !== prevProps.userId) {
      this.fetchUserData(this.props.userId);
    }
  }
  
  componentWillUnmount() {
    // Отменяем текущий запрос при размонтировании
    if (this.request) {
      this.request.cancel();
    }
  }
  
  fetchUserData = async (userId) => {
    this.setState({ loading: true, error: null });
    
    try {
      const response = await fetch(`/api/users/${userId}`);
      const user = await response.json();
      this.setState({ user, loading: false });
    } catch (error) {
      this.setState({ error: error.message, loading: false });
    }
  };
  
  shouldComponentUpdate(nextProps, nextState) {
    // Оптимизация: не перерисовывать если данные те же
    if (this.state.loading && nextState.loading) return false;
    return true;
  }
  
  render() {
    const { user, loading, error } = this.state;
    
    if (loading) return <div>Загрузка...</div>;
    if (error) return <div>Ошибка: {error}</div>;
    if (!user) return <div>Пользователь не найден</div>;
    
    return (
      <div>
        <h1>{user.name}</h1>
        <p>Email: {user.email}</p>
        <p>Город: {user.city}</p>
      </div>
    );
  }
}
```

---

### **Резюме**

Жизненный цикл компонента — это набор методов, которые React вызывает в определённые моменты существования компонента: при создании (монтировании), обновлении и удалении (размонтировании).

**Ключевые нюансы:**
1. **Три основные фазы:** Монтирование (создание), обновление (изменение props/state) и размонтирование (удаление). Каждая фаза имеет свои методы, которые вызываются в строгом порядке.
2. **Ключевые методы:** `componentDidMount` (идеально для начальной загрузки данных), `componentDidUpdate` (для реакций на изменения), `componentWillUnmount` (для очистки ресурсов). `shouldComponentUpdate` используется для оптимизации производительности.
3. **Современный подход:** В функциональных компонентах методы жизненного цикла заменены хуком `useEffect`, который объединяет логику `componentDidMount`, `componentDidUpdate` и `componentWillUnmount` в одну декларативную API.

**Выжимка по объяснению:**
Жизненный цикл React-компонента состоит из трёх этапов: монтирование (компонент создаётся и добавляется в DOM), обновление (компонент перерисовывается при изменении props или state) и размонтирование (компонент удаляется из DOM). На каждом этапе вызываются соответствующие методы, позволяющие выполнять код в нужный момент: инициализировать данные, обновлять UI, очищать ресурсы. В современных функциональных компонентах эта логика реализуется через хук useEffect.

**Как сказать своими словами кратко:**
>«Жизненный цикл — это как история жизни компонента: рождение (монтирование), взросление (обновление) и смерть (размонтирование). При рождении (componentDidMount) компонент загружает данные, при изменениях (componentDidUpdate) реагирует на новые props/state, а перед смертью (componentWillUnmount) убирает за собой (очищает таймеры, подписки). Сейчас вместо этих методов используют хук useEffect, который делает то же самое, но проще.»