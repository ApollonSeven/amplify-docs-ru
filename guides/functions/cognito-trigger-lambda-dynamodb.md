# Вызов DynamoDB с использованием AWS Cognito triggers

Как добавить запись информацией пользователя в DynamoDB после подтверждения регистрации

Если вы используете AWS Cogniton для обработки аутентификации в своем приложении, вы можете использовать триггеры для обработки событий аутентификации. К примеру вы можете отправлять приветственное письмо после авторизации пользователя.
Полную документацию по триггерам AWS cognito вы можете найти [здесь](https://docs.aws.amazon.com/cognito/latest/developerguide/cognito-user-identity-pools-working-with-aws-lambda-triggers.html).

В этой инструкции вы научитесь как использовать триггер после подтверждения для сохранения информации о пользователе в DynamoDB таблицу. 
Как упомянуто в прошло инструкции самый простой способ взаиомодействия с DynamoDB из Lambda функции в Node.js окружении это использовать [DynamoDB document клиент](https://docs.aws.amazon.com/AWSJavaScriptSDK/latest/AWS/DynamoDB/DocumentClient.html).

Используя этот подход вы можете подключить Cogntio Identity к профилю пользователя в вашем приложении и иметь возможность выводить список постов по автору и отображать их имя, email, дату создания и т.к. вместо их идентификаторов.

Главное преимущество такого метода это то что вам не нужно вручную создавать пользователя в вашем GraphQL API используя мутации, что является еще одной альтернативой.

Главная проблема такого решения это то, что если вы удалите пользователя из AWS Cognito, ваше приложение не будет знать об этом.

### Сценарий

После регистрации пользователя вы хотите создать запись в DynamoDB таблицу с информацией о пользователе.

### Создание GraphQL API

На этом этапе создайте вашу User таблицу, где будут сохраняться записи с информацией пользователя. 
In this step you will create your User table, where the entry with user's information will be saved. Это можно сделать используя Amplify GraphQL API.

Вы можете пропустить эту часть если вы уже имеете GraphQL API с таюлицей User.

```sh
amplify add api

? Please select from one of the below mentioned services: GraphQL
? Provide API name: myapi
? Choose the default authorization type for the API: API key
? Enter a description for the API key: public
? After how many days from now the API key should expire (1-365): 365
? Do you want to configure advanced settings for the GraphQL API: No, I am done.
? Do you have an annotated GraphQL schema? No
? Choose a schema template: Single object with fields (e.g., “Todo” with ID, name, description)
? Do you want to edit the schema now? Yes
```

CLI должна открыть схему GraphQL, расположенную в `amplify/backend/api/contactapi/schema.graphql`, в вашем текстовом редакторе. Обновите схему со следующими данными и сохраните файл:

```graphql
type User
  @model
{
  id: ID!
  name: String
  email: String
}
```


### Создание lambda функции

Эта функция будет вызвана после подтверждения пользователя.

```sh
amplify add function

? Provide a friendly name for your resource to be used as a label for this category in the project: mylambda
? Provide the AWS Lambda function name: mylambda
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? Y
? Select the category: storage
? Select the operations you want to permit for UserTable: create, read, update, delete
? Do you want to invoke this function on a recurring schedule? N
? Do you want to configure Lambda layers for this function? N
? Do you want to edit the local lambda function now? N
```

Затем откройте файл index.js относящийся к вашей новосозданной функции lambda и вставьте следующий код: 

```js
var aws = require('aws-sdk');
var ddb = new aws.DynamoDB();

exports.handler = async (event, context) => {
    
    let date = new Date();

    if (event.request.userAttributes.sub) {

        let params = {
            Item: {
                'id': {S: event.request.userAttributes.sub},
                '__typename': {S: 'User'},
                'name': {S: event.request.userAttributes.name},
                'email': {S: event.request.userAttributes.email},
                'createdAt': {S: date.toISOString()},
                'updatedAt': {S: date.toISOString()},
            },
            TableName: process.env.API_{YOUR_APP_NAME}_USERTABLE_NAME
        };

        // Call DynamoDB
        try {
            await ddb.putItem(params).promise()
            console.log("Success");
        } catch (err) {
            console.log("Error", err);
        }

        console.log("Success: Everything executed correctly");
        context.done(null, event);

    } else {
        // Nothing to do, the user's email ID is unknown
        console.log("Error: Nothing was written to DynamoDB");
        context.done(null, event);
    }
};
```

Вы можете получить доступ к названию таблице вызвав переменную окружения `API_{APP_NAME}_USERTABLE_NAME`.


Загррузите функцию на сервер:

```sh
amplify push
```
Теперь ваша lambda функция готова к использованию! 

### Настройка тригерра пост-подтверждения

Чтобы настроить ваш AWS Cognito триггер для вызова уже созданной lambda функции вы должны выполнить следующее:

- Зайдите в [AWS Console](https://console.aws.amazon.com/console/home)
- Перейдите к сервису AWS Cognito, и выберите 'Manage User Pools'
- Выберите User Pool (пул пользователей) связанный с вашим приложением
- Идите во вкладку 'Triggers' и найдите `Post Confirmation Trigger`, затем выберите вашу Lambda функцию
