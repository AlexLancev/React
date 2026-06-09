### Какие проблемы производительности возникают при использовании Context API и почему?

**Шпаргалка:**

> Главная проблема Context API — при изменении `value` все компоненты, которые подписаны на этот контекст через `useContext`, получают обновление и ререндерятся. Чем больше подписчиков и чем чаще меняется значение, тем сильнее влияние на производительность.

---

#### Как работает обновление Context

```jsx
const ThemeContext = createContext();

function App() {
  const [theme, setTheme] = useState('dark');

  return (
    <ThemeContext.Provider value={theme}>
      <Layout />
    </ThemeContext.Provider>
  );
}
```

Подписчик:

```jsx
function Button() {
  const theme = useContext(ThemeContext);

  return <button>{theme}</button>;
}
```

При:

```jsx
setTheme('light');
```

React обновит всех подписчиков контекста.

---

#### Проблема большого количества подписчиков

Например:

```text
App
├── Header
├── Sidebar
├── Menu
├── UserPanel
├── Notifications
└── Settings
```

Если все используют:

```jsx
useContext(AppContext)
```

то изменение контекста может привести к ререндеру большого количества компонентов.

---

#### Проблема "огромного" контекста

Плохой пример:

```jsx
<AppContext.Provider
  value={{
    user,
    theme,
    notifications,
    settings,
    cart,
    language
  }}
>
```

Компоненту нужен только:

```jsx
theme
```

но он всё равно подписан на весь объект.

Если изменится:

```jsx
cart
```

подписчик также получит обновление.

---

#### Проблема новых объектов в value

Плохой пример:

```jsx
<AppContext.Provider
  value={{
    theme,
    toggleTheme
  }}
>
```

Каждый рендер создаёт новый объект:

```jsx
{} !== {}
```

Для React:

```text
value изменился
↓
обновить подписчиков
```

Даже если:

```jsx
theme
```

остался прежним.

---

#### Решение через useMemo

```jsx
const contextValue = useMemo(() => {
  return {
    theme,
    toggleTheme
  };
}, [theme, toggleTheme]);

<AppContext.Provider value={contextValue}>
```

Теперь объект меняется только при изменении зависимостей.

---

#### Почему React.memo не спасает

```jsx
const UserInfo = React.memo(function UserInfo() {
  const user = useContext(UserContext);

  return <div>{user.name}</div>;
});
```

Если контекст обновился:

```text
Context update
↓
UserInfo re-render
```

Даже несмотря на:

```jsx
React.memo
```

Потому что обновление пришло не через props, а через Context.

---

#### Как оптимизируют большие приложения

##### Разделяют контексты

Вместо:

```jsx
<AppContext>
```

делают:

```jsx
<UserContext>
<ThemeContext>
<CartContext>
```

Чтобы обновления были локальными.

---

##### Используют мемоизацию value

```jsx
useMemo(...)
```

для объектов

```jsx
useCallback(...)
```

для функций

---

##### Используют селекторы или state-менеджеры

Например:

* [Redux Toolkit](https://redux-toolkit.js.org?utm_source=chatgpt.com)
* [Zustand](https://zustand.docs.pmnd.rs?utm_source=chatgpt.com)
* [Jotai](https://jotai.org?utm_source=chatgpt.com)

Они позволяют подписываться только на нужный кусок состояния.

---

### Нюансы и выжимка для собеса

* изменение `Context value` обновляет всех подписчиков этого контекста
* чем больше подписчиков, тем дороже обновление
* большой общий контекст часто вызывает лишние ререндеры
* новые объекты и функции в `value` могут провоцировать ненужные обновления
* `React.memo` не защищает от обновлений, пришедших через `useContext`
* для оптимизации используют разделение контекстов по ответственности
* `value` обычно мемоизируют через `useMemo`
* для сложного глобального состояния часто используют store-библиотеки с селективными подписками

---

### Своими словами

Context отлично решает проблему prop drilling, но он не является бесплатным глобальным хранилищем. Когда значение контекста меняется, React уведомляет всех подписчиков. Если в одном контексте хранить всё приложение целиком, любое изменение может затронуть большое количество компонентов. Поэтому в крупных проектах контексты обычно делают маленькими и специализированными: отдельно для пользователя, темы, настроек и т.д. Это уменьшает область обновлений и снижает количество лишних ререндеров.
