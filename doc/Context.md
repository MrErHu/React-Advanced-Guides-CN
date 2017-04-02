# Context

在React中，在React组件中很容易追踪数据流。当你观察组件时，你可以找出哪些属性(props)被传递，这使得你的应用非常容易理解。

在某些场景下，你想在整个组件树中传递数据，但却不想手动地在每一层传递属性。你可以直接在React中使用强大的`context` API解决上述问题。

## 为什么不要使用Context

绝大多数的应用程序不需要使用`context`。

如果你希望使用应用程序更加稳定就不要使用context。这只是一个实验性的API并且可能在未来的React版本中移除。

如果你不熟悉[React](https://github.com/reactjs/redux)或者[Mobx](https://github.com/mobxjs/mobx)这类state管理库，就不要使用`context`。对于许多应用程序，上述库和`state`绑定是管理`state`不错的选择。`Redux`相比`context`是更好的解决方法。

如果你不是一个有经验的React开发者，就不要使用`context`。更好的方式是使用`props`和`state`。

如果你不顾这些警告仍然坚持使用`context`，尝试着将`context`的使用隔离在一个将小的范围内，并且在可能的情况下直接使用`context`，以便在API改变的时候进行升级。

## 如何使用Context

假定有下面的结构:

```javascript
class Button extends React.Component {
  render() {
    return (
      <button style={{'{{'}}background: this.props.color}}>
        {this.props.children}
      </button>
    );
  }
}

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button color={this.props.color}>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  render() {
    const color = "purple";
    const children = this.props.messages.map((message) =>
      <Message text={message.text} color={color} />
    );
    return <div>{children}</div>;
  }
}
```

在这个例子中，我们手动地传递`color`属性使得`Button`和`Message`设置正确的样式。使用`context`，我们可以自动在组件树中传递属性。

```javascript{4,11-13,19,26-28,38-40}
class Button extends React.Component {
  render() {
    return (
      <button style={{'{{'}}background: this.context.color}}>
        {this.props.children}
      </button>
    );
  }
}

Button.contextTypes = {
  color: React.PropTypes.string
};

class Message extends React.Component {
  render() {
    return (
      <div>
        {this.props.text} <Button>Delete</Button>
      </div>
    );
  }
}

class MessageList extends React.Component {
  getChildContext() {
    return {color: "purple"};
  }

  render() {
    const children = this.props.messages.map((message) =>
      <Message text={message.text} />
    );
    return <div>{children}</div>;
  }
}

MessageList.childContextTypes = {
  color: React.PropTypes.string
};
```

通过给`MessageList`添加`childContextTypes`和`childContextTypes`(`context`提供者)，React自动地向下传递信息，任何子树(例如:`Button`)可以通过定义`contextTypes`访问到属性。

如果没有定义`contextTypes`,`context`将是一个空的object。

## 父子耦合

Context可以构建API使得父组价和子组件进行相互通信。例如：[React Router V4](https://reacttraining.com/react-router)工作机制如下:

```javascript
import { BrowserRouter as Router, Route, Link } from 'react-router-dom';

const BasicExample = () => (
  <Router>
    <div>
      <ul>
        <li><Link to="/">Home</Link></li>
        <li><Link to="/about">About</Link></li>
        <li><Link to="/topics">Topics</Link></li>
      </ul>

      <hr />

      <Route exact path="/" component={Home} />
      <Route path="/about" component={About} />
      <Route path="/topics" component={Topics} />
    </div>
  </Router>
);
```

通过从`Router`中传递相关信息，`Router`中的每一个`Link`和`Route`都可以与之通信。

在你构建包含类似于上述的API的组件之前，考虑是否有其他的更清晰的选择。例如，你可以传递整个React组件作为props传递。

## 在生命周期函数中使用`Context`

如果`contextTypes`在组件中定义，下列的[生命周期函数](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle)将接受一个额外的参数:`context`对象

- [`constructor(props, context)`](https://facebook.github.io/react/docs/react-component.html#constructor)
- [`componentWillReceiveProps(nextProps, nextContext)`](https://facebook.github.io/react/docs/react-component.html#componentwillreceiveprops)
- [`shouldComponentUpdate(nextProps, nextState, nextContext)`](https://facebook.github.io/react/docs/react-component.html#shouldcomponentupdate)
- [`componentWillUpdate(nextProps, nextState, nextContext)`](https://facebook.github.io/react/docs/react-component.html#componentwillupdate)
- [`componentDidUpdate(prevProps, prevState, prevContext)`](https://facebook.github.io/react/docs/react-component.html#componentdidupdate)

## 在无状态的函数式组件中使用`Context`

如果`contextType`被定义为函数的属性，无状态函数式组件也能够引用`context`。下面的代码演示了一个`Button`状态的函数式组件。

```javascript
const Button = ({children}, context) =>
  <button style={{'{{'}}background: context.color}}>
    {children}
  </button>;

Button.contextTypes = {color: React.PropTypes.string};
```

## 更新Context

别这么做！

React有一个API更新context，但是它打破了基本流程，不应该使用。

`getChildContext`函数将会在每次`state`或者`props`改变时调用。为了更新`context`中的数据，使用`this.setState`触发本地状态的更新。这将触发一个的`context`并且数据的改变可以被子元素收到。

```javascript
class MediaQuery extends React.Component {
  constructor(props) {
    super(props);
    this.state = {type:'desktop'};
  }

  getChildContext() {
    return {type: this.state.type};
  }

  componentDidMount() {
    const checkMediaQuery = () => {
      const type = window.matchMedia("(min-width: 1025px)").matches ? 'desktop' : 'mobile';
      if (type !== this.state.type) {
        this.setState({type});
      }
    };

    window.addEventListener('resize', checkMediaQuery);
    checkMediaQuery();
  }

  render() {
    return this.props.children;
  }
}

MediaQuery.childContextTypes = {
  type: React.PropTypes.string
};
```

问题在于，组件提供的`context`值改变，后代元素如果`shouldComponentUpdate`返回`false`那么`context`的将不会更新。这使得使用`context`的组件完全失控，所以基本上没有办法可靠的更新`context`。[这篇blog](https://medium.com/@mweststrate/how-to-safely-use-react-context-b7e343eff076)很好的解释了为什么这是一个问题并如果绕过它。
