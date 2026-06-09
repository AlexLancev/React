### Как tree shaking и способ импортов (barrel vs direct import) влияют на итоговый бандл?

**Шпаргалка:**

> Tree shaking — это механизм сборщика, который удаляет неиспользуемый код из итогового бандла. Насколько эффективно он сработает, сильно зависит от структуры экспортов и способа импортирования модулей.

---

#### Что такое tree shaking

Допустим есть модуль:

```js
export const add = () => {};
export const subtract = () => {};
export const multiply = () => {};
```

Используем только:

```js
import { add } from './math';
```

После сборки:

```text
add     ✅ останется
subtract ❌ удалится
multiply ❌ удалится
```

Сборщик понимает, что остальной код нигде не используется.

---

#### Когда tree shaking работает хорошо

Лучше всего работает с ES Modules:

```js
export const foo = () => {};
export const bar = () => {};
```

```js
import { foo } from './utils';
```

Сборщик может статически определить зависимости ещё на этапе сборки.

---

#### Когда tree shaking работает плохо

CommonJS:

```js
const utils = require('./utils');
```

или

```js
module.exports = {
  foo,
  bar
};
```

Анализировать такие зависимости сложнее.

Поэтому современные проекты предпочитают:

```text
ESM (import/export)
```

---

#### Barrel Export

Допустим есть структура:

```text
components/
├─ Button.js
├─ Input.js
├─ Modal.js
└─ index.js
```

index.js:

```js
export * from './Button';
export * from './Input';
export * from './Modal';
```

Импорт:

```js
import { Button } from './components';
```

Это называется:

```text
barrel export
```

---

#### Плюсы barrel

Удобный API:

```js
import {
  Button,
  Input,
  Modal
} from './components';
```

Вместо:

```js
import Button from './components/Button';
import Input from './components/Input';
import Modal from './components/Modal';
```

---

#### Минусы barrel

В некоторых случаях сборщик вынужден анализировать всю цепочку реэкспортов:

```text
index
↓
Button
Input
Modal
...
```

На больших проектах это:

```text
увеличивает время сборки
усложняет tree shaking
```

---

#### Direct Import

Вместо:

```js
import { Button } from '@/components';
```

используют:

```js
import Button from '@/components/Button';
```

Сборщик сразу знает нужный файл.

---

#### Классический пример с lodash

Плохо:

```js
import _ from 'lodash';
```

или:

```js
import { debounce } from 'lodash';
```

В некоторых конфигурациях может попасть большая часть библиотеки.

---

Лучше:

```js
import debounce from 'lodash/debounce';
```

или:

```js
import { debounce } from 'lodash-es';
```

---

#### Проблема Side Effects

Tree shaking удаляет только безопасный код.

Если модуль содержит:

```js
console.log('loaded');
```

или:

```js
window.someGlobal = {};
```

сборщик не сможет просто выбросить файл.

---

Поэтому многие библиотеки указывают:

```json
{
  "sideEffects": false
}
```

Это сигнал сборщику:

```text
неиспользуемые модули можно удалять
```

---

#### Реальный кейс UI-библиотек

Плохо:

```js
import {
  Button
} from 'huge-ui-library';
```

Если библиотека плохо подготовлена, можно подтянуть:

```text
Button
Modal
Table
DatePicker
Charts
...
```

---

Лучше:

```js
import Button from 'huge-ui-library/Button';
```

или использовать библиотеку с хорошей поддержкой tree shaking.

---

#### Что смотреть в Bundle Analyzer

Если импортировали:

```js
import { Button } from 'ui-kit';
```

а в отчёте видно:

```text
DatePicker
Charts
Modal
Editor
```

значит tree shaking работает неэффективно либо библиотека имеет side effects.

---

### Нюансы и выжимка для собеса

* tree shaking удаляет неиспользуемый код из итогового бандла
* лучше всего работает с ES Modules (`import/export`)
* barrel exports удобны, но могут усложнять анализ зависимостей и увеличивать время сборки
* direct imports делают зависимости более явными и иногда дают меньший бандл
* важно учитывать side effects — они могут помешать удалению кода
* некоторые библиотеки tree-shake-ятся хорошо, некоторые — плохо
* Bundle Analyzer помогает проверить, действительно ли лишний код был исключён
* способ импортов напрямую влияет на размер итогового бандла

---

### Своими словами

Tree shaking нужен для того, чтобы в продакшен попадал только реально используемый код. Но эффективность этого механизма зависит от того, как написана библиотека и как мы её импортируем. Barrel-файлы удобны для разработки, но иногда усложняют работу сборщика, особенно в больших проектах. Direct imports делают зависимости более явными и могут уменьшить размер бандла. На практике после добавления крупной библиотеки полезно открыть Bundle Analyzer и убедиться, что вместе с одной кнопкой случайно не подтянулась половина UI-кита.
