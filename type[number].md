## Indexed Access Types в TypeScript

### Что это такое?

**`FriendList['friends'][number]`** — это **Indexed Access Type** (тип индексного доступа), который позволяет извлекать типы из других типов по ключам или индексам.

---

## Разберем по частям:

### Базовый пример:
```typescript
type FriendList = {
  friends: Array<{
    name: string;
    age: number;
    email: string;
  }>;
  totalCount: number;
};

// Извлекаем тип элемента массива friends
type Friend = FriendList['friends'][number];
// Эквивалентно: { name: string; age: number; email: string; }
```

### Что дает `[number]`?

**`[number]`** — это специальный синтаксис для получения типа элемента массива/кортежа:

```typescript
// Для обычного массива
type StringArray = string[];
type StringArrayElement = StringArray[number]; // string

// Для типизированного массива
type UserArray = { id: number; name: string }[];
type User = UserArray[number]; // { id: number; name: string }
```

---

## Когда это использовать?

### 1. **DRY (Don't Repeat Yourself) - избегание дублирования**

```typescript
// ПЛОХО: дублирование типа
type User = {
  id: number;
  name: string;
  email: string;
};

type UserResponse = {
  data: User[];
  total: number;
};

// ХОРОШО: переиспользование типа
type UserResponse = {
  data: User[];
  total: number;
};

type UserFromResponse = UserResponse['data'][number];
// UserFromResponse === User
```

### 2. **Извлечение типов из сложных структур**

```typescript
type ApiResponse = {
  status: 'success' | 'error';
  data: {
    users: Array<{
      id: number;
      profile: {
        name: string;
        avatar: string;
      };
    }>;
    pagination: {
      page: number;
      total: number;
    };
  };
};

// Извлекаем тип пользователя
type User = ApiResponse['data']['users'][number];
// { id: number; profile: { name: string; avatar: string; } }

// Извлекаем тип профиля
type Profile = User['profile'];
// { name: string; avatar: string; }
```

### 3. **Работа с кортежами (tuples)**

```typescript
type ColorTuple = [string, string, string]; // [red, green, blue]
type ColorComponent = ColorTuple[number]; // string

type MixedTuple = [string, number, boolean];
type Mixed = MixedTuple[number]; // string | number | boolean
```

### 4. **Извлечение типов из объединений (unions)**

```typescript
type ApiResponses = 
  | { type: 'user'; data: { id: number; name: string } }
  | { type: 'product'; data: { id: number; price: number } };

type ResponseData = ApiResponses['data'];
// { id: number; name: string } | { id: number; price: number }
```

---

## Практические примеры использования

### Пример 1: Типы для форм на основе API
```typescript
type User = {
  id: number;
  name: string;
  email: string;
  address: {
    street: string;
    city: string;
  };
};

// Форма редактирования использует те же типы, но без id
type EditUserForm = Omit<User, 'id'>;
type AddressForm = User['address'];

// Автодополнение работает!
const form: EditUserForm = {
  name: "John",
  email: "john@example.com",
  address: {
    street: "123 Main St", // ✅ автодополнение
    city: "New York"       // ✅ автодополнение
  }
};
```

### Пример 2: Типы для компонентов
```typescript
type TableProps<T> = {
  data: T[];
  columns: Array<{
    key: keyof T;
    title: string;
    render: (value: T[keyof T], row: T) => React.ReactNode;
  }>;
};

type User = {
  id: number;
  name: string;
  email: string;
};

// Автоматически получаем тип для данных таблицы
type TableUser = TableProps<User>['data'][number]; // User

// Автоматически получаем тип для колонок
type TableColumn = TableProps<User>['columns'][number];
```

### Пример 3: Конфигурации и настройки
```typescript
type AppConfig = {
  database: {
    host: string;
    port: number;
    tables: Array<{
      name: string;
      columns: string[];
    }>;
  };
  cache: {
    enabled: boolean;
    ttl: number;
  };
};

// Извлекаем только настройки БД
type DatabaseConfig = AppConfig['database'];

// Извлекаем тип таблицы
type TableConfig = DatabaseConfig['tables'][number];
```

---

## Почему именно `[number]`?

### Альтернативы и почему они хуже:

**1. Рукописное дублирование (плохо):**
```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

type UserArray = User[]; // Дублирование
```

**2. Вспомогательные типы (лучше):**
```typescript
type ArrayElement<T> = T extends (infer U)[] ? U : never;
type User = ArrayElement<UserArray>;
```

**3. `[number]` (идеально для простых случаев):**
```typescript
type User = UserArray[number]; // Просто и понятно
```

---

## Продвинутые техники

### Комбинация с `keyof`:
```typescript
type User = {
  id: number;
  name: string;
  email: string;
};

type UserPropertyTypes = User[keyof User]; // number | string
```

### Извлечение типов из функций:
```typescript
type AsyncFunction = (...args: any[]) => Promise<any>;
type PromiseResult<T> = T extends Promise<infer U> ? U : never;

type ApiCall = () => Promise<{ data: User[] }>;
type ApiResponse = PromiseResult<ReturnType<ApiCall>>; // { data: User[] }
type ApiData = ApiResponse['data'][number]; // User
```

---

## Итог

**`TypeName['key'][number]` полезен когда:**
- Нужно избежать дублирования типов
- Работаете с ответами API и сложными структурами
- Хотите сохранить связь между типами
- Нужно автоматическое обновление типов при изменении базового типа

**Почему `number`?** Потому что массивы индексируются числами, и это специальный синтаксис TypeScript для получения типа элементов массива.

Это мощный инструмент для создания поддерживаемой и типобезопасной codebase!
