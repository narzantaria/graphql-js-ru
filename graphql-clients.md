# GraphQL-клиенты

Поскольку API-интерфейс GraphQL имеет более основополагающую структуру, чем REST API, существуют более мощные клиенты, такие как [Relay](https://relay.dev/en), которые могут автоматически обрабатывать сортировку, кэширование и другие функции. Но вам не нужен сложный клиент для вызова сервера GraphQL. С помощью ```express-graphql``` вы можете просто отправить HTTP-запрос POST на эндпойнт, на котором вы установили свой сервер GraphQL, передав запрос GraphQL в качестве поля ```query``` в полезной нагрузке JSON.

Например, скажем, мы смонтировали сервер GraphQL на [http://localhost:4000/graphql](http://localhost:4000/graphql), как в примере кода для [запуска сервера Express GraphQL](running-express-server.md), и мы хотим отправить запрос GraphQL ```{ hello }```. Мы можем сделать это из командной строки с помощью ```curl```. Если вы вставите это в терминал:

```bash
$ curl -X POST \
$ -H "Content-Type: application/json" \
$ -d '{"query": "{ hello }"}' \
$ http://localhost:4000/graphql
```

Вы должны увидеть выходные данные в виде JSON:

```graphql
{"data":{"hello":"Hello world!"}}
```

Если вы предпочитаете использовать графический интерфейс пользователя для отправки тестового запроса, вы можете использовать такие клиенты, как [GraphiQL](https://github.com/graphql/graphiql) и [Insomnia](https://github.com/getinsomnia/insomnia).

Также просто отправить GraphQL из браузера. Откройте http://localhost:4000/graphql, откройте консоль разработчика и вставьте:

```javascript
fetch('/graphql', {
  method: 'POST',
  headers: {
    'Content-Type': 'application/json',
    'Accept': 'application/json',
  },
  body: JSON.stringify({query: "{ hello }"})
})
  .then(r => r.json())
  .then(data => console.log('data returned:', data));
```

Вы должны увидеть возвращенные данные, логированные в консоли:

```javascript
data returned: Object { hello: "Hello world!" }
```

В этом примере запрос был просто жестко закодированной строкой. Поскольку ваше приложение становится более сложным, и вы добавляете эндпойнты GraphQL, которые принимают аргументы, как описано в разделе [Передача аргументов](passing-arguments.md), вы захотите создавать запросы GraphQL с использованием переменных в клиентском коде. Вы можете сделать это, включив ключевое слово с префиксом в запросе со знаком доллара и передав дополнительное поле переменных в полезную нагрузку.

Например, предположим, что вы запускаете пример сервера из [Передачи аргументов](passing-arguments.md), который имеет схему

```graphql
type Query {
  rollDice(numDice: Int!, numSides: Int): [Int]
}
```

Вы можете получить доступ к этому из JavaScript с помощью кода:

```javascript
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

Использование этого синтаксиса для переменных является хорошей идеей, потому что он автоматически предотвращает ошибки из-за экранирования и облегчает мониторинг вашего сервера.

В целом, для настройки клиента GraphQL, такого как Relay, потребуется немного больше времени, но оно того стоит, чтобы получить больше возможностей по мере роста вашего приложения. Возможно, вы захотите начать с использования HTTP-запросов в качестве базового транспортного уровня и переключения на более сложного клиента по мере усложнения приложения.

На этом этапе вы можете написать клиент и сервер в GraphQL для API, который получает одну строку. Чтобы сделать больше, вы захотите узнать, как использовать [другие основные типы данных](basic-types.md).
