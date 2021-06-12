Вы можете вызывать AppSync GraphQL API из NodeJS-приложения или Lambda-функции. Возьмем базовое `Todo`-приложение для примера:

```graphql
type Todo @model {
  id: ID!
  name: String
  description: String
}
```

Для этого API буду доступны `create`, `read`, `update`, и `delete` операции. Давайте посмотрим как выполнить __query__ и __mutation__ запросы из Lambda-функции используя Node.js.

## Query запрос

```javascript
const axios = require('axios');
const gql = require('graphql-tag');
const graphql = require('graphql');
const { print } = graphql;

const listTodos = gql`
  query listTodos {
    listTodos {
      items {
        id
        name
        description
      }
    }
  }
`

exports.handler = async (event) => {
  try {
    const graphqlData = await axios({
      url: process.env.API_URL,
      method: 'post',
      headers: {
        'x-api-key': process.env.API_<YOUR_API_NAME>_GRAPHQLAPIKEYOUTPUT
      },
      data: {
        query: print(listTodos),
      }
    });
    const body = {
        graphqlData: graphqlData.data.data.listTodos
    }
    return {
        statusCode: 200,
        body: JSON.stringify(body),
        headers: {
            "Access-Control-Allow-Origin": "*",
        }
    }
  } catch (err) {
    console.log('error posting to appsync: ', err);
  } 
}
```

## Mutation запрос

В этом примере мы создаем мутацию показывающую как передавать переменные в качестве аргументов.

```js
const axios = require('axios');
const gql = require('graphql-tag');
const graphql = require('graphql');
const { print } = graphql;

const createTodo = gql`
  mutation createTodo($input: CreateTodoInput!) {
    createTodo(input: $input) {
      id
      name
      description
    }
  }
`

exports.handler = async (event) => {
  try {
    const graphqlData = await axios({
      url: process.env.API_URL,
      method: 'post',
      headers: {
        'x-api-key': process.env.API_<YOUR_API_NAME>_GRAPHQLAPIKEYOUTPUT
      },
      data: {
        query: print(createTodo),
        variables: {
          input: {
            name: "Hello world!",
            description: "My first todo"
          }
        }
      }
    });
    const body = {
      message: "successfully created todo!"
    }
    return {
      statusCode: 200,
      body: JSON.stringify(body),
      headers: {
          "Access-Control-Allow-Origin": "*",
      }
    }
  } catch (err) {
    console.log('error creating todo: ', err);
  } 
}
```

## Подписание запроса из Lambda

Что если мы работаем с кастомной подписью запроса?
Давайте взглянем на другой пример схемы, которая использует `iam` авторизацию.

```graphql
type Todo @model @auth (
    rules: [
        { allow: private, provider: iam }
    ]
) {
  id: ID!
  name: String
  description: String
}
```

Создайте Lambda-функцию командой `amplify add function` и убедитесь что дали доступ к вашему GraphQL API при вызове `amplify add function` потока.

CLI автоматически настроит IAM-роль необходимую Lambda-функции для вызова GraphQL API. Следующая функция будет подписывать запрос и использует переменные окружения для доступа в AppSync и Region созданные при потоке `amplify add function`.

```javascript
const https = require('https');
const AWS = require("aws-sdk");
const urlParse = require("url").URL;
const appsyncUrl = process.env.API_<YOUR_API_NAME>_GRAPHQLAPIENDPOINTOUTPUT;
const region = process.env.REGION;
const endpoint = new urlParse(appsyncUrl).hostname.toString();
const graphqlQuery = require('./query.js').mutation;
const apiKey = process.env.API_<YOUR_API_NAME>_GRAPHQLAPIKEYOUTPUT;

exports.handler = async (event) => {
    const req = new AWS.HttpRequest(appsyncUrl, region);

    const item = {
        input: {
            name: "Lambda Item",
            description: "Item Generated from Lambda"
        }
    };

    req.method = "POST";
    req.path = "/graphql";
    req.headers.host = endpoint;
    req.headers["Content-Type"] = "application/json";
    req.body = JSON.stringify({
        query: graphqlQuery,
        operationName: "createTodo",
        variables: item
    });

    if (apiKey) {
        req.headers["x-api-key"] = apiKey;
    } else {
        const signer = new AWS.Signers.V4(req, "appsync", true);
        signer.addAuthorization(AWS.config.credentials, AWS.util.date.getDate());
    }

    const data = await new Promise((resolve, reject) => {
        const httpRequest = https.request({ ...req, host: endpoint }, (result) => {
            result.on('data', (data) => {
                resolve(JSON.parse(data.toString()));
            });
        });

        httpRequest.write(req.body);
        httpRequest.end();
    });

    return {
        statusCode: 200,
        body: data
    };
};
```

Наконец вы можете объявить выполняемую вами операцию GraphQL, в этом случае мутацию `createTodo`, в отдельной файле `query.js`:

```javascript
module.exports = {
    mutation: `mutation createTodo($input: CreateTodoInput!) {
      createTodo(input: $input) {
        id
        name
        description
      }
    }
    `
}
```
