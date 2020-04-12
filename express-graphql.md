Модуль express-graphql предоставляет простой способ создания сервера [Express](https://expressjs.com/ru/), на котором работает API GraphQL.

```
import graphqlHTTP from 'express-graphql'; // ES6
var graphqlHTTP = require('express-graphql'); // CommonJS
```

## graphqlHTTP 

```
graphqlHTTP({
  schema: GraphQLSchema,
  graphiql?: ?boolean,
  rootValue?: ?any,
  context?: ?any,
  pretty?: ?boolean,
  formatError?: ?Function,
  validationRules?: ?Array<any>,
}): Middleware
```

Создает приложение Express на основе схемы GraphQL.

См. [Руководство по express-graphql](running-express-server.md) для примера использования.

См. [GitHub README](https://github.com/graphql/express-graphql) для более подробной документации деталей этого метода.