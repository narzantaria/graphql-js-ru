# Аутентификация и Middleware Express
Использовать любое middleware Express в сочетании с express-graphql просто. В частности, это отличный шаблон для обработки аутентификации.

Чтобы использовать middleware с resolver-ом GraphQL, просто используйте middleware, как в обычном приложении Express. Затем объект ```request``` доступен в качестве второго аргумента в любом resolver-е.

Например, скажем, мы хотели, чтобы наш сервер регистрировал IP-адрес каждого запроса, и мы также хотим написать API, который возвращает IP-адрес вызывающей стороны. Первое можно сделать с помощью middleware, а второе - путем обращения к объекту ```request``` в resolver-е. Вот код сервера, который реализует это:

```javascript
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

var schema = buildSchema(`
  type Query {
    ip: String
  }
`);

const loggingMiddleware = (req, res, next) => {
  console.log('ip:', req.ip);
  next();
}

var root = {
  ip: function (args, request) {
    return request.ip;
  }
};

var app = express();
app.use(loggingMiddleware);
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
}));
app.listen(4000);
console.log('Running a GraphQL API server at localhost:4000/graphql');
```

В REST API аутентификация часто обрабатывается с помощью заголовка, который содержит токен авторизации, который подтверждает, какой пользователь делает этот запрос. Промежуточное ПО (middleware) Express обрабатывает эти заголовки и помещает данные аутентификации в объект ```request``` Express. Некоторые модули middleware, которые обрабатывают такого рода аутентификацию, - это [Passport](http://passportjs.org/), [express-jwt](https://github.com/auth0/express-jwt) и [express-session](https://github.com/expressjs/session). Каждый из этих модулей работает с ```express-graphql```.

Если вы не знакомы ни с одним из этих механизмов аутентификации, мы рекомендуем использовать ```express-jwt```, потому что это просто без ущерба для будущей гибкости.

Если вы прочитали документы линейно, чтобы добраться до этого момента, поздравляем! Теперь вы знаете все, что нужно для создания практического сервера API GraphQL.