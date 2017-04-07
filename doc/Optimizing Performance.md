# Optimizing Performance

在更新UI时，React内部使用了多种技术最小化了必要的DOM操作。对于大多数应用，使用React可以达到一个更快的用户交互并且不需要额外的特定性能优化。然而，下面有几种方式能够加快你的React应用。

## 使用生产环境

如果你在React应用中遇到性能的瓶颈，请确保你是在生产环境下测试：

* 对于单文件应用，我们提供了`.min.js`版本
* 对 Brunch，你需要在`build`命令前加`-p`参数
* 对于 Browserify，你需要以`NODE_ENV=production`方式运行
* 对于 Create React App，你需要按照相关说明并运行`npm run build`命令。
* 对于 Rollup，你需要在[commonjs](https://github.com/rollup/rollup-plugin-commonjs)插件前使用[replace](https://github.com/rollup/rollup-plugin-replace)插件，使得开发先关的模块不会被引入。更详细的例子请[阅读](https://gist.github.com/Rich-Harris/cb14f4bc0670c47d00d191565be36bf0)。

```js
plugins: [
  require('rollup-plugin-replace')({
    'process.env.NODE_ENV': JSON.stringify('production')
  }),
  require('rollup-plugin-commonjs')(),
  // ...
]
```


* 对于 Webpack, 在你的生产环节配置中添加下面的插件:

```js
new webpack.DefinePlugin({
  'process.env': {
    NODE_ENV: JSON.stringify('production')
  }
}),
new webpack.optimize.UglifyJsPlugin()
```

开发版本中包含额外的警告，这对开发应用很有帮助，但是由于额外的日志记录会相应地降低性能。

## 使用Chrome Timeline分析组件性能

在开发模式中，你可以在支持相关功能的浏览器中使用性能工具来可视化组件mount，update和unmount的各个过程。

<center><img src="/react/img/blog/react-perf-chrome-timeline.png" width="651" height="228" alt="React components in Chrome timeline" /></center>

在Chrome操作如下:

1. 通过添加`?react_perf`查询字段加载你的应用(例如：`http://localhost:3000/?react_perf`)

2. 打开Chrome DevTools **[Timeline](https://developers.google.com/web/tools/chrome-devtools/evaluate-performance/timeline-tool)** 并点击**Record**。

3. 执行你想要分析的操作，不要超过20秒，否则Chrome可能会挂起。

4. 停止记录。

5. 在**User Timing**下，React事件将会分组列出。

注意，上述数据是相对的，组件会在生产环境中有更好的性能。然而，这对你分析由于错误导致不相关的组件的更新、分析组件更新的深度和频率很有帮助。

目前Chrome，Edge和IE支持该特性，但是我们使用了标准的 [User Timing API](https://developer.mozilla.org/en-US/docs/Web/API/User_Timing_API),因此我们期待将来会有更多的浏览器支持。

## 避免Reconciliation

React创建和维护了渲染UI的内部状态。其包括了组件返回的React元素。这些内部状态使得React只有在必要的情况下才会创建DOM节点和访问存在DOM节点，因为对JavaScript对象的操作是比DOM操作更快。这被称为"虚拟DOM"，React Native的基于上述原理。

当组件的`props`和`state`更新时,React通过比较新返回的元素和之前渲染的元素来决定是否有必要更新DOM元素。如果二者不相等，则更新DOM元素。

在部分场景下，组件可以通过重写生命周期函数`shouldComponentUpdate`来优化性能。`shouldComponentUpdate`函数会在重新渲染流程前触发。`shouldComponentUpdate`的默认实现中返回的是`true`，使得React执行更新操作。

```javascript
shouldComponentUpdate(nextProps, nextState) {
  return true;
}
```

如果你的组件在部分场景下不需要更行，你可以在`shouldComponentUpdate`返回`false`来跳过整个渲染流程(包括调用`render`和之后流程)。

## shouldComponentUpdate

下面有一个组件子树

Here's a subtree of components. For each one, `SCU` indicates what `shouldComponentUpdate` returned, and `vDOMEq` indicates whether the rendered React elements were equivalent. Finally, the circle's color indicates whether the component had to be reconciled or not.

<figure><img src="/react/img/docs/should-component-update.png" /></figure>

Since `shouldComponentUpdate` returned `false` for the subtree rooted at C2, React did not attempt to render C2, and thus didn't even have to invoke `shouldComponentUpdate` on C4 and C5.

For C1 and C3, `shouldComponentUpdate` returned `true`, so React had to go down to the leaves and check them. For C6 `shouldComponentUpdate` returned `true`, and since the rendered elements weren't equivalent React had to update the DOM.

The last interesting case is C8. React had to render this component, but since the React elements it returned were equal to the previously rendered ones, it didn't have to update the DOM.

Note that React only had to do DOM mutations for C6, which was inevitable. For C8, it bailed out by comparing the rendered React elements, and for C2's subtree and C7, it didn't even have to compare the elements as we bailed out on `shouldComponentUpdate`, and `render` was not called.

## Examples

If the only way your component ever changes is when the `props.color` or the `state.count` variable changes, you could have `shouldComponentUpdate` check that:

```javascript
class CounterButton extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  shouldComponentUpdate(nextProps, nextState) {
    if (this.props.color !== nextProps.color) {
      return true;
    }
    if (this.state.count !== nextState.count) {
      return true;
    }
    return false;
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

In this code, `shouldComponentUpdate` is just checking if there is any change in `props.color` or `state.count`. If those values don't change, the component doesn't update. If your component got more complex, you could use a similar pattern of doing a "shallow comparison" between all the fields of `props` and `state` to determine if the component should update. This pattern is common enough that React provides a helper to use this logic - just inherit from `React.PureComponent`. So this code is a simpler way to achieve the same thing:

```js
class CounterButton extends React.PureComponent {
  constructor(props) {
    super(props);
    this.state = {count: 1};
  }

  render() {
    return (
      <button
        color={this.props.color}
        onClick={() => this.setState(state => ({count: state.count + 1}))}>
        Count: {this.state.count}
      </button>
    );
  }
}
```

Most of the time, you can use `React.PureComponent` instead of writing your own `shouldComponentUpdate`. It only does a shallow comparison, so you can't use it if the props or state may have been mutated in a way that a shallow comparison would miss.

This can be a problem with more complex data structures. For example, let's say you want a `ListOfWords` component to render a comma-separated list of words, with a parent `WordAdder` component that lets you click a button to add a word to the list. This code does *not* work correctly:

```javascript
class ListOfWords extends React.PureComponent {
  render() {
    return <div>{this.props.words.join(',')}</div>;
  }
}

class WordAdder extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      words: ['marklar']
    };
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    // This section is bad style and causes a bug
    const words = this.state.words;
    words.push('marklar');
    this.setState({words: words});
  }

  render() {
    return (
      <div>
        <button onClick={this.handleClick} />
        <ListOfWords words={this.state.words} />
      </div>
    );
  }
}
```

The problem is that `PureComponent` will do a simple comparison between the old and new values of `this.props.words`. Since this code mutates the `words` array in the `handleClick` method of `WordAdder`, the old and new values of `this.props.words` will compare as equal, even though the actual words in the array have changed. The `ListOfWords` will thus not update even though it has new words that should be rendered.

## The Power Of Not Mutating Data

The simplest way to avoid this problem is to avoid mutating values that you are using as props or state. For example, the `handleClick` method above could be rewritten using `concat` as:

```javascript
handleClick() {
  this.setState(prevState => ({
    words: prevState.words.concat(['marklar'])
  }));
}
```

ES6 supports a [spread syntax](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Operators/Spread_operator) for arrays which can make this easier. If you're using Create React App, this syntax is available by default.

```js
handleClick() {
  this.setState(prevState => ({
    words: [...prevState.words, 'marklar'],
  }));
};
```

You can also rewrite code that mutates objects to avoid mutation, in a similar way. For example, let's say we have an object named `colormap` and we want to write a function that changes `colormap.right` to be `'blue'`. We could write:

```js
function updateColorMap(colormap) {
  colormap.right = 'blue';
}
```

To write this without mutating the original object, we can use [Object.assign](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Object/assign) method:

```js
function updateColorMap(colormap) {
  return Object.assign({}, colormap, {right: 'blue'});
}
```

`updateColorMap` now returns a new object, rather than mutating the old one. `Object.assign` is in ES6 and requires a polyfill.

There is a JavaScript proposal to add [object spread properties](https://github.com/sebmarkbage/ecmascript-rest-spread) to make it easier to update objects without mutation as well:

```js
function updateColorMap(colormap) {
  return {...colormap, right: 'blue'};
}
```

If you're using Create React App, both `Object.assign` and the object spread syntax are available by default.

## Using Immutable Data Structures

[Immutable.js](https://github.com/facebook/immutable-js) is another way to solve this problem. It provides immutable, persistent collections that work via structural sharing:

* *Immutable*: once created, a collection cannot be altered at another point in time.
* *Persistent*: new collections can be created from a previous collection and a mutation such as set. The original collection is still valid after the new collection is created.
* *Structural Sharing*: new collections are created using as much of the same structure as the original collection as possible, reducing copying to a minimum to improve performance.

Immutability makes tracking changes cheap. A change will always result in a new object so we only need to check if the reference to the object has changed. For example, in this regular JavaScript code:

```javascript
const x = { foo: 'bar' };
const y = x;
y.foo = 'baz';
x === y; // true
```

Although `y` was edited, since it's a reference to the same object as `x`, this comparison returns `true`. You can write similar code with immutable.js:

```javascript
const SomeRecord = Immutable.Record({ foo: null });
const x = new SomeRecord({ foo: 'bar' });
const y = x.set('foo', 'baz');
x === y; // false
```

In this case, since a new reference is returned when mutating `x`, we can safely assume that `x` has changed.

Two other libraries that can help use immutable data are [seamless-immutable](https://github.com/rtfeldman/seamless-immutable) and [immutability-helper](https://github.com/kolodny/immutability-helper).

Immutable data structures provide you with a cheap way to track changes on objects, which is all we need to implement `shouldComponentUpdate`. This can often provide you with a nice performance boost.
