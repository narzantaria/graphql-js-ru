# Передача аргументов
Как и в случае REST API, в API GraphQL обычно передаются аргументы к конечной точке. Определяя аргументы на языке схемы, проверка типов происходит автоматически. Каждый аргумент должен быть назван и иметь тип. Например, в документации по [базовым типам](basic-types.md) у нас был эндпойнт с именем ```rollThreeDice```:

```
type Query {
  rollThreeDice: [Int]
}
```

Вместо жесткого кодирования "three" нам может потребоваться более общая функция, которая бросает кубики ```numDice```, каждая из которых имеет стороны ```numSides```. Мы можем добавить аргументы в язык схемы GraphQL следующим образом:

```
type Query {
  rollDice(numDice: Int!, numSides: Int): [Int]
}
```

Восклицательный знак в Int! указывает, что ```numDice``` не может быть нулевым, что означает, что мы можем пропустить немного логики проверки, чтобы сделать наш серверный код проще. Мы можем позволить ```numSides``` быть нулевыми и предположить, что по умолчанию у кубика есть 6 сторон.

До сих пор наши функции resolver не принимали аргументов. Когда resolver принимает аргументы, они передаются как один объект "args", как первый аргумент функции. Таким образом, ```rollDice``` может быть реализован как:

```
var root = {
  rollDice: (args) => {
    var output = [];
    for (var i = 0; i < args.numDice; i++) {
      output.push(1 + Math.floor(Math.random() * (args.numSides || 6)));
    }
    return output;
  }
};
```

Для этих параметров удобно использовать [деструктурирующее присваивание ES6](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Operators/Destructuring_assignment), поскольку вы знаете, в каком формате они будут. Таким образом, мы можем также написать ```rollDice``` как

```
var root = {
  rollDice: ({numDice, numSides}) => {
    var output = [];
    for (var i = 0; i < numDice; i++) {
      output.push(1 + Math.floor(Math.random() * (numSides || 6)));
    }
    return output;
  }
};
```

Если вы знакомы с деструктуризацией, это немного лучше, потому что строка кода, в которой определен ```rollDice```, говорит вам о аргументах.

Весь код сервера, на котором размещен этот ```rollDice``` API:

```
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  type Query {
    rollDice(numDice: Int!, numSides: Int): [Int]
  }
`);

// The root provides a resolver function for each API endpoint
var root = {
  rollDice: ({numDice, numSides}) => {
    var output = [];
    for (var i = 0; i < numDice; i++) {
      output.push(1 + Math.floor(Math.random() * (numSides || 6)));
    }
    return output;
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

Когда вы вызываете этот API, вы должны передавать каждый аргумент по имени. Таким образом, для вышеприведенного сервера вы можете выполнить этот запрос GraphQL, чтобы бросить три шестигранных кубика:

```
{
  rollDice(numDice: 3, numSides: 6)
}
```

Если вы запустите этот код с ```node server.js``` и перейдете по адресу [http://localhost:4000/graphql](http://localhost:4000/graphql), вы можете попробовать этот API.

Когда вы передаете аргументы в коде, обычно лучше избегать построения всей строки запроса самостоятельно. Вместо этого вы можете использовать синтаксис ```$``` для определения переменных в запросе и передачи переменных в виде отдельной карты.

Например, некоторый код JavaScript, который вызывает наш сервер выше:

```
var dice = 3;
var sides = 6;
var query = `query RollDice($dice: Int!, $sides: Int) {
  rollDice(numDice: $dice, numSides: $sides)
}`;

fetch('/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
  body: JSON.stringify({
    query,
    variables: { dice, sides },
  })
})
  .then(r => r.json())
  .then(data => console.log('data returned:', data));
```

Использование ```$dice``` и ```$sides``` в качестве переменных в GraphQL означает, что нам не нужно беспокоиться о экранировании на стороне клиента.

С помощью базовых типов и передачи аргументов вы можете реализовать все, что можете реализовать в REST API. Но GraphQL поддерживает еще более мощные запросы. Вы можете заменить несколько вызовов API одним вызовом API, если вы научитесь [определять свои собственные типы объектов](object-types.md).