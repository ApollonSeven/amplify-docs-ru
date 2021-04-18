# GraphQL запрос с сортировкой по дате

В этом руководстве вы узнаете, как реализовать сортировку в GraphQL API. В нашем примере вы сделаете сортировку результатов по дате в возрастающем или убывающем порядке, реализуя дополнительный шаблон доступа к данным с использованием глобального вторичного индекса DynamoDB с помощью директивы GraphQL Transform `@key`.

### Обзор

Для начала давайте начнем с базовой схемы GraphQL для приложения Todo:

```graphql
type Todo @model {
  id: ID!
  title: String!
}
```
Когда API создается с помощью директивы `@model`, для вас будут автоматически созданы следующие запросы:

```graphql
type Query {
  getTodo(id: ID!): Todo
  listTodos(filter: ModelTodoFilterInput, limit: Int, nextToken: String): ModelTodoConnection
}
```
Затем взгляните на тип `ModelTodoConnection`, чтобы получить представление о данных, которые будут возвращены при выполнении запроса `listTodos`:

```graphql
type ModelTodoConnection {
  items: [Todo]
  nextToken: String
}
```

По умолчанию запрос `listTodos` возвращает неупорядоченный массив `items`. Часто вам нужно будет отсортировать эти элементы по названию, по дате создания или каким-либо другим способом.

Чтобы включить это, вы можете использовать директиву [@key](~/cli/graphql-transformer/key.md). Эта директива позволит вам установить собственный ключ `sortKey` для любого поля вашего API.

### Реализация

В этом примере вы включите сортировку по полю `createdAt`. По умолчанию Amplify заполняет поле `createdAt` отметкой времени, если она не передана.

Чтобы включить это, обновите свою схему следующим образом:

```graphql
type Todo @model
  @key(name: "todosByDate", fields: ["type", "createdAt"], queryField: "todosByDate") {
  id: ID!
  title: String!
  type: String!
  createdAt: String!
}
```

> При создании Todo теперь вы должны заполнить поле `type`, чтобы это работало правильно.

Затем создайте несколько `todos`, обязательно заполните поле `type`:

```graphql
mutation createTodo {
  createTodo(input: {
    title: "Todo 1"
    type: "Todo"
  }) {
    id
    title
  }
}
```
Теперь вы можете запрашивать задачи по дате в порядке возрастания или убывания, используя новый запрос `todosByDate`:

```graphql
query todosByDate {
  todosByDate(
    type: "Todo"
    sortDirection: ASC
  ) {
    items {
      id
      title
      createdAt
    }
  }
}

query todosByDateDescending {
  todosByDate(
    type: "Todo"
    sortDirection: DESC
  ) {
    items {
      id
      title
      createdAt
    }
  }
}
```

Чтобы узнать больше о директиве `@key`, ознакомьтесь с документацией [здесь](~/cli/graphql-transformer/key.md)
