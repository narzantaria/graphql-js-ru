# graphql/validation

Модуль ```graphql/validation``` выполняет этап проверки выполнения результата GraphQL. Вы можете импортировать либо из модуля ```graphql/validation```, либо из корневого модуля graphql. Например:

```
import { validate } from 'graphql/validation'; // ES6
var { validate } = require('graphql/validation'); // CommonJS
```

## Обзор

### [function validate](#validate)
Проверяет AST по предоставленной схеме.
### [var specifiedRules](#specifiedRules)
Список стандартных правил проверки, описанных в спецификации GraphQL.

## Validation 

### validate

```
function validate(
  schema: GraphQLSchema,
  ast: Document,
  rules?: Array<any>
): Array<GraphQLError>
```

Реализует раздел «Проверка» спецификации.

Проверка выполняется синхронно, возвращая массив обнаруженных ошибок или пустой массив, если ошибок не обнаружено и документ действителен.

Список конкретных правил проверки может быть предоставлен. Если не указан, будет использован список правил по умолчанию, определенный спецификацией GraphQL.

Каждое правило проверки - это функция, которая возвращает visitor  (см. API language/visitor). Ожидается, что методы visitor будут возвращать GraphQLErrors или Arrays of GraphQLErrors, если они недействительны.

Visitors могут также предоставить visitSpreadFragments: true, который изменит поведение visitor, чтобы пропустить определенные фрагменты верхнего уровня, и вместо этого посещать эти фрагменты в каждой точке, где встречается распространение(?).

### specifiedRules 

```
var specifiedRules: Array<(context: ValidationContext): any>
```

Этот набор включает в себя все правила проверки, определенные спецификацией GraphQL.