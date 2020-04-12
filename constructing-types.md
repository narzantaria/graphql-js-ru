# Конструирование типов

Для многих приложений вы можете определить фиксированную схему при запуске приложения и определить ее с помощью языка схемы GraphQL. В некоторых случаях полезно создавать схему программно. Вы можете сделать это используя конструктор ```GraphQLSchema```.

Когда вы используете конструктор ```GraphQLSchema``` для создания схемы, вместо того, чтобы определять типы ```Query``` и ```Mutation``` исключительно с использованием языка схем, вы создаете их как отдельные типы объектов.

Например, скажем, мы создаем простой API, который позволяет вам получать пользовательские данные для нескольких пользователей с жестким кодом на основе идентификатора. Используя ```buildSchema```, мы могли бы написать сервер с:

```
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

var schema = buildSchema(`
  type User {
    id: String
    name: String
  }

  type Query {
    user(id: String): User
  }
`);

// Maps id to User object
var fakeDatabase = {
  'a': {
    id: 'a',
    name: 'alice',
  },
  'b': {
    id: 'b',
    name: 'bob',
  },
};

var root = {
  user: ({id}) => {
    return fakeDatabase[id];
  }
};

var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
}));
app.listen(4000);
console.log('Running a GraphQL API server at localhost:4000/graphql');
```

Мы можем реализовать этот же API без использования языка схемы GraphQL:

```
var express = require('express');
var graphqlHTTP = require('express-graphql');
var graphql = require('graphql');

// Maps id to User object
var fakeDatabase = {
  'a': {
    id: 'a',
    name: 'alice',
  },
  'b': {
    id: 'b',
    name: 'bob',
  },
};

// Define the User type
var userType = new graphql.GraphQLObjectType({
  name: 'User',
  fields: {
    id: { type: graphql.GraphQLString },
    name: { type: graphql.GraphQLString },
  }
});

// Define the Query type
var queryType = new graphql.GraphQLObjectType({
  name: 'Query',
  fields: {
    user: {
      type: userType,
      // `args` describes the arguments that the `user` query accepts
      args: {
        id: { type: graphql.GraphQLString }
      },
      resolve: (_, {id}) => {
        return fakeDatabase[id];
      }
    }
  }
});

var schema = new graphql.GraphQLSchema({query: queryType});

var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,
  graphiql: true,
}));
app.listen(4000);
console.log('Running a GraphQL API server at localhost:4000/graphql');
```

Когда мы используем этот метод создания API, resolver-ы корневого уровня реализуются для типов ```Query``` и ```Mutation```, а не для корневых объектов.

Это особенно полезно, если вы хотите автоматически создать схему GraphQL из чего-то другого, например, из схемы базы данных. У вас может быть общий формат для чего-то вроде создания и обновления записей базы данных. Это также полезно для реализации таких функций, как объединяемые типы, которые не соответствуют чисто классам ES6 и языку схем.