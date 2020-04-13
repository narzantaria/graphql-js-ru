# graphql/type

Модуль ```graphql/type``` отвечает за определение типов и схемы GraphQL. Вы можете импортировать либо из модуля ```graphql/type```, либо из корневого модуля ```graphql```. Например:

```javascript
import { GraphQLSchema } from 'graphql'; // ES6
var { GraphQLSchema } = require('graphql'); // CommonJS
```

## Обзор

### Schema

### [class GraphQLSchema](#GraphQLSchema)
Представление возможностей GraphQL Server.

### Definitions

### [class GraphQLScalarType](#GraphQLScalarType)
Тип scalar в GraphQL.
### [class GraphQLObjectType](#GraphQLObjectType)
Тип object в GraphQL, который содержит fields.
### [class GraphQLInterfaceType](#GraphQLInterfaceType)
Тип interface в GraphQL который определяет представления полей, которые будет содержать.
### [class GraphQLUnionType](#GraphQLUnionType)
Тип union в GraphQL который определяет список представлений.
### [class GraphQLEnumType](#GraphQLEnumType)
Тип enum в GraphQL который определяет список допустимых значений.
### [class GraphQLInputObjectType](#GraphQLInputObjectType)
Тип input object в GraphQL, который представляет структурированные input.
### [class GraphQLList](#GraphQLList)
Обертка типов вокруг других типов, представляющая список этих типов.
### [class GraphQLNonNull](#GraphQLNonNull)
Обертка типов вокруг других типов, которая представляет ненулевую версию этих типов.

### Predicates

### [function isInputType](#isInputType)
Возвращает, если тип может использоваться в качестве входных типов для аргументов и директив.
### [function isOutputType](#isOutputType)
Возвращает, если тип может использоваться в качестве выходных типов как результат полей.
### [function isLeafType](#isLeafType)
Возвращает, если тип может быть конечным значением в ответе.
### [function isCompositeType](#isCompositeType)
Возвращает, если тип может быть родительским контекстом набора selection.
### [function isAbstractType](#isAbstractType)
Возвращает, если тип является комбинацией типов объектов.

### Un-modifiers

### [function getNullableType](#getNullableType)
Удаляет любые ненулевые wrappers из типа.
### [function getNamedType](#getNamedType)
Удаляет любые ненулевые или списки wrappers из типа.

### Scalars

#### [var GraphQLInt](#GraphQLInt)
Скалярный тип, представляющий целые числа.
#### [var GraphQLFloat](#GraphQLFloat)
A scalar type representing floats.
#### [var GraphQLString](#GraphQLString)
Скалярный тип, представляющий числа с плавающей точкой.
#### [var GraphQLBoolean](#GraphQLBoolean)
Скалярный тип, представляющий логические значения.
#### [var GraphQLID](#GraphQLID)
Скалярный тип, представляющий ID.

## Schema

### GraphQLSchema

```javascript
class GraphQLSchema {
  constructor(config: GraphQLSchemaConfig)
}

type GraphQLSchemaConfig = {
  query: GraphQLObjectType;
  mutation?: ?GraphQLObjectType;
}
```

Schema создается путем предоставления корневых типов каждого типа операции, запроса и мутации (необязательно). Затем определение схемы предоставляется валидатору и исполнителю.

### Пример

```javascript
var MyAppSchema = new GraphQLSchema({
  query: MyAppQueryRootType
  mutation: MyAppMutationRootType
});
```

## Определения

### GraphQLScalarType 

```javascript
class GraphQLScalarType<InternalType> {
  constructor(config: GraphQLScalarTypeConfig<InternalType>)
}

type GraphQLScalarTypeConfig<InternalType> = {
  name: string;
  description?: ?string;
  serialize: (value: mixed) => ?InternalType;
  parseValue?: (value: mixed) => ?InternalType;
  parseLiteral?: (valueAST: Value) => ?InternalType;
}
```

Конечные значения любого запроса и входные значения аргументов являются скалярами (или перечислениями) и определяются с помощью имени и ряда функций сериализации, используемых для обеспечения достоверности.

### Пример

```javascript
var OddType = new GraphQLScalarType({
  name: 'Odd',
  serialize: oddValue,
  parseValue: oddValue,
  parseLiteral(ast) {
    if (ast.kind === Kind.INT) {
      return oddValue(parseInt(ast.value, 10));
    }
    return null;
  }
});

function oddValue(value) {
  return value % 2 === 1 ? value : null;
}
```

### GraphQLObjectType

```javascript
class GraphQLObjectType {
  constructor(config: GraphQLObjectTypeConfig)
}

type GraphQLObjectTypeConfig = {
  name: string;
  interfaces?: GraphQLInterfacesThunk | Array<GraphQLInterfaceType>;
  fields: GraphQLFieldConfigMapThunk | GraphQLFieldConfigMap;
  isTypeOf?: (value: any, info?: GraphQLResolveInfo) => boolean;
  description?: ?string
}

type GraphQLInterfacesThunk = () => Array<GraphQLInterfaceType>;

type GraphQLFieldConfigMapThunk = () => GraphQLFieldConfigMap;

// See below about resolver functions.
type GraphQLFieldResolveFn = (
  source?: any,
  args?: {[argName: string]: any},
  context?: any,
  info?: GraphQLResolveInfo
) => any

type GraphQLResolveInfo = {
  fieldName: string,
  fieldNodes: Array<Field>,
  returnType: GraphQLOutputType,
  parentType: GraphQLCompositeType,
  schema: GraphQLSchema,
  fragments: { [fragmentName: string]: FragmentDefinition },
  rootValue: any,
  operation: OperationDefinition,
  variableValues: { [variableName: string]: any },
}

type GraphQLFieldConfig = {
  type: GraphQLOutputType;
  args?: GraphQLFieldConfigArgumentMap;
  resolve?: GraphQLFieldResolveFn;
  deprecationReason?: string;
  description?: ?string;
}

type GraphQLFieldConfigArgumentMap = {
  [argName: string]: GraphQLArgumentConfig;
};

type GraphQLArgumentConfig = {
  type: GraphQLInputType;
  defaultValue?: any;
  description?: ?string;
}

type GraphQLFieldConfigMap = {
  [fieldName: string]: GraphQLFieldConfig;
};
```

Почти все определяемые вами типы GraphQL будут типами object. Типы object имеют имя, но наиболее важно описывают их поля.

Когда два типа должны ссылаться друг на друга или тип должен ссылаться на себя в поле, вы можете использовать функциональное выражение (также называемое замыканием или thunk) для ленивой подачи полей.

Обратите внимание, что функции resolver предоставляются объектом ```source``` в качестве первого параметра. Однако, если функция resolver не предусмотрена, используется resolver по умолчанию, который ищет метод в источнике с тем же именем, что и у поля. Если найдено, метод вызывается с помощью (**args**, **context**, **info**). Поскольку это метод ```source```, на это значение всегда можно ссылаться с помощью ```this```.

### Примеры

```javascript
var AddressType = new GraphQLObjectType({
  name: 'Address',
  fields: {
    street: { type: GraphQLString },
    number: { type: GraphQLInt },
    formatted: {
      type: GraphQLString,
      resolve(obj) {
        return obj.number + ' ' + obj.street
      }
    }
  }
});

var PersonType = new GraphQLObjectType({
  name: 'Person',
  fields: () => ({
    name: { type: GraphQLString },
    bestFriend: { type: PersonType },
  })
});
```

### GraphQLInterfaceType

```javascript
class GraphQLInterfaceType {
  constructor(config: GraphQLInterfaceTypeConfig)
}

type GraphQLInterfaceTypeConfig = {
  name: string,
  fields: GraphQLFieldConfigMapThunk | GraphQLFieldConfigMap,
  resolveType?: (value: any, info?: GraphQLResolveInfo) => ?GraphQLObjectType,
  description?: ?string
};
```

Когда поле может возвращать один из разнородных наборов типов, для описания возможных типов используется тип Interface, какие поля являются общими для всех типов, а также функция для определения того, какой тип фактически используется, когда поле распознано (resolved).

### Пример

```javascript
var EntityType = new GraphQLInterfaceType({
  name: 'Entity',
  fields: {
    name: { type: GraphQLString }
  }
});
```

### GraphQLUnionType

```javascript
class GraphQLUnionType {
  constructor(config: GraphQLUnionTypeConfig)
}

type GraphQLUnionTypeConfig = {
  name: string,
  types: GraphQLObjectsThunk | Array<GraphQLObjectType>,
  resolveType?: (value: any, info?: GraphQLResolveInfo) => ?GraphQLObjectType;
  description?: ?string;
};

type GraphQLObjectsThunk = () => Array<GraphQLObjectType>;
```

Когда поле может возвращать один из неоднородного набора типов, тип Union используется для описания возможных типов, а также для предоставления функции, позволяющей определить, какой тип фактически используется при распознавании (resolve) поля.

### Пример

```javascript
var PetType = new GraphQLUnionType({
  name: 'Pet',
  types: [ DogType, CatType ],
  resolveType(value) {
    if (value instanceof Dog) {
      return DogType;
    }
    if (value instanceof Cat) {
      return CatType;
    }
  }
});
```

### GraphQLEnumType

```javascript
class GraphQLEnumType {
  constructor(config: GraphQLEnumTypeConfig)
}

type GraphQLEnumTypeConfig = {
  name: string;
  values: GraphQLEnumValueConfigMap;
  description?: ?string;
}

type GraphQLEnumValueConfigMap = {
  [valueName: string]: GraphQLEnumValueConfig;
};

type GraphQLEnumValueConfig = {
  value?: any;
  deprecationReason?: string;
  description?: ?string;
}

type GraphQLEnumValueDefinition = {
  name: string;
  value?: any;
  deprecationReason?: string;
  description?: ?string;
}
```

Некоторые конечные значения запросов и входных значений являются Enums. GraphQL сериализует значения Enum в виде строк, однако внутренне Enum может быть представлен любым типом, часто целыми числами.

Примечание. Если значение не указано в определении, имя значения enum будет использоваться в качестве его внутреннего значения.

### Пример

```javascript
var RGBType = new GraphQLEnumType({
  name: 'RGB',
  values: {
    RED: { value: 0 },
    GREEN: { value: 1 },
    BLUE: { value: 2 }
  }
});
```

### GraphQLInputObjectType

```javascript
class GraphQLInputObjectType {
  constructor(config: GraphQLInputObjectConfig)
}

type GraphQLInputObjectConfig = {
  name: string;
  fields: GraphQLInputObjectConfigFieldMapThunk | GraphQLInputObjectConfigFieldMap;
  description?: ?string;
}

type GraphQLInputObjectConfigFieldMapThunk = () => GraphQLInputObjectConfigFieldMap;

type GraphQLInputObjectFieldConfig = {
  type: GraphQLInputType;
  defaultValue?: any;
  description?: ?string;
}

type GraphQLInputObjectConfigFieldMap = {
  [fieldName: string]: GraphQLInputObjectFieldConfig;
};

type GraphQLInputObjectField = {
  name: string;
  type: GraphQLInputType;
  defaultValue?: any;
  description?: ?string;
}

type GraphQLInputObjectFieldMap = {
  [fieldName: string]: GraphQLInputObjectField;
};
```

Входной объект определяет структурированную коллекцию полей, которые могут быть предоставлены в качестве аргумента поля.

Использование ```NonNull``` гарантирует, что значение должно быть предоставлено запросом

### Пример

```javascript
var GeoPoint = new GraphQLInputObjectType({
  name: 'GeoPoint',
  fields: {
    lat: { type: new GraphQLNonNull(GraphQLFloat) },
    lon: { type: new GraphQLNonNull(GraphQLFloat) },
    alt: { type: GraphQLFloat, defaultValue: 0 },
  }
});
```

### GraphQLList

```javascript
class GraphQLList {
  constructor(type: GraphQLType)
}
```

Список - это своего рода маркер типа, оберточный (wrapper) тип, который указывает на другой тип. Списки часто создаются в контексте определения полей типа объекта.

### Пример

```javascript
var PersonType = new GraphQLObjectType({
  name: 'Person',
  fields: () => ({
    parents: { type: new GraphQLList(PersonType) },
    children: { type: new GraphQLList(PersonType) },
  })
});
```

### GraphQLNonNull

```javascript
class GraphQLNonNull {
  constructor(type: GraphQLType)
}
```

Ненулевое значение - это разновидность маркера типа, оберточный (wrapper) тип, который указывает на другой тип. Ненулевые типы гарантируют, что их значения никогда не равны NULL и могут гарантировать возникновение ошибки, если это когда-либо произойдет во время запроса. Это полезно для полей, в которых вы можете дать строгую гарантию необнуляемости, например, обычно поле id строки базы данных никогда не будет нулевым.

### Пример

```javascript
var RowType = new GraphQLObjectType({
  name: 'Row',
  fields: () => ({
    id: { type: new GraphQLNonNull(String) },
  })
});
```

## Предикаты

### isInputType

```javascript
function isInputType(type: ?GraphQLType): boolean
```

Эти типы могут использоваться в качестве типов ввода для аргументов и директив.

### isOutputType

```javascript
function isOutputType(type: ?GraphQLType): boolean
```

Эти типы могут использоваться как выходные типы как результат полей

### isLeafType

```javascript
function isLeafType(type: ?GraphQLType): boolean
```

Эти типы могут описывать типы, которые могут быть конечными значениями.

### isCompositeType

```javascript
function isCompositeType(type: ?GraphQLType): boolean
```

Эти типы могут описывать родительский контекст набора selection

### isAbstractType

```javascript
function isAbstractType(type: ?GraphQLType): boolean
```

Эти типы могут описывать комбинацию типов объектов

## Un-modifiers

### getNullableType

```javascript
function getNullableType(type: ?GraphQLType): ?GraphQLNullableType
```

Если данный тип non-nullable, это удаляет non-nullability и возвращает основополагающий тип.

### getNamedType

```javascript
function getNamedType(type: ?GraphQLType): ?GraphQLNamedType
```

Если данный тип non-nullable или является списком, это повторяет удаление non-nullability и list wrappers, и возвращает основополагающий тип.

## Скаляры

### GraphQLInt

```javascript
var GraphQLInt: GraphQLScalarType;
```

GraphQLScalarType, представляющий int.

### GraphQLFloat 

```javascript
var GraphQLFloat: GraphQLScalarType;
```

GraphQLScalarType представляющий float.

### GraphQLString 

```javascript
var GraphQLString: GraphQLScalarType;
```

GraphQLScalarType представляющий string.

### GraphQLBoolean 

```javascript
var GraphQLBoolean: GraphQLScalarType;
```

GraphQLScalarType представляющий boolean.

### GraphQLID 

```javascript
var GraphQLID: GraphQLScalarType;
```

GraphQLScalarType представляющийn ID.