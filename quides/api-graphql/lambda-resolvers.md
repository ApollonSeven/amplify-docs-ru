# Как использовать Lambda GraphQL Resolvers

Как использовать Lambda GraphQL Resolvers для взаимодействия с другими сервисами

[Библиотека преобразований GraphQL](~/cli/graphql-transformer/function.md) предоставляет директиву `@function`, которая позволяет настраивать преобразователи функций AWS Lambda в вашем GraphQL API. В этом руководстве вы узнаете, как использовать Lambda-функции в качестве преобразователей GraphQL для взаимодействия с другими службами и API с помощью директивы `@function`.

## Создание базовых преобразователей функций запросов и мутаций

Для начала давайте взглянем на схему GraphQL с запросом и мутацией, в которой источник данных установлен как Lambda функция.

```graphql
# Запрос, возвращающий аргументы
type Query {
  echo(msg: String): String @function(name: "functionName-${env}")
}

# Мутация, складывающая два числа
type Mutation {
  add(number1: Int, number2: Int): Int @function(name: "functionName-${env}")
}
```
Используя директиву `@function`, вы можете указать Lambda функцию, которая будет вызываться как преобразователь GraphQL.

В этом руководстве вы узнаете, как включить преобразователи Lambda функций в GraphQL API.

### Создание функций

Для начала создадим первую Lambda функцию:

```sh
amplify add function

? Provide a friendly name for your resource to be used as a label for this category in the project: addingfunction
? Provide the AWS Lambda function name: echofunction
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? No
? Do you want to invoke this function on a recurring schedule? No
? Do you want to edit the local lambda function now? Yes
```

Обновите код функции (расположен в __amplify/backend/function/echofunction/src/index.js__) на следующее и нажмите enter:

```js
exports.handler = async (event) => {
    const response = event.arguments.msg
    return response;
};
```
Эта функция просто вернет значение свойства `msg`, переданное в качестве аргумента.

#### Информация об объекте event в Lambda

Объект `event` будет содержать следующие свойства:

```js
/*
event = {
  "typeName": "Query" or "Mutation", Filled dynamically based on @function usage location
  "fieldName": Filled dynamically based on @function usage location
  "arguments": { msg }, GraphQL field arguments
  "identity": {}, AppSync identity object
  "source": {}, The object returned by the parent resolver. E.G. if resolving field 'Post.comments', the source is the Post object
  "request": {}, AppSync request object. Contains things like headers
  "prev": {} If using the built-in pipeline resolver support, this contains the object returned by the previous function.
}
*/
```

В приведенной выше функции мы использовали свойство `arguments`, чтобы получить значения, переданные в качестве аргументов функции.

Затем создайте еще одну Lambda функцию:

```sh
amplify add function

? Provide a friendly name for your resource to be used as a label for this category in the project: addingfunction
? Provide the AWS Lambda function name: addfunction
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? No
? Do you want to invoke this function on a recurring schedule? No
? Do you want to edit the local lambda function now? Yes
```

Затем обновите код функции (расположен в __amplify/backend/function/addingfunction/src/index.js__) на следующее и нажмите enter:

```js
exports.handler = async (event) => {
    /* Складываем number1 и number2, возвращаем результат */
    const response = event.arguments.number1 + event.arguments.number2
    return response;
};
```

Эта функция сложит два числа и вернет результат.

### Создание GraphQL API

Теперь, когда функции созданы, вы можете создать GraphQL API:

```sh
amplify add api

? Please select from one of the below mentioned services: GraphQL
? Provide API name: gqllambda
? Choose the default authorization type for the API: API Key
? Enter a description for the API key: public (or some description)
? After how many days from now the API key should expire: 365 (or your preferred expiration)
? Do you want to configure advanced settings for the GraphQL API: N
? Do you have an annotated GraphQL schema? N
? Choose a schema template: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? Y
```

Затем обновите базовую схему GraphQL (расположена в __amplify/backend/api/gqllambda/schema.graphql__) на следующее и нажмите enter:

```graphql
type Query {
  echo(msg: String): String @function(name: "echofunction-${env}")
}

type Mutation {
  add(number1: Int, number2: Int): Int @function(name: "addfunction-${env}")
}
```

Теперь загрузите функции и GraphQL API:

```sh
amplify push
```

### Запрос GraphQL API

Теперь вы можете запустить следующие запросы и мутации для взаимодействия с API:

```sh
query echo {
  echo(msg: "Hello world!")
}

mutation add {
  add(number1: 1100, number2:100)
}
```

## Создание преобразователя, который взаимодействует с другим API

Теперь мы создадим функцию, которая будет взаимодействовать с публичным REST API криптовалюты.

Создадим новую функцию:

```sh
amplify add function
```

Затем обновите код функции (расположен в __amplify/backend/function/cryptofunction/src/index.js__) на следующее и нажмите enter:

```javascript
const axios = require('axios')

exports.handler = async (event) => {
    let limit = 10
    if (event.arguments.limit) {
        limit = event.arguments.limit
    }
    const url = `https://api.coinlore.net/api/tickers/?limit=${limit}`
    let response = await axios.get(url)
    return JSON.stringify(response.data.data);
};
```

Затем установите библиотеку axios в папку функции __src__ и вернитесь в корневой каталог:

```sh
cd amplify/backend/function/cryptofunction/src
npm install axios
cd ../../../../../
```
Теперь обновите схему GraphQL и добавьте преобразователь `getCoins` в схему запроса:

```graphql
type Query {
  echo(msg: String): String @function(name: "gqlfunc-${env}")
  getCoins(limit: Int): String @function(name: "cryptofunction-${env}")
}
```

Теперь загрузим обновления:

```sh
amplify push
```
Теперь вы можете запрашивать GraphQL API с помощью нового запроса `getCoins`.

#### Базовый запрос

```graphql
query getCoins {
  getCoins
}
```

#### Запрос с лимитом

```graphql
query getCoins {
  getCoins(limit: 100)
}
```

Чтобы узнать больше о директиве `@function`, ознакомьтесь с документацией GraphQL Transform [здесь](~/cli/graphql-transformer/function.md).
