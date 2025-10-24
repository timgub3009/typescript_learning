### Суть разницы: `void` vs `never`

**`void`** - функция возвращает **отсутствие значения** (undefined)  
**`never`** - функция **никогда не возвращает управление** (бесконечный цикл, ошибка, исключение)

---

### 10 примеров использования

**1. `void` - обычная функция без return**
```typescript
function logMessage(message: string): void {
    console.log(message);
    // Неявно возвращает undefined
}
```

**2. `never` - функция, которая всегда выбрасывает ошибку**
```typescript
function throwError(message: string): never {
    throw new Error(message);
    // Код после throw никогда не выполнится
}
```

**3. `never` - бесконечный цикл**
```typescript
function infiniteLoop(): never {
    while (true) {
        // Бесконечно...
    }
    // Управление никогда не вернется
}
```

**4. `void` - callback в forEach**
```typescript
[1, 2, 3].forEach((item): void => {
    console.log(item);
    // return не нужен
});
```

**5. `never` - сужение типов в type guards**
```typescript
function assertNever(value: never): never {
    throw new Error(`Unexpected value: ${value}`);
}

function handleShape(shape: "circle" | "square") {
    switch (shape) {
        case "circle": return "round";
        case "square": return "angular";
        default: return assertNever(shape); // Если добавить новый тип, TypeScript предупредит
    }
}
```

**6. `void` - стрелочные функции**
```typescript
const onClick = (): void => {
    // Обработчик события не возвращает значение
    console.log("clicked");
};
```

**7. `never` - обработка исключений**
```typescript
function fail(): never {
    throw new Error("Something went wrong");
    // Функция завершается исключением, а не возвратом
}
```

**8. `void` - методы классов без возврата**
```typescript
class User {
    private name: string;
    
    setName(newName: string): void {
        this.name = newName;
        // Метод модифицирует состояние, но не возвращает значение
    }
}
```

**9. `never` - в условных типах для исключений**
```typescript
type NonNullable<T> = T extends null | undefined ? never : T;
// Если T extends null | undefined, то возвращается never (исключается)
```

**10. `void` vs `never` в обработчиках Promise**
```typescript
// void - асинхронная функция без значимого возврата
async function fetchData(): Promise<void> {
    await fetch('/api');
    // Возвращает Promise<undefined>
}

// never - Promise, который никогда не разрешится
async function eternalWait(): Promise<never> {
    while (true) {
        await new Promise(resolve => setTimeout(resolve, 1000));
    }
}
```

---

### Ключевые отличия

| Аспект | `void` | `never` |
|--------|--------|---------|
| **Возвращаемое значение** | `undefined` | Ничего (функция не завершается) |
| **Когда используется** | Функция завершается, но нет значимого возврата | Функция никогда не завершает выполнение |
| **Присваиваемость** | Можно присвоить любому типу | Никакой тип нельзя присвоить `never` |
| **Использование в коде** | Обычные функции, методы | Исключения, бесконечные циклы, exhaustive checks |

### Практическое правило:
- Используй `void`, когда функция **завершается**, но ничего не возвращает
- Используй `never`, когда функция **никогда не завершается** или всегда прерывается исключением

```typescript
// void - можно игнорировать возвращаемое значение
const result: void = logMessage("test"); // OK

// never - нельзя даже присвоить
const x: never = throwError("error"); // ❌ Код недостижим
```
