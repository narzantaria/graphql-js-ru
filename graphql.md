# graphql

Модуль ```graphql``` экспортирует основное подмножество функций GraphQL для создания систем и серверов типа GraphQL.

```javascript
import { graphql } from 'graphql'; // ES6
var { graphql } = require('graphql'); // CommonJS
```

## Обзор

### *Точка входа*

#### [функция graphql](#graphql)
Lex-ит, парсит, проверяет и выполняет запрос GraphQL к схеме.

### *Схема*

#### [класс GraphQLSchema](type.md#GraphQLSchema)
Представление возможностей сервера GraphQL.

### Определения типов

#### [класс GraphQLScalarType](type.md#GraphQLScalarType)
Скалярный тип в GraphQL.
#### [класс GraphQLObjectType](type.md#GraphQLObjectType)
Тип объекта в GraphQL, который содержит поля.
#### [класс GraphQLInterfaceType](type.md#GraphQLInterfaceType)
Тип интерфейса в GraphQL, который определяет реализации полей, будет содержать.
#### [класс GraphQLUnionType](type.md#GraphQLUnionType)
Тип объединения в GraphQL, который определяет список реализаций.
#### [класс GraphQLEnumType](type.md#GraphQLEnumType)
Тип enum в GraphQL, который определяет список допустимых значений.
#### [класс GraphQLInputObjectType](type.md#GraphQLInputObjectType)
Тип входного объекта в GraphQL, который представляет структурированные входные данные.
#### [класс GraphQLList](type.md#GraphQLList)
Обертка типов вокруг других типов, представляющая список этих типов.
#### [класс GraphQLNonNull](type.md#GraphQLNonNull)
Обертка типов вокруг других типов, которая представляет ненулевую версию этих типов.

### Scalars

#### [var GraphQLInt](type.md#GraphQLInt)
Скалярный тип, представляющий целые числа.
#### [var GraphQLFloat](type.md#GraphQLFloat)
Скалярный тип, представляющий числа с плавающей точкой.
#### [var GraphQLString](type.md#GraphQLString)
Скалярный тип, представляющий строки.
#### [var GraphQLBoolean](type.md#GraphQLBoolean)
Скалярный тип, представляющий логические значения.
#### [var GraphQLID](type.md#GraphQLID)
Скалярный тип, представляющий идентификаторы.

### Errors

#### [функция formatError](error.md#formatError)
Форматировать ошибку в соответствии с правилами, описанными в формате ответа.

# Точка входа

## graphql

```javascript
graphql(
  schema: GraphQLSchema,
  requestString: string,
  rootValue?: ?any,
  contextValue?: ?any,
  variableValues?: ?{[key: string]: any},
  operationName?: ?string
): Promise<GraphQLResult>
```

Функция **graphql** осуществляет лексирование, парсинг, проверку и выполнение запроса GraphQL. Она требует **schema** и **requestString**. Необязательные аргументы включают rootValue, который будет передан исполнителю в качестве корневого значения, **contextValue**, который будет передан всем resolve-функциям, **variableValues**, который будет передан исполнителю для предоставления значений для любых переменных в **requestString** и **operationName**, что позволяет вызывающей стороне указывать, какая операция в **requestString** будет выполняться, в тех случаях, когда **requestString** содержит несколько операций верхнего уровня.

## Schema
См. [Справочник API системы типов](type.md#schema).

## Определения типов
См. [Справочник API системы типов](type.md#definitions).

## Скаляры
См. [Справочник API системы типов](type.md#scalars).

## Ошибки
См. [Справочник API ошибок](error.md)