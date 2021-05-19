# Установка
Как установить и настроить Amplify CLI

## Установка Amplify CLI

Интерфейс командной строки Amplify (CLI) - это унифицированный набор инструментов для создания облачных сервисов AWS для вашего приложения. Давайте продолжим и установим Amplify CLI.

__NPM__

```bash
npm install -g @aws-amplify/cli
```

**Примечание:** Поскольку мы устанавливаем Amplify CLI глобально, вам может потребоваться выполнить указанную выше команду с помощью `sudo` в зависимости от вашей системной политики.

__cURL (Mac and Linux)__

```bash
curl -sL https://aws-amplify.github.io/amplify-cli/install | bash && $SHELL
```

__cURL (Windows)__

```bash
curl -sL https://aws-amplify.github.io/amplify-cli/install-win -o install.cmd && install.cmd
```

### Требования для установки

* [Установите Node.js®](https://nodejs.org/en/download/) и [NPM](https://www.npmjs.com/get-npm) если они еще не установлены.
* Убедитесь, что вы используете как минимум Node.js версии 10.x и npm версии 6.x или выше, запустив `node -v` или `npm -v` в окне терминала/консоли.
* [Создайте аккаунт AWS](https://portal.aws.amazon.com/billing/signup?redirect_url=https%3A%2F%2Faws.amazon.com%2Fregistration-confirmation#/start). Если у вас еще нет учетной записи AWS, вам необходимо создать ее, чтобы выполнить шаги, описанные в этом руководстве.


## Настройка Amplify CLI

Чтобы настроить Amplify CLI на локальном компьютере, вам необходимо настроить его для подключения к вашей учетной записи AWS. 

### Вариант 1: Посмотрите видеоинструкцию

Посмотрите видео ниже, чтобы узнать, как установить и настроить Amplify CLI, или перейдите к следующему разделу, чтобы следовать пошаговым инструкциям.

<iframe src="https://www.youtube-nocookie.com/embed/fWbM5DLh25U" frameborder="0" allow="accelerometer; autoplay; clipboard-write; encrypted-media; gyroscope; picture-in-picture" allowfullscreen></iframe>

### Вариант 2: Следуйте инструкциям

Настройте Amplify, выполнив следующую команду:

```bash
amplify configure
```

`amplify configure` попросит вас войти в Консоль AWS.

Как только вы войдете в систему, Amplify CLI попросит вас создать пользователя IAM.
> Amazon IAM (Identity and Access Management) позволяет управлять пользователями и разрешениями пользователей в AWS. Вы можете узнать больше об Amazon IAM [здесь](https://aws.amazon.com/iam/).

```console
Specify the AWS Region
? region:  # Your preferred region
Specify the username of the new IAM user:
? user name:  # User name for Amplify IAM user
Complete the user creation using the AWS console
```

Создайте пользователя с доступом администратора (`AdministratorAccess`) к своей учетной записи, чтобы предоставить вам ресурсы AWS, такие как AppSync, Cognito и т.д.

![image](../../images/user-creation.gif)

После создания пользователя Amplify CLI попросит вас предоставить `accessKeyId` и `secretAccessKey` для соединения Amplify CLI с вашим вновь созданным пользователем IAM.

```console
Enter the access key of the newly created user:
? accessKeyId:  # YOUR_ACCESS_KEY_ID
? secretAccessKey:  # YOUR_SECRET_ACCESS_KEY
This would update/create the AWS Profile in your local machine
? Profile Name:  # (default)

Successfully set up the new user.
```


### Работайте в своем веб-проекте

После установки CLI перейдите в корень проекта JavaScript, iOS или Android, инициализируйте AWS Amplify в новом каталоге, запустив `ampify init`. После нескольких вопросов о конфигурации вы можете в любой момент воспользоваться расширенной справкой, чтобы увидеть общую структуру команд. Когда вы будете готовы добавить функцию, запустите `ampify add <category>`.