### Суть разницы: `&` vs `|`

**`&` (Intersection type) - "И"**  
Объединяет типы, создавая новый тип, который содержит ВСЕ свойства из всех исходных типов.

**`|` (Union type) - "ИЛИ"**  
Создает тип, который может быть ЛЮБЫМ из исходных типов, но не всеми одновременно.

---

### Пример 1: Базовое использование с объектами

```typescript
type User = { name: string; age: number };
type Admin = { role: string; permissions: string[] };

// & - должен иметь ВСЕ свойства
const superUser: User & Admin = {
    name: "Alice",
    age: 30,
    role: "admin", 
    permissions: ["read", "write"] // Все свойства обязательны!
};

// | - может быть ЛЮБЫМ из типов
const person: User | Admin = {
    name: "Bob", 
    age: 25
    // Может быть только User
};
// ИЛИ
const person2: User | Admin = {
    role: "user",
    permissions: ["read"]
    // Может быть только Admin
};
```

---

### Пример 2: Работа с функциями

```typescript
type StringProcessor = (input: string) => string;
type NumberProcessor = (input: number) => number;

// & - функция должна обрабатывать И строки И числа
const universalProcessor: StringProcessor & NumberProcessor = (input: any) => {
    return input; // Должна работать с обоими типами
};

// | - функция может обрабатывать строки ИЛИ числа
const specificProcessor: StringProcessor | NumberProcessor = (input: string) => {
    return input.toUpperCase(); // Достаточно реализовать один вариант
};
```

---

### Пример 3: Практический кейс - конфигурация

```typescript
type DatabaseConfig = { host: string; port: number };
type CacheConfig = { redisUrl: string; ttl: number };
type AuthConfig = { secret: string; expiresIn: string };

// & - полная конфигурация приложения (должна включать ВСЕ настройки)
type FullConfig = DatabaseConfig & CacheConfig & AuthConfig;
const config: FullConfig = {
    host: "localhost",
    port: 5432,
    redisUrl: "redis://localhost",
    ttl: 3600,
    secret: "secret-key",
    expiresIn: "7d"
    // Все свойства обязательны!
};

// | - частичная конфигурация (может быть любой комбинацией)
type PartialConfig = DatabaseConfig | CacheConfig | AuthConfig;
const dbOnly: PartialConfig = { host: "localhost", port: 5432 }; // Только база
const cacheOnly: PartialConfig = { redisUrl: "redis://...", ttl: 3600 }; // Только кэш
```

---

### Ключевые отличия

**`&` (Intersection):**
- Комбинирует типы
- Результат имеет ВСЕ свойства
- Используется для создания сложных типов из простых
- "МUST HAVE ALL"

**`|` (Union):**
- Предоставляет варианты
- Результат может быть ЛЮБЫМ из типов
- Используется для гибкости и обработки разных случаев
- "CAN BE ANY"

### Когда что использовать:
- **`&`** - когда нужен объект, сочетающий несколько ролей
- **`|`** - когда переменная может принимать разные формы

**Совершенно верно! Это ключевое ограничение Union Types (`|`).**

### Проблема Union Types:

Когда у тебя `TFirstType | TSecType`, TypeScript разрешает доступ **только к общим свойствам** этих типов.

```typescript
type TFirstType = { id: number; value: string };
type TSecType = { id: number; name: string; age: number };

// Проблема:
const array: (TFirstType | TSecType)[] = [
    { id: 1, value: "test" },
    { id: 2, name: "John", age: 30 }
];

// ОШИБКА: Property 'name' does not exist on type 'TFirstType | TSecType'
console.log(array[0].name); // ❌ Не компилируется!
console.log(array[1].name); // ❌ Тоже ошибка!
```

**TypeScript не знает, какой именно тип в каждом элементе массива.**

---

### Решения:

**1. Type Guards (Проверки типов)**
```typescript
// Проверяем наличие свойства
if ('name' in array[0]) {
    console.log(array[0].name); // ✅ Теперь TypeScript знает, что это TSecType
}

// Или пользовательская type guard
function isTSecType(item: TFirstType | TSecType): item is TSecType {
    return 'name' in item;
}

if (isTSecType(array[1])) {
    console.log(array[1].name); // ✅ Безопасно
    console.log(array[1].age);  // ✅ Тоже работает
}
```

**2. Discriminated Unions (Размеченные объединения)**
```typescript
// Добавляем общее поле-метку
type TFirstType = { type: "first"; id: number; value: string };
type TSecType = { type: "second"; id: number; name: string; age: number };

const array: (TFirstType | TSecType)[] = [
    { type: "first", id: 1, value: "test" },
    { type: "second", id: 2, name: "John", age: 30 }
];

// Теперь можно безопасно обращаться
array.forEach(item => {
    if (item.type === "second") {
        console.log(item.name); // ✅ TypeScript знает, что это TSecType
        console.log(item.age);  // ✅
    } else {
        console.log(item.value); // ✅ А это TFirstType
    }
});
```

---

### Вывод:

**Union Types (`|`) хороши, когда:**
- У типов есть общие свойства, к которым нужно обращаться
- Используются Discriminated Unions
- Планируется проверка типов перед использованием специфичных свойств

**Intersection Types (`&`) лучше, когда:**
- Нужно гарантированно иметь все свойства из всех типов
- Создается комбинированный тип, а не вариант выбора
