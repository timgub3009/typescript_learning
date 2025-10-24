### Суть `unknown`

`unknown` — это **безопасный `any`**. Он принимает любое значение, но запрещает его использование без проверки типа. Это защита от случайных ошибок.

---

### 10 конкретных примеров пользы

**1. Данные из API**
```typescript
const response: unknown = await fetch('/api/data');
// Без проверки TypeScript не даст работать с response
if (typeof response === 'object' && response && 'users' in response) {
    response.users // Теперь безопасно
}
```

**2. JSON-парсинг**
```typescript
// JSON.parse() возвращает any, но лучше unknown
const data: unknown = JSON.parse('{"id": 1}');
// Вынуждает проверить структуру
if (data && typeof data === 'object' && 'id' in data) {
    console.log(data.id);
}
```

**3. Обработка ошибок**
```typescript
try {
    // ...
} catch (error: unknown) { // error теперь unknown, а не any
    if (error instanceof Error) {
        console.log(error.message); // Безопасно
    }
}
```

**4. Функции-обертки**
```typescript
function safeStringify(data: unknown): string {
    return JSON.stringify(data); // Принимает любой тип, но явно это показывает
}
```

**5. Валидация данных**
```typescript
function isValidUser(data: unknown): data is {name: string} {
    return !!(
        typeof data === 'object' && 
        data && 
        'name' in data && 
        typeof (data as any).name === 'string'
    );
}
```

**6. Универсальные обработчики**
```typescript
function processInput(input: unknown): string {
    if (typeof input === 'string') return input;
    if (typeof input === 'number') return input.toString();
    return 'default';
}
```

**7. Миграция с JavaScript**
```typescript
// Вместо any при переносе JS кода
const legacyData: unknown = getOldJsData();
// Теперь придется явно проверять типы
```

**8. Колбэки с произвольными данными**
```typescript
function withCallback(callback: (data: unknown) => void) {
    const randomData = Math.random() > 0.5 ? 'text' : {value: 1};
    callback(randomData); // Колбэк должен уметь работать с unknown
}
```

**9. Хранилища состояний**
```typescript
class Store {
    private state: unknown;
    
    getState<T>(): T {
        return this.state as T; // Пользователь сам указывает тип
    }
}
```

**10. Интерсепторы**
```typescript
function apiInterceptor(response: unknown): unknown {
    // Можем логировать, трансформировать, но не знаем точную структуру
    console.log('Received:', response);
    return response; // Возвращаем тоже unknown
}
```

---

### Главное правило

**Используй `unknown`, когда тип действительно неизвестен на этапе компиляции. Используй проверки типов, чтобы "превратить" `unknown` в конкретный тип.**

Это защищает от:
- Ошибок времени выполнения
- Случайного доступа к несуществующим свойствам
- Неправильных предположений о структуре данных

`unknown` — это не ограничение, а инструмент для написания более надежного кода.
