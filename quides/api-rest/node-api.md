# NodeJS API

В этом руководстве вы узнаете, как развернуть API Node.js.

## 1. Инициализируем новый проект Amplify

```sh
amplify init

# Следуйте инструкциям, чтобы дать проекту имя, имя среды и установить текстовый редактор по умолчанию.
# Примите значения по умолчанию для всего остального и выберите свой профиль AWS.
```

## 2. Добавьте API и функцию

```sh
amplify add api

? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: nodeapi
? Provide a path (e.g., /book/{isbn}): /hello
? Choose a Lambda source: Create a new Lambda function
? Provide a friendly name for your resource to be used as a label for this category in the project: greetingfunction
? Provide the AWS Lambda function name: greetingfunction
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? N
? Do you want to invoke this function on a recurring schedule? N
? Do you want to edit the local lambda function now? N
? Restrict API access: N
? Do you want to add another path? N
```

CLI должен создать новую функцию расположенную в **amplify/backend/function/greetingfunction**.

## 3. Обновление кода функции

Затем откройте  **amplify/backend/function/greetingfunction/src/index.js** и обновите код на следующий:

```js
exports.handler = async (event) => {
  const body = {
      message: "Hello from Lambda"
  }
  const response = {
      statusCode: 200,
      body: JSON.stringify(body),
      headers: {
          "Access-Control-Allow-Origin": "*",
      }
  };
  return response;
};
```

## 4. Загрузите API

Для загрузки API запустите команду `push`:

```sh
amplify push
```

## 5. Использование API

Here is how you can send a GET request to the API.

__JS__
```js
import { API } from 'aws-amplify';

const response = await API.get('nodeapi', '/hello');
```
__IOS__
```swift
func getGreeting() {
    let request = RESTRequest(path: "/hello", body: nil)
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
__Android__
```kotlin
suspend fun getMessage() {
    val request = RestOptions.builder()
        .addPath("/hello")
        .build()
    try {
        val response = Amplify.API.get(request)
        Log.i("MyAmplifyApp", "GET succeeded: $response")
    } catch (failure: ApiException) {
        Log.e("MyAmplifyApp", "GET failed", failure)
    }
}
```

Чтобы узнать больше о взаимодействии с REST API с помощью Amplify, ознакомьтесь с полной документацией [здесь](~/lib//restapi/getting-started.md).

Конечная точка API находится в папке `aws-exports.js`.

Вы также можете напрямую взаимодействовать с API, используя этот URL-адрес и указанный путь:

```sh
curl https://<api-id>.execute-api.<api-region>.amazonaws.com/<your-env-name>/hello
```
