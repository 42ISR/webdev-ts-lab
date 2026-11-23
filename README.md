# Лабораторная работа: Типизация данных в TypeScript

> **Цель:** освоить описание предметной области типами — объединения, интерфейсы, сужение типов и утилитарные типы — при запуске кода в Node.js.

# [Документация](https://www.typescriptlang.org/docs/handbook/2/everyday-types.html)

В заданиях моделируется предметная область **сервиса доставки еды**: меню, заказы, статусы и способы доставки. Примеры в теоретических разделах построены на другой предметной области — **онлайн-школе**, чтобы не дублировать решения заданий.

## Что такое TypeScript

TypeScript — это JavaScript с системой типов. Код, допустимый в JavaScript, как правило, допустим и в TypeScript, однако перед запуском TypeScript выполняет статическую проверку типов: не передана ли строка вместо числа, не пропущено ли поле объекта, не передан ли недопустимый вариант значения.

```typescript
let attempts: number = 2
attempts = "два"   // ❌ ошибка: Type 'string' is not assignable to type 'number'
```

Сам TypeScript в Node.js не выполняется: перед запуском он преобразуется в обычный JavaScript. Поэтому требуются два инструмента — средство запуска файлов `.ts` и средство проверки типов.

Основное правило работы: использование `any` и `@ts-ignore` запрещено. Если тип значения неизвестен, следует применять `unknown` и проверять значение перед использованием.

---

## Настройка проекта

Работа выполняется в отдельной папке, чтобы не изменять файлы React-приложения, расположенного в репозитории.

1. Создайте папку `ts-lab` в корне репозитория и выполните команды:

```bash
mkdir ts-lab
cd ts-lab
npm init -y
npm i -D typescript tsx
```

2. Создайте файл `tsconfig.json` следующего содержания:

```json
{
  "compilerOptions": {
    "target": "ES2020",
    "module": "NodeNext",
    "moduleResolution": "NodeNext",
    "strict": true,
    "noEmit": true,
    "skipLibCheck": true
  },
  "include": ["*.ts"]
}
```

Параметр `"strict": true` включает строгий режим проверки типов и должен оставаться включённым.

3. Создайте файл `index.ts` и поместите в него следующий код:

```typescript
console.log("Привет, мир!")      // Привет, мир!
console.log(2 + 2)               // 4
```

4. Запустите файл командой:

```bash
npx tsx index.ts
```

5. Для проверки типов без запуска используйте команду:

```bash
npx tsc --noEmit
```

При отсутствии ошибок команда не выводит ничего. Сообщение об ошибке типа имеет следующий вид:

```
index.ts:2:1 - error TS2322: Type 'string' is not assignable to type 'number'.
```

В первую очередь следует прочитать номер строки и текст ошибки. При необходимости текст ошибки можно найти в документации.

---

## Раздел 1. Типы, объединения и функции

### Теория

**Простые типы.** Тип указывается после имени переменной через двоеточие:

```typescript
const lessonMinutes: number = 90
const courseTitle: string = "Основы TypeScript"
const isPublished: boolean = true
```

Если тип очевиден из значения, аннотацию можно опустить: TypeScript определяет тип автоматически. Переменная, объявленная через `let`, может изменять значение, однако её тип фиксируется при объявлении:

```typescript
let attempts = 2     // TypeScript определил тип number
attempts = 3         // ✅
attempts = "три"     // ❌ ошибка: строку нельзя присвоить переменной типа number
```

**Литеральные типы и объединения.** Литеральный тип — это конкретное значение, используемое в качестве типа. Объединение (`|`) допускает одно значение из списка. Псевдоним `type` даёт такому набору имя:

```typescript
type LessonStatus = "planned" | "inProgress" | "finished" | "cancelled"

let lessonStatus: LessonStatus = "planned"   // ✅
lessonStatus = "archived"                    // ❌ ошибка: такого значения нет в типе
```

Литеральными могут быть и числа:

```typescript
type Rating = 1 | 2 | 3 | 4 | 5

let rating: Rating = 4   // ✅
rating = 6               // ❌ ошибка
```

**Функции.** Типы параметров и возвращаемого значения указываются явно:

```typescript
function repeatTitle(title: string, times: number = 1): string {
  return title.repeat(times)
}

function printLesson(title: string): void {
  console.log(`Занятие: ${title}`)
}
```

- `: string` после скобок задаёт тип возвращаемого значения.
- `void` означает, что функция ничего не возвращает.
- Значение по умолчанию делает параметр необязательным при вызове: если аргумент не передан, используется указанное значение.

**`unknown`.** Тип `unknown` означает «значение неизвестного типа». Перед использованием такое значение необходимо проверить, например с помощью `typeof`:

```typescript
function parseRating(raw: unknown): number | null {
  if (typeof raw === "number" && Number.isInteger(raw) && raw >= 1 && raw <= 5) {
    return raw
  }
  return null
}
```

Тип `any` отключает проверку и в данной работе не используется.

---

### Задание 1

Создайте файл `basics.ts` и выполните следующее:

1. Объявите литеральный тип `OrderStatus`, который допускает только пять значений: `"new"`, `"cooking"`, `"delivering"`, `"done"`, `"cancelled"`. Напишите функцию `statusLabel(status: OrderStatus): string`, которая по значению статуса возвращает русскую подпись: «Новый», «Готовится», «В доставке», «Завершён», «Отменён». Для выбора подписи используйте `switch`. Выведите результат для каждого из пяти статусов.
2. Напишите функцию `orderTotal(price: number, quantity: number, deliveryFee: number = 0): number`, которая вычисляет итоговую стоимость заказа. Параметр `price` — цена одной позиции в рублях, `quantity` — количество позиций, `deliveryFee` — стоимость доставки. Итог вычисляется по формуле `price * quantity + deliveryFee`. Третий параметр необязателен: если он не передан, стоимость доставки считается равной нулю. Выведите результат двух вызовов: с указанием стоимости доставки и без неё.
3. Напишите функцию `isFinalStatus(status: OrderStatus): boolean`. Функция должна возвращать `true`, если заказ находится в конечном статусе (`"done"` или `"cancelled"`), и `false` для остальных статусов.
4. Напишите функцию `parseQuantity(raw: unknown): number | null`. Функция принимает значение неизвестного типа и возвращает количество или `null`. Допустимым считается целое положительное число, а также строка, которую можно преобразовать в такое число, например `"2"`. Во всех остальных случаях функция возвращает `null`. Проверьте работу функции на значениях `"2"`, `2`, `2.5`, `-1`.
5. Объявите литеральный числовой тип `Discount`, который допускает только значения `0`, `5`, `10`, `15`. Напишите функцию `applyDiscount(total: number, discount: Discount): number`, которая возвращает сумму после скидки. `total` — исходная сумма заказа, `discount` — размер скидки в процентах; итог вычисляется по формуле `total - total * discount / 100`. Выведите результаты для скидок `0`, `10` и `15`.

После выполнения добавьте в конец файла строку `statusLabel("paid")`, убедитесь, что `npx tsc --noEmit` выдаёт ошибку, и удалите эту строку.

---

## Раздел 2. Объекты и интерфейсы

### Теория

В примерах раздела используется тип `LessonStatus`, объявленный в разделе 1.

**Интерфейс** описывает форму объекта:

```typescript
interface IStudent {
  readonly id: string
  name: string
  email?: string
}

const student: IStudent = {
  id: "s1",
  name: "Алексей",
}
```

- `?` делает поле необязательным.
- `readonly` запрещает изменять поле после создания объекта.
- Тип `interface` используется для описания объектов, `type` — для объединений, примитивов и псевдонимов.

**Метод объекта** описывается как функция. Внутри метода доступен `this`:

```typescript
interface ITeacher {
  name: string
  subject: string
  introduce(): string
}

const teacher: ITeacher = {
  name: "Ирина",
  subject: "TypeScript",
  introduce() {
    return `${this.name}, преподаватель ${this.subject}`
  },
}
```

**Вложенные интерфейсы.** Полем интерфейса может быть другой интерфейс или массив:

```typescript
interface ICourse {
  id: string
  teacher: ITeacher
  status: LessonStatus
}
```

**Структурная типизация.** TypeScript сравнивает не имена типов, а наборы полей. Объект с лишними полями совместим с типом, если в нём есть все необходимые поля. Исключение — объектный литерал, переданный напрямую: для него проверяется отсутствие лишних свойств.

```typescript
function fitsSchedule(slot: { minutes: number }, lessonMinutes: number): boolean {
  return slot.minutes >= lessonMinutes
}

const morningSlot = { minutes: 120, label: "утро" }
fitsSchedule(morningSlot, 90)                            // ✅
fitsSchedule({ minutes: 120, label: "утро" }, 90)        // ❌ лишнее свойство label
```

---

### Задание 2

Создайте файл `objects.ts`. В начале файла объявите литеральный тип статуса заказа:

```typescript
type OrderStatus = "new" | "cooking" | "delivering" | "done" | "cancelled"
```

Выполните следующее:

1. Объявите `interface IMenuItem` с полями `readonly id: string`, `title: string`, `price: number`, `tags?: string[]` и методом `label(): string`. Метод возвращает строку вида `"Капучино — 250 ₽"`: название позиции, разделитель `—` и цена со знаком `₽`. Создайте массив `menu` из трёх позиций: «Капучино» (`id: "m1"`, цена `250`), «Круассан» (`id: "m2"`, цена `180`), «Чизкейк» (`id: "m3"`, цена `320`). Выведите результат вызова `label()` для каждой позиции.
2. Объявите `interface IOrderItem` с полями `menuItemId: string`, `title: string`, `price: number`, `quantity: number`, `comment?: string`. Напишите функцию `itemTotal(item: IOrderItem): number`, которая вычисляет стоимость позиции: `price * quantity`. Выведите результат для позиции «Капучино» ценой `250` в количестве `2`.
3. Объявите `interface ICustomer` с полями `name: string` и `phone: string`, затем `interface IOrder` с полями `id: string`, `customer: ICustomer`, `items: IOrderItem[]`, `status: OrderStatus`, `createdAt: string`. Напишите функцию `orderTotal(order: IOrder): number`, которая суммирует стоимости всех позиций заказа: для каждой позиции вычисляется `price * quantity`, результаты складываются. Создайте один заказ с двумя позициями и выведите результат.
4. Из созданного заказа с помощью деструктуризации получите `id`, `status` и `customer.name`; из первой позиции — `title` и `comment` со значением по умолчанию `"без комментария"`. Выведите полученные значения.
5. Напишите функцию `canAfford(wallet: { balance: number }, price: number): boolean`, которая возвращает `true`, если `wallet.balance >= price`, и `false` в противном случае. Вызовите функцию двумя способами: через переменную `wallet = { balance: 500, currency: "RUB" }` (вызов `canAfford(wallet, 300)` должен компилироваться) и через объектный литерал `canAfford({ balance: 500, currency: "RUB" }, 300)` (этот вызов должен давать ошибку из-за лишнего свойства). Ошибочный вызов удалите после проверки.

Добавьте в конец файла строку `menu[0].id = "m4"`, убедитесь, что `npx tsc --noEmit` выдаёт ошибку, и удалите эту строку.

---

## Раздел 3. Массивы, union-типы и сужение

### Теория

В примерах раздела используется тип `LessonStatus`, объявленный в разделе 1.

**Массивы интерфейсов** и методы `filter`, `find`:

```typescript
interface ILessonRef {
  id: string
  status: LessonStatus
}

const lessons: ILessonRef[] = [
  { id: "l1", status: "planned" },
  { id: "l2", status: "finished" },
]

const upcoming = lessons.filter((lesson) => lesson.status === "planned")   // ILessonRef[]
const found = lessons.find((lesson) => lesson.id === "l2")                 // ILessonRef | undefined
```

Метод `filter` возвращает новый массив только с теми элементами, для которых колбэк вернул `true`. Метод `find` возвращает первый подходящий элемент или `undefined`, если элемент не найден. Результат `find` необходимо проверять перед использованием.

**Discriminated union.** Объединение объектов с общим полем-дискриминантом позволяет TypeScript определять конкретный вариант после проверки этого поля:

```typescript
type LessonFormat =
  | { kind: "offline"; room: string }
  | { kind: "online"; link: string }

function formatDetails(format: LessonFormat): string {
  if (format.kind === "offline") {
    return `Аудитория: ${format.room}`
  }
  return `Ссылка: ${format.link}`
}
```

В ветке `if` TypeScript знает, что `format` имеет тип `{ kind: "offline"; room: string }`, а после неё — тип `{ kind: "online"; link: string }`.

**`Record` с union-ключами** описывает объект, в котором должны присутствовать все ключи объединения:

```typescript
const statusLabels: Record<LessonStatus, string> = {
  planned: "Запланировано",
  inProgress: "Идёт",
  finished: "Завершено",
  cancelled: "Отменено",
}
```

Если пропустить один из статусов, TypeScript сообщит об ошибке.

---

### Задание 3

Создайте файл `unions.ts`. В начале файла разместите объявления типов:

```typescript
type OrderStatus = "new" | "cooking" | "delivering" | "done" | "cancelled"

interface ICustomer {
  name: string
  phone: string
}

interface IOrderItem {
  menuItemId: string
  title: string
  price: number
  quantity: number
  comment?: string
}

interface IOrder {
  id: string
  customer: ICustomer
  items: IOrderItem[]
  status: OrderStatus
  createdAt: string
}
```

Каждый файл работы должен быть самодостаточным, поэтому объявления типов повторяются.

Выполните следующее:

1. Дан массив заказов:

   ```typescript
   const orders: IOrder[] = [
     {
       id: "o1",
       customer: { name: "Анна", phone: "+7 900 000-00-01" },
       items: [{ menuItemId: "m1", title: "Капучино", price: 250, quantity: 2 }],
       status: "new",
       createdAt: "2026-11-24T10:00:00",
     },
     {
       id: "o2",
       customer: { name: "Иван", phone: "+7 900 000-00-02" },
       items: [{ menuItemId: "m2", title: "Круассан", price: 180, quantity: 1 }],
       status: "delivering",
       createdAt: "2026-11-24T10:15:00",
     },
     {
       id: "o3",
       customer: { name: "Мария", phone: "+7 900 000-00-03" },
       items: [{ menuItemId: "m3", title: "Чизкейк", price: 320, quantity: 1 }],
       status: "done",
       createdAt: "2026-11-24T09:40:00",
     },
     {
       id: "o4",
       customer: { name: "Пётр", phone: "+7 900 000-00-04" },
       items: [{ menuItemId: "m2", title: "Круассан", price: 180, quantity: 3 }],
       status: "new",
       createdAt: "2026-11-24T10:20:00",
     },
   ]
   ```

1. Напишите функцию `filterByStatus(orders: IOrder[], status: OrderStatus): IOrder[]`, которая возвращает массив заказов с указанным статусом. Выведите идентификаторы заказов со статусом `"new"`.
2. Напишите функцию `findOrder(orders: IOrder[], id: string): IOrder | undefined`, которая находит заказ по идентификатору. Выведите идентификатор найденного заказа; если заказ с таким идентификатором отсутствует, выведите сообщение «Заказ не найден». Используйте `if`.
3. Напишите функцию `countByStatus(orders: IOrder[]): Record<OrderStatus, number>`, которая подсчитывает количество заказов в каждом статусе. Внутри создайте объект с ключами `new`, `cooking`, `delivering`, `done`, `cancelled`, начальными значениями `0`, затем переберите заказы и увеличьте счётчик соответствующего статуса на единицу.
4. Объявите тип `Delivery` как объединение двух вариантов: `{ kind: "pickup" }` и `{ kind: "courier"; address: string; fee: number }`. Напишите функцию `deliveryCost(delivery: Delivery): number`, которая возвращает стоимость доставки: для самовывоза — `0`, для доставки курьером — значение поля `fee`. Проверьте функцию на двух значениях: `{ kind: "pickup" }` и `{ kind: "courier", address: "ул. Ленина, 5", fee: 150 }`.
5. Напишите функцию `deliveryLabel(delivery: Delivery): string`, которая возвращает описание способа доставки: для самовывоза — строку `"Самовывоз"`, для доставки курьером — строку вида `"Курьер: ул. Ленина, 5"`, где после двоеточия указан адрес из поля `address`. Используйте сужение типа через `if`.

---

## Раздел 4. Утилитарные типы

### Теория

Утилитарные типы создают новый тип на основе существующего интерфейса и избавляют от дублирования полей. В примерах используются типы `LessonStatus` и `IStudent` из предыдущих разделов.

| Тип | Результат |
|---|---|
| `Partial<T>` | все поля становятся необязательными |
| `Pick<T, K>` | остаются только перечисленные поля |
| `Omit<T, K>` | перечисленные поля удаляются |
| `Readonly<T>` | все поля становятся доступными только для чтения |

```typescript
interface ILesson {
  id: string
  status: LessonStatus
  student: IStudent
}

type LessonPatch = Partial<ILesson>
// { id?: string; status?: LessonStatus; student?: IStudent }

type LessonCard = Pick<ILesson, "id" | "status">
// { id: string; status: LessonStatus }

type PublicLesson = Omit<ILesson, "student">
// { id: string; status: LessonStatus }
```

**`Partial`** применяется при частичном обновлении объекта: объект такого типа может содержать любое подмножество полей исходного типа.

```typescript
const patch: LessonPatch = { status: "finished" }   // ✅ достаточно одного поля
```

**`Readonly`** запрещает изменение полей после создания объекта:

```typescript
const lesson: ILesson = {
  id: "l1",
  status: "planned",
  student: { id: "s1", name: "Алексей" },
}

const archivedLesson: Readonly<ILesson> = lesson
archivedLesson.status = "cancelled"   // ❌ ошибка: Cannot assign to 'status' because it is a read-only property
```

**`Record`** описывает объект с ключами заданного типа:

```typescript
const lessonsById: Record<string, ILesson> = {}
```

---

### Задание 4

Создайте файл `utility.ts`. В начале файла разместите объявления типов и исходные данные:

```typescript
type OrderStatus = "new" | "cooking" | "delivering" | "done" | "cancelled"

interface ICustomer {
  name: string
  phone: string
}

interface IOrderItem {
  menuItemId: string
  title: string
  price: number
  quantity: number
  comment?: string
}

interface IOrder {
  id: string
  customer: ICustomer
  items: IOrderItem[]
  status: OrderStatus
  createdAt: string
}

interface IMenuItem {
  readonly id: string
  title: string
  price: number
  tags?: string[]
  label(): string
}

const orders: IOrder[] = [
  {
    id: "o1",
    customer: { name: "Анна", phone: "+7 900 000-00-01" },
    items: [
      { menuItemId: "m1", title: "Капучино", price: 250, quantity: 2 },
      { menuItemId: "m2", title: "Круассан", price: 180, quantity: 1 },
    ],
    status: "new",
    createdAt: "2026-11-24T10:00:00",
  },
  {
    id: "o2",
    customer: { name: "Иван", phone: "+7 900 000-00-02" },
    items: [{ menuItemId: "m3", title: "Чизкейк", price: 320, quantity: 1 }],
    status: "delivering",
    createdAt: "2026-11-24T10:15:00",
  },
]

const menuItem: IMenuItem = {
  id: "m1",
  title: "Капучино",
  price: 250,
  label() {
    return `${this.title} — ${this.price} ₽`
  },
}
```

Выполните следующее:

1. Напишите функцию `updateOrder(order: IOrder, patch: Partial<IOrder>): IOrder`, которая возвращает новый объект заказа с частично обновлёнными полями. Объект `patch` может содержать любое подмножество полей заказа; значения из `patch` должны перекрывать значения из `order`. Исходный объект изменяться не должен. Используйте spread-оператор. Проверьте на заказе `orders[0]`, передав в `patch` только статус `"done"`.
2. Напишите функцию `orderSummary(order: IOrder): Pick<IOrder, "id" | "status">`, которая возвращает объект только с идентификатором и статусом заказа.
3. Напишите функцию `publicOrder(order: IOrder): Omit<IOrder, "customer">`, которая возвращает заказ без персональных данных клиента: все поля, кроме `customer`, должны сохраниться. Используйте деструктуризацию с rest-оператором.
4. Напишите функцию `indexOrdersById(orders: IOrder[]): Record<string, IOrder>`, которая создаёт справочник заказов: ключом является `id` заказа, значением — сам заказ. Создайте пустой объект типа `Record<string, IOrder>`, заполните его в цикле и верните. Выведите список ключей результата.
5. Напишите функцию `freezeItem(item: IMenuItem): Readonly<IMenuItem>`, которая возвращает позицию меню, доступную только для чтения. После вызова попробуйте изменить поле `title` у результата, убедитесь, что `npx tsc --noEmit` выдаёт ошибку, и удалите эту строку.

# Как сдавать

- Создайте форк репозитория в вашей организации с названием вида `<название-этого-репозитория>-<ваша-фамилия>`.
- Выполните задание в ветке `wip`.
- Зафиксируйте изменения в вашем репозитории.
- После завершения работы создайте пул-реквест из вашей ветки `wip` в ветку `main` (тоже вашу) и укажите ревьюером `ktkv419`.
- Не выполняйте слияние коммита самостоятельно: это будет сделано после проверки задания.
