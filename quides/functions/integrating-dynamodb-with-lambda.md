# Интеграция DynamoDB с Lambda

В этом руководстве вы узнаете, как сделать три вещи:

1. Создать новую функцию Lambda и базу данных DynamoDB, которые будут интегрированы вместе
2. Создать новую базу данных DynamoDB и интегрировать ее с существующей функцией Lambda
3. Создать новую функцию Lambda и интегрировать ее с существующей базой данных DynamoDB

### Создание новой функции Lambda и базы данных DynamoDB, которые будут интегрированы вместе

Первое, что нужно сделать, это создать таблицу DynamoDB:

```sh
amplify add storage

? Please select from one of the below mentioned services: NoSQL Database
? Please provide a friendly name for your resource that will be used to label this category in the project: testtable
? Please provide table name: testtable

# Теперь вы можете добавлять столбцы в таблицу.
? What would you like to name this column: id
? Please choose the data type: String
? Would you like to add another column? N
? Please choose partition key for the table: id
? Do you want to add a sort key to your table? N
? Do you want to add global secondary indexes to your table? N
? Do you want to add a Lambda Trigger for your Table? N
```

Затем создайте функцию:

```sh
amplify add function

? Provide a friendly name for your resource to be used as a label for this category in the project: mylambda
? Provide the AWS Lambda function name: mylambda
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? Y
? Select the category: storage
? Select the operations you want to permit for testtable: create, read, update, delete
? Do you want to invoke this function on a recurring schedule? N
? Do you want to edit the local lambda function now? N
```

Загрузите функцию и базу данных:

```sh
amplify push
```

Теперь ваша функция и база данных готовы к использованию!

Чтобы узнать, как взаимодействовать с DynamoDB из Lambda, ознакомьтесь с [Вызов DynamoDB из Lambda в Node.js](~/guides/functions/dynamodb-from-js-lambda.md).

### Создание новой базы данных DynamoDB и интеграция с существующей функцией Lambda

Сначала создайте базу данных, используя категорию __хранилища__:

```sh
amplify add storage

? Please select from one of the below mentioned services: NoSQL Database
? Please provide a friendly name for your resource that will be used to label this category in the project: testtable
? Please provide table name: testtable

# Теперь вы можете добавлять столбцы в таблицу.
? What would you like to name this column: id
? Please choose the data type: String
? Would you like to add another column? N
? Please choose partition key for the table: id
? Do you want to add a sort key to your table? N
? Do you want to add global secondary indexes to your table? N
? Do you want to add a Lambda Trigger for your Table? N
```

Затем обновите разрешения функции:

```sh
amplify update function

? Please select the Lambda Function you would want to update: <your-function>
? Do you want to update permissions granted to this Lambda function to perform on other resources in your project? Y
? Select the category: storage
? Select the operations you want to permit for testtable: create, read, update, delete
? Do you want to invoke this function on a recurring schedule? N
? Do you want to edit the local lambda function now? N
```

Загрузите базу данных и обновите разрешения Lambda:

```sh
amplify push
```

Теперь ваша функция и база данных готовы к использованию!

### Создание новой функции Lambda и интеграция с существующей базой данных DynamoDB

Чтобы создать новую лямбда-функцию, интегрированную с существующей базой данных DynamoDB, вам необходимо предоставить доступ к базе данных в процессе создания функции:

```sh
amplify add function

? Provide a friendly name for your resource to be used as a label for this category in the project: mylambda
? Provide the AWS Lambda function name: mylambda
? Choose the function runtime that you want to use: NodeJS
? Choose the function template that you want to use: Hello World
? Do you want to access other resources created in this project from your Lambda function? Y
? Select the category: storage

# Выберите базу данных, для которой вы хотите предоставить разрешения, или перейдите к следующему шагу, если в проекте только одна база данных

? Select the operations you want to permit for <your-table-name>: create, read, update, delete
? Do you want to invoke this function on a recurring schedule? N
? Do you want to edit the local lambda function now? N
```

Загрузите функцию:

```sh
amplify push
```

Теперь ваша функция и база данных готовы к использованию!

Чтобы узнать, как взаимодействовать с DynamoDB из Lambda, ознакомьтесь с [Вызов DynamoDB из Lambda в Node.js](~/guides/functions/dynamodb-from-js-lambda.md).