---
title: "Как создавать устойчивые компоненты во фронтенде: уроки из Чистой архитектуры"
description: "Применяем принципы Роберта Мартина из «Чистой архитектуры» — SOLID и метрику нестабильности — к фронтенд-компонентам на примерах React и TypeScript."
pubDate: 2025-06-30
category: Frontend
---

Перечитывал "Чистую архитектуру" Роберта Мартина и задумался, как применить её принципы к фронтенду. Особенно зацепила идея **устойчивых компонентов**. Разберём, что это и как их создавать.

## Что такое устойчивые компоненты?

Устойчивые компоненты, по Мартину:

- **Независимы**: не ломаются при изменении других модулей.
- **Инкапсулируют логику**: бизнес-правила отделены от UI и API.
- **Имеют чёткие границы**: взаимодействуют через строгие интерфейсы.

Они опираются на принципы SOLID:

1. **SRP**: один компонент — одна задача.
2. **OCP**: расширяем компонент без изменения кода.
3. **LSP**: компоненты взаимозаменяемы без последствий.

Мартин предлагает **метрику нестабильности** (I):

```
I = Ce / (Ca + Ce)
```

- **Ce**: сколько компонентов зависит от вашего.
- **Ca**: от скольки компонентов зависит ваш.

**I ≈ 0** — компонент устойчив, **I ≈ 1** — нестабилен.

## Как применить во фронтенде?

Фронтенд часто хаотичен и такого стремления к порядку как в бэкенде мы не добьемся никогда, но попытка не пытка. Выделим пару правил и постараемся их не забывать, пока будем разрабатывать свои приложения.

### 1. Инкапсуляция: отделяем UI от логики

Не смешивайте рендеринг, API и данные в одном компоненте. Разделяйте:

- **UI-компоненты**: только рендеринг.
- **Логика**: хуки или сервисы.

**Пример (React):**

```jsx
// UI-компонент
const UserList = ({ users, onSelect }) => (
  <ul>
    {users.map(user => (
      <li key={user.id} onClick={() => onSelect(user)}>
        {user.name}
      </li>
    ))}
  </ul>
);

// Контейнер (логика)
const UserListContainer = () => {
  const [users, setUsers] = useState([]);

  useEffect(() => {
    fetchUsers().then(setUsers);
  }, []);

  const handleSelect = user => {
    console.log(`Выбран: ${user.name}`);
  };

  return <UserList users={users} onSelect={handleSelect} />;
};
```

**Почему устойчиво?**

- `UserList` работает с props, не зная об источнике данных.
- `UserListContainer` изолирует логику и легко заменяется.

### 2. Чёткие границы: интерфейсы и типизация

Компоненты общаются через контракты (props, события). TypeScript фиксирует ожидания.

**Пример (TypeScript):**

```tsx
interface User {
  id: string;
  name: string;
}

interface UserListProps {
  users: User[];
  onSelect: (user: User) => void;
}

const UserList: React.FC<UserListProps> = ({ users, onSelect }) => (
  <ul>
    {users.map(user => (
      <li key={user.id} onClick={() => onSelect(user)}>
        {user.name}
      </li>
    ))}
  </ul>
);
```

**Почему устойчиво?**

- `UserListProps` — контракт, минимизирующий зависимости.
- Компонент не обращается к глобальному состоянию.

### 3. Устойчивость к изменениям: SOLID в деле

Применяем SOLID:

- **SRP**: форма рендерит UI, логика — в хуке.
- **OCP**: хуки и композиция для расширения.
- **LSP**: вариации компонентов (например, `<PrimaryButton>`) работают как базовый `<Button>`.

**Пример (OCP с хуком):**

```jsx
const useForm = initialValues => {
  const [values, setValues] = useState(initialValues);
  const handleChange = e => {
    setValues({ ...values, [e.target.name]: e.target.value });
  };
  return { values, handleChange };
};

const LoginForm = () => {
  const { values, handleChange } = useForm({ email: '', password: '' });

  const handleSubmit = () => {
    console.log('Отправка:', values);
  };

  return (
    <form onSubmit={handleSubmit}>
      <input name="email" value={values.email} onChange={handleChange} />
      <input name="password" value={values.password} onChange={handleChange} />
      <button type="submit">Войти</button>
    </form>
  );
};
```

### 4. Измеряем устойчивость

Для `UserListContainer`:

- **Ca = 2** (зависит от `fetchUsers` и `UserList`).
- **Ce = 0** (никто не зависит напрямую).
- **I = 0 / (2 + 0) = 0** — устойчив!

**Как сохранить устойчивость?**

- Минимизируйте зависимости через сервисы.
- Изолируйте через props и хуки.
- Проверяйте **Ca** и **Ce** при рефакторинге.

## Практические советы

1. **Структура проекта**:
   - `components/` — UI.
   - `hooks/` — логика.
   - `services/` — API.
   - `utils/` — функции.
2. **Тестирование**:
   - Юнит-тесты для хуков (Jest/Vitest).
   - UI-тесты.
3. **Избегайте хрупких зависимостей**
4. **Документация**:
   - Storybook для UI.
   - TypeScript/JSDoc для API.

## Зачем это нужно?

Устойчивые компоненты:

- Упрощают масштабирование.
- Делают рефакторинг безопасным.
- Помогают писать понятный код.
