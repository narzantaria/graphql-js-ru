# Основные типы

В большинстве случаев все, что вам нужно сделать, это указать типы для вашего API, используя язык схемы GraphQL, взятый в качестве аргумента функции ```buildSchema```.

Язык схемы GraphQL поддерживает скалярные типы ```String```, ```Int```, ```Float```, ```Boolean``` и ```ID```, поэтому вы можете использовать их непосредственно в схеме, передаваемой в ```buildSchema```.

По умолчанию каждый тип может быть нулевым - разрешено возвращать NULL как любой из скалярных типов. Используйте восклицательный знак, чтобы указать, что тип не может быть нулевым, поэтому String! является ненулевой строкой

Чтобы использовать тип списка, заключите его в квадратные скобки, поэтому ```[Int]``` - это список целых чисел.

Каждый из этих типов напрямую сопоставлен с JavaScript, поэтому вы можете просто возвращать простые старые объекты JavaScript в API, которые возвращают эти типы. Вот пример, который показывает, как использовать некоторые из этих основных типов:

```javascript
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  type Query {
    quoteOfTheDay: String
    random: Float!
    rollThreeDice: [Int]
  }
`);

// The root provides a resolver function for each API endpoint
var root = {
  quoteOfTheDay: () => {
    return Math.random() < 0.5 ? 'Take it easy' : 'Salvation lies within';
  },
  random: () => {
    return Math.random();
  },
  rollThreeDice: () => {
    return [1, 2, 3].map(_ => 1 + Math.floor(Math.random() * 6));
  },
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

Если вы запустите этот код с ```node server.js``` и перейдете по адресу [http://localhost:4000/graphql](http://localhost:4000/graphql), вы можете попробовать эти API.

Эти примеры показывают, как вызывать API, которые возвращают разные типы. Чтобы отправлять различные типы данных в API, вам также необходимо узнать о [передаче аргументов в API GraphQL](passing-arguments.md).