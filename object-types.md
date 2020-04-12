# Типы объектов

Во многих случаях вы не хотите возвращать число или строку из API. Вы хотите вернуть объект, который имеет свое сложное поведение. GraphQL идеально подходит для этого.

В языке схем GraphQL вы определяете новый тип объекта так же, как мы определяли тип ```Query``` в наших примерах. Каждый объект может иметь поля, которые возвращают определенный тип, и методы, которые принимают аргументы. Например, в документации [Передача аргументов](passing-arguments.md) у нас был метод бросить несколько случайных кубиков:

```
type Query {
  rollDice(numDice: Int!, numSides: Int): [Int]
}
```

Если бы мы хотели иметь все больше и больше методов, основанных на случайном кубике с течением времени, мы могли бы реализовать это с типом объекта ```RandomDie```.

```
type RandomDie {
  roll(numRolls: Int!): [Int]
}

type Query {
  getDie(numSides: Int): RandomDie
}
```

Вместо resolver корневого уровня для типа ```RandomDie``` мы можем вместо этого использовать класс ES6, где resolver-ы являются методами экземпляра. Этот код показывает, как приведенная выше схема ```RandomDie``` может быть реализована:

```
class RandomDie {
  constructor(numSides) {
    this.numSides = numSides;
  }

  rollOnce() {
    return 1 + Math.floor(Math.random() * this.numSides);
  }

  roll({numRolls}) {
    var output = [];
    for (var i = 0; i < numRolls; i++) {
      output.push(this.rollOnce());
    }
    return output;
  }
}

var root = {
  getDie: ({numSides}) => {
    return new RandomDie(numSides || 6);
  }
}
```

Для полей, которые не используют никаких аргументов, вы можете использовать либо свойства объекта, либо методы экземпляра. Таким образом, для приведенного выше примера кода и ```numSides```, и ```rollOnce``` могут фактически использоваться для реализации полей **GraphQL**, так что код также реализует схему:

```
type RandomDie {
  numSides: Int!
  rollOnce: Int!
  roll(numRolls: Int!): [Int]
}

type Query {
  getDie(numSides: Int): RandomDie
}
```

Собрав все это вместе, вот пример кода, который запускает сервер с этим API GraphQL:

```
var express = require('express');
var graphqlHTTP = require('express-graphql');
var { buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  type RandomDie {
    numSides: Int!
    rollOnce: Int!
    roll(numRolls: Int!): [Int]
  }

  type Query {
    getDie(numSides: Int): RandomDie
  }
`);

// This class implements the RandomDie GraphQL type
class RandomDie {
  constructor(numSides) {
    this.numSides = numSides;
  }

  rollOnce() {
    return 1 + Math.floor(Math.random() * this.numSides);
  }

  roll({numRolls}) {
    var output = [];
    for (var i = 0; i < numRolls; i++) {
      output.push(this.rollOnce());
    }
    return output;
  }
}

// The root provides the top-level API endpoints
var root = {
  getDie: ({numSides}) => {
    return new RandomDie(numSides || 6);
  }
}

var app = express();
app.use('/graphql', graphqlHTTP({
  schema: schema,
  rootValue: root,
  graphiql: true,
}));
app.listen(4000);
console.log('Running a GraphQL API server at localhost:4000/graphql');
```

Когда вы выполняете запрос GraphQL к API, который возвращает типы объектов, вы можете вызывать несколько методов для объекта одновременно, вложив имена полей GraphQL. Например, если вы хотите вызвать оба ```rollOnce```, чтобы бросить кубик один раз, и ```roll```, чтобы бросить кубик три раза, вы можете сделать это с помощью следующего запроса:

```
{
  getDie(numSides: 6) {
    rollOnce
    roll(numRolls: 3)
  }
}
```

Если вы запустите этот код с узлом server.js и перейдете по адресу [http://localhost:4000/graphql](http://localhost:4000/graphql), вы можете опробовать эти API с помощью GraphiQL.

Этот способ определения типов объектов часто дает преимущества по сравнению с традиционным API REST. Вместо выполнения одного запроса API для получения базовой информации об объекте, а затем нескольких последующих запросов API для получения дополнительной информации об этом объекте, вы можете получить всю эту информацию в одном запросе API. Это экономит полосу пропускания, ускоряет работу приложения и упрощает логику на стороне клиента.

Пока что каждый API, на который мы смотрели, предназначен для возврата данных. Чтобы изменить сохраненные данные или обработать сложный ввод, это помогает узнать о [мутациях и типах ввода](mutations-and-input-types.md).