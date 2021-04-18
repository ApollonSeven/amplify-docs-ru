# Как создать подписку GraphQL по идентификатору

В этом руководстве вы узнаете, как создать настраиваемую подписку GraphQL, которая будет подключаться и запускаться только при мутации, содержащей в качестве аргумента конкретный идентификатор.

При использовании библиотеки преобразования Amplify GraphQL часто возникают моменты, когда вам нужно расширить схему GraphQL и операции, созданные директивой `@model`. Типичный вариант использования - когда для подписок GraphQL требуется детальный контроль.

Возьмем, например, следующую схему GraphQL:

```graphql
type Post @model {
  id: ID!
  title: String!
  content: String
  comments: [Comment] @connection
}

type Comment @model {
  id: ID!
  content: String
}
```

По умолчанию подписки будут созданы для следующих мутаций:

```graphql
# Post type
onCreatePost
onUpdatePost
onDeletePost

# Comment type
onCreateComment
onUpdateComment
onDeleteComment
```

Одна операция, которая не описана, - это если вы хотите подписаться только на комментарии к определенной публикации.

Поскольку в схеме включена связь "один ко многим" между сообщениями и комментариями, вы можете использовать автоматически созданное поле `postCommentsId`, которое определяет связь между сообщением и комментарием, чтобы установить это в новом определении подписки.

Чтобы реализовать это, вы можете обновить схему следующим образом:

```graphql
type Post @model {
  id: ID!
  title: String!
  content: String
  comments: [Comment] @connection
}

type Comment @model {
  id: ID!
  content: String
  postCommentsId: ID!
}

type Subscription {
  onCommentByPostId(postCommentsId: ID!): Comment
    @aws_subscribe(mutations: ["createComment"])
}

```

__JS__
```js
import { API } from 'aws-amplify';
import { onCommentByPostId } from './graphql/subscriptions';

API.graphql({
  query: onCommentByPostId,
  variables: {
    postCommentsId: "12345"
  }
})
.subscribe({
  next: data => {
    console.log('data: ', data)
  }
})
```
[__IOS__](~/guides/api-graphql/fragments/ios/subscriptions-by-id.md)
[__Android__](~/guides/api-graphql/fragments/android/subscriptions-by-id.md)