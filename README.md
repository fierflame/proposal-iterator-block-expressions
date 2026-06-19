Proposal For Iterator Block Expressions
========

## Status

> **Stage 0 (Strawperson)**  
> This proposal is currently at Stage 0. It is an early-stage idea intended to spark discussion within the TC39 committee regarding a specific pain point in JavaScript iteration. The syntax and semantics are highly subject to change and may be rejected. Community feedback and prior art comparisons are highly encouraged.

## Proposal Details

The `block {}` can be defined as a syntactic sugar similar to `for..of`, but it differs from `for..of` in the following ways:
- `block {}` will pass the result of the last statement to `iterator.next`. 
- `block {}` is an expression rather than a statement, so it’s allowed to be nested within expressions.
- The arguments of `block {|prm| }` are variable, even those of variables wrapped in `using` or `await using`.
- `block {}` now allows an optional trailing `else {}` to form a `block {} else {}` construct.
- `block {}` introduces a new `return break` syntax, along with an optional `iteratorReturnResult.break` property.
- `block {} else {}` is an expression.

The commonalities between `block {}` and `for..of`:

- Both utilize `Symbol.iterator` and `Symbol.asyncIterator`
- All support the use of control statements such as `yield`, `yield*`, `return`, `throw`, `continue`, `break`, `continue label`, `break label`, and `await` internally. 

### Comparison of `block {}` vs. `for...of` Syntax

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


### Handling labels when a single statement contains multiple `block {}`s

For the following usage:
```javascript
label: let result = [iterable1 { continue label; }, iterable2 { continue label; }]
```

Its equivalent to

```javascript
label1: const __result1 = iterable1 { continue label1; }
label2: const __result2 = iterable2 { continue label2; }
let result = [__result1, __result2];
```


### How to Determine Whether It Is a `block {}`

- In cases where `()` and `[]` are used within expressions rather than as statements, they are always treated as a block `{}`.
- Otherwise, the `block` will only be considered a `block {}` if it appears on the same line as at least one of the following: `await using`, `using`, `await`, or `{`.

Can be regarded as an example of `block {}`:

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
* Due to the placement of line breaks, the above-mentioned cases cannot be considered as sentence endings and are therefore all treated as `block {}` usage.
* The following types can only be used as expressions and cannot be split into two statements; therefore, they are all treated as `block {}` usage.
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

Examples that cannot be regarded as `block {}`:

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
In these examples, the portion preceding the line break at the point of line breaking can already be considered a complete statement; therefore, from a parsing perspective, it cannot be treated as a `block {}` usage.

### Definition of Multiple `await` Statements

```javascript
let result = await (await fn()) await using await {}
```

- The first `await` is treated as an `await` on the return value of the `block {}`, because `await` has lower priority than `block {}`, and it’s executed last. 
- The second `await`, being enclosed in parentheses, belongs to the `block` section and is the first `await` to be executed. 
- The third `await`, since it is immediately followed by `using`, should be treated as a single unit with `using`—that is, as an `await using`. It falls under the `await` in `for (await using _ of block)`.
- After the fourth `await`, since it is immediately followed by `{`, it belongs to the `await` within `for await (let _ of block)`.

### Handling Return Values

- If execution is interrupted by `return`, `throw`, a block label (the label comes from an outer scope), or a continue label (the label comes from an outer scope), there’s no need to consider the return value since it will no longer be used.
- By default, the return value is the `iterator.next().value` when `iterator.next().done === true`, or the generator’s return value.
- If interrupted by a `block`, the return value should be either `undefined` or the return value of the last actually executed statement within the `else {}` block.
- `continue` merely ends the current iteration; since the loop hasn't ended yet, there's no need to discuss its return value.

#### Considerations for the Return Value of `block{}` by Default

At this stage, if you execute the generator and only want to obtain its final return value, there is no simple syntax available. You’ll need to implement it using multiple statements as follows:

```javascript
let result = generator();
while(!result.done){
  result = result.next();
}
result = result.value;
```

However, if you define the return value of `block{}` as `iterator.next().value` when `iterator.next().done === true`, or as the generator’s return value, it can be simplified to:

```javascript
let result = generator() {};
```

#### Considerations for Returning `undefined` When Using `break`

- Although the community is not currently considering combining `else` with statements other than `if`, the implementation of `block {} else {}` would allow `break` to fall through to the `else {}` block. In this case, the return value of the `block` would be defined as the value of the last statement executed within the `else {}` block (returning `undefined` if there is no `else` block or if the `else {}` block contains no statements).
- If the community futurely supports carrying expressions after a `break`, the return value of `block{}` could be defined as the value of the expression carried after the `break`.
- If both of the above are implemented simultaneously, consider omitting the `else{}` block when a statement follows the `break`.

### `return break` Syntax and the Optional `iteratorReturnResult.break` Property

The `return break` syntax is used to set `iteratorReturnResult.break` to `true`.

Even if there is no `break` statement within the `block {}`, if `return break` is encountered or `iteratorReturnResult.break == true`, the `else {}` block will still be executed following the `break` logic. The result of this execution becomes the actual return value (if there is no `else {}` block, `undefined` is returned).

### Why not treat `block {}` as syntactic sugar for `block(() => {})`?

- The flow control statements such as `break`, `break label`, and `continue label` will be unavailable.
- `return` is equivalent to `continue` within a loop, except that it includes a return value. This means that `continue` and `return` overlap significantly. 
- If `break`, `break label`, `continue label`, and `return` are forcibly treated as standard flow-control constructs, this implies that, in implementation, special errors that are not caught by `catch` must also be handled. Moreover, if an asynchronous `block {}` is invoked in a synchronous context, flow control still remains unavailable.


### Application Examples

#### Declarative UI and Declarative Configuration

Functions for declarative use

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
    throw new Error(`[Render Error]: The node <${tag}> cannot be created outside the UI tree!`); 
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
    throw new Error(`[Render Error]: Text node "${content}" cannot be created outside the UI tree!`);
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

Usage Example

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

Approximate fallback to the following form:

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

#### Reactive UI

For reactive UI, if only the attributes are reactive, you can pass a `ref` directly into the attribute.

If you need reactivity for control flows like `if` or `for`, you can use the `reactive` function as shown below:

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


Usage Example

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


### `unless() {}`

Definition:

```javascript
function* unless(condition) {
  if (!condition) {
    return yield;
  } else {
    return break;
  }
}
```

Usage:

```javascript
unless(a > b) {
  
} else {
  
}
```

#### `where() { when() {}}` `where() { is() {}}` `select() { when() {}}` `select() { is() {}}` `select() { other() {}}`

Definition:

```javascript
const stack = [];

function is(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: 'is' must be used inside 'where' or 'select'`);
  }
  if (Object.is(item[0], condition)) { return item[1] = yield item[0]; }
}

function when(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: 'when' must be used inside 'where' or 'select'`);
  }
  if (condition(item[0])) { return item[1] = yield item[0]; }
}
function other(condition) {
  const item = stack[stack.length - 1];
  if (!item) {
    throw new Error(`[Error]: 'when' must be used inside 'where' or 'select'`);
  }
  if (item.length === 1) { return item[1] = yield item[0]; }
}
function *where(condition) {
  const item = [condition];
  stack.push(item);
  try {
    const result = yield condition;
    if (item.length === 1) { return break; }
    return result;
  } finally {
    stack.pop();
  }
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

Usage:

```javascript
where(n) {
  is(1) {
    
  }
  when(k => 2< k && k < 10) {
    
  }
  when(k => 20< k && k < 50) {
    
  }
  other {
    // There are some differences in usage between `other` and `else`. The `is` and `when` clauses following `other` will not be executed. Furthermore, if the community were to accept it, `other` could also be used as a substitute for `block {} else {}`.
  }
} else {

}
const result = select(n) {
  is(1) {
    
  }
  when(k => 2< k && k < 10) {
    
  }
  when(k => 20< k && k < 50) {
    
  }
  other {
    
  }
} else {

}
```


### Fallback Example

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

Fallback to:

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
