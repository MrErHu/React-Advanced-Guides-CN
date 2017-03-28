# React Without JSX

对于React来说，并不一定需要使用JSX. 如果不想在构建环境下设置编译器，使用React而不使用JSX非常的方便。

每一个JSX元素都是调用`React.createElement(component, props, ...children)`的语法糖，因此，使用JSX所做的任何事都可以通过纯JavaScript实现。

例如，下面代码是通过JSX实现的:

```js
class Hello extends React.Component {
  render() {
    return <div>Hello {this.props.toWhat}</div>;
  }
}

ReactDOM.render(
  <Hello toWhat="World" />,
  document.getElementById('root')
);
```
可以被编译成不使用JSX的代码:

```js
class Hello extends React.Component {
  render() {
    return React.createElement('div', null, `Hello ${this.props.toWhat}`);
  }
}

ReactDOM.render(
  React.createElement(Hello, {toWhat: 'World'}, null),
  document.getElementById('root')
);
```

如果你想查看更多JSX如果转化为JavaScript的实例，你可以尝试[在线Babel编译器](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-0&code=function%20hello()%20%7B%0A%20%20return%20%3Cdiv%3EHello%20world!%3C%2Fdiv%3E%3B%0A%7D)

组件可以通过字符串提供，也可以通过`React.Component`的子类提供，或者通过普通函数实现的无状态组件。

如果你厌倦了使用`React.createElement`，另一个常见的模式是将其赋值给一个缩写:

```js
const e = React.createElement;

ReactDOM.render(
  e('div', null, 'Hello World'),
  document.getElementById('root')
);
```
如果你使用`React.createElement`的缩写形式，就可以很方便的在不通过JSX情况下，使用React。