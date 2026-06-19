迭代器块表达式提案
========

## 提案内容

可以将 `block {}` 定义为类似 `for..of` 的语法糖，但与 `for..of` 有以下区别：

- `block {}` 会将最后一条语句的结果传递给 `iterator.next`
- `block {}` 是表达式，而不是语句，所以允许嵌套在表达式中
- `block {|prm| }` 的参数是可变的，即便是被 `using` 或 `await using` 的变量
- `block {}` 允许后跟 `else {}` 形成 `block {} else {}` 结构
- `block {}` 新引入 `return break` 语法与 `iteratorReturnResult.break` 可选属性
- `block {} else {}` 是表达式

`block {}` 与 `for..of` 的共同点：

- 都是利用 `Symbol.iterator` 和 `Symbol.asyncIterator`
- 都支持在内部使用 `yield`、 `yield*`、`return`、 `throw`、`continue`、`break`、`continue label`、`break label`、 `await` 等控制语句

### `block {}` 近似 `for..of` 语法对比

| `block {}`                                                | `for..of`                                                                 |
| --------------------------------------------------------- | ------------------------------------------------------------------------- |
| `iterable { doSameThing(); }`                             | `for(let _ of iterable){ doSameThing(); }`                                |
| `iterable await { doSameThing(); }`                       | `for await(let _ of iterable){ doSameThing(); }`                          |
| `iterable using { doSameThing(); }`                       | `for (using _ of iterable){ doSameThing(); }`                             |
| `iterable await using { doSameThing(); }`                 | `for (await using _ of iterable){ doSameThing(); }`                       |
| `iterable using await { doSameThing(); }`                 | `for await (using _ of iterable){ doSameThing(); }`                       |
| `iterable await using await { doSameThing(); }`           | `for await (await using _ of iterable){ doSameThing(); }`                 |
| `generator() { doSameThing(); }`                          | `for(let _ of generator()){ doSameThing(); }`                             |
| `generator() await { doSameThing(); }`                    | `for await(let _ of generator()){ doSameThing(); }`                       |
| `generator() using { doSameThing(); }`                    | `for (using _ of generator()){ doSameThing(); }`                          |
| `generator() await using { doSameThing(); }`              | `for (await using _ of generator()){ doSameThing(); }`                    |
| `generator() using await { doSameThing(); }`              | `for await (using _ of generator()){ doSameThing(); }`                    |
| `generator() await using await { doSameThing(); }`        | `for await (await using _ of generator()){ doSameThing(); }`              |
| `[...array] { doSameThing(); }`                           | `for(let _ of [...array]){ doSameThing(); }`                              |
| `[...array] using { doSameThing(); }`                     | `for (using _ of [...array]){ doSameThing(); }`                           |
| `[...array] await using { doSameThing(); }`               | `for (await using _ of [...array]){ doSameThing(); }`                     |
| `iterable {\|prm\| doSameThing(); }`                      | `for(let prm of iterable){ doSameThing(); }`                              |
| `iterable await {\|prm\| doSameThing(); }`                | `for await(let prm of iterable){ doSameThing(); }`                        |
| `iterable using {\|prm\| doSameThing(); }`                | `for (using _ of iterable){ let prm = _; doSameThing(); }`                |
| `iterable await using {\|prm\| doSameThing(); }`          | `for (await using _ of iterable){ let prm = _; doSameThing(); }`          |
| `iterable using await {\|prm\| doSameThing(); }`          | `for await (using _ of iterable){ let prm = _; doSameThing(); }`          |
| `iterable await using await {\|prm\| doSameThing(); }`    | `for await (await using _ of iterable){ let prm = _; doSameThing(); }`    |
| `generator() {\|prm\| doSameThing(); }`                   | `for(let prm of generator()){ doSameThing(); }`                           |
| `generator() await {\|prm\| doSameThing(); }`             | `for await(let prm of generator()){ doSameThing(); }`                     |
| `generator() using {\|prm\| doSameThing(); }`             | `for (using _ of generator()){ let prm = _; doSameThing(); }`             |
| `generator() await using {\|prm\| doSameThing(); }`       | `for (await using _ of generator()){ let prm = _; doSameThing(); }`       |
| `generator() using await {\|prm\| doSameThing(); }`       | `for await (using _ of generator()){ let prm = _; doSameThing(); }`       |
| `generator() await using await {\|prm\| doSameThing(); }` | `for await (await using _ of generator()){ let prm = _; doSameThing(); }` |
| `[...array] {\|prm\| doSameThing(); }`                    | `for(let prm of [...array]){ doSameThing(); }`                            |
| `[...array] using {\|prm\| doSameThing(); }`              | `for (using _ of [...array]){ let prm = _; doSameThing(); }`              |
| `[...array] await using {\|prm\| doSameThing(); }`        | `for (await using _ of [...array]){ let prm = _; doSameThing(); }`        |


### 一条语句中有多个 `block {}`时，对于 label 处理

对于以下用法：

```javascript
label: let result = [iterable1 { continue label; }, iterable2 { continue label; }]
```

其等价于

```javascript
label1: const __result1 = iterable1 { continue label1; }
label2: const __result2 = iterable2 { continue label2; }
let result = [__result1, __result2];
```


### 如何界定是否为 `block {}`

- 在 `()` `[]` 等内被是为表达式的而非语句的情况，一律视作 `block {}`
- 否则 `block` 与 `await using`、 `using`、`await`、`{` 中至少一者在同一行，才会被视作 `block {}`

可以被视作 `block {}` 的例子：

```javascript
block {}

block await {}

block using {}

block await using {}

block await using await {}


block await
{}

block using
{}

block await using
{}

block await using await
{}

block await using
await {}

/**
 * 上面集中情况，因换行所在位置，无法视为语句结束，所以均被视为 `block {}` 用法
 * 以下几种都只能作为表达式，而不能拆分成两条语句，所以也均被视为 `block {}` 用法
 */

(block
{});

[block
await {}];

let obj = {val: block
using {}};

func(block
await using {});

for (let x of block
await using await {}) {

};


```

不能被视作 `block {}` 的例子：

```javascript
block
{}

block
await {}

block
using {}

block
await using {}

block
await using await {}

```
这些例子中，正在换行处，换行前的部分，已经可以视作完整语句，所以在解析上，无法视作 `block {}` 用法

### 多处 await 的界定

```javascript
let result = await (await fn()) await using await {}
```

- 第一个 `await` 因为 `await` 优先级低于 `block {}` 所以被视作对 `block{}` 的返回值的 `await`，最后执行
- 第二个 `await` 因为被括号包裹，所以属于 `block` 部分，是最先执行的 `await`
- 第三个 `await` 因为其后紧跟 `using`，所以应与 `using` 视作一体 `await using`，属于 `for (await using _ of block)` 中的 `await`
- 第四个 `await` 后面，因为其后紧跟着 `{`，其属于 `for await (let _ of block)` 中的 `await`

### 返回值的处理

- 如果被 `return`、`throw`、`block label`(label 来自更外层)、`continue label`(label 来自更外层) 等中断执行，因其不再使用返回值，所以无需考虑返回值
- 默认情况下，返回值为 `iterator.next().done === true` 时的 `iterator.next().value`，或生成器的返回值
- 如果是被 `block`中断，则返回值应为 `undefined` 或 `else {}` 中的最后一条实际执行语句的返回值
- `continue` 只是结束当前循环，循环未结束，所以也无需讨论返回值

#### 默认情况下 `block{}` 返回值的考量

现阶段，如果执行生成器，只取生成器最终的返回值，没有简单的语法，需要通过以下多条语句实现：

```javascript
let result = generator();
while(!result.done){
  result = result.next();
}
result = result.value;
```

但将 `block{}` 返回值定为 `iterator.next().done === true` 时的 `iterator.next().value`，或生成器的返回值，则可以简化为：

```javascript
let result = generator() {};
```

#### `break` 时，返回值为 `undefined` 的考量

- 虽然当前阶段，社区没有考虑 `else` 与 `if` 之外的语句结合的情况，但实现 `block{}else{}` 后，让 `break` 走`else{}`，且将 `block` 的返回值定义为`else{}` 最后一条语句的值（没有 `else`情况或`else{}`内为空语句，则返回值为 `undefined`）。
- 若社区未来实现 `break` 后支持携带表达式，则可以将 `block{}` 的返回值定义为 `break` 后携带的表达式的值。
- 若同时实现以上两种，可考虑在 `break` 后带有表达式时，不走 `else{}`

###  `return break` 语法与 `iteratorReturnResult.break` 可选属性

`return break` 用于实现将 `iteratorReturnResult.break` 设为 `true`

即便 `block{}` 中没有 `break`，但倘若是 `return break` 或 `iteratorReturnResult.break == true`则依然按照 `break` 的逻辑执行 `else {}` 并将其结果作为实际返回值（如果没有 `else {}` 则返回 `undefined`）

### 为何不将 `block {}` 作为 `block(() => {})` 的语法糖

- 将无法使用 `break`、 `break label`、`continue label`等流程控制语句
- `return`将相当于循环中的 `continue` 只是带有返回值，这意味着 `continue` 与 `return` 高度重叠
- 如强行将`break`、 `break label`、`continue label`、`return` 作为标准流程控制，这意味着，实现上，还需要实现不被 `catch` 捕获的特殊错误。而且，如果是在同步环境下调用异步`block {}`，流程控制依然不可用。


### 应用举例

#### 声明式 UI 与声明式配置

用于声明式使用的函数

```javascript
/** @type {{children: any[]; tag: string}[]} */
const stack = [];

/**
 * 
 * @param {string} tag 
 * @returns 
 */
function element(tag) {
  const parent = stack[stack.length - 1];
  if (!parent) {
    throw new Error(`[Render Error]: 节点 <${tag}> 不能在 UI 树外部创建！`);
  }
  const node = {
    type: 'element',
    tag,
    props: {},
    children: [],
    /** @param {string} value */
    id(value) {
      this.props.id = value;
      return this;
    },

    *[Symbol.iterator]() {
      stack.push(this);
      yield this;
      stack.pop();
      return this;
    }
  };
  parent.children.push(node);

  return node;
}

/**
 * 
 * @param {string} content 
 * @returns 
 */
function text(content) {
  const parent = stack[stack.length - 1];
  if (!parent) {
    throw new Error(`[Render Error]: 文本节点 "${content}" 不能在 UI 树外部创建！`);
  }
  const textNode = {
    type: 'text',
    content, 
    *[Symbol.iterator]() { return this; }, };
  parent.children.push(textNode);

  return textNode;
}
function root() {
  return {
    type: 'root',
    tag: 'Root',
    children: [],

    *[Symbol.iterator]() {
      stack.push(this);
      yield this;
      stack.pop();
      return this;
    },
  };
}
```

使用实例

```javascript

const ui = root() {
  element('div').id('123') {
    element('span').id('456');
    element('ul').id('789') {
      element('li').id('item1') {
        text('Hello World');
      }
    }
  }
}

console.log(JSON.stringify(ui, null, 2));
```

近似降级为以下形式：

```javascript
const ui = root();
for (const _ of ui) {
  for (const _ of element('div').id('123')) {
    element('span').id('456');
    for (const _ of element('ul').id('789')) {
      for (const _ of element('li').id('item1')) {
        text('Hello World');
      }
    }
  }
}

console.log(JSON.stringify(ui, null, 2));
```

#### 响应式 UI

对于响应式 UI，如果只是属性的响应式，可以直接将 ref 传入属性。

如果是是 if / for 等的响应式，则可以引入 reactive 函数，如下：

```javascript

function reactive(fn) {
  const parent = stack[stack.length - 1];
  if (!parent) {
    throw new Error(`[Render Error]: reactive 不能在 UI 树外部运行！`);
  }
  const reactive =() => (root() {fn()}).children;
  reactive.type = 'reactive';
  reactive[Symbol.iterator] = function*() {return this;};
  parent.children.push(reactive);

  return textNode;

}

```


使用实例

```javascript

const id = ref(123);

const ui = root() {
  element('div').id(id) {
    element('span').id('456');
    reactive(() => {
      element('ul').id('789') {
        element('li').id('item1') {
          text('Hello World');
        }
      }
    });
  }
}

```


### `unless() {} else {}`

定义：

```javascript
function* unless(condition) {
  if (!condition) {
    return yield;
  } else {
    return break;
  }
}
```

用例：

```javascript
unless(a > b) {
  
} else {
  
}
```

#### `where() { when() {}}` `where() { is() {}}` `select() { when() {}}` `select() { is() {}}` `select() { other() {}}`

定义：

```javascript
const stack = [];

function is(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: is 必须在 where/select 内部`);
  }
  if (Object.is(item[0], condition)) { return item[1] = yield item[0]; }
}

function when(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: when 必须在 where/select 内部`);
  }
  if (condition(item[0])) { return item[1] = yield item[0]; }
}
function other(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: when 必须在 where/select 内部`);
  }
  if (item.length === 1) { return item[1] = yield item[0]; }
}
function *select(condition) {
  const item = [condition];
  stack.push(item);
  try {
    yield condition;
    if (item.length === 1) { return break; }
    return item[1];
  } finally {
    stack.pop();
  }
}
```

用例：

```javascript
const result = select(n) {
  is(1) {
    
  }
  when(k => 2< k && k < 10) {
    
  }
  when(k => 20< k && k < 50) {
    
  }
  other {
    // other 与 else 使用上有一定却别，other 之后的 is 和 when 依然执行，而且若不是社区不接受 `block {} else {}` 则也可以用 other 代替
  }
} else {

}
```


### 降级实例

```javascript
label: const result = generator() await using await {|prm|
  if (if1()) {
    val1()
  } else if (if2()) {
    break;
  } else if (if3()) {
    val3();
    continue;
  } else if (if4()) {
    continue label;
  } else if (if5()) {
    continue otherLabel;
  } else if (if6()) {
    break otherLabel;
  } else if (if7()) {
    return val7();
  } else if (if8()) {
    break val8();
  }
} else {
  val9()
}
```

降级为：

```javascript
let __result;
{
  let _toRunElse = false;
  let _nextInput = undefined;
  const _iterator = generator()[Symbol.asyncIterator]();

  label: while (true) {
    const { value: prm, done, break: returnBreak } = await _iterator.next(_nextInput);
    if (done) {
      if (returnBreak) {
        _toRunElse = true;
        break;
      }
      __result = prm;
      break;
    }
    _nextInput = undefined;
    let _lastResult = undefined;
    try {
      await using _ = prm;

      if (if1()) {
        _lastResult = val1();
      } else if (if2()) {
        _toRunElse = true;
        break;
      } else if (if3()) {
        _lastResult = val3();
        _nextInput = _lastResult;
        continue;
      } else if (if4()) {
        _lastResult = if4();
        _nextInput = _lastResult;
        continue label;

      } else if (if5()) {
        _lastResult = if5();
        _nextInput = _lastResult;
        continue otherLabel;

      } else if (if6()) {
        _toRunElse = true;
        break otherLabel;

      } else if (if7()) {
        return val7();
      } else if (if8()) {
        __result = val8();
        _toRunElse = false;
        return val7();
      }
      _nextInput = _lastResult;

    } catch (err) {
      if (_iterator.throw) {
        await _iterator.throw(err);
      } else {
        throw err;
      }

    } finally {
      if (_iterator.return) {
        await _iterator.return();
      }
    }
  }

  if (_toRunElse) {
    __result = undefined;
    {
      __result = val9();
    }
  }
}
const result = __result;
```
