# Управление атрибутами пользователя

В Cognito у вас есть возможность управлять как __стандартными__, так и __настраиваемыми__ пользовательскими атрибутами для ваших пользователей.

## Стандартные атрибуты

### Настройка стандартных атрибутов

По умолчанию в Cognito доступно множество пользовательских атрибутов. Вот их список:

- address
- birthdate
- email
- family name
- gender
- given name
- locale
- middle name
- name
- nickname
- phone number
- picture
- preferred username
- profile
- zoneinfo
- updated at
- website

Чтобы настроить и включить стандартные пользовательские атрибуты в вашем приложении, вы можете запустить команду Amplify `update auth` и выбрать __Walkthrough all the auth configurations__. Когда будет предложено __Specify read attributes__ и __Specify write attributes__, выберите атрибуты, которые вы хотите включить в своем приложении.

### Создание и обновление стандартных атрибутов

Вы можете создать атрибуты пользователя при регистрации или после регистрации.

#### Создание атрибутов пользователя при регистрации

Чтобы установить атрибуты пользователя во время регистрации, вы можете заполнить поле `attributes`:

```js
await Auth.signUp({
  username: 'someuser', password: 'mycoolpassword',
  attributes: {
    email: 'someuser@somedomain.com', address: '105 Main St. New York, NY 10001'
  }
});
```

#### Управление атрибутами пользователей после регистрации

Для управления атрибутами пользователя после регистрации вы можете использовать метод `updateUserAttributes` класса __Auth__:

```js
async function updateUser() {
  const user = await Auth.currentAuthenticatedUser();
  await Auth.updateUserAttributes(user, {
    'address': '105 Main St. New York, NY 10001'
  });
}
```

### Чтение атрибутов пользователя

Для чтения атрибутов пользователя вы можете использовать метод `currentAuthenticatedUser` класса __Auth__:

```js
async function getUserInfo() {
  const user = await Auth.currentAuthenticatedUser();
  console.log('attributes:', user.attributes);
}
```

## Настраиваемые атрибуты

Чтобы установить настраиваемый атрибут, необходимо сначала открыть панель управления Amazon Cognito:

```sh
amplify console auth

? Which console: User Pool
```

Затем нажмите __Attributes__ в левой навигационной панели и нажмите __Add custom attribute__.

### Создание и обновление пользовательских атрибутов

Вы можете создать настраиваемые атрибуты пользователя при регистрации или после регистрации. При управлении настраиваемыми атрибутами перед этим атрибутом необходимо указать `custom`:

#### Создание настраиваемых атрибутов пользователя при регистрации

Чтобы создать атрибут пользователя во время регистрации, вы можете заполнить поле `attributes`:

```js
await Auth.signUp({
  username: 'someuser', password: 'mycoolpassword',
  attributes: {
    email: 'someuser@somedomain.com', 'custom:favorite_ice_cream': 'chocolate'
  }
})
```

#### Управление настраиваемыми атрибутами пользователя после регистрации

Для управления пользовательскими атрибутами после регистрации вы можете использовать метод `updateUserAttributes` класса __Auth__:

```js
async function updateUser() {
  const user = await Auth.currentAuthenticatedUser();
  await Auth.updateUserAttributes(user, {
    'custom:favorite_ice_cream': 'vanilla'
  });
}
```
