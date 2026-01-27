### **Объяснение с примерами**

**Состояние (state)** — это внутренние данные компонента, которые могут изменяться со временем в ответ на действия пользователя, ответы сервера или другие события. В отличие от props, состояние контролируется и обновляется самим компонентом.

---

#### **1. Базовый пример состояния**

```jsx
// Функциональный компонент с состоянием
function Counter() {
  // Объявляем состояние: count - текущее значение, setCount - функция для обновления
  const [count, setCount] = useState(0);
  
  return (
    <div>
      <p>Счетчик: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        Увеличить
      </button>
      <button onClick={() => setCount(count - 1)}>
        Уменьшить
      </button>
    </div>
  );
}

// Классовый компонент с состоянием
class CounterClass extends React.Component {
  constructor(props) {
    super(props);
    this.state = { count: 0 };
  }
  
  render() {
    return (
      <div>
        <p>Счетчик: {this.state.count}</p>
        <button onClick={() => this.setState({ count: this.state.count + 1 })}>
          Увеличить
        </button>
      </div>
    );
  }
}
```

---

#### **2. Основные отличия State от Props**

##### **A. Кто контролирует данные**
```jsx
// PROPS: Контролируются родительским компонентом
function ChildComponent({ message }) {
  // message приходит извне, компонент НЕ может его изменить
  return <p>{message}</p>;
}

function ParentComponent() {
  const [text, setText] = useState('Привет');
  
  // Только ParentComponent может изменить message для ChildComponent
  return (
    <div>
      <ChildComponent message={text} />
      <button onClick={() => setText('Новый текст')}>
        Изменить текст
      </button>
    </div>
  );
}

// STATE: Контролируется самим компонентом
function StatefulComponent() {
  const [count, setCount] = useState(0); // Полный контроль внутри компонента
  
  return (
    <div>
      <p>Счет: {count}</p>
      <button onClick={() => setCount(count + 1)}>
        // Только этот компонент решает, как изменить count
        Увеличить
      </button>
    </div>
  );
}
```

##### **B. Изменяемость (Mutability)**
```jsx
// PROPS: НЕИЗМЕНЯЕМЫ (immutable)
function UserProfile({ user }) {
  // ❌ НЕЛЬЗЯ изменять props
  // user.name = 'Новое имя'; // Ошибка!
  
  // ✅ Можно только читать
  return <h1>{user.name}</h1>;
}

// STATE: ИЗМЕНЯЕМО (mutable) через специальные функции
function UserForm() {
  const [formData, setFormData] = useState({ name: '', email: '' });
  
  const handleChange = (e) => {
    // ✅ Можно изменять через setFormData
    setFormData({
      ...formData,
      [e.target.name]: e.target.value
    });
  };
  
  return (
    <form>
      <input 
        name="name"
        value={formData.name}
        onChange={handleChange}
      />
    </form>
  );
}
```

##### **C. Назначение и использование**
```jsx
// PROPS: Для конфигурации и передачи данных СВЕРХУ ВНИЗ
function App() {
  const users = [
    { id: 1, name: 'Анна' },
    { id: 2, name: 'Петр' }
  ];
  
  return (
    <div>
      {/* users передаются как props для настройки UserList */}
      <UserList users={users} title="Список пользователей" />
    </div>
  );
}

function UserList({ users, title }) {
  return (
    <div>
      <h2>{title}</h2>
      <ul>
        {users.map(user => (
          <UserItem key={user.id} user={user} />
        ))}
      </ul>
    </div>
  );
}

// STATE: Для внутреннего управления данными компонента
function LoginForm() {
  // Внутреннее состояние формы
  const [username, setUsername] = useState('');
  const [password, setPassword] = useState('');
  const [isLoading, setIsLoading] = useState(false);
  const [error, setError] = useState(null);
  
  const handleSubmit = async (e) => {
    e.preventDefault();
    setIsLoading(true);
    setError(null);
    
    try {
      // Логика отправки формы
      await login(username, password);
    } catch (err) {
      setError(err.message);
    } finally {
      setIsLoading(false);
    }
  };
  
  return (
    <form onSubmit={handleSubmit}>
      <input value={username} onChange={(e) => setUsername(e.target.value)} />
      <input type="password" value={password} onChange={(e) => setPassword(e.target.value)} />
      {error && <div className="error">{error}</div>}
      <button disabled={isLoading}>
        {isLoading ? 'Загрузка...' : 'Войти'}
      </button>
    </form>
  );
}
```

---

#### **3. Полный пример, демонстрирующий разницу**

```jsx
// Родительский компонент
function UserDashboard() {
  // STATE родителя: может меняться и влиять на детей
  const [theme, setTheme] = useState('light');
  const [user, setUser] = useState({ name: 'Иван', role: 'admin' });
  
  // PROPS для дочерних компонентов
  const dashboardData = {
    stats: { posts: 42, followers: 100 },
    notifications: ['Новое сообщение', 'Обновление системы']
  };
  
  return (
    <div className={`dashboard ${theme}`}>
      {/* theme и user передаются как PROPS */}
      <Header 
        theme={theme} 
        user={user} 
        onThemeToggle={() => setTheme(theme === 'light' ? 'dark' : 'light')}
      />
      
      {/* dashboardData передается как PROPS */}
      <MainContent data={dashboardData} />
      
      {/* Внутренний компонент со своим STATE */}
      <UserSettings />
    </div>
  );
}

// Дочерний компонент 1: получает PROPS
function Header({ theme, user, onThemeToggle }) {
  // Не имеет своего состояния, только отображает props
  return (
    <header>
      <h1>Добро пожаловать, {user.name}!</h1>
      <button onClick={onThemeToggle}>
        Переключить тему ({theme})
      </button>
    </header>
  );
}

// Дочерний компонент 2: имеет собственное STATE
function UserSettings() {
  // Внутреннее состояние компонента
  const [notificationsEnabled, setNotificationsEnabled] = useState(true);
  const [emailFrequency, setEmailFrequency] = useState('daily');
  
  return (
    <div className="settings">
      <h3>Настройки</h3>
      
      <label>
        <input
          type="checkbox"
          checked={notificationsEnabled}
          onChange={(e) => setNotificationsEnabled(e.target.checked)}
        />
        Уведомления
      </label>
      
      <select 
        value={emailFrequency} 
        onChange={(e) => setEmailFrequency(e.target.value)}
      >
        <option value="daily">Ежедневно</option>
        <option value="weekly">Еженедельно</option>
        <option value="monthly">Ежемесячно</option>
      </select>
      
      {/* Локальное состояние влияет только на этот компонент */}
      <p>Текущие настройки: уведомления {notificationsEnabled ? 'включены' : 'выключены'}</p>
    </div>
  );
}
```

---

#### **4. Когда использовать State vs Props**

**Используйте State, когда:**
- Данные меняются со временем
- Изменения происходят внутри компонента
- Компоненту нужно реагировать на пользовательский ввод
- Нужно хранить временные данные (например, значение формы)

**Используйте Props, когда:**
- Данные приходят из родительского компонента
- Компонент должен быть переиспользуемым с разными данными
- Нужно передать callback-функцию для общения с родителем
- Данные являются конфигурационными параметрами

```jsx
// Пример: компонент с props и state
function ProductCard({ product, onAddToCart }) {
  // Props: product (данные о товаре), onAddToCart (функция от родителя)
  // State: локальное состояние компонента
  const [quantity, setQuantity] = useState(1);
  const [isFavorite, setIsFavorite] = useState(false);
  
  const handleAdd = () => {
    // Используем и props, и state
    onAddToCart(product.id, quantity);
    setQuantity(1); // Сбрасываем локальное состояние
  };
  
  return (
    <div className="product-card">
      <h3>{product.name}</h3> {/* Данные из props */}
      <p>Цена: {product.price} руб.</p>
      
      <div>
        <button onClick={() => setQuantity(q => q - 1)}>-</button>
        <span>Количество: {quantity}</span> {/* Локальное state */}
        <button onClick={() => setQuantity(q => q + 1)}>+</button>
      </div>
      
      <button onClick={handleAdd}>
        Добавить в корзину
      </button>
      
      <button onClick={() => setIsFavorite(!isFavorite)}>
        {isFavorite ? '★ В избранном' : '☆ В избранное'}
      </button>
    </div>
  );
}
```

---

#### **5. Важные особенности State**

**Асинхронность обновления состояния:**
```jsx
function Counter() {
  const [count, setCount] = useState(0);
  
  const handleClick = () => {
    // ❌ Проблема: несколько вызовов setCount подряд
    setCount(count + 1);
    setCount(count + 1); // count всё ещё 0 здесь
    setCount(count + 1); // и здесь
    
    // ✅ Решение: использовать функциональную форму
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
    setCount(prevCount => prevCount + 1);
    // Результат: увеличение на 3
  };
  
  return <button onClick={handleClick}>Увеличить</button>;
}
```

**Пакетные обновления (Batching):**
```jsx
function UserForm() {
  const [user, setUser] = useState({ name: '', email: '' });
  
  const updateUser = () => {
    // React объединит эти обновления в один ререндер
    setUser(prev => ({ ...prev, name: 'Иван' }));
    setUser(prev => ({ ...prev, email: 'ivan@mail.ru' }));
    // Только один ререндер компонента
  };
}
```

**Инициализация состояния на основе props:**
```jsx
function CounterWithInitial({ initialValue }) {
  // State инициализируется на основе props
  const [count, setCount] = useState(initialValue);
  
  // Но если props изменятся, state НЕ обновится автоматически!
  // Для синхронизации используйте useEffect:
  useEffect(() => {
    setCount(initialValue);
  }, [initialValue]);
  
  return <div>Счет: {count}</div>;
}
```

---

### **Резюме**

**State (состояние)** — это внутренние, изменяемые данные компонента, которые контролируются самим компонентом и приводят к его повторному рендеру при обновлении. **Props (свойства)** — это внешние, неизменяемые данные, которые передаются от родительского компонента дочернему.

**Ключевые нюансы:**
1. **Контроль и изменение:** State контролируется и изменяется самим компонентом через `setState` или функции обновления состояния. Props контролируются родительским компонентом и являются **read-only** для получателя.
2. **Направление данных:** Props передаются **сверху вниз** (однонаправленный поток), а state является **локальным** для каждого компонента. Для общения между компонентами на одном уровне используется поднятие состояния (lifting state up).
3. **Влияние на рендер:** Изменение **любого props** или **любого state** приводит к повторному рендеру компонента. Но обновление state в родителе не обновляет state в дочернем компоненте автоматически, если они не связаны через props.

**Выжимка по объяснению:**
Props — это входные параметры компонента (как аргументы функции), которые передаются от родителя и не могут быть изменены внутри компонента. State — это внутреннее состояние компонента (как локальные переменные в функции), которое может изменяться и управляется самим компонентом. Props обеспечивают конфигурируемость и повторное использование, state — интерактивность и динамическое поведение.

**Как сказать своими словами кратко:**
>«Props — это настройки, которые родительский компонент передаёт дочернему (как данные или callback-функции). Дочерний компонент не может их менять — только читать. State — это внутренняя память компонента, его личные изменяемые данные. Компонент сам решает, когда и как менять своё состояние. Если данные приходят извне — это props, если создаются и меняются внутри — это state. Изменение и props, и state заставляет компонент перерисовываться.»