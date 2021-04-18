# Вызов DynamoDB из Lambda в Node.js

Как взаимодействовать с базой данных DynamoDB из лямбда-функции в Node.js

Самый простой способ взаимодействия с DynamoDB из Lambda в среде Node.js - использовать [клиент документа DynamoDB](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html). В этом руководстве вы узнаете, как взаимодействовать с базой данных DynamoDB из функции Lambda, используя среду выполнения Node.js.

Вы научитесь выполнять операции [put](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#put-property), [get](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#get-property), [scan](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#scan-property), и [query](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html#query-property).

### Создание элемента в DynamoDB из Lambda

Чтобы создать элемент в DynamoDB, вы можете использовать метод `put`:

```js
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();

const params = {
  TableName : 'your-table-name',
  /* Свойства элемента будут зависеть от задач вашего приложения */
  Item: {
     id: '12345',
     price: 100.00
  }
}

async function createItem(){
  try {
    await docClient.put(params).promise();
  } catch (err) {
    return err;
  }
}

exports.handler = async (event) => {
  try {
    await createItem()
    return { body: 'Successfully created item!' }
  } catch (err) {
    return { error: err }
  }
};
```

### Получение элемента по первичному ключу в DynamoDB из Lambda

Чтобы получить элемент по первичному ключу в DynamoDB, вы можете использовать метод `get`. Запрос `get` возвращает один элемент с учетом первичного ключа этого элемента:

```js
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();

const params = {
  TableName : 'your-table-name',
  /* Свойства элемента будут зависеть от задач вашего приложения */
  Key: {
    id: '12345'
  }
}

async function getItem(){
  try {
    const data = await docClient.get(params).promise()
    return data
  } catch (err) {
    return err
  }
}

exports.handler = async (event, context) => {
  try {
    const data = await getItem()
    return { body: JSON.stringify(data) }
  } catch (err) {
    return { error: err }
  }
}
```

### Сканирование таблицы

Метод `scan` возвращает один или несколько элементов и атрибутов элементов, обращаясь к каждому элементу в таблице или вторичном индексе (ограничение в 1 МБ данных).

```js
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();

const params = {
  TableName : 'your-table-name'
}

async function listItems(){
  try {
    const data = await docClient.scan(params).promise()
    return data
  } catch (err) {
    return err
  }
}

exports.handler = async (event, context) => {
  try {
    const data = await listItems()
    return { body: JSON.stringify(data) }
  } catch (err) {
    return { error: err }
  }
}
```

### Запрос таблицы

Метод `query` возвращает один или несколько элементов и атрибутов элементов, запрашивая элементы из таблицы по первичному ключу или вторичному индексу.

```js
const AWS = require('aws-sdk');
const docClient = new AWS.DynamoDB.DocumentClient();

var params = {
  TableName: 'your-table-name',
  IndexName: 'some-index',
  KeyConditionExpression: '#name = :value',
  ExpressionAttributeValues: { ':value': 'shoes' },
  ExpressionAttributeNames: { '#name': 'name' }
}

async function queryItems(){
  try {
    const data = await docClient.query(params).promise()
    return data
  } catch (err) {
    return err
  }
}

exports.handler = async (event, context) => {
  try {
    const data = await queryItems()
    return { body: JSON.stringify(data) }
  } catch (err) {
    return { error: err }
  }
}
```