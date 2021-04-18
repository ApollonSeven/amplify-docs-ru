# Прослушивание событий аутентификации

В этом руководстве вы узнаете, как использовать утилиту `Hub` для прослушивания различных событий аутентификации.

События `signIn`, `signUp` и `signOut` публикуются в канале аутентификации. Вы можете слушать эти уведомления о событиях и действовать в соответствии с ними.

```js
import { Hub } from 'aws-amplify';

Hub.listen('auth', (data) => {
  switch (data.payload.event) {
    case 'signIn':
        console.log('user signed in');
        break;
    case 'signUp':
        console.log('user signed up');
        break;
    case 'signOut':
        console.log('user signed out');
        break;
    case 'signIn_failure':
        console.log('user sign in failed');
        break;
    case 'configured':
        console.log('the Auth module is configured');
  }
});
```

Чтобы узнать больше о том, как работает `Hub`, ознакомьтесь с документацией по API [здесь](~/lib/utilities/hub.md)
