# Начало работы с GraphQL.js

## Предпосылки

Перед началом работы у вас должен быть установлен Node v6, хотя примеры в основном должны работать и в предыдущих версиях Node. В этом руководстве мы не будем использовать какие-либо языковые функции, требующие транспиляции, но мы будем использовать некоторые функции ES6, такие как [промисы](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Global_Objects/Promise){:target="_blank"}, [классы](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Classes){:target="_blank"} и [стрелочные функции](https://developer.mozilla.org/ru/docs/Web/JavaScript/Reference/Functions/Arrow_functions){:target="_blank"}, поэтому, если вы не знакомы с ними, вы можете сначала прочитать о них.

Чтобы создать новый проект и установить GraphQL.js в текущем каталоге:

```bash
$ npm init
$ npm install graphql --save
```

## Написание кода

Для обработки запросов GraphQL нам нужна схема, которая определяет тип запроса, и нам нужен корень API с функцией под названием «resolver» для каждой конечной точки API. Для API, который просто возвращает «Hello world!», Мы можем поместить этот код в файл с именем ```server.js```:

```
var { graphql, buildSchema } = require('graphql');

// Construct a schema, using GraphQL schema language
var schema = buildSchema(`
  type Query {
    hello: String
  }
`);

// The root provides a resolver function for each API endpoint
var root = {
  hello: () => {
    return 'Hello world!';
  },
};

// Run the GraphQL query '{ hello }' and print out the response
graphql(schema, '{ hello }', root).then((response) => {
  console.log(response);
});
```
Если вы запустите это с:
```bash
node server.js
```
Вы должны увидеть выведенный ответ GraphQL:

```
{ data: { hello: 'Hello world!' } }
```
Поздравляем - вы только что выполнили запрос GraphQL!

Для практических приложений вы, вероятно, захотите запускать запросы GraphQL с сервера API, а не выполнять GraphQL с помощью инструмента командной строки. Чтобы использовать GraphQL для сервера API через HTTP, проверьте [Запуск Сервера Express GraphQL](running-express-server.md).