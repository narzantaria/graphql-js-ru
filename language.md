# graphql/language

Модуль ```graphql/language``` отвечает за синтаксический анализ и работу на языке GraphQL. Вы можете импортировать либо из модуля ```graphql/language```, либо из корневого модуля ```graphql```. Например:

```javascript
import { Source } from 'graphql'; // ES6
var { Source } = require('graphql'); // CommonJS
```

## Обзор

### Источник

#### [class Source](#Source)
Представляет входную строку для сервера GraphQL
#### [function getLocation](#getLocation)
Преобразует смещение символа в строку и столбец в Source

### Lexer

#### [function lex](#lex)
Лексирует источник GraphQL в соответствии с GraphQL Grammar

### Parser

#### [function parse](#parse)
Парсит GraphQL Source в соответствии с GraphQL Grammar
#### [function parseValue](#parseValue)
Разбирает значение в соответствии с GraphQL Grammar
#### [var Kind](#Kind)
Представляет различные виды анализируемых узлов AST.

### Visitor

#### [function visit](#visit)
Универсальный visitor для прохождения проанализированного GraphQL AST
#### [var BREAK](#BREAK)
Токен, позволяющий вырваться из посетителя.

### Printer

#### [function print](#print)
Печатает AST в стандартном формате.

## Source 

### Source

```javascript
export class Source {
  constructor(body: string, name?: string)
}
```

Представление исходного ввода в GraphQL. Имя необязательно, но в основном полезно для клиентов, которые хранят документы GraphQL в исходных файлах; например, если входные данные GraphQL находятся в файле Foo.graphql, может быть полезно, чтобы name было "Foo.graphql".

### getLocation

```javascript
function getLocation(source: Source, position: number): SourceLocation

type SourceLocation = {
  line: number;
  column: number;
}
```

Принимает источник и смещение символа UTF-8 и возвращает соответствующую строку и столбец как SourceLocation.

## Lexer

### lex

```javascript
function lex(source: Source): Lexer;

type Lexer = (resetPosition?: number) => Token;

export type Token = {
  kind: number;
  start: number;
  end: number;
  value: ?string;
};
```

С учетом Source он возвращает Lexer для данного source. Lexer - это функция, которая действует как генератор, и каждый раз, когда она вызывается, она возвращает следующий токен в Source. При условии использования исходных лексов, последний токен, испускаемый лексером, будет иметь вид EOF, после чего лексер будет повторно возвращать токены EOF при каждом вызове.

Аргумент функции lexer является необязательным и может использоваться для возврата или быстрой перемотки лексера на новую позицию в source.

## Парсер

### parse

```javascript
export function parse(
  source: Source | string,
  options?: ParseOptions
): Document
```

Учитывая исходный код GraphQL, анализирует его в Document.

Выдает GraphQLError, если обнаружена синтаксическая ошибка.

### parseValue

```javascript
export function parseValue(
  source: Source | string,
  options?: ParseOptions
): Value
```

Учитывая строку, содержащую значение GraphQL, парсить AST для этого значения.

Выдает GraphQLError, если обнаружена синтаксическая ошибка.

Это полезно в инструментах, которые работают со значениями GraphQL напрямую и в отрыве от полных документов GraphQL.

### Kind 

Перечисление, описывающее различные виды AST nodes.

## Visitor 

### visit

```javascript
function visit(root, visitor, keyMap)
```

Метод visit () будет проходить через AST с использованием первого прохода глубины, вызывая функцию ввода visitor на каждом node прохода и вызывая функцию leave после посещения этого node и всех его дочерних node.

Возвращая различные value из функций enter и leave, можно изменить поведение visitor, в том числе пропуская поддерево AST (возвращая false), редактируя AST за счет возврата value или null для удаления value, или остановить весь проход, вернув BREAK.

При использовании visit() для редактирования AST, исходный AST не будет изменен, и новая функция AST с внесенными изменениями будет возвращена из функции visitor.

```javascript
var editedAST = visit(ast, {
  enter(node, key, parent, path, ancestors) {
    // @return
    //   undefined: no action
    //   false: skip visiting this node
    //   visitor.BREAK: stop visiting altogether
    //   null: delete this node
    //   any value: replace this node with the returned value
  },
  leave(node, key, parent, path, ancestors) {
    // @return
    //   undefined: no action
    //   false: no action
    //   visitor.BREAK: stop visiting altogether
    //   null: delete this node
    //   any value: replace this node with the returned value
  }
});
```

В качестве альтернативы предоставлению функций enter() и exit(), visitor может вместо этого предоставить функции, названные так же, как виды узлов AST, или ввести/оставить посетителей по названному ключу, что приводит к четырем перестановкам API visitor:

1) Именованные посетители срабатывают при входе в node определенного вида.

```javascript
visit(ast, {
  Kind(node) {
    // enter the "Kind" node
  }
})
```

2) Именованные посетители, которые срабатывают при входе и выходе из node определенного вида.

```javascript
visit(ast, {
  Kind: {
    enter(node) {
      // enter the "Kind" node
    }
    leave(node) {
      // leave the "Kind" node
    }
  }
})
```

3) Общие visitors, которые срабатывают при входе и выходе из любого node.

```javascript
visit(ast, {
  enter(node) {
    // enter any node
  },
  leave(node) {
    // leave any node
  }
})
```

4) Параллельные visitors для входа и выхода из node определенного вида.

```javascript
visit(ast, {
  enter: {
    Kind(node) {
      // enter the "Kind" node
    }
  },
  leave: {
    Kind(node) {
      // leave the "Kind" node
    }
  }
})
```

## BREAK

Часовой показатель BREAK описан в документации посетителя.

## Printer

### print 

```javascript
function print(ast): string
```

Преобразует AST в строку, используя один набор разумных правил форматирования.

