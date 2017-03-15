# React Without ES6

通常情况下你可以用纯JavaScript类定义一个组件:

```javascript
class Greeting extends React.Component {
  render() {
    return <h1>Hello, {this.props.name}</h1>;
  }
}
```

如果你不使用ES6，你可以使用`React.createClass`来定义:

```javascript
var Greeting = React.createClass({
  render: function() {
    return <h1>Hello, {this.props.name}</h1>;
  }
});
```
除了部分特性，ES6中类的API非常类似于函数`React.createClass`

## 声明属性类型和默认属性

在函数和class类中,`propTypes`和`defaultProps`被定义为组件内部的属性:

```javascript
class Greeting extends React.Component {
  // ...
}

Greeting.propTypes = {
  name: React.PropTypes.string
};

Greeting.defaultProps = {
  name: 'Mary'
};
```

在`React.createClass()`函数中,你需要在所传递的对象中定义`propTypes`属性和`getDefaultProps()`方法:

```javascript
var Greeting = React.createClass({
  propTypes: {
    name: React.PropTypes.string
  },

  getDefaultProps: function() {
    return {
      name: 'Mary'
    };
  },

  // ...

});
```

## 设置初始化状态

在ES6类中，你可以在构造函数通过给`this.state`辅助定义初始状态:

```javascript
class Counter extends React.Component {
  constructor(props) {
    super(props);
    this.state = {count: props.initialCount};
  }
  // ...
}
```

在`React.createClass()`函数中，你可以使用`getInitialState`方法返回初始状态:

```javascript
var Counter = React.createClass({
  getInitialState: function() {
    return {count: this.props.initialCount};
  },
  // ...
});
```

## 自动绑定

在以ES6 class方式声明的React组件中，方法遵循与普通ES6的class中相同的语义。也就是说方法不会自动绑定到实例中，你必须在构造函数中显式的使用`.bind(this)`:

```javascript
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
    // This line is important!
    this.handleClick = this.handleClick.bind(this);
  }

  handleClick() {
    alert(this.state.message);
  }

  render() {
    // Because `this.handleClick` is bound, we can use it as an event handler.
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```

在`React.createClass()`方式中，并不需要这么做，因为方法可以自动绑定。

```javascript
var SayHello = React.createClass({
  getInitialState: function() {
    return {message: 'Hello!'};
  },

  handleClick: function() {
    alert(this.state.message);
  },

  render: function() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
});
```

这意味着在使用ES6 class方式下对于事件处理函数你需要编写更多的样本代码,但是在大型应用中具有更好的性能。

如果你不想使用样本代码，你可以使用Babel中**实验性***的[提案](https://babeljs.io/docs/plugins/transform-class-properties/)

```javascript
class SayHello extends React.Component {
  constructor(props) {
    super(props);
    this.state = {message: 'Hello!'};
  }
  // WARNING: this syntax is experimental!
  // Using an arrow here binds the method:
  handleClick = () => {
    alert(this.state.message);
  }

  render() {
    return (
      <button onClick={this.handleClick}>
        Say hello
      </button>
    );
  }
}
```
请注意，上述语法是**实验性**的，可能将来会发生改变，或者这个提案可能不会纳入语言范畴。

如果你想更稳妥的方法，你有一下的选择：

* 在构造函数中绑定方法
* 使用箭头函数 `onClick={(e) => this.handleClick(e)}`.
* 使用 `React.createClass()`.

## Mixins

>**注意:**
>
> ES6是不支持mixin的，因此，当你用ES6 class编写React程序时是不支持mixins的
>
>**我们也在使用mixins的情况下发现了部分问题，所以我们不推荐目前使用**
>
>以下部分仅用来参考

有时不同的组件可能会共用部分方法，这些方法会被称为[横切关注点(cross-cutting concerns)](https://en.wikipedia.org/wiki/Cross-cutting_concern)

[`React.createClass`](/react/docs/top-level-api.html#react.createclass) 可以允许你使用mixins。

一个常见的使用场景是组件间隔一段时间自我更新。使用`setInterval()`很容易实现，但是为了节省内存空间必须在不使用时取消。React提供了[生命周期方法](/react/docs/working-with-the-browser.html#component-lifecycle),可以通知你组件创建和销毁。我们编写一个简单的mixin，执行方法可以提供`setInterval()`方法，并且在组件销毁时可以自动被清除。

```javascript
var SetIntervalMixin = {
  componentWillMount: function() {
    this.intervals = [];
  },
  setInterval: function() {
    this.intervals.push(setInterval.apply(null, arguments));
  },
  componentWillUnmount: function() {
    this.intervals.forEach(clearInterval);
  }
};

var TickTock = React.createClass({
  mixins: [SetIntervalMixin], // Use the mixin
  getInitialState: function() {
    return {seconds: 0};
  },
  componentDidMount: function() {
    this.setInterval(this.tick, 1000); // Call a method on the mixin
  },
  tick: function() {
    this.setState({seconds: this.state.seconds + 1});
  },
  render: function() {
    return (
      <p>
        React has been running for {this.state.seconds} seconds.
      </p>
    );
  }
});

ReactDOM.render(
  <TickTock />,
  document.getElementById('example')
);
```

如果一个组件使用多个mixin，不同的mixin中定义了相同的生命周期方法(例如，不容的mixin中都想要在组件销毁时做相应的清理)，这些生命周期函数都会被调用。在组件内部的生命周期方法执行完毕后，mixin中的方法将会按照mixin的顺序依次执行。
