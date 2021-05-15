# ExpressJS сервер

В этом руководстве вы узнаете, как развернуть веб-сервер [Express](https://expressjs.com/) с маршрутизацией.

## Инициализация проекта Amplify

Инициализируем новый проект Amplify:

```sh
amplify init

# Следуйте инструкциям, чтобы дать проекту имя, имя среды и установить текстовый редактор по умолчанию.
# Примите значения по умолчанию для всего остального и выберите свой профиль AWS.
```

### Создание API и функции

Затем создайте API и веб-сервер. Для этого вы можете использовать команду Amplify `add`:

```sh
amplify add api

? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: myapi
? Provide a path (e.g., /items): /items (or whatever path you would like)
? Choose a Lambda source: Create a new Lambda function
? Provide a friendly name for your resource to be used as a label for this category in the project: mylambda
? Provide the AWS Lambda function name: mylambda
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Serverless express function
? Do you want to access other resources created in this project from your Lambda function? N
? Do you want to invoke this function on a recurring schedule? N
? Do you want to edit the local lambda function now? N
? Restrict API access: N
? Do you want to add another path? N
```

CLI создал для вас несколько вещей:

- API эндпоинт
- Lambda функцию
- Веб-сервер с использованием [Serverless Express](https://github.com/awslabs/aws-serverless-express) в функции
- Некоторый шаблонный код для разных методов на маршруте `/items`

### Обновление кода функции

Откроем код для сервера.

Откройте __amplify/backend/function/mylambda/src/index.js__.

В этом файле вы увидите основной обработчик функции с `event` и` context`, проксируемыми на экспресс-сервер, расположенный в `./app.js` (не вносите никаких изменений в этот файл):

```js
const awsServerlessExpress = require('aws-serverless-express');
const app = require('./app');

const server = awsServerlessExpress.createServer(app);

exports.handler = (event, context) => {
  console.log(`EVENT: ${JSON.stringify(event)}`);
  awsServerlessExpress.proxy(server, event, context);
};

```

Далее откройте __amplify/backend/function/mylambda/src/app.js__.

Здесь вы увидите код экспресс-сервера и некоторый шаблон для различных HTTP-методов для объявленного вами маршрута.

Найдите маршрут для `app.get('/items')` и обновите его до следующего:

```js
// amplify/backend/function/mylambda/src/app.js
app.get('/items', function(req, res) {
  const items = ['hello', 'world']
  res.json({ success: 'get call succeed!', items });
});
```

### Развертывание сервиса

Чтобы развернуть API и функцию, мы можем запустить команду `push`:

```sh
amplify push
```

Теперь вы можете начать взаимодействие с API:

__JS__

```js
// get request
const items = await API.get('myapi', '/items')

// post with data
const data = { body: { items: ['some', 'new', 'items'] } }
await API.post('myapi', '/items', data)
```

__Android__

```kotlin
suspend fun getItems() {
    val options = RestOptions.builder()
        .addPath("/items")
        .build()
    try {
        val response = Amplify.API.get(options)
        Log.i("MyAmplifyApp", "GET succeeded: $response")
    } catch (failure: ApiException) {
        Log.e("MyAmplifyApp", "GET failed", failure)
    }
}
```

__IOS__

```swift
func getItems() {
    let request = RESTRequest(path: "/items", body: nil)
    _ = Amplify.API.get(request: request) { result in
        switch result {
        case .success(let data):
            let str = String(decoding: data, as: UTF8.self)
            print("Success \(str)")
        case .failure(let apiError):
            print("Failed", apiError)
        }
    }
}
```

Вы можете добавить дополнительный путь. Для этого запустите команду обновления:

```sh
amplify update api
```

Оттуда вы можете добавлять, обновлять или удалять пути. Чтобы узнать больше о взаимодействии с REST API с помощью Amplify, ознакомьтесь с полной документацией [здесь](~/lib/restapi/getting-started.md).

Эндпоинт API находится в папке `aws-exports.js`.

Вы также можете напрямую взаимодействовать с API, используя этот URL-адрес и указанный путь:

```sh
curl https://<api-id>.execute-api.<api-region>.amazonaws.com/<your-env-name>/items
```
