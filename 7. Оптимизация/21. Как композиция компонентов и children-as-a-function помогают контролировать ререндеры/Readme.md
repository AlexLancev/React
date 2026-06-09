### Как композиция компонентов и children-as-a-function помогают контролировать ререндеры?

**Шпаргалка:**

Композиция помогает локализовать состояние и уменьшать область обновлений. Вместо передачи большого количества props и хранения state высоко в дереве, логику выносят в отдельные компоненты. Паттерн children-as-a-function (render props) позволяет передавать данные только тем компонентам, которым они действительно нужны.

### Нюансы и выжимка для собеса

* композиция помогает локализовать состояние и уменьшать область обновлений
* изменение локального state затрагивает меньше компонентов
* часто производительность улучшается не мемоизацией, а правильной композицией
* children позволяет инкапсулировать состояние внутри компонента-обёртки
* children-as-a-function (render props) передаёт данные потребителю через функцию
* render props помогают переиспользовать логику без поднятия состояния
* этот паттерн уменьшает prop drilling и делает обновления более локальными
* сегодня многие сценарии render props реализуются через кастомные хуки

---

### Своими словами

Одна из частых причин лишних ререндеров — состояние хранится слишком высоко в дереве компонентов. Композиция помогает переместить это состояние ближе к месту использования и тем самым сократить количество обновляемых компонентов. Render props работают похожим образом: компонент владеет состоянием и логикой, а внешний код только получает нужные данные для отображения. В результате управление состоянием становится более локальным, а область ререндеров — меньше. На практике это часто даёт больший эффект, чем попытки обернуть всё подряд в `React.memo`, `useMemo` и `useCallback`.

---

#### Композиция как инструмент оптимизации

Плохой вариант:

```jsx
function App() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <Header />
      <Sidebar />
      <Content />
      <Modal isOpen={isOpen} />
    </>
  );
}
```

При:

```jsx
setIsOpen(true);
```

получаем:

```text
App render
↓
Header render
Sidebar render
Content render
Modal render
```

---

Лучше выделить отдельный компонент:

```jsx
function ModalContainer() {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>
        Open
      </button>

      <Modal isOpen={isOpen} />
    </>
  );
}
```

```jsx
function App() {
  return (
    <>
      <Header />
      <Sidebar />
      <Content />
      <ModalContainer />
    </>
  );
}
```

Теперь:

```text
setIsOpen
↓
ModalContainer render
↓
Modal render
```

Остальная часть приложения не затрагивается.

---

#### Композиция через children

Пример:

```jsx
function Dialog({ children }) {
  const [isOpen, setIsOpen] = useState(false);

  return (
    <>
      <button onClick={() => setIsOpen(true)}>
        Open
      </button>

      {isOpen && children}
    </>
  );
}
```

Использование:

```jsx
<Dialog>
  <UserProfile />
</Dialog>
```

Состояние модалки инкапсулировано внутри Dialog.

---

#### Что такое children-as-a-function

Render Props:

```jsx
<DataLoader>
  {(data) => (
    <UserList users={data} />
  )}
</DataLoader>
```

или

```jsx
function DataLoader({ children }) {
  const [data, setData] = useState([]);

  return children(data);
}
```

---

#### Зачем это нужно

Компонент управляет логикой:

```jsx
DataLoader
```

а UI остаётся снаружи:

```jsx
UserList
```

Получается разделение ответственности.

---

#### Как это связано с ререндерами

Допустим есть:

```jsx
<DataLoader>
  {(data) => <UserList users={data} />}
</DataLoader>
```

Когда меняется:

```jsx
data
```

обновляется только цепочка, связанная с DataLoader.

Нет необходимости поднимать состояние в App:

```text
App
↓
всё дерево
```

---

#### Частый паттерн: управление состоянием через обёртку

```jsx
<Toggle>
  {({ isOpen, toggle }) => (
    <>
      <Button onClick={toggle} />
      {isOpen && <Menu />}
    </>
  )}
</Toggle>
```

Компонент:

```jsx
Toggle
```

управляет состоянием.

Потребитель получает только нужные данные.

---

#### Современная альтернатива

Сегодня многие задачи render props решаются через кастомные хуки:

```jsx
const { isOpen, toggle } = useToggle();
```

Но на собеседовании важно понимать, что render props исторически решали те же задачи:

* переиспользование логики
* изоляция состояния
* уменьшение prop drilling

---
