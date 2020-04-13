# Мутации и типы Input

Если у вас есть эндпойнт API, который изменяет данные, как включение данных в базу или изменение данных, уже находящихся в базе, вы должны сделать этот эндпойнт мутацией, а не запросом. Это так же просто, как сделать эндпойнт API частью типа ```Mutation``` верхнего уровня вместо типа ```Query``` верхнего уровня.

Допустим, у нас есть сервер “message of the day”, где каждый может обновить сообщение дня, и любой может прочитать текущее. Схема GraphQL для этого - просто:

```graphql
type Mutation {
  setMessage(message: String): String
}

type Query {
  getMessage: String
}
```

Часто удобно иметь мутацию, которая сопоставляется с операцией create или update базы данных, например ```setMessage```, возвращая то же самое, что хранил сервер. Таким образом, если вы измените данные на сервере, клиент может узнать об этих изменениях.

И мутации, и запросы могут обрабатываться корневыми resolver-ами, поэтому корнем, который реализует эту схему, может быть просто:

```javascript
var fakeDatabase = {};
var root = {
  setMessage: ({message}) => {
    fakeDatabase.message = message;
    return message;
  },
  getMessage: () => {
    return fakeDatabase.message;
  }
};
```

Вам не нужно ничего больше, чтобы реализовать мутации. Но во многих случаях вы найдете множество различных мутаций, которые все принимают одинаковые входные параметры. Типичным примером является то, что создание объекта в базе данных и обновление объекта в базе данных часто принимают одни и те же параметры. Чтобы упростить вашу схему, вы можете использовать для этого «типы ввода», используя ключевое слово ```input``` вместо ключевого слова ```type```.

Например, вместо одного сообщения дня, скажем, у нас есть много сообщений, проиндексированных в базе данных с помощью поля ```id```, и каждое сообщение имеет как строку ```content```, так и строку ```author```. Мы хотим, чтобы API мутации создавался как для нового сообщения, так и для обновления старого. Мы могли бы использовать схему:

```graphql
input MessageInput {
  content: String
  author: String
}

type Message {
  id: ID!
  content: String
  author: String
}

type Query {
  getMessage(id: ID!): Message
}

type Mutation {
  createMessage(input: MessageInput): Message
  updateMessage(id: ID!, input: MessageInput): Message
}
```

Здесь мутации возвращают тип ```Message```, так что клиент может получить больше информации о недавно измененном ```Message``` в том же запросе, что и запрос, который его "мутирует".

Типы Input не могут иметь поля, являющиеся другими объектами, только базовые скалярные типы, типы списков и другие типы ввода.

Называть типы ввода с помощью ```Input``` на конце - это полезное соглашение, потому что вам часто нужно, чтобы тип ввода и тип вывода немного отличались для одного концептуального объекта.

Вот некоторый исполняемый код, который реализует эту схему, сохраняя данные в памяти:

```javascript
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  input MessageInput {
    content: String
    author: String
  }

  type Message {
    id: ID!
    content: String
    author: String
  }

  type Query {
    getMessage(id: ID!): Message
  }

  type Mutation {
    createMessage(input: MessageInput): Message
    updateMessage(id: ID!, input: MessageInput): Message
  }
`);

// If Message had any complex fields, we'd put them on this object.
class Message {
  constructor(id, {content, author}) {
    this.id = id;
    this.content = content;
    this.author = author;
  }
}

// Maps username to content
var fakeDatabase = {};

var root = {
  getMessage: ({id}) => {
    if (!fakeDatabase[id]) {
      throw new Error('no message exists with id ' + id);
    }
    return new Message(id, fakeDatabase[id]);
  },
  createMessage: ({input}) => {
    // Create a random id for our "database".
    var id = require('crypto').randomBytes(10).toString('hex');

    fakeDatabase[id] = input;
    return new Message(id, input);
  },
  updateMessage: ({id, input}) => {
    if (!fakeDatabase[id]) {
      throw new Error('no message exists with id ' + id);
    }
    // This replaces all old data, but some apps might want partial update.
    fakeDatabase[id] = input;
    return new Message(id, input);
  },
};

var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
}));
app.listen(4000, () => {
  console.log('Running a GraphQL API server at localhost:4000/graphql');
});
```

Чтобы вызвать мутацию, вы должны использовать ключевое слово ```mutation``` перед запросом GraphQL. Чтобы передать тип ввода, предоставьте данные, написанные так, как будто это объект JSON. Например, на указанном выше сервере вы можете создать новое сообщение и вернуть ```id``` нового сообщения с помощью этой операции:

```graphql
mutation {
  createMessage(input: {
    author: "andy",
    content: "hope is a good thing",
  }) {
    id
  }
}
```

Вы можете использовать переменные, чтобы упростить клиентскую логику мутации, так же, как и с запросами. Например, некоторый код JavaScript, который вызывает сервер для выполнения этой мутации:

```javascript
var author = 'andy';
var content = 'hope is a good thing';
var query = `mutation CreateMessage($input: MessageInput) {
  createMessage(input: $input) {
    id
  }
}`;

fetch('/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
  body: JSON.stringify({
    query,
    variables: {
      input: {
        author,
        content,
      }
    }
  })
})
  .then(r => r.json())
  .then(data => console.log('data returned:', data));
```

Один конкретный тип мутации - это операции, которые меняют пользователей, например, регистрация нового пользователя. Хотя вы можете реализовать это с помощью мутаций GraphQL, вы можете использовать многие существующие библиотеки, если узнаете о GraphQL с [Аутентификацией и middleware Express](authentication-and-express-middleware.md).

