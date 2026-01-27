### **Объяснение с примерами**

**JSX (JavaScript XML)** — это синтаксическое расширение для JavaScript, которое позволяет писать HTML-подобную разметку внутри JavaScript-кода. Это **не** шаблонизатор и не отдельный язык — это **синтаксический сахар** для вызовов `React.createElement()`.

---

#### **1. Базовый пример JSX**

```jsx
// Это JSX
const element = <h1 className="greeting">Hello, world!</h1>;

// Компилятор (например, Babel) преобразует это в:
const element = React.createElement(
  'h1',
  { className: 'greeting' },
  'Hello, world!'
);

// Что в результате создаёт объект:
{
  type: 'h1',
  props: {
    className: 'greeting',
    children: 'Hello, world!'
  }
}
```

---

#### **2. Особенности синтаксиса JSX**

**Встраивание выражений JavaScript:**
```jsx
const name = 'Анна';
const element = <h1>Привет, {name}!</h1>;
// Результат: <h1>Привет, Анна!</h1>

const user = { firstName: 'Иван', lastName: 'Петров' };
const element = (
  <div>
    Полное имя: {user.firstName} {user.lastName}
  </div>
);
```

**JSX — тоже выражение:**
```jsx
// JSX можно присваивать переменным
const greeting = <div>Привет</div>;

// Использовать в условиях
function getGreeting(isLoggedIn) {
  return isLoggedIn ? <span>Добро пожаловать!</span> : <span>Войдите</span>;
}

// Возвращать из функций
function renderItem(item) {
  return <li key={item.id}>{item.name}</li>;
}
```

**Атрибуты в JSX:**
```jsx
// class становится className (т.к. class - зарезервированное слово в JS)
<div className="container"></div>

// for становится htmlFor
<label htmlFor="email">Email:</label>

// Стиль передаётся объектом
<div style={{ color: 'red', fontSize: '20px' }}></div>

// Атрибуты без значения становятся true
<button disabled>Неактивная кнопка</button>
// Эквивалентно <button disabled={true}>
```

**Самозакрывающиеся теги:**
```jsx
// В JSX всегда нужно закрывать теги
const img = <img src="image.jpg" alt="Описание" />; // Правильно
const br = <br />; // Правильно
// <br> или <img ...> без / вызовет ошибку
```

---

#### **3. JSX предотвращает инъекции атак**

```jsx
const userInput = '<script>alert("xss")</script>';

// JSX автоматически экранирует содержимое
const element = <div>{userInput}</div>;
// На странице отобразится текст: <script>alert("xss")</script>
// А НЕ выполнится скрипт

// Для безопасного вставки HTML нужно использовать dangerouslySetInnerHTML
const htmlContent = { __html: '<span>Безопасный HTML</span>' };
const element = <div dangerouslySetInnerHTML={htmlContent} />;
```

---

#### **4. JSX с компонентами**

```jsx
// Пользовательские компоненты должны начинаться с заглавной буквы
function Welcome(props) {
  return <h1>Привет, {props.name}</h1>;
}

// Использование компонента
const element = <Welcome name="Мария" />;

// Вложенность компонентов
function App() {
  return (
    <div>
      <Welcome name="Алексей" />
      <Welcome name="Елена" />
      <Welcome name="Дмитрий" />
    </div>
  );
}
```

---

#### **5. Фрагменты**

```jsx
// JSX требует одного корневого элемента
// Но можно использовать фрагменты
function List() {
  return (
    <>
      <li>Пункт 1</li>
      <li>Пункт 2</li>
    </>
  );
  // Или так:
  return (
    <React.Fragment>
      <li>Пункт 1</li>
      <li>Пункт 2</li>
    </React.Fragment>
  );
}
```

---

#### **6. Как работает компиляция**

```jsx
// Исходный JSX
const element = (
  <div className="container">
    <h1>Заголовок</h1>
    <p>Текст {count}</p>
  </div>
);

// После компиляции Babel
const element = React.createElement(
  'div',
  { className: 'container' },
  React.createElement('h1', null, 'Заголовок'),
  React.createElement('p', null, 'Текст ', count)
);
```

---

### **Резюме**

JSX — это синтаксическое расширение JavaScript, которое позволяет писать HTML-подобную разметку в React-коде, делая его более читаемым и удобным для описания UI.

**Ключевые нюансы:**
1. **Не HTML, а JavaScript:** JSX компилируется в вызовы `React.createElement()`, создавая объекты (React-элементы), а не строки. Это позволяет React эффективно сравнивать и обновлять DOM.
2. **Строгие правила:** Используются `className` вместо `class`, `htmlFor` вместо `for`, обязательное закрытие всех тегов. Стили передаются как объекты, а не строки.
3. **Безопасность и выражения:** JSX автоматически экранирует содержимое, предотвращая XSS-атаки. В фигурных скобках `{}` можно вставлять любое JavaScript-выражение.

**Выжимка по объяснению:**
JSX — это специальный синтаксис, похожий на HTML, но работающий внутри JavaScript-кода. Он делает описание React-компонентов интуитивно понятным. На этапе сборки JSX преобразуется в обычные JavaScript-вызовы, которые создают виртуальные DOM-элементы. Это не шаблонизатор, а синтаксический сахар для `React.createElement()`.

**Как сказать своими словами кратко:**
>«JSX — это способ писать HTML прямо в JavaScript-коде React. Он выглядит как HTML, но под капотом превращается в вызовы `React.createElement()`. Это делает код компонентов более читаемым, при этом даёт всю мощь JavaScript (можно вставлять переменные, вызывать функции прямо в разметке). Главное помнить про небольшие отличия от обычного HTML, например `className` вместо `class`».