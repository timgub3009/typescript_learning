## Оператор `key in` в TypeScript

**`key in`** — это оператор, используемый в **Mapped Types** (сопоставленных типах) для итерации по ключам другого типа.

---

## Базовый синтаксис

```typescript
// Общий вид:
type NewType = {
  [Key in KeyType]: ValueType;
};
```

---

## Основные варианты использования

### 1. **Создание типа из union литералов**

```typescript
type Status = 'pending' | 'success' | 'error';

// Создаем объектный тип, где каждый ключ - элемент union
type StatusObject = {
  [Key in Status]: boolean;
};
// Эквивалентно:
// type StatusObject = {
//   pending: boolean;
//   success: boolean;
//   error: boolean;
// };
```

### 2. **Итерация по ключам существующего типа (`keyof`)**

```typescript
interface User {
  id: number;
  name: string;
  email: string;
}

// Делаем все поля опциональными
type PartialUser = {
  [Key in keyof User]?: User[Key];
};
// Эквивалентно:
// type PartialUser = {
//   id?: number;
//   name?: string;
//   email?: string;
// };
```

### 3. **Изменение модификаторов (`readonly`, `?`)**

```typescript
interface Config {
  readonly apiUrl: string;
  timeout?: number;
}

// Убираем readonly и делаем все поля обязательными
type MutableConfig = {
  -readonly [Key in keyof Config]-?: Config[Key];
};
// Эквивалентно:
// type MutableConfig = {
//   apiUrl: string;
//   timeout: number;
// };
```

---

## Практические примеры

### Пример 1: **Создание формы на основе модели**

```typescript
interface Product {
  id: number;
  name: string;
  price: number;
  category: string;
}

// Форма для создания продукта (без id, все поля обязательные)
type ProductForm = {
  [Key in Exclude<keyof Product, 'id'>]: Product[Key];
};
// Эквивалентно:
// type ProductForm = {
//   name: string;
//   price: number;
//   category: string;
// };
```

### Пример 2: **Генерация селекторов для Redux/Zustand**

```typescript
type State = {
  user: { name: string; email: string };
  products: { id: number; title: string }[];
  cart: { items: number; total: number };
};

// Автоматически генерируем тип для селекторов
type Selectors = {
  [Key in keyof State]: (state: State) => State[Key];
};
// Эквивалентно:
// type Selectors = {
//   user: (state: State) => { name: string; email: string };
//   products: (state: State) => { id: number; title: string }[];
//   cart: (state: State) => { items: number; total: number };
// };
```

### Пример 3: **Создание API клиента**

```typescript
type Endpoints = 'users' | 'products' | 'orders';

// Генерируем методы для каждого endpoint
type ApiClient = {
  [Key in Endpoints]: {
    get: (id: string) => Promise<any>;
    list: () => Promise<any[]>;
    create: (data: any) => Promise<any>;
  };
};
// Эквивалентно:
// type ApiClient = {
//   users: { get: (id: string) => Promise<any>; ... };
//   products: { get: (id: string) => Promise<any>; ... };
//   orders: { get: (id: string) => Promise<any>; ... };
// };
```

---

## Продвинутые техники

### 1. **Комбинация с условными типами**

```typescript
interface Entity {
  id: number;
  name: string;
  createdAt: Date;
  updatedAt: Date;
}

// Преобразуем Date в string, остальные типы оставляем как есть
type EntityDTO = {
  [Key in keyof Entity]: Entity[Key] extends Date 
    ? string 
    : Entity[Key];
};
// Эквивалентно:
// type EntityDTO = {
//   id: number;
//   name: string;
//   createdAt: string;
//   updatedAt: string;
// };
```

### 2. **Фильтрация ключей**

```typescript
interface UserSettings {
  theme: 'light' | 'dark';
  language: string;
  notifications: boolean;
  // Внутренние поля, не для API
  _version: number;
  _syncStatus: string;
}

// Только публичные настройки (исключаем поля, начинающиеся с _)
type PublicSettings = {
  [Key in keyof UserSettings as Key extends `_${string}` 
    ? never 
    : Key]: UserSettings[Key];
};
// Эквивалентно:
// type PublicSettings = {
//   theme: 'light' | 'dark';
//   language: string;
//   notifications: boolean;
// };
```

### 3. **Префиксы/суффиксы для ключей**

```typescript
interface Actions {
  load: () => void;
  save: (data: any) => void;
  delete: (id: number) => void;
}

// Добавляем префикс к каждому ключу
type ActionTypes = {
  [Key in keyof Actions as `ACTION_${Uppercase<Key>}`]: Actions[Key];
};
// Эквивалентно:
// type ActionTypes = {
//   ACTION_LOAD: () => void;
//   ACTION_SAVE: (data: any) => void;
//   ACTION_DELETE: (id: number) => void;
// };
```

---

## Встроенные Mapped Types в TypeScript

TypeScript уже включает несколько полезных mapped types:

### `Readonly<T>`
```typescript
type Readonly<T> = {
  readonly [Key in keyof T]: T[Key];
};
```

### `Partial<T>`
```typescript
type Partial<T> = {
  [Key in keyof T]?: T[Key];
};
```

### `Required<T>`
```typescript
type Required<T> = {
  [Key in keyof T]-?: T[Key];
};
```

### `Pick<T, K>`
```typescript
type Pick<T, K extends keyof T> = {
  [Key in K]: T[Key];
};
```

### `Record<K, T>`
```typescript
type Record<K extends keyof any, T> = {
  [Key in K]: T;
};
```

---

## Пример: Создание собственного utility type

```typescript
// Делает все поля nullable (могут быть null)
type Nullable<T> = {
  [Key in keyof T]: T[Key] | null;
};

interface Product {
  id: number;
  name: string;
  price: number;
}

type NullableProduct = Nullable<Product>;
// Эквивалентно:
// type NullableProduct = {
//   id: number | null;
//   name: string | null;
//   price: number | null;
// };
```

---

## Когда использовать `key in`?

**Используйте когда:**
- Нужно создать тип на основе ключей другого типа
- Хотите автоматически генерировать связанные типы
- Работаете с конфигурациями, формами, API
- Нужно изменить модификаторы полей (readonly, optional)

**Не используйте когда:**
- Можно обойтись простыми типами
- Структура статична и не будет меняться
- Избыточная сложность для простой задачи

---

## Итог

**`key in`** — это мощный инструмент для:
- **Автоматизации** создания связанных типов
- **Согласованности** между различными частями системы типов
- **Поддержки** DRY (Don't Repeat Yourself) принципов
- **Гибкости** при работе с динамическими структурами данных

Это основа для создания переиспользуемых utility types и поддержания типобезопасности в больших проектах!
