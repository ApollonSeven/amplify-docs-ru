# Подключение REST API к Lambda функциям

В этом руководстве вы узнаете, как подключить REST API к существующей Lambda функции.

Для начала создайте новый API:

```sh
amplify add api

? Please select from one of the below mentioned services: REST
? Provide a friendly name for your resource to be used as a label for this category in the project: myapi
? Provide a path (e.g., /book/{isbn}): /hello
? Choose a Lambda source: Use a Lambda function already added in the current Amplify project
? Choose the Lambda function to invoke by this path: <your-function-name>
? Restrict API access: N
? Do you want to add another path: N
```

Сделайте деплой API:

```sh
amplify push
```

Теперь ваш API готов к использованию!

Чтобы узнать больше о том, как взаимодействовать с API из клиентского приложения, ознакомьтесь с документацией [здесь](~/lib/restapi/getting-started.md)!