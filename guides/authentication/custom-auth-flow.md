# Создание настраиваемого потока аутентификации

[Библиотека UI-компонентов Amplify](~/ui/auth/authenticator.md) позволяет выстроить сквозной поток аутентификации всего в несколько строк кода. Но зачастую приходится создавать собственный поток аутентификации с нуля.

В этом руководстве вы узнаете, как использовать класс `Auth` для управления идентификацией пользователя и утилиту `Hub` для прослушивания событий аутентификации.

## Класс Auth

[Auth class](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html) - это API, предоставляемый библиотекой JavaScript Amplify, который имеет более 30 методов для управления идентификацией пользователей в вашем приложении.

В этом руководстве мы будем работать со следующими методами:

- [signUp](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html#signup) - Создание новой учетной записи пользователя
- [confirmSignUp](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html#confirmsignup) - Подтверждение пользователя верификационным кодом для мультифакторной аутентификации (MFA)
- [signIn](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html#signin) - Авторизация пользователя
- [currentAuthenticatedUser](https://aws-amplify.github.io/amplify-js/api/classes/authclass.html#currentauthenticateduser) - Если пользователь вошел в систему, возвращает метаданные о текущем вошедшем пользователе. Если ни один пользователь не вошел в систему, возвращается null.

## Hub

[Утилита Hub](~/lib/utilities/hub.md) - это локальная система событий, используемая для обмена данными между модулями и компонентами в вашем приложении. В этом руководстве вы узнаете, как прослушивать события аутентификации с помощью утилиты `Hub`:

## Концепции

При построении потока аутентификации пользователя необходимо управлять несколькими частями состояния:

1. __Form state / Состояние формы__ - состояние формы определяет, какая форма должна быть показана пользователю. Это новый пользователь? Какую регистрационную форму отображать для этого пользователя? Он в процессе регистрации? Следует ли показывать форму подтверждения регистрации для потока MFA?
2. __Form input state / Состояние ввода формы__ - Состояние ввода формы это значение фактических полей формы при вводе пользователем. Например, у вас может быть форма, позволяющая пользователю вводить свой адрес электронной почты, пароль и номер телефона. Состояние ввода формы будет содержать эти значения и позволит вам отправить их в запросе API.
3. __Routing state / Состояние маршрутизации__ - Состояние маршрутизации определяет, какой URL или страницу просматривает пользователь. Во многих случаях это будет определять, может ли пользователь просматривать определенный маршрут или страницу. Реализация во многом зависит от типа создаваемого вами приложения и используемой среды (если таковая имеется).
4. __User state / Состояние пользователя__ - это текущий вошедший в систему пользователь (если он есть). Используя это состояние, вы можете определить состояние маршрутизации и включить более детальный контроль в зависимости от того, вошел ли пользователь в систему, или на основе данных текущего пользователя.

<!-- This guide will cover strategies for handling __form state__, __form input state__, and __user state__ but will not be covering routing state as this is very dependent on the framework. -->

## Реализация настраиваемого потока аутентификации

Используя комбинацию состояния приложения и API Amplify, вы можете легко управлять потоком аутентификации вашего приложения. Давайте посмотрим, как этого добиться с помощью псевдокода:

```javascript
/* Импортируем Amplify Auth API */
import { Auth } from 'aws-amplify';

/* Создайем состояние формы и состояние ввода формы */
let formState = "signUp";
let formInputState = { username: '', password: '', email: '', verificationCode: '' };

/* обработчик onChange для ввода формы */
function onChange(e) {
  formInputState = { ...formInputState, [e.target.name]: e.target.value };
}

/* Функция регистрации (signUp) */
async function signUp() {
  try {
    await Auth.signUp({
      username: formInputState.username,
      password: formInputState.password,
      attributes: {
        email: formInputState.email
      }});
    /* После успешной регистрации пользователя обновите состояние формы, чтобы отобразить форму подтверждения регистрации для MFA. */
    formState = "confirmSignUp";
  } catch (err) { console.log({ err }); }
}

/* Подтверждение регистрации для MFA */
async function confirmSignUp() {
  try {
    await Auth.confirmSignUp(formInputState.username, formInputState.verificationCode);
    /* После того, как пользователь успешно подтвердит свою учетную запись, обновите состояние формы, чтобы отобразить форму авторизации.*/
    formState = "signIn";
  } catch (err) { console.log({ err }); }
}

/* Функция авторизации */
async function signIn() {
  try {
    await Auth.signIn(formInputState.username, formInputState.password);
    /* После успешного входа пользователя обновите состояние формы, чтобы отобразить состояние входа. */
    formState = "signedIn";
  } catch (err) { console.log({ err }); }
}


/* В пользовательском интерфейсе приложения отображайте формы на основе состояния формы */
/* Если состояние формы - «signUp», покажите форму регистрации */
if (formState === "signUp") {
  return (
    <div>
      <input
        name="username"
        onChange={onChange}
      />
      <input
        name="password"
        type="password"
        onChange={onChange}
      />
      <input
        name="email"
        onChange={onChange}
      />
      <button onClick={signUp}>Sign Up</button>
    </div>
  )
}

/* Если состояние формы - «confirmSignUp», покажите форму подтверждения */
if (formState === "confirmSignUp") {
  return (
    <div>
      <input
        name="username"
        onChange={onChange}
      />
      <input
        name="verificationCode"
        onChange={onChange}
      />
      <button onClick={confirmSignUp}>Confirm Sign Up</button>
    </div>
  )
}

/* Если состояние формы - «signIn», покажите форму авторизации */
if (formState === "signIn") {
  return (
    <div>
      <input
        name="username"
        onChange={onChange}
      />
      <input
        name="password"
        onChange={onChange}
      />
      <button onClick={signIn}>Sign In</button>
    </div>
  )
}

/* Если состояние формы - "signedIn", покажите приложение */
if (formState === "signedIn") {
  return (
    <div>
      <h1>Welcome to my app!</h1>
    </div>
  )
}
```

## Управление состоянием пользователя

Для управления состоянием пользователя вы можете вызвать метод `currentAuthenticatedUser` класса `Auth`. В этом примере вы можете вызвать метод, чтобы показать или скрыть форму регистрации при загрузке приложения:

```javascript
async function onAppLoad() {
  const user = await Auth.currentAuthenticatedUser();
  console.log('user:', user)
  if (user) {
    formState = "signedIn";
  } else {
    formState = "signUp";
  }
}
```

## Прослушивание событий аутентификации

Для прослушивания событий авторизации (регистрация, вход и т.д.) Вы можете использовать утилиту `Hub`. Допустим, вам нужно было прослушивать событие выхода из любого места в приложении. Для этого вы можете использовать следующий код:

```javascript
Hub.listen('auth', (data) => {
  const event = data.payload.event;
  console.log('event:', event);
  if (event === "signOut") {
    console.log('user signed out...');
  }
});
```

