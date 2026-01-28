### **Объяснение**

**useContext** — хук для доступа к значению React Context, позволяет получить данные из контекста без использования Consumer компонентов.

```jsx
const value = useContext(MyContext);
```

---

### **Создание и использование контекста**

#### **1. Создание контекста**
```jsx
const ThemeContext = createContext('light'); // Значение по умолчанию
```

#### **2. Предоставление значения (Provider)**
```jsx
function App() {
  return (
    <ThemeContext.Provider value="dark">
      <Toolbar />
    </ThemeContext.Provider>
  );
}
```

#### **3. Получение значения (Consumer через useContext)**
```jsx
function Button() {
  const theme = useContext(ThemeContext); // Получаем значение
  
  return (
    <button className={theme}>
      Текущая тема: {theme}
    </button>
  );
}
```

---

### **Полный пример**

```jsx
import { createContext, useContext, useState } from 'react';

// 1. Создаём контекст
const UserContext = createContext(null);

// 2. Компонент-провайдер
function App() {
  const [user, setUser] = useState({ name: 'Иван', age: 30 });
  
  return (
    <UserContext.Provider value={{ user, setUser }}>
      <Dashboard />
    </UserContext.Provider>
  );
}

// 3. Компонент-потребитель
function Dashboard() {
  return (
    <div>
      <UserProfile />
      <Settings />
    </div>
  );
}

// 4. Получаем контекст через useContext
function UserProfile() {
  const { user } = useContext(UserContext);
  
  return <h1>Привет, {user.name}!</h1>;
}

function Settings() {
  const { user, setUser } = useContext(UserContext);
  
  const updateName = () => {
    setUser({ ...user, name: 'Петр' });
  };
  
  return <button onClick={updateName}>Сменить имя</button>;
}
```

---

### **Отличие от Consumer компонента**

#### **Старый способ (Consumer)**
```jsx
function OldWay() {
  return (
    <ThemeContext.Consumer>
      {theme => (
        <button className={theme}>
          Тема: {theme}
        </button>
      )}
    </ThemeContext.Consumer>
  );
}
```

#### **Новый способ (useContext)**
```jsx
function NewWay() {
  const theme = useContext(ThemeContext);
  
  return (
    <button className={theme}>
      Тема: {theme}
    </button>
  );
}
```

---

### **Оптимизация с помощью useMemo**

```jsx
function App() {
  const [user, setUser] = useState({ name: '', age: 0 });
  const [theme, setTheme] = useState('light');
  
  // Мемоизируем значение контекста
  const userContextValue = useMemo(() => ({ 
    user, 
    setUser 
  }), [user]); // Пересоздаётся только при изменении user
  
  return (
    <UserContext.Provider value={userContextValue}>
      <Content theme={theme} />
    </UserContext.Provider>
  );
}
```

---

### **Распространённые ошибки**

#### **1. Забытый Provider**
```jsx
function Component() {
  const value = useContext(MyContext); // ❌ undefined, если нет Provider выше
  return <div>{value}</div>;
}
```

#### **2. Ненужное вложение**
```jsx
// ❌ Избыточно
function BadExample() {
  const user = useContext(UserContext);
  const theme = useContext(ThemeContext);
  const language = useContext(LanguageContext);
  
  return <div>{user.name}</div>;
}

// ✅ Лучше: кастомный хук
function useAppContext() {
  const user = useContext(UserContext);
  const theme = useContext(ThemeContext);
  const language = useContext(LanguageContext);
  
  return { user, theme, language };
}
```

---

### **Когда использовать**

**✅ Хорошие случаи:**
- Тема оформления (light/dark)
- Язык интерфейса
- Данные аутентификации
- Глобальные настройки

**❌ Плохие случаи:**
- Часто изменяемые данные (лучше стейт-менеджер)
- Множество независимых значений (лучше разделить контексты)
- Пропс дриллинг на 2-3 уровня (можно просто передать пропсы)

---

### **Резюме**

**useContext** — хук для чтения значения из React Context, упрощает доступ к глобальным данным.

**Ключевые нюансы:**
1. **Упрощает Consumer:** Заменяет громоздкий `<Context.Consumer>` на простой вызов хука
2. **Перерисовывает компоненты:** Компонент перерисовывается при изменении значения контекста
3. **Только для чтения:** Для изменения нужны функции в значении контекста

**Как сказать своими словами:**
«useContext — это способ получить глобальные данные в любом месте компонента без передачи через props. Создаёшь контекст, оборачиваешь часть приложения в Provider, а в дочерних компонентах получаешь значение через useContext. Удобно для темы, языка, данных пользователя.»