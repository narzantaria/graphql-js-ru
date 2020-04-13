# graphql/utilities

Модуль ```graphql/utilities``` содержит общие полезные вычисления для использования с языком GraphQL и объектами типов. Вы можете импортировать либо из модуля ```graphql/utilities```, либо из корневого модуля ```graphql```. Например:

```
import { introspectionQuery } from 'graphql'; // ES6
var { introspectionQuery } = require('graphql'); // CommonJS
```

## Обзор

### Introspection

#### var introspectionQuery
Запрос introspection GraphQL, содержащий достаточно информации для воспроизведения системы типов.

#### function buildClientSchema
Создает клиентскую схему с учетом результат запроса схемы с `introspectionQuery`.

### Schema Language

#### function buildSchema
Создает объект Schema из языка схем GraphQL.

#### function printSchema
Выводит(?) схему в стандартном формате.

#### function printIntrospectionSchema
Выводит(?) функции introspection схемы в стандартном формате.

#### function buildASTSchema
Создает схему из разобранной(parsed) схемы AST.

#### function typeFromAST
Ищет тип, указанный в AST в GraphQLSchema.

#### function astFromValue
Создает значение Input GraphQL AST с учетом значения JavaScript.

### Visitors

#### class TypeInfo
Отслеживает тип и определения полей во время прохода(traversal) AST visitor.

### Value Validation

#### function isValidJSValue
Определяет, допустимо ли значение JavaScript для типа GraphQL.

#### function isValidLiteralValue
Определяет, допустимо ли литеральное значение из AST для типа GraphQL.

## Introspection

### introspectionQuery

```
var introspectionQuery: string
```

Запрос GraphQL, который запрашивает у системы introspection сервера достаточное количество информации для воспроизведения системы типов этого сервера.

### buildClientSchema

```
function buildClientSchema(
  introspection: IntrospectionQuery
): GraphQLSchema
```

Создание GraphQLSchema для использования клиентскими инструментами.

Учитывая результат выполнения клиентом запроса introspection, он создает и возвращает экземпляр GraphQLSchema, который затем можно использовать со всеми инструментами GraphQL.js, но нельзя использовать для выполнения запроса, поскольку introspection не представляет функции "resolver", "parse" или "serialize" или любые другие внутренние механизмы сервера.

## Schema Representation

### buildSchema 

```
function buildSchema(source: string | Source): GraphQLSchema {
```

Создает объект GraphQLSchema из языка схемы GraphQL. Схема будет использовать *resolvers* по умолчанию. Для получения более подробной информации о языке схемы GraphQL см. [Документацию по языку схемы](https://graphql.org/learn/schema/) или эту [шпаргалку по языку схемы](https://wehavefaces.net/graphql-shorthand-notation-cheatsheet-17cd715861b6#.9oztv0a7n).

### printSchema

```
function printSchema(schema: GraphQLSchema): string {
```

Печатает предоставленную схему в формате языка схемы.

### printIntrospectionSchema

```
function printIntrospectionSchema(schema: GraphQLSchema): string {
```

Печатает встроенную схему introspection в формате языка схемы.

### buildASTSchema

```
function buildASTSchema(
  ast: SchemaDocument,
  queryTypeName: string,
  mutationTypeName: ?string
): GraphQLSchema
```

Это берет весь документ схемы, созданный ```parseSchemaIntoAST``` в ```graphql/language/schema```, и создает экземпляр GraphQLSchema, который затем можно использовать со всеми инструментами GraphQL.js, но нельзя использовать для выполнения запроса, так как introspection не представляет функции "resolver", "parse" или "serialize" или любые другие внутренние механизмы сервера.

### typeFromAST

```
function typeFromAST(
  schema: GraphQLSchema,
  inputTypeAST: Type
): ?GraphQLType
```

Учитывая имя Type, как оно отображается в GraphQL AST и Schema, вернуть соответствующий GraphQLType из этой схемы.

### astFromValue

```
function astFromValue(
  value: any,
  type?: ?GraphQLType
): ?Value
```

Создает входное значение GraphQL AST с учетом значения JavaScript.

По желанию может быть предоставлен тип GraphQL, который будет использоваться для устранения неоднозначности между примитивами значений.

## Visitors 

### TypeInfo

```
class TypeInfo {
  constructor(schema: GraphQLSchema)
  getType(): ?GraphQLOutputType {
  getParentType(): ?GraphQLCompositeType {
  getInputType(): ?GraphQLInputType {
  getFieldDef(): ?GraphQLFieldDefinition {
  getDirective(): ?GraphQLDirective {
  getArgument(): ?GraphQLArgument {
}
```

TypeInfo - это служебный класс, который, учитывая схему GraphQL, может отслеживать текущее поле и определения типов в любой точке документа GraphQL AST во время рекурсивного спуска с помощью вызова enter(node) и leave(node).

## Value Validation 

### isValidJSValue 

```
function isValidJSValue(value: any, type: GraphQLInputType): string[]
```

Учитывая значение JavaScript и тип GraphQL, определить, будет ли это значение принято для этого типа. Это в первую очередь полезно для проверки значений времени выполнения переменных запроса.

### isValidLiteralValue

```
function isValidLiteralValue(
  type: GraphQLInputType,
  valueAST: Value
): string[]
```

Утилита для validators, которая определяет, является ли значение литерала AST допустимым для данного типа ввода.

Обратите внимание, что это проверяет только литеральные значения, предполагается, что переменные предоставляют значения правильного типа.