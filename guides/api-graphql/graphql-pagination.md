# GraphQL пагинация

В этом руководстве вы узнаете, как реализовать пагинацию в вашем GraphQL API.

При работе с большим набором записей вам может понадобиться получить только первое __N__ количество элементов. Например, давайте начнем с базовой схемы GraphQL для приложения Todo:

```graphql
type Todo @model {
  id: ID!
  title: String!
  description: String 
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
При запросе API с помощью метода `listTodos` типом возвращаемого значения будет `ModelTodoConnection`, то есть вы можете вернуть как массив `Todos`, так и `nextToken`.

`nextToken` - это то, что используется для разбивки на страницы. Если `nextToken` имеет значение `null`, это означает, что больше нет данных для возврата из API. Если присутствует `nextToken`, вы можете использовать значение в качестве аргумента для следующего запроса` listTodos`, чтобы вернуть следующий набор выбора из API.

Чтобы проверить это, попробуйте создать 5 записей, используя такую ​​мутацию:

```sh
mutation createTodo {
  createTodo(input: {
    title: "Todo 1"
    description: "My first todo"
  }) {
    id
    title
    description
  }
}
```
Затем вы можете установить ограничение на количество записей в запросе, передав аргумент `limit`. В этом запросе вы установите ограничение на 2 элемента и запросите `nextToken` в качестве возвращаемого значения:

```graphql
query listTodos {
  listTodos(limit: 2) {
    items {
      id
      title
      description
    }
    nextToken
  }
}
```
Теперь, чтобы запросить следующие 2 элемента из API, вы можете передать этот `nextToken` в качестве аргумента:

```graphql
query listTodos {
  listTodos(limit: 2, nextToken: <your_token>) {
    items {
      id
      title
      description
    }
    nextToken
  }
}
```

Когда не осталось других элементов для возврата, для параметра `nextToken` в ответе будет установлено значение `null`.

## Запросы из JavaScript

Запрос `listTodos` должен был создаться для вас автоматически с помощью CLI, но для справки он должен выглядеть примерно так:

```js
const listTodos = `
  query listTodos($limit: Int) {
    listTodos(limit: $limit) {
      items {
        id
        title
        description
      }
      nextToken
    }
  }
`
```

Чтобы передать `limit` в запросе из JavaScript, вы можете использовать следующий код, установив ограничение как переменную:

```js
import { API } from 'aws-amplify';

async function fetchTodos() {
  const todoData = await API.graphql({
    query: listTodos,
    variables: {
      limit: 2
    }
  })
  console.log({ todoData })
}
```

Данные, возвращаемые из запроса API, должны выглядеть следующим образом (с массивом элементов, содержащим количество созданных элементов):

```graphql
{
  "data" {
    "listTodos" {
      "items": [{ id: "001", title: "Todo 1", description: "My first todo" }],
      "nextToken": "<token-id>"
    }
  }
}
```
Чтобы установить `nextToken` в запросе из JavaScript, вы можете использовать следующий код:

```js
import { API } from 'aws-amplify';

async function fetchTodos() {
  const todoData = await API.graphql({
    query: listTodos,
    variables: {
      limit: 2,
      nextToken: "<token-id>"
    }
  })
  console.log({ todoData })
}
```