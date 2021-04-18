# Регистрация и вход только по электронной почте

Вместо того, чтобы требовать от пользователей создания имени пользователя, многие приложения будут использовать адрес электронной почты пользователя для аутентификации. В этом руководстве вы узнаете, как включить эту функцию с помощью Amplify.

## Начало работы - Создание сервиса

Чтобы включить аутентификацию, указав `email` в качестве основного свойства аутентификации, выполните следующие действия:

```sh
amplify add auth

? Do you want to use the default authentication and security configuration? Default configuration
? How do you want users to be able to sign in? Email
? Do you want to configure advanced settings? No, I am done.
```
Затем загрузите службу аутентификации:

```sh
amplify push
```

## Клиентская интеграция

Теперь, когда служба развернута, вы можете настроить клиентскую часть для взаимодействия со службой аутентификации. Вы узнаете, как взаимодействовать со службой, используя как компоненты пользовательского интерфейса, так и класс `Auth` библиотеки JavaScript Amplify.

### UI Components

В компоненте пользовательского интерфейса вам необходимо указать, что вы хотите использовать адрес электронной почты в качестве свойства регистрации и входа для пользователей. Для этого вам необходимо установить для параметра `usernameAlias` ​​значение `email`.

__React__

```js
import { AmplifyAuthenticator } from '@aws-amplify/ui-react';

<AmplifyAuthenticator usernameAlias="email" />
```

__Angular__

```js
<amplify-authenticator usernameAlias="email"></amplify-authenticator>
```

__Vue__

```js
<amplify-authenticator username-alias="email"></amplify-authenticator>
```

__React Native__

```js
import { withAuthenticator, Authenticator } from 'aws-amplify-react';

// When using Authenticator
class App {
  // ...

  render() {
    return (
      <Authenticator usernameAttributes='email'/>
    );
  }
}

export default App;

// When using withAuthenticator
class App2 {
  // ...
}

export default withAuthenticator(App2, { usernameAttributes: 'email' });
```

### Вызов напрямую из Auth API

Вы также можете вызвать службу аутентификации напрямую, используя категорию `Auth`:

**Регистрация**

```js
import { Auth } from 'aws-amplify';

await Auth.signUp({
  username: "youremail@yourdomain.com",
  password: "your-secure-password",
  attributes: {
    email: "youremail@yourdomain.com"
  }
});
```

**Подтверждение регистрации**

```js
import { Auth } from 'aws-amplify';

await Auth.confirmSignUp("youremail@yourdomain.com", "123456");
```

**Авторизация**

```js
import { Auth } from 'aws-amplify';

await Auth.signIn("youremail@yourdomain.com", "your-secure-password");
```