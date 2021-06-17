# Действия администратора

Узнайте как предоставить доступ к действиям администратора для Cognito User Pool в вашем приложении.

Действия администратора позволяют вам выполнять запросы и операции с пользователями и группами в вашем Cognito user pool.

К примеру возможность вывести список всех пользователей в Cognito User Pool может оказаться полезным для административной панели приложения, если вошедший в систему пользователь является членом определенной группы под названием «Администраторы».

> Это продвинутая функция которая не рекомендуется без понимания базовой архитектуры. Созданная связанная инфраструктура является базой, разработанной для вас, чтобы вы могли настроить ее в соответствии с конкретными бизнес-потребностями. Мы рекомендуем удалить все функции, которые не требуются вашему приложению.

Amplify CLI может настроить REST с безопасным доступом к функции Lambda, работающей с ограниченными разрешениями для пользовательского пула (User Pool) если вы хотите иметь эти возможности в вашем приложени и вы выбрали возможность предоставить действия для всех пользователей с валидным аккаунтом или ограничить для определенных групп (User Pool Group)

## Включение запросов администратора

```bash
amplify add auth
```

```console
? Do you want to add an admin queries API? Yes
? Do you want to restrict access to a specific Group Yes
? Select the group to restrict access with: (Use arrow keys)
❯ Admins 
  Editors 
  Enter a custom group 
```

Это настроит эндпоинт API Gateway с помощью Cognito Authorizer который принимает Access Token, который используется функцией Lambda для выполнения действий к User Pool. Функция представляет собой пример кода который вы можете использовать для удаления, добавления или изменения функциональности в зависимости от вашего бизнес-сценария, отредактировав его в `./amplify/backend/function/AdminQueriesXXX/src` и запустив `amplify push` для развертывания ваших изменений. Если вы выберите ограничение действий для определенной группы, настраиваемый промежуточный процесс (middleware) в функции предотвратит действия пока пользователь является членом этой группы. 

## Admin Queries API

The default routes and their functions, HTTP methods, and expected parameters are below
- `addUserToGroup`: Adds a user to a specific Group. Expects `username` and `groupname` in the POST body.
- `removeUserFromGroup`: Removes a user from a specific Group. Expects `username` and `groupname` in the POST body.
- `confirmUserSignUp`: Confirms a users signup. Expects `username` in the POST body.
- `disableUser`: Disables a user. Expects `username` in the POST body.
- `enableUser`: Enables a user. Expects `username` in the POST body.
- `getUser`: Gets specific user details. Expects `username` as a GET query string.
- `listUsers`: Lists all users in the current Cognito User Pool. You can provide an OPTIONAL `limit` (between 0 and 60) as a GET query string, which returns a `NextToken` that can be provided as a `token` query string for pagination.
- `listGroups`: Lists all groups in the current Cognito User Pool. You can provide an OPTIONAL `limit` (between 0 and 60) as a GET query string, which returns a `NextToken` that can be provided as a `token` query string for pagination.
- `listGroupsForUser`: Lists groups to which current user belongs to. Expects `username` as a GET query string. You can provide an OPTIONAL `limit` (between 0 and 60) as a GET query string, which returns a `NextToken` that can be provided as a `token` query string for pagination.
- `listUsersInGroup`: Lists users that belong to a specific group. Expects `groupname` as a GET query string. You can provide an OPTIONAL `limit` (between 0 and 60) as a GET query string, which returns a `NextToken` that can be provided as a `token` query string for pagination.
- `signUserOut`: Signs a user out from User Pools, but only if the call is originating from that user. Expects `username` in the POST body.

## Example

To leverage this functionality in your app you would call the appropriate route in your [JavaScript](~/lib/restapi/authz.md#cognito-user-pools-authorization), [iOS, or Android](~/sdk/api/rest.md#cognito-user-pools-authorization) application after signing in. For example to add a user "richard" to the Editors Group and then list all members of the Editors Group with a pagination limit of 10 you could use the following React code below:

```js
import React from 'react'
import Amplify, { Auth, API } from 'aws-amplify';
import { withAuthenticator } from 'aws-amplify-react';
import awsconfig from './aws-exports';
Amplify.configure(awsconfig);

async function addToGroup() { 
  let apiName = 'AdminQueries';
  let path = '/addUserToGroup';
  let myInit = {
      body: {
        "username" : "richard",
        "groupname": "Editors"
      }, 
      headers: {
        'Content-Type' : 'application/json',
        Authorization: `${(await Auth.currentSession()).getAccessToken().getJwtToken()}`
      } 
  }
  return await API.post(apiName, path, myInit);
}


let nextToken;

async function listEditors(limit){
  let apiName = 'AdminQueries';
  let path = '/listUsersInGroup';
  let myInit = { 
      queryStringParameters: {
        "groupname": "Editors",
        "limit": limit,
        "token": nextToken
      },
      headers: {
        'Content-Type' : 'application/json',
        Authorization: `${(await Auth.currentSession()).getAccessToken().getJwtToken()}`
      }
  }
  const { NextToken, ...rest } =  await API.get(apiName, path, myInit);
  nextToken = NextToken;
  return rest;
}

function App() {
  return (
    <div className="App">
      <button onClick={addToGroup}>Add to Group</button>
      <button onClick={() => listEditors(10)}>List Editors</button>
    </div>
  );
}

export default withAuthenticator(App, true);
```
