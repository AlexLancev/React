### **Объяснение с примерами**

**Props (properties, свойства)** — это объект, содержащий данные, которые передаются от родительского компонента к дочернему. Props доступны только для чтения (read-only) и позволяют компонентам быть переиспользуемыми с разными данными.

---

#### **1. Базовый пример передачи props**

```jsx
// Дочерний компонент, принимающий props
function Welcome(props) {
  return <h1>Привет, {props.name}!</h1>;
}

// Родительский компонент, передающий props
function App() {
  return (
    <div>
      <Welcome name="Анна" />
      <Welcome name="Петр" />
      <Welcome name="Мария" />
    </div>
  );
}

// Результат на странице:
// Привет, Анна!
// Привет, Петр!
// Привет, Мария!
```

---

#### **2. Различные способы передачи props**

**Строки:**
```jsx
<Component title="Заголовок" />
```

**Числа:**
```jsx
<Component count={42} rating={4.5} />
```

**Булевы значения (true можно опускать):**
```jsx
<Component isActive={true} />
// Сокращённая запись:
<Component isActive />
```

**Массивы:**
```jsx
<Component items={['яблоко', 'банан', 'апельсин']} />
<Component numbers={[1, 2, 3, 4, 5]} />
```

**Объекты:**
```jsx
<Component user={{ name: 'Иван', age: 30 }} />
```

**Функции (callback-функции):**
```jsx
<Component onClick={() => console.log('Клик!')} />
<Component onChange={handleChange} />
```

**JSX и React-элементы:**
```jsx
<Component icon={<StarIcon />} />
<Component 
  header={<h2>Специальный заголовок</h2>}
  footer={<button>Сохранить</button>}
/>
```

**Дети (children) — специальный prop:**
```jsx
// Компонент Button принимает содержимое как children
function Button({ children }) {
  return <button className="btn">{children}</button>;
}

// Использование с текстом
<Button>Нажми меня</Button>

// Использование с другими элементами
<Button>
  <span>🔔</span>
  Уведомления
  <span className="badge">3</span>
</Button>
```

---

#### **3. Деструктуризация props**

```jsx
// Без деструктуризации
function UserCard(props) {
  return (
    <div>
      <h2>{props.name}</h2>
      <p>{props.email}</p>
      <p>Возраст: {props.age}</p>
    </div>
  );
}

// С деструктуризацией (рекомендуется)
function UserCard({ name, email, age }) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>Возраст: {age}</p>
    </div>
  );
}

// Деструктуризация с значениями по умолчанию
function UserCard({ 
  name = 'Гость', 
  email = 'не указан', 
  age = 18 
}) {
  return (
    <div>
      <h2>{name}</h2>
      <p>{email}</p>
      <p>Возраст: {age}</p>
    </div>
  );
}
```

---

#### **4. Передача всех props с помощью spread оператора**

```jsx
function InputField(props) {
  return <input {...props} />;
}

function Form() {
  const inputProps = {
    type: 'text',
    placeholder: 'Введите имя',
    className: 'form-input',
    onChange: handleChange
  };
  
  return <InputField {...inputProps} />;
}

// Это эквивалентно:
// <InputField 
//   type="text" 
//   placeholder="Введите имя" 
//   className="form-input" 
//   onChange={handleChange} 
// />
```

---

#### **5. Props в классовых компонентах**

```jsx
class Welcome extends React.Component {
  render() {
    return <h1>Привет, {this.props.name}!</h1>;
  }
}

// Использование
<Welcome name="Алексей" />
```

**С деструктуризацией в классовых компонентах:**
```jsx
class Welcome extends React.Component {
  render() {
    const { name, greeting = 'Привет' } = this.props;
    return <h1>{greeting}, {name}!</h1>;
  }
}
```

---

#### **6. Валидация props с PropTypes**

```jsx
import PropTypes from 'prop-types';

function Product({ name, price, inStock }) {
  return (
    <div>
      <h3>{name}</h3>
      <p>Цена: {price} руб.</p>
      <p>{inStock ? 'В наличии' : 'Нет в наличии'}</p>
    </div>
  );
}

// Валидация props
Product.propTypes = {
  name: PropTypes.string.isRequired,
  price: PropTypes.number.isRequired,
  inStock: PropTypes.bool,
  onSale: PropTypes.bool,
  discount: PropTypes.number
};

// Значения по умолчанию
Product.defaultProps = {
  inStock: true,
  onSale: false,
  discount: 0
};

// Использование
<Product name="Ноутбук" price={50000} />
<Product name="Мышь" price={1500} inStock={false} discount={10} />
```

**Современная альтернатива — TypeScript:**
```typescript
interface ProductProps {
  name: string;
  price: number;
  inStock?: boolean;
  discount?: number;
}

function Product({ name, price, inStock = true }: ProductProps) {
  // ...
}
```

---

#### **7. Важные особенности props**

**Props доступны только для чтения:**
```jsx
function UserCard({ user }) {
  // НЕЛЬЗЯ изменять props!
  // user.name = 'Новое имя'; // ❌ Ошибка!
  
  // Можно использовать значения для вычислений
  const displayName = user.name.toUpperCase(); // ✅
  
  return <div>{displayName}</div>;
}
```

**Передача props через несколько уровней (Prop Drilling):**
```jsx
// Проблема: передача через промежуточные компоненты
function App() {
  const [user, setUser] = useState({ name: 'Иван' });
  
  return (
    <Layout>
      <Header user={user} />
    </Layout>
  );
}

function Layout({ children }) {
  return <div className="layout">{children}</div>;
}

function Header({ user }) {
  return (
    <header>
      <UserMenu user={user} />
    </header>
  );
}

function UserMenu({ user }) {
  return <div>Привет, {user.name}!</div>;
}
```

**Решение проблемы через Context API:**
```jsx
const UserContext = createContext();

function App() {
  const [user, setUser] = useState({ name: 'Иван' });
  
  return (
    <UserContext.Provider value={user}>
      <Layout>
        <Header />
      </Layout>
    </UserContext.Provider>
  );
}

function Header() {
  return (
    <header>
      <UserMenu />
    </header>
  );
}

function UserMenu() {
  const user = useContext(UserContext); // Получаем напрямую
  return <div>Привет, {user.name}!</div>;
}
```

---

#### **8. Практический пример компонента с props**

```jsx
function ProductCard({
  id,
  name,
  price,
  discount = 0,
  rating,
  onAddToCart,
  children
}) {
  const finalPrice = price - (price * discount) / 100;
  
  return (
    <div className="product-card">
      <img 
        src={`/images/products/${id}.jpg`} 
        alt={name}
      />
      <h3>{name}</h3>
      
      <div className="price">
        {discount > 0 && (
          <span className="old-price">{price} руб.</span>
        )}
        <span className="current-price">{finalPrice} руб.</span>
      </div>
      
      {rating && (
        <div className="rating">
          {'★'.repeat(Math.floor(rating))}
          {'☆'.repeat(5 - Math.floor(rating))}
        </div>
      )}
      
      <button onClick={() => onAddToCart(id)}>
        Добавить в корзину
      </button>
      
      {children}
    </div>
  );
}

// Использование
function App() {
  const handleAddToCart = (productId) => {
    console.log('Добавлен товар:', productId);
  };
  
  return (
    <ProductCard
      id={123}
      name="Смартфон"
      price={30000}
      discount={15}
      rating={4.5}
      onAddToCart={handleAddToCart}
    >
      <p className="description">Новый флагман с отличной камерой</p>
      <button className="quick-view">Быстрый просмотр</button>
    </ProductCard>
  );
}
```

---

### **Резюме**

Props — это механизм передачи данных от родительских компонентов к дочерним в React. Они делают компоненты переиспользуемыми и настраиваемыми.

**Ключевые нюансы:**
1. **Однонаправленный поток:** Props передаются только сверху вниз (от родителя к потомку) и являются **неизменяемыми (immutable)** внутри компонента-получателя.
2. **Декларативность:** Передача props происходит декларативно через атрибуты JSX. Можно передавать любые типы данных: строки, числа, объекты, массивы, функции и даже другие React-элементы.
3. **Children как особый prop:** Содержимое между открывающим и закрывающим тегами компонента передаётся как `props.children`, что позволяет создавать компоненты-контейнеры.

**Выжимка по объяснению:**
Props — это параметры, которые родительский компонент передаёт дочернему. Они доступны только для чтения и позволяют настраивать поведение и внешний вид компонентов при их повторном использовании. Props могут содержать любые данные: от простых строк до сложных объектов и callback-функций. Декларативная передача через атрибуты JSX делает код предсказуемым и понятным.

**Как сказать своими словами кратко:**
>«Props — это как параметры функции, но для React-компонентов. Родительский компонент передаёт данные дочернему через атрибуты в JSX (например, `<User name="Иван" age={30}>`). Внутри компонента эти данные доступны только для чтения. Можно передавать что угодно: строки, числа, массивы, объекты и даже функции для обратной связи. Это делает компоненты переиспользуемыми — один и тот же компонент может отображать разные данные в зависимости от переданных props.»