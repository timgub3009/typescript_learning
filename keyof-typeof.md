**Отличный вопрос! `typeof` и `keyof` — это основа работы с типами в TypeScript. Давайте разберем по полочкам.**

## **1. `keyof` — ключи типа**

### **Что делает:**
Получает все ключи типа в виде union типа.

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

type UserKeys = keyof User;
// ^? "id" | "name" | "email"
```

### **Когда использовать отдельно:**
```typescript
// ✅ Валидация ключей
function getUserField<T>(user: T, key: keyof T) {
  return user[key];
}

const user: User = { id: 1, name: "John", email: "john@test.com" };
getUserField(user, "name");    // ✅
// getUserField(user, "age");  // ❌ Ошибка - нет такого ключа

// ✅ Создание типов на основе ключей
type UserOptional = {
  [K in keyof User]?: User[K]
};
// { id?: number; name?: string; email?: string; }
```

---

## **2. `typeof` — тип значения**

### **Что делает:**
Получает тип значения (переменной, константы, функции).

```typescript
const user = {
  id: 1,
  name: "John",
  email: "john@test.com"
};

type UserType = typeof user;
// ^? { id: number; name: string; email: string }
```

### **Когда использовать отдельно:**
```typescript
// ✅ Получение типа из существующей переменной
const config = {
  apiUrl: "https://api.test.com",
  timeout: 5000,
  retries: 3
};

type Config = typeof config;
// Используем где-то в коде
function setupApp(config: Config) { }

// ✅ Получение типа функции
const fetchUser = async (id: number) => ({ id, name: "John" });
type FetchUserFn = typeof fetchUser;
// ^? (id: number) => Promise<{ id: number; name: string }>
```

---

## **3. `keyof typeof` — комбинация**

### **Что делает:**
Получает ключи из типа значения.

```typescript
const user = {
  id: 1,
  name: "John", 
  email: "john@test.com"
};

type UserKeys = keyof typeof user;
// ^? "id" | "name" | "email"
```

### **Когда использовать вместе:**

#### **Пример 1: Валидация ключей объекта**
```typescript
const theme = {
  primary: "#007bff",
  secondary: "#6c757d", 
  success: "#28a745"
};

type ThemeKeys = keyof typeof theme;
// "primary" | "secondary" | "success"

function getColor(key: ThemeKeys) {
  return theme[key];
}

getColor("primary");   // ✅
// getColor("error");  // ❌ Ошибка
```

#### **Пример 2: Создание union из ключей конфига**
```typescript
const routes = {
  home: "/",
  about: "/about",
  contact: "/contact"
} as const;

type RoutePaths = typeof routes[keyof typeof routes];
// ^? "/" | "/about" | "/contact"

type RouteNames = keyof typeof routes;
// ^? "home" | "about" | "contact"
```

#### **Пример 3: Типизация компонентов**
```typescript
const buttonVariants = {
  primary: "bg-blue-500",
  secondary: "bg-gray-500",
  danger: "bg-red-500"
} as const;

type ButtonVariant = keyof typeof buttonVariants;

function Button({ variant }: { variant: ButtonVariant }) {
  const className = buttonVariants[variant];
  return <button className={className}>Click</button>;
}

<Button variant="primary" />     // ✅
// <Button variant="custom" />   // ❌ Ошибка
```

---

## **4. Практические паттерны**

### **Паттерн 1: Enum-подобные объекты**
```typescript
const STATUS = {
  PENDING: "pending",
  SUCCESS: "success", 
  ERROR: "error"
} as const;

type Status = keyof typeof STATUS;
// "PENDING" | "SUCCESS" | "ERROR"

type StatusValue = typeof STATUS[keyof typeof STATUS];
// "pending" | "success" | "error"
```

### **Паттерн 2: Типизация событий**
```typescript
const EVENTS = {
  CLICK: "click",
  SUBMIT: "submit",
  CHANGE: "change"
} as const;

type EventHandlerMap = {
  [K in keyof typeof EVENTS]: (event: Event) => void;
};

const handlers: EventHandlerMap = {
  click: (e) => console.log("clicked"),
  submit: (e) => console.log("submitted"),
  change: (e) => console.log("changed")
  // auto-complete работает!
};
```

### **Паттерн 3: Конфигурация приложения**
```typescript
const appConfig = {
  database: {
    host: "localhost",
    port: 5432
  },
  api: {
    baseUrl: "https://api.example.com",
    timeout: 5000
  }
} as const;

type ConfigKeys = keyof typeof appConfig;
// "database" | "api"

type ApiConfig = typeof appConfig["api"];
// { baseUrl: "https://api.example.com"; timeout: 5000 }
```

### **Паттерн 4: Полиморфные компоненты**
```typescript
const components = {
  button: "button",
  input: "input",
  textarea: "textarea"
} as const;

type ComponentType = keyof typeof components;

function Polymorphic<T extends ComponentType>({
  as,
  ...props
}: { as: T } & React.ComponentProps<typeof components[T]>) {
  const Component = components[as] as any;
  return <Component {...props} />;
}

<Polymorphic as="button" onClick={() => {}}>Click</Polymorphic>     // ✅
<Polymorphic as="input" placeholder="Type..." />                    // ✅
// <Polymorphic as="div" /> // ❌ Ошибка
```

---

## **5. Когда что использовать — шпаргалка**

### **Используй `keyof` когда:**
- Есть интерфейс/тип и нужны его ключи
- Нужно создать mapped type
- Валидация ключей в generic-функциях

```typescript
interface User { id: number; name: string }
type Keys = keyof User;
```

### **Используй `typeof` когда:**
- Есть JavaScript переменная и нужен её тип
- Нужно создать тип на основе существующего значения
- Работаешь с сторонним JS кодом

```typescript
const user = { id: 1, name: "John" }
type UserType = typeof user;
```

### **Используй `keyof typeof` когда:**
- Есть JavaScript объект и нужны его ключи как тип
- Создаешь type-safe конфиги, темы, варианты
- Нужен union type из ключей объекта

```typescript
const colors = { red: "#f00", blue: "#00f" }
type ColorKeys = keyof typeof colors;
```

---

## **6. Частые ошибки**

### **Ошибка 1: `keyof` с примитивами**
```typescript
type StringKeys = keyof string;
// ^? number | typeof Symbol.iterator | "toString" | "charAt" | ...
// Это редко нужно!
```

### **Ошибка 2: Забыли `as const`**
```typescript
const colors = { red: "#f00", blue: "#00f" };
type Colors = keyof typeof colors; // "red" | "blue" ✅

// Но для значений:
type ColorValues = typeof colors[keyof typeof colors]; 
// ^? string - слишком широко!

const colorsConst = { red: "#f00", blue: "#00f" } as const;
type ColorValuesConst = typeof colorsConst[keyof typeof colorsConst];
// ^? "#f00" | "#00f" - точно!
```

---

## **Итог:**

- **`keyof`** — от типа к ключам
- **`typeof`** — от значения к типу  
- **`keyof typeof`** — от значения к ключам через тип

**Используй эту комбинацию для type-safe конфигов, вариантов компонентов, enum-подобных структур и любых случаев когда нужно сохранить синхронизацию между runtime значениями и типами!**
