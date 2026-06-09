### Для чего нужен useCallback и как он связан с оптимизацией дочерних компонентов?

**Шпаргалка:**

> `useCallback` — это хук для мемоизации функции. Он сохраняет ссылку на функцию между рендерами и создаёт новую только при изменении зависимостей.

---

#### Пример без useCallback

```jsx id="5p4z2j"
const Child = React.memo(({ onClick }) => {
  console.log('Child render');

  return <button onClick={onClick}>Click</button>;
});

function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = () => {
    console.log('click');
  };

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        {count}
      </button>

      <Child onClick={handleClick} />
    </>
  );
}
```

При каждом рендере `Parent` создаётся новая функция:

```jsx id="6glq3t"
const handleClick = () => {};
```

Для React:

```jsx id="v0j8ns"
oldFn !== newFn
```

Поэтому `React.memo` не сможет пропустить ререндер `Child`.

---

#### Пример с useCallback

```jsx id="w9bkjq"
function Parent() {
  const [count, setCount] = useState(0);

  const handleClick = useCallback(() => {
    console.log('click');
  }, []);

  return (
    <>
      <button onClick={() => setCount(c => c + 1)}>
        {count}
      </button>

      <Child onClick={handleClick} />
    </>
  );
}
```

Теперь ссылка на функцию остаётся прежней:

```jsx id="m9lzy8"
oldFn === newFn
```

и `Child` не будет ререндериться без необходимости.

---

#### Как это работает

Без `useCallback`:

```jsx id="ykz8x8"
const fn = () => {};
```

На каждом рендере создаётся новый объект функции.

С `useCallback`:

```jsx id="ap51rn"
const fn = useCallback(() => {}, []);
```

React сохраняет функцию и возвращает ту же ссылку, пока зависимости не изменятся.

---

#### Связь с React.memo

Самый частый сценарий:

```jsx id="57f6dy"
const Child = React.memo(...)
```

```jsx id="z2qaqs"
<Child onClick={handleClick} />
```

Если `handleClick` создаётся заново на каждом рендере:

```jsx id="8e0ec9"
oldFn !== newFn
```

то `React.memo` становится бесполезным.

Поэтому часто встречается связка:

```jsx id="9q6m9u"
React.memo
+
useCallback
```

---

#### useCallback и useMemo

По сути:

```jsx id="97i97d"
useCallback(fn, deps)
```

эквивалентен:

```jsx id="2zzr6m"
useMemo(() => fn, deps)
```

Разница только в удобстве API.

---

#### Когда useCallback не нужен

Плохой пример:

```jsx id="k2zv2k"
const handleClick = useCallback(() => {
  setOpen(true);
}, []);
```

если функция:

* не передаётся в дочерние компоненты
* не участвует в зависимостях других хуков

то мемоизация обычно не даёт выгоды.

---

### Нюансы и выжимка для собеса

* `useCallback` мемоизирует функцию
* сохраняет ссылку между рендерами
* создаёт новую функцию только при изменении зависимостей
* нужен в первую очередь для работы с `React.memo`
* помогает избежать лишних ререндеров дочерних компонентов
* полезен для функций в зависимостях `useEffect`, `useMemo`
* не стоит использовать для каждой функции подряд
* `useCallback(fn, deps)` ≈ `useMemo(() => fn, deps)`

---

### Своими словами

Каждый рендер компонента создаёт новые функции. Для JavaScript это новые объекты в памяти, даже если код функции не изменился. Если такая функция передаётся в memo-компонент через props, React считает, что props изменились, и выполняет новый рендер. `useCallback` позволяет сохранить одну и ту же ссылку на функцию между рендерами, благодаря чему `React.memo` начинает работать эффективно и может пропускать лишние обновления дочерних компонентов.
