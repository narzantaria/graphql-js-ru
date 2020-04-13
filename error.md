# graphql/error

Модуль ```graphql/error``` отвечает за создание и форматирование ошибок GraphQL. Вы можете импортировать либо из модуля ```graphql/error```, либо из корневого модуля ```graphql```. Например:

```
import { GraphQLError } from 'graphql'; // ES6
var { GraphQLError } = require('graphql'); // CommonJS
```

## Обзор

#### [class GraphQLError](#GraphQLError)
Представление ошибки, которая произошла в GraphQL.
#### [функция syntaxError](#syntaxError)
Создает GraphQLError, представляющую синтаксическую ошибку.
#### [функция localError](#localError)
Создает новый GraphQLError, осведомленный о местоположении, ответственном за ошибку.
#### [функция formatError](#formatError)
Отформатируйте ошибку в соответствии с правилами, описанными в формате ответа.

## Ошибки

### GraphQLError

```
class GraphQLError extends Error {
 constructor(
   message: string,
   nodes?: Array<any>,
   stack?: ?string,
   source?: Source,
   positions?: Array<number>,
   originalError?: ?Error,
   extensions?: ?{ [key: string]: mixed }
 )
}
```

Представление ошибки, которая произошла в GraphQL. Содержит информацию о том, где в запросе произошла ошибка при отладке. Чаще всего строится с помощью ```locatedError``` ниже.

#### syntaxError

```
function syntaxError(
  source: Source,
  position: number,
  description: string
): GraphQLError;
```

Создает GraphQLError, представляющую синтаксическую ошибку, содержащую полезную описательную информацию о позиции синтаксической ошибки в источнике.

#### locatedError

```
function locatedError(error: ?Error, nodes: Array<any>): GraphQLError {
```

При произвольной ошибке, которая, вероятно, выдается при попытке выполнить операцию GraphQL, генерирует новый GraphQLError, в котором содержится информация о местоположении в документе, ответственном за исходную ошибку.

#### formatError

```
function formatError(error: GraphQLError): GraphQLFormattedError

type GraphQLFormattedError = {
  message: string,
  locations: ?Array<GraphQLErrorLocation>
};

type GraphQLErrorLocation = {
  line: number,
  column: number
};
```

При наличии GraphQLError отформатируйте его в соответствии с правилами, описанными в разделе «Формат ответа» в разделе «Ошибки» спецификации GraphQL.


