Отличный вопрос! Давайте разберем **совместимость типов** в TypeScript - это ключ к пониманию `extends`.

## **1. Структурная типизация (Structural Typing)**

TypeScript использует **структурную типизацию**, а не номинальную (как в Java/C#). Это значит:

**Типы совместимы, если они имеют одинаковую структуру, независимо от имен!**

```typescript
interface Point {
  x: number;
  y: number;
}

interface Coordinate {
  x: number;
  y: number;
}

// Совместимы, потому что структура одинаковая!
let point: Point = { x: 1, y: 2 };
let coord: Coordinate = point; // ✅ OK
```

---

## **2. Когда работает совместимость с `extends`**

### **a. Примитивы и литералы**
```typescript
// Литералы совместимы с базовыми типами
type T1 = "hello" extends string ? true : false;          // true
type T2 = 42 extends number ? true : false;               // true
type T3 = true extends boolean ? true : false;            // true

// Но не наоборот!
type T4 = string extends "hello" ? true : false;          // false
type T5 = number extends 42 ? true : false;               // false
```

### **b. Объекты и интерфейсы**
```typescript
interface Animal {
  name: string;
}

interface Dog extends Animal {
  breed: string;
}

interface Cat {
  name: string;
  age: number;
}

// Наследование
type T1 = Dog extends Animal ? true : false;              // true

// Структурная совместимость (даже без extends)
type T2 = Cat extends Animal ? true : false;              // true
type T3 = { name: string } extends Animal ? true : false; // true

// Больше свойств = совместимо
type T4 = Dog extends { name: string } ? true : false;    // true

// Меньше свойств = НЕ совместимо
type T5 = Animal extends Dog ? true : false;              // false
```

### **c. Функции**
```typescript
type F1 = (a: string, b: number) => void;
type F2 = (a: string) => void;

// Меньше параметров = совместимо
type T1 = F2 extends F1 ? true : false;                   // true

// Больше параметров = НЕ совместимо  
type T2 = F1 extends F2 ? true : false;                   // false
```

---

## **3. Правила совместимости**

### **Правило 1: Ковариантность свойств объекта**
```typescript
interface Source {
  value: string | number;
}

interface Target {
  value: string;
}

// Источник имеет БОЛЕЕ ШИРОКИЙ тип => совместим
type T1 = Source extends Target ? true : false;           // true

// Источник имеет БОЛЕЕ УЗКИЙ тип => НЕ совместим  
type T2 = Target extends Source ? true : false;           // false
```

### **Правило 2: Контравариантность параметров функций**
```typescript
type SourceFunc = (param: string) => void;
type TargetFunc = (param: string | number) => void;

// Источник принимает БОЛЕЕ УЗКИЙ тип => совместим
type T1 = SourceFunc extends TargetFunc ? true : false;   // true

// Источник принимает БОЛЕЕ ШИРОКИЙ тип => НЕ совместим
type T2 = TargetFunc extends SourceFunc ? true : false;   // false
```

---

## **4. Практические примеры с `extends`**

### **Пример 1: Проверка на наличие свойства**
```typescript
type HasProperty<T, K extends string> = T extends { [P in K]: any } ? true : false;

type T1 = HasProperty<{ name: string }, "name">;          // true
type T2 = HasProperty<{ name: string }, "age">;           // false
```

### **Пример 2: Извлечение из объединения**
```typescript
type ExtractStrings<T> = T extends string ? T : never;

type T1 = ExtractStrings<string | number | boolean>;      // string
type T2 = ExtractStrings<"hello" | 42 | true>;            // "hello"
```

### **Пример 3: Рекурсивная обработка**
```typescript
type DeepPartial<T> = {
  [P in keyof T]?: T[P] extends object ? DeepPartial<T[P]> : T[P];
};

type Example = DeepPartial<{
  name: string;
  address: {
    street: string;
    city: string;
  };
}>;

// Результат:
// {
//   name?: string;
//   address?: {
//     street?: string;
//     city?: string;  
//   };
// }
```

---

## **5. Сложные случаи**

### **Union types распределение:**
```typescript
type Distributed<T> = T extends any ? T[] : never;
type T1 = Distributed<string | number>; // string[] | number[]

// Без распределения:
type NonDistributed<T> = [T] extends [any] ? T[] : never;
type T2 = NonDistributed<string | number>; // (string | number)[]
```

### **Never type:**
```typescript
// never распространяется через extends
type T1 = never extends string ? true : false;            // true
type T2 = string extends never ? true : false;            // false
```

### **Any type:**
```typescript
// any совместим со всем
type T1 = any extends string ? true : false;              // boolean (true | false)
type T2 = string extends any ? true : false;              // true
```

---

## **6. Практическое применение в utility types**

### **Extract<T, U> - извлечь совместимые типы**
```typescript
type Extract<T, U> = T extends U ? T : never;

type T1 = Extract<"a" | "b" | 1 | 2, string>;            // "a" | "b"
```

### **Exclude<T, U> - исключить совместимые типы**  
```typescript
type Exclude<T, U> = T extends U ? never : T;

type T1 = Exclude<"a" | "b" | 1 | 2, string>;            // 1 | 2
```

### **Parameters<T> - получить параметры функции**
```typescript
type Parameters<T> = T extends (...args: infer P) => any ? P : never;

type T1 = Parameters<(a: string, b: number) => void>;     // [string, number]
```

---

## **7. Когда НЕ работает совместимость**

### **Private/protected поля:**
```typescript
class A {
  private secret = 1;
}

class B {
  private secret = 1;  
}

// ❌ НЕ совместимы, даже если структура одинаковая!
type T1 = A extends B ? true : false;                     // false
```

### **Разные имена свойств с одинаковой структурой:**
```typescript
type Branded<T, Brand> = T & { __brand: Brand };

type UserId = Branded<number, "UserId">;
type ProductId = Branded<number, "ProductId">;

// ❌ НЕ совместимы из-за брендирования
type T1 = UserId extends ProductId ? true : false;        // false
```

---

## **Итог по совместимости:**

1. **Структурная типизация** - главный принцип
2. **`A extends B`** = "можно ли безопасно присвоить A переменной типа B"
3. **Ковариантность** для свойств объектов и возвращаемых типов
4. **Контравариантность** для параметров функций  
5. **Дистрибутивность** для union типов в условных типах

**Простое правило:** Тип A совместим с B, если A имеет **все обязательные свойства** B с **совместимыми типами**.

Конечно! Вот 10 кратких практических примеров где `extends` незаменим:

## **1. Валидация пропсов компонента**
```typescript
type ButtonProps<T extends React.ElementType> = {
  as?: T;
} & React.ComponentProps<T>;

// ✅ Можно использовать с любым HTML элементом
// ❌ Нельзя с несуществующими тегами
```

## **2. Ограничение дженериков в API функциях**
```typescript
function getById<T extends { id: string }>(items: T[], id: string): T | undefined {
  return items.find(item => item.id === id);
}
// Гарантирует, что у объектов есть id
```

## **3. Фильтрация типов из union**
```typescript
type StringsOnly<T> = T extends string ? T : never;
type Result = StringsOnly<"a" | 1 | "b" | true>; // "a" | "b"
```

## **4. Условный рендеринг в компонентах**
```typescript
type ShowWhen<T extends boolean> = T extends true 
  ? { children: React.ReactNode } 
  : never;

function Conditional({ show, children }: { show: boolean } & ShowWhen<typeof show>) {
  return show ? <div>{children}</div> : null;
}
```

## **5. Валидация ключей объекта**
```typescript
function getValue<T, K extends keyof T>(obj: T, key: K): T[K] {
  return obj[key];
}
// ❌ Не даст передать несуществующий ключ
```

## **6. Создание theme-провайдера**
```typescript
type ThemeColors = "primary" | "secondary" | "error";
type ColorVariants<T extends ThemeColors> = `${T}-light` | `${T}-dark`;

const variant: ColorVariants<"primary"> = "primary-light"; // ✅
// const variant: ColorVariants<"unknown"> = ""; // ❌
```

## **7. Типизация ивентов**
```typescript
type ExtractEvent<T, K extends keyof T> = T[K] extends (...args: any[]) => any 
  ? Parameters<T[K]>[0] 
  : never;

// Получаем тип первого параметра обработчика
```

## **8. Конфигурация с дефолтами**
```typescript
type WithDefaults<T extends object, D extends Partial<T>> = Omit<T, keyof D> & Partial<D>;

type UserConfig = { theme: string; language: string };
type DefaultConfig = WithDefaults<UserConfig, { theme: "dark" }>;
// theme теперь опциональный с дефолтом "dark"
```

## **9. Валидация статусов state machine**
```typescript
type ValidTransition<State extends string, From extends State, To extends State> = 
  From extends "idle" ? "loading" :
  From extends "loading" ? "success" | "error" : never;

function transition<From extends State, To extends ValidTransition<State, From, To>>(
  from: From, to: To
) {}
```

## **10. Полиморфные компоненты**
```typescript
type PolymorphicProps<T extends React.ElementType> = {
  component?: T;
} & Omit<React.ComponentProps<T>, "component">;

function Box<T extends React.ElementType = "div">({
  component: Component = "div",
  ...props
}: PolymorphicProps<T>) {
  return <Component {...props} />;
}
// <Box component="button" onClick={...} /> ✅
```

**Итог:** `extends` везде где нужны ограничения типов, условная логика на уровне типов или валидация на этапе компиляции!
