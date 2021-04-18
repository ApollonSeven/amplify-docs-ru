# Настройка параметров Lambda функции

Возможно, вы захотите переопределить конфигурации Amplify CLI по умолчанию для вашей Lambda функции или изменить настройки, недоступные в рабочем процессе `amplify add function`.

*Пример*: При создании функции в `Node.js` CLI автоматически настраивает версию среды выполнения, размер памяти по умолчанию и многое другое. Есть несколько параметров, которые вы можете переопределить или настроить:

1. Runtime/Среда выполнения
2. Объем памяти
3. Переменные окружения

Давайте посмотрим как изменить эти параметры!

## Обновление среды выполнения (Runtime)

Возможно, вы захотите настроить версию среды выполнения, чтобы она была либо более новой, либо более старой, чем версия по умолчанию, созданная Amplify.

Допустим, мы развернули функцию Lambda, используя среду выполнения Node.js, и мы хотим изменить версию среды выполнения на `14.x`.

Чтобы сделать это откройте __amplify/backend/function/function-name/function-name-cloudformation-template.json__ и измените параметр `Runtime` в `LambdaFunction` ресурсе:

```json
"Resources": {
  "LambdaFunction": {
      ...
      "Runtime": "nodejs14.x", // Среда выполнения сейчас установлена на 14.x
      "Layers": [],
      ...
    }
  },
}
```

Затем загрузите обновления используя Amplify CLI:

```sh
amplify push
```

## Обновление размера памяти по умолчанию

Когда вы создаете функцию с помощью Amplify, размер памяти по умолчанию будет установлен на низкий уровень (128 МБ). Зачастую вам может потребоваться увеличить размер памяти по умолчанию, чтобы повысить производительность. Популярный параметр памяти в Lambda - 1024 МБ, так как он заметно ускоряет функцию, обычно сохраняя при этом стоимость такой же или близкой к ней.

За изменения размера памяти откройте __amplify/backend/function/function-name/function-name-cloudformation-template.json__ и измените параметр `MemorySize` в `LambdaFunction` ресурсе:

```json
"Resources": {
  "LambdaFunction": {
      ...
      "Runtime": "nodejs14.x",
      "MemorySize": "1024", // Объем памяти сейчас установлен на 1024 МБ
      "Layers": [],
      ...
    }
  },
}
```

Затем загрузите обновления используя Amplify CLI:

```sh
amplify push
```

_Чтобы узнать больше об оптимизации выделения ресурсов для Lambda функций, ознакомьтесь с [этой статьей](https://dev.to/aws/deep-dive-finding-the-optimal-resources-allocation-for-your-lambda-functions-35a6)._


## Установка переменных окружения

Очень распространенный сценарий - необходимость установить и использовать переменную среды в вашей Lambda функции.

Обычно существует два типа переменных среды:
- Секретные значения (пример: ключи доступа, ключи API и т.д.)
- Несекретные значения

### 1. Настройка секретных значений

Если ваше значение является секретным, вы можете использовать [Secrets Manager](https://aws.amazon.com/secrets-manager/).

Для этого вам сначала нужно создать секретное значение в [secrets manager](https://console.aws.amazon.com/secretsmanager) консоли.

Затем добавьте оператор `PolicyDocument` в __amplify/backend/function/function-name/function-name-cloudformation-template.json__ чтобы дать разрешение Lambda функции использовать секретные значения:

```json
{
  "Effect": "Allow",
  "Action": [
    "secretsmanager:GetSecretValue"
  ],
  "Resource": {
    "Fn::Sub": [
      "arn:aws:secretsmanager:${region}:${account}:secret:key_id",
      {
        "region": {
          "Ref": "AWS::Region"
        },
        "account": {
          "Ref": "AWS::AccountId"
        }
      }
    ]
  }
}
```

Затем получите доступ к токену в вашей функции:

```js
const AWS = require('aws-sdk')

const secretsManager = new AWS.SecretsManager()
const secret = await secretsManager.getSecretValue({ SecretId: 'YOUR_KEY' }).promise()

console.log(secret.SecretString)
```

### 2. Настройка несекретных значений

Если ваше значение является просто значением конфигурации, вы можете настроить конфигурацию CloudFormation локально, чтобы установить значение - в __amplify/backend/function/function-name/function-name-cloudformation-template.json__

Для этого в шаблоне есть раздел - `Parameters` - который вы можете изменить.

```json
"Parameters" : {
  "MyKey" : {
    "Type" : "String",
    "Default" : "my-environment-variable"
  }
}
```

А затем используйте эти параметры в объявлении `Environment`:

```json
"Environment":{
   "Variables":{
      "MY_ENV_VAR":{
         "Ref":"MyKey"
      }
   }
}
```

> Чтобы просмотреть все параметры конфигурации, доступные в AWS Lambda, ознакомьтесь с документацией [здесь](https://docs.aws.amazon.com/AWSCloudFormation/latest/UserGuide/aws-properties-lambda-function-environment.html)
> 
> Чтобы узнать больше о расширении Amplify CLI с помощью пользовательских ресурсов, ознакомьтесь с документацией [здесь](~/cli/usage/customcf.md)
