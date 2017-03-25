# Web Components

React和[Web Component](https://developer.mozilla.org/en-US/docs/Web/Web_Components)是为了解决不同的问题建立的。Web Component为可重用组件提供了强大的封装，然而React提供声明库，可以使得DOM和数据保持同步。两者的目标是互补的。作为开发者，你可以在你的Web Component中自由使用React，或者在React中使用Web Component，或者都使用。

大多数使用React的开发者不使用Web Component，但是你可能想要使用Web Component，尤其是如果你正在使用Web Component编写的第三方的UI库的情况下。

## 在React中使用Web Components

```javascript
class HelloMessage extends React.Component {
  render() {
    return <div>Hello <x-search>{this.props.name}</x-search>!</div>;
  }
}
```

> 注意:
>
> Web Components通常都会对外暴露一个必须的API，例如，对于一个`video` Web Component可能会暴露`play()`和`pause()`为了访问Web Component的命令式API,您需要`ref`与DOM节点直接交互。如果你使用的是第三方的Web Component，最好的解决方案是编写React组件，作为Web Component的包装器。
>
> 由Web Component发出的事件可能不会沿着React渲染树正确传播。
>
> 因此在你的React组件中，你需要手动的添加事件处理程序来处理这些事件。

一个常见的困惑是Web Component使用`class`而不是`className`

```javascript
function BrickFlipbox() {
  return (
    <brick-flipbox class="demo">
      <div>front</div>
      <div>back</div>
    </brick-flipbox>
  );
}
```

## 在你的Web Component中使用React

```javascript
const proto = Object.create(HTMLElement.prototype, {
  attachedCallback: {
    value: function() {
      const mountPoint = document.createElement('span');
      this.createShadowRoot().appendChild(mountPoint);

      const name = this.getAttribute('name');
      const url = 'https://www.google.com/search?q=' + encodeURIComponent(name);
      ReactDOM.render(<a href={url}>{name}</a>, mountPoint);
    }
  }
});
document.registerElement('x-search', {prototype: proto});
```
您还可以查看[Github完整Web组件示例](https://github.com/facebook/react/tree/master/examples/webcomponents)
