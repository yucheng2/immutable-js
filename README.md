Immutable collections for JavaScript

简体中文 | [English](README-en.md)

不可变集合用于 JavaScript

[![Build Status](https://github.com/immutable-js/immutable-js/actions/workflows/ci.yml/badge.svg?branch=main)](https://github.com/immutable-js/immutable-js/actions/workflows/ci.yml?query=branch%3Amain) [Chat on slack](https://immutable-js.slack.com)

[阅读文档](https://immutable-js.com/docs/) 并吃你的蔬菜。

文档自动从 [README.md][] 和 [immutable.d.ts][] 生成。
请贡献！同样不要错过 [wiki][]，其中包含有关其他特定主题的文章。
找不到东西？打开 [问题][]。

**目录：**

- [介绍](#introduction)
- [入门](#getting-started)
- [不可变性的案例](#the-case-for-immutability)
- [JavaScript优先API](#javascript-first-api)
- [嵌套结构](#nested-structures)
- [平等将集合视为值](#equality-treats-collections-as-values)
- [批量变更](#batching-mutations)
- [懒惰序列](#lazy-seq)
- [额外的工具和资源](#additional-tools-and-resources)
- [贡献](#contributing)

## 介绍

[Immutable][] 数据一旦创建就不能更改，这导致应用程序开发更简单，
无需防御性复制，并启用高级备忘录和变更检测技术，使用简单的逻辑。
[Persistent][] 数据呈现一个变异的 API，它不更新数据就地，而是始终
产生新的更新数据。

Immutable.js 提供了许多持久不可变数据结构，包括：
`List`, `Stack`, `Map`, `OrderedMap`, `Set`, `OrderedSet` 和 `Record`。

这些数据结构通过使用 [哈希映射树][] 和 [向量树][] 通过结构共享来实现高度效率，
正如 Clojure 和 Scala 所推广的，最小化了复制或缓存数据的需求。

Immutable.js 还提供了一个懒惰的 `Seq`，允许高效地
链接集合方法，如 `map` 和 `filter`，而无需创建中间表示。使用 `Range` 和 `Repeat` 创建一些 `Seq`。

想了解更多吗？观看有关 Immutable.js 的演示：

[![Immutable Data and React](website/public/Immutable-Data-and-React-YouTube.png)](https://youtu.be/I7IdS-PbEgI)

[README.md]: https://github.com/immutable-js/immutable-js/blob/main/README.md
[immutable.d.ts]: https://github.com/immutable-js/immutable-js/blob/main/type-definitions/immutable.d.ts
[wiki]: https://github.com/immutable-js/immutable-js/wiki
[issue]: https://github.com/immutable-js/immutable-js/issues
[Persistent]: https://en.wikipedia.org/wiki/Persistent_data_structure
[Immutable]: https://en.wikipedia.org/wiki/Immutable_object
[hash maps tries]: https://en.wikipedia.org/wiki/Hash_array_mapped_trie
[vector tries]: https://hypirion.com/musings/understanding-persistent-vector-pt-1

## 入门

使用 npm 安装 `immutable`。

```shell
# 使用 npm
npm install immutable

# 使用 Yarn
yarn add immutable

# 使用 pnpm
pnpm add immutable

# 使用 Bun
bun add immutable
```

然后将其引入到任何模块。

```js
const { Map } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3 });
const map2 = map1.set('b', 50);
map1.get('b') + ' 对比 ' + map2.get('b'); // 2 对比 50
```

### 浏览器

Immutable.js 没有依赖项，这使得它在浏览器中包含是可预测的。

强烈建议使用像 [webpack](https://webpack.github.io/)、
[rollup](https://rollupjs.org/) 或 [browserify](https://browserify.org/) 这样的模块打包器。
`immutable` npm 模块无需额外考虑即可工作。
整个文档中的所有示例都将假定使用此类工具。

或者，Immutable.js 可以直接作为脚本标签包含。下载或链接到 CDN 例如 [CDNJS](https://cdnjs.com/libraries/immutable)
或 [jsDelivr](https://www.jsdelivr.com/package/npm/immutable)。

使用脚本标签直接将 `Immutable` 添加到全局作用域：

```html
<script src="https://cdn.jsdelivr.net/npm/immutable@4.0.0-rc.12/dist/immutable.min.js"></script>
<script>
  var map1 = Immutable.Map({ a: 1, b: 2, c: 3 });
  var map2 = map1.set('b', 50);
  console.log(map1.get('b')); // 2
  console.log(map2.get('b')); // 50
</script>
```

## 不可变性的案例

使应用程序开发变得困难的许多事情是跟踪变异和维护状态。
使用不可变数据开发鼓励您以不同的方式思考数据如何流经您的应用程序。

在应用程序中订阅数据事件会创建巨大的记账开销，这可能会损害性能，
有时是戏剧性的，并为应用程序的不同区域由于容易犯的程序员错误而变得不同步创造机会。
由于不可变数据永远不会改变，因此在整个模型中订阅更改是死路一条，
新数据只能从上面传递。

这种数据流模型与 [React][] 的架构非常契合，
特别是与使用 [Flux][] 思想设计的应用程序非常契合。

当数据是从上面传递而不是被订阅时，并且您只对在某些事情发生变化时进行工作感兴趣，
您可以使用等式。

不可变集合应该被视为 _值_ 而不是 _对象_。
虽然对象表示可能随时间变化的事物，但值表示该事物在特定时间点的状态。
这个原则对于理解不可变数据的适当使用至关重要。为了将 Immutable.js 集合视为值，
重要的是使用 `Immutable.is()` 函数或 `.equals()` 方法来确定 _值等式_
而不是使用 `===` 操作符，它确定对象 _引用身份_。

```js
const { Map } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3 });
const map2 = Map({ a: 1, b: 2, c: 3 });
map1.equals(map2); // true
map1 === map2; // false
```

注意：作为一种性能优化，Immutable.js 尝试在操作将产生相同集合时返回现有集合，
允许使用 `===` 引用等式来快速确定某事物肯定没有变化。
这可以在 memoization 函数中非常有用，如果更深层次的等式检查可能成本更高，
则宁愿重新运行函数。`===` 等式检查也作为性能优化在 `Immutable.is` 和 `.equals()` 内部使用。

```js
const { Map } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3 });
const map2 = map1.set('b', 2); // 设置为相同值
map1 === map2; // true
```

如果一个对象是不可变的，那么可以通过简单地制作另一个引用而不是复制整个对象来 "复制" 它。
因为引用比对象本身小得多，这可以在内存节省和潜在的加速执行速度方面带来好处，
对于依赖副本的程序（例如撤销栈）。

```js
const { Map } = require('immutable');
const map = Map({ a: 1, b: 2, c: 3 });
const mapCopy = map; // 看，"副本" 是免费的！
```

[React]: https://reactjs.org/
[Flux]: https://facebook.github.io/flux/docs/in-depth-overview/

## JavaScript-first API

虽然 Immutable.js 受到 Clojure、Scala、Haskell 以及其他函数式编程环境的启发，
但它的设计理念是将这些强大的概念带给 JavaScript，
因此它具有面向对象的 API，与 ES2015 的 [Array][]、[Map][] 和 [Set][] 非常相似。

[es2015]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/New_in_JavaScript/ECMAScript_6_support_in_Mozilla
[array]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Array
[map]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Map
[set]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Set

不可变集合与可变集合的不同之处在于，那些会变异集合的方法，
如 `push`、`set`、`unshift` 或 `splice`，反而返回一个新的不可变集合。
那些返回新数组的方法，如 `slice` 或 `concat`，反而返回新的不可变集合。

```js
const { List } = require('immutable');
const list1 = List([1, 2]);
const list2 = list1.push(3, 4, 5);
const list3 = list2.unshift(0);
const list4 = list1.concat(list2, list3);
assert.equal(list1.size, 2);
assert.equal(list2.size, 5);
assert.equal(list3.size, 6);
assert.equal(list4.size, 13);
assert.equal(list4.get(0), 1);
```

几乎所有的 [Array][] 方法都可以在 `Immutable.List` 上找到相似的形式，[Map][] 的方法可以在 `Immutable.Map` 上找到，[Set][] 的方法可以在 `Immutable.Set` 上找到，包括集合操作如 `forEach()` 和 `map()`。

```js
const { Map } = require('immutable');
const alpha = Map({ a: 1, b: 2, c: 3, d: 4 });
alpha.map((v, k) => k.toUpperCase()).join();
// 'A,B,C,D'
```

### 从原生 JavaScript 对象和数组转换

为了与现有的 JavaScript 互操作，Immutable.js 接受普通的 JavaScript 数组和对象作为任何期望 `Collection` 的方法的输入。

```js
const { Map, List } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3, d: 4 });
const map2 = Map({ c: 10, a: 20, t: 30 });
const obj = { d: 100, o: 200, g: 300 };
const map3 = map1.merge(map2, obj);
// Map { a: 20, b: 2, c: 10, d: 100, t: 30, o: 200, g: 300 }
const list1 = List([1, 2, 3]);
const list2 = List([4, 5, 6]);
const array = [7, 8, 9];
const list3 = list1.concat(list2, array);
// List [ 1, 2, 3, 4, 5, 6, 7, 8, 9 ]
```

这是因为 Immutable.js 可以将任何 JavaScript 数组或对象视为 Collection。您可以利用这一点，在 JavaScript 对象上使用复杂的集合方法，这些对象本身具有非常稀疏的原生 API。因为 Seq 延迟求值并不缓存中间结果，这些操作可以非常高效。

```js
const { Seq } = require('immutable');
const myObject = { a: 1, b: 2, c: 3 };
Seq(myObject)
  .map(x => x * x)
  .toObject();
// { a: 1, b: 4, c: 9 }
```

请注意，当使用 JS 对象构造 Immutable Maps 时，JavaScript 对象属性始终是字符串，即使在不带引号的简写中编写，而 Immutable Maps 接受任何类型的键。

```js
const { fromJS } = require('immutable');

const obj = { 1: 'one' };
console.log(Object.keys(obj)); // [ "1" ]
console.log(obj['1'], obj[1]); // "one", "one"

const map = fromJS(obj);
console.log(map.get('1'), map.get(1)); // "one", undefined
```

属性访问对 JavaScript 对象首先将键转换为字符串，但由于 Immutable Map 键可以是任何类型，`get()` 的参数不会被改变。

### 转换回原生 JavaScript 对象

所有 Immutable.js 集合都可以使用 `toArray()` 和 `toObject()` 或者 `toJS()` 深度转换为普通的 JavaScript 数组和对象。

所有 Immutable 集合还实现了 `toJSON()`，允许它们直接传递给 `JSON.stringify`。它们还尊重嵌套对象的自定义 `toJSON()` 方法。

```js
const { Map, List } = require('immutable');
const deep = Map({ a: 1, b: 2, c: List([3, 4, 5]) });
console.log(deep.toObject()); // { a: 1, b: 2, c: List [ 3, 4, 5 ] }
console.log(deep.toArray()); // [ 1, 2, List [ 3, 4, 5 ] ]
console.log(deep.toJS()); // { a: 1, b: 2, c: [ 3, 4, 5 ] }
JSON.stringify(deep); // '{"a":1,"b":2,"c":[3,4,5]}'
```

### 拥抱 ES2015

Immutable.js 支持所有 JavaScript 环境，包括旧版浏览器（甚至是 IE11）。
然而，它也利用了 JavaScript 在 [ES2015][] 中添加的功能，这是 JavaScript 的最新标准版本，
包括 [迭代器][]、[箭头函数][]、[类][] 和 [模块][]。它受到 ES2015 中添加的原生 [Map][] 和 [Set][] 集合的启发。

所有文档中的示例都以 ES2015 呈现。要在所有浏览器中运行，它们需要转换为 ES5。

```js
// ES2015
const mapped = foo.map(x => x * x);
// ES5
var mapped = foo.map(function (x) {
  return x * x;
});
```

所有 Immutable.js 集合都是 [可迭代的][iterators]，这允许它们在期望可迭代对象的地方使用，
例如在展开到数组时。

```js
const { List } = require('immutable');
const aList = List([1, 2, 3]);
const anArray = [0, ...aList, 4, 5]; // [ 0, 1, 2, 3, 4, 5 ]
```

注意：集合始终以相同的顺序迭代，但顺序可能并不总是明确定义的，就像 `Map` 和 `Set` 的情况一样。

[迭代器]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Guide/The_Iterator_protocol
[箭头函数]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Functions/Arrow_functions
[类]: https://wiki.ecmascript.org/doku.php?id=strawman:maximally_minimal_classes
[模块]: https://www.2ality.com/2014/09/es6-modules-final.html

## 嵌套结构

Immutable.js 中的集合旨在嵌套，允许深层的数据树，类似于 JSON。

```js
const { fromJS } = require('immutable');
const nested = fromJS({ a: { b: { c: [3, 4, 5] } } });
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ] } } }
```

一些强大的工具允许读取和操作嵌套数据。最有用的包括 `mergeDeep`、`getIn`、`setIn` 和 `updateIn`，在 `List`、`Map` 和 `OrderedMap` 上找到。

```js
const { fromJS } = require('immutable');
const nested = fromJS({ a: { b: { c: [3, 4, 5] } } });

const nested2 = nested.mergeDeep({ a: { b: { d: 6 } } });
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 6 } } }

console.log(nested2.getIn(['a', 'b', 'd'])); // 6

const nested3 = nested2.updateIn(['a', 'b', 'd'], value => value + 1);
console.log(nested3);
// Map { a: Map { b: Map { c: List [ 3, 4, 5 ], d: 7 } } }

const nested4 = nested3.updateIn(['a', 'b', 'c'], list => list.push(6));
// Map { a: Map { b: Map { c: List [ 3, 4, 5, 6 ], d: 7 } } }
```

## 将集合视为值的平等性

Immutable.js 集合被视为纯数据 _值_。两个不可变集合被认为是 _值相等_（通过 `.equals()` 或 `is()`）如果它们表示相同的值集合。这与 JavaScript 的典型 _引用相等_（通过 `===` 或 `==`）不同，后者只确定两个变量是否表示指向同一对象实例的引用。

考虑下面的例子，其中两个相同的 `Map` 实例不是 _引用相等_，但是是 _值相等_。

```js
// 首先考虑：
const obj1 = { a: 1, b: 2, c: 3 };
const obj2 = { a: 1, b: 2, c: 3 };
obj1 !== obj2; // 两个不同的实例永远不等于 ===

const { Map, is } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3 });
const map2 = Map({ a: 1, b: 2, c: 3 });
map1 !== map2; // 两个不同的实例不是引用相等
map1.equals(map2); // 但如果它们具有相同的值，它们就是值相等
is(map1, map2); // 另外可以使用 is() 函数
```

值相等允许 Immutable.js 集合用作 Maps 或 Sets 中的键，并且可以用不同但等价的集合检索：

```js
const { Map, Set } = require('immutable');
const map1 = Map({ a: 1, b: 2, c: 3 });
const map2 = Map({ a: 1, b: 2, c: 3 });
const set = Set().add(map1);
set.has(map2); // 为 true，因为它们值相等
```

注意：`is()` 使用与 [Object.is][] 对于标量字符串和数字相同的等式度量，但对 Immutable 集合使用值等式，确定两者是否都是不可变的，并且所有键和值使用相同的等式度量都是相等的。

[Object.is]: https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/is

#### 性能折衷

尽管值等式在许多情况下都很有用，但它与引用等式有不同的性能特点。了解这些折衷可能有助于您决定在每种情况下使用哪一个，特别是在用于记忆化操作时。

当比较两个集合时，值等式可能需要考虑每个集合中的每个项目，具有 `O(N)` 时间复杂度。对于大型值集合，这可能是一个成本较高的操作。尽管如果两个集合不相等并且几乎不相似，不等式会很快被确定。相比之下，当使用引用等式比较两个集合时，只需要比较两个内存的初始引用，这与集合的大小无关，具有 `O(1)` 时间复杂度。检查引用等式总是非常快的，然而，仅仅因为两个集合不是引用相等，并不能排除它们可能值相等的可能性。

#### 无操作优化返回自身

尽可能地，Immutable.js 避免在更新中创建新对象，当没有发生 _值_ 变化时，允许使用高效的 _引用等式_ 检查来快速确定是否没有发生变化。

```js
const { Map } = require('immutable');
const originalMap = Map({ a: 1, b: 2, c: 3 });
const updatedMap = originalMap.set('b', 2);
updatedMap === originalMap; // 无操作 .set() 返回原始引用。
```

然而，确实导致变化的更新将返回新引用。这些操作是独立的，所以两个相似的更新不会返回相同的引用：

```js
const { Map } = require('immutable');
const originalMap = Map({ a: 1, b: 2, c: 3 });
const updatedMap = originalMap.set('b', 1000);
// 新实例，保留原始不可变性。
updatedMap !== originalMap;
const anotherUpdatedMap = originalMap.set('b', 1000);
// 尽管两个都是相同操作的结果，但每个都创建了新引用。
anotherUpdatedMap !== updatedMap;
// 然而，这两个值是相等的。
anotherUpdatedMap.equals(updatedMap);
```

## 批量变更

> 如果一棵树在森林中倒下，它会发出声音吗？
> 
> 如果一个纯函数为了产生不可变返回值而变异一些局部数据，那可以吗？
> 
> — Rich Hickey, Clojure

应用变异以创建一个新的不可变对象会产生一些开销，这可能会累积成轻微的性能惩罚。如果您需要在返回之前局部应用一系列变异，Immutable.js 允许您通过使用 `withMutations` 创建集合的临时可变（瞬态）副本，并以高效的方式应用一批变异。实际上，这正是 Immutable.js 自身应用复杂变异的方式。

例如，构建 `list2` 的结果是创建了 1 个而不是 3 个新的不可变 Lists。

```js
const { List } = require('immutable');
const list1 = List([1, 2, 3]);
const list2 = list1.withMutations(function (list) {
  list.push(4).push(5).push(6);
});
assert.equal(list1.size, 3);
assert.equal(list2.size, 6);
```

注意：Immutable.js 还提供了 `asMutable` 和 `asImmutable`，但仅在 `withMutations` 不足以满足需求时鼓励使用它们。小心使用它们，不要返回一个可变副本，这可能会导致不期望的行为。

_重要！_：只有少数几种方法可以在 `withMutations` 中使用，包括 `set`、`push` 和 `pop`。这些方法可以直接应用于持久数据结构，而其他方法如 `map`、`filter`、`sort` 和 `splice` 总是返回新的不可变数据结构，并且永远不会变异一个可变集合。

## 懒惰序列

`Seq` 描述了一个延迟操作，允许它们通过不创建中间集合来有效链接使用所有高阶集合方法（例如 `map` 和 `filter`）。

**Seq 是不可变的** —— 一旦创建了 Seq，它就不能被改变、追加、重新排序或以其他方式修改。相反，Seq 上调用的任何变异方法都会返回一个新的 Seq。

**Seq 是懒惰的** —— Seq 尽可能少地做工作以响应任何方法调用。值通常是在迭代期间创建的，包括在减少或转换为具体数据结构（如 `List` 或 JavaScript `Array`）时的隐式迭代。

例如，以下操作不执行任何工作，因为结果 Seq 的值从未被迭代：

```js
const { Seq } = require('immutable');
const oddSquares = Seq([1, 2, 3, 4, 5, 6, 7, 8])
  .filter(x => x % 2 !== 0)
  .map(x => x * x);
```

一旦使用了 Seq，它只执行必要的工作。在这个例子中，从未创建过任何中间数组，filter 被调用了三次，map 只被调用了一次：

```js
oddSquares.get(1); // 9
```

任何集合都可以通过 `Seq()` 转换为懒惰 Seq。

```js
const { Map, Seq } = require('immutable');
const map = Map({ a: 1, b: 2, c: 3 });
const lazySeq = Seq(map);
```

`Seq` 允许有效链接操作，允许表达可能非常繁琐的逻辑：

```js
lazySeq
  .flip()
  .map(key => key.toUpperCase())
  .flip();
// Seq { A: 1, B: 2, C: 3 }
```

以及表达逻辑，否则似乎受内存或时间限制，例如 `Range` 是一种特殊的懒惰序列。

```js
const { Range } = require('immutable');
Range(1, Infinity)
  .skip(1000)
  .map(n => -n)
  .filter(n => n % 2 === 0)
  .take(2)
  .reduce((r, n) => r * n, 1);
// 1006008
```

## 过滤（filter）、分组（groupBy）和分区（partition）的比较

`filter()`、`groupBy()` 和 `partition()` 方法相似，因为它们都基于对每个元素应用函数来将集合划分为部分。这三种方法都对输入集合中的每个项目调用一次谓词或分组函数。这三种方法都返回零个或多个与其输入相同类型的集合。返回的集合始终与输入不同（根据 `===`），即使内容相同。

在这些方法中，`filter()` 是唯一懒惰的，也是唯一一个从输入集合中丢弃项目的方法。它是最简单易用的，并且它返回恰好一个集合的事实使它很容易与其他方法结合形成操作流水线。

`partition()` 方法类似于 `filter()` 的急切版本，但它返回两个集合；第一个包含将被 `filter()` 丢弃的项目，第二个包含将被保留的项目。它总是返回恰好两个集合的数组，这可能使它比 `groupBy()` 更容易使用。与分别进行两次 `filter()` 调用相比，`partition()` 使其谓词调用次数减半。

`groupBy()` 方法是 `partition()` 的更通用版本，可以按任意函数而不是仅仅是谓词进行分组。它返回一个具有零个或多个条目的映射，其中键是由分组函数返回的值，值是相应的非空集合。尽管 `groupBy()` 比 `partition()` 更强大，但由于无法事先预测返回的映射将有多少个条目以及它们的键将是什么，所以使用起来可能更困难。

| 摘要                       | `filter` | `partition` | `groupBy`      |
|:---------------------------|:---------|:-----------|:---------------|
| 易用性                     | 最简单   | 中等       | 最难          |
| 通用性                     | 最少     | 中等       | 最多          |
| 惰性                       | 懒惰     | 急切       | 急切          |
| 返回子集合的数量           | 1        | 2          | 0 或更多      |
| 子集合可能为空             | 是       | 是         | 否            |
| 可以丢弃项目               | 是       | 否         | 否            |
| 包装容器                   | 无       | 数组       | Map/OrderedMap |

## 附加工具和资源

- [Atom-store](https://github.com/jameshopkins/atom-store/)
  - 一个受 Clojure 启发的 Javascript 中的原子实现，具有可配置的外部持久性。

- [Chai Immutable](https://github.com/astorije/chai-immutable)
  - 如果您正在使用 [Chai Assertion Library](https://chaijs.com/)，这个库提供了一组针对 Immutable.js 集合的断言。

- [Fantasy-land](https://github.com/fantasyland/fantasy-land)
  - 这是JavaScript中通用代数结构互操作性的规范。

- [Immutagen](https://github.com/pelotom/immutagen)
  - 一个在 JavaScript 中模拟不可变生成器的库。

- [Immutable-cursor](https://github.com/redbadger/immutable-cursor)
  - 基于 Clojure 启发的原子的 Immutable 光标。

- [Immutable-ext](https://github.com/DrBoolean/immutable-ext)
  - 为 immutablejs 增加的 Fantasyland 扩展。

- [Immutable-js-tools](https://github.com/madeinfree/immutable-js-tools)
  - 用于 immutable.js 的实用工具。

- [Immutable-Redux](https://github.com/gajus/redux-immutable)
  - redux-immutable 用于创建与 Redux 结合使用 Immutable.js 状态的工作 combineReducers 等效函数。

- [Immutable-Treeutils](https://github.com/lukasbuenger/immutable-treeutils)
  - 用于 ImmutableJS 数据结构的函数式树遍历助手。

- [Irecord](https://github.com/ericelliott/irecord)
  - 一个不可变存储，公开了 RxJS 可观察对象。非常适合 React。

- [Mudash](https://github.com/brianneisler/mudash)
  - 提供 Immutable.JS 支持的 Lodash 包装器。

- [React-Immutable-PropTypes](https://github.com/HurricaneJames/react-immutable-proptypes)
  - 与 Immutable.js 配合使用的 PropType 验证器。

- [Redux-Immutablejs](https://github.com/indexiatech/redux-immutablejs)
  - Redux Immutable 工具。

- [Rxstate](https://github.com/yamalight/rxstate)
  - 基于 RxJS 和 Immutable.js 的简单、有意见的状态管理库。

- [Transit-Immutable-js](https://github.com/glenjamin/transit-immutable-js)
  - Immutable.js 的 Transit 序列化。
  - 另见：[Transit-js](https://github.com/cognitect/transit-js)

如果您有其他为 Immutable.js 设计的工具？
提交一个 PR 将它们按字母顺序添加到此列表中。

## 贡献

使用 [Github 问题](https://github.com/immutable-js/immutable-js/issues) 进行请求。

我们积极欢迎拉取请求，了解如何 [贡献](https://github.com/immutable-js/immutable-js/blob/main/.github/CONTRIBUTING.md)。

Immutable.js 在 [Contributor Covenant 的行为准则](https://www.contributor-covenant.org/version/2/0/code_of_conduct/) 下进行维护。

### 更新日志

更改以 [Github 版本](https://github.com/immutable-js/immutable-js/releases) 跟踪。

### 许可证

Immutable.js 是 [MIT 许可](./LICENSE)。

### 致谢

[Phil Bagwell](https://www.youtube.com/watch?v=K2NYwP90bNs)，感谢他在持久数据结构方面的启发和研究。

[Hugh Jackson](https://github.com/hughfdjackson/)，感谢他提供 npm 包名。如果您正在寻找他不支持的包，请看 [这个仓库](https://github.com/hughfdjackson/immutable)。