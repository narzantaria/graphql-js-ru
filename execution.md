# graphql/execution

Модуль ```graphql/execution``` отвечает за этап выполнения выполнения запроса GraphQL. Вы можете импортировать либо из модуля ```graphql/execution```, либо из корневого модуля ```graphql```. Например:

```javascript
import { execute } from 'graphql'; // ES6
var { execute } = require('graphql'); // CommonJS
```

## Обзор

#### [function execute](#execute)
Выполняет запрос GraphQL по предоставленной схеме.

## Execution 

### execute 

```javascript
export function execute(
  schema: GraphQLSchema,
  documentAST: Document,
  rootValue?: mixed,
  contextValue?: mixed,
  variableValues?: ?{[key: string]: mixed},
  operationName?: ?string
): MaybePromise<ExecutionResult>

type MaybePromise<T> = Promise<T> | T;

type ExecutionResult = {
  data: ?Object;
  errors?: Array<GraphQLError>;
}
```

Реализует раздел «Оценка запросов» спецификации GraphQL.

Возвращает обещание, которое в конечном итоге будет решено и никогда не будет отклонено.

Если аргументы этой функции не приводят к допустимому контексту выполнения, немедленно генерируется GraphQLError, объясняя неверный ввод.

```ExecutionResult``` представляет результат выполнения. ```data``` являeтся результатом выполнения запроса, ```errors``` равны нулю, если ошибок не было, и являются непустым массивом, если произошла ошибка.