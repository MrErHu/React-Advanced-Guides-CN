# Typechecking With PropTypes

随着应用规模的提升，你可以通过类型检测捕捉更多的bug。对于部分应用，你可能需要需要使用类似于[Flow](https://flowtype.org/)或者[TypeScript](https://www.typescriptlang.org/)的JavaScript扩展来对你整个应用类型进行类型检测。但即使你不使用这些，React内置了类型检测的功能。要在组件中运行类型检测，你可以赋值`propTypes`属性。

```javascript
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

Greeting.propTypes = {
  name: React.PropTypes.string
};
```

`React.PropTypes` 输出了一系列的验证器，可以用来确保接收到的参数是有效的。例如，我们可以使用`React.PropTypes.string`语句。当给prop传递了一个不正确的值时，JavaScript控制台将会显示一条警告。出于性能的原因，`propTypes`仅在开发模式中检测。

### React.PropTypes

下面给出不同验证器的示例:

```javascript
MyComponent.propTypes = {
  // 声明prop一个特定的基本类型，默认情况，是可选的。
  optionalArray: React.PropTypes.array,
  optionalBool: React.PropTypes.bool,
  optionalFunc: React.PropTypes.func,
  optionalNumber: React.PropTypes.number,
  optionalObject: React.PropTypes.object,
  optionalString: React.PropTypes.string,
  optionalSymbol: React.PropTypes.symbol,

  // 任何可以被渲染:numbers, strings, elements,或者是包含这些类型的数组(或者是片段)
  optionalNode: React.PropTypes.node,

  // A React element.
  optionalElement: React.PropTypes.element,

  // 声明props是一个类，使用JS的instanceof操作符
  optionalMessage: React.PropTypes.instanceOf(Message),

  // 声明prop是特定的值，类似于枚举
  optionalEnum: React.PropTypes.oneOf(['News', 'Photos']),

  // 多种类型其中之一
  optionalUnion: React.PropTypes.oneOfType([
    React.PropTypes.string,
    React.PropTypes.number,
    React.PropTypes.instanceOf(Message)
  ]),

  // 包含测定类型的数组
  optionalArrayOf: React.PropTypes.arrayOf(React.PropTypes.number),

  // 值为特定类型的对象
  optionalObjectOf: React.PropTypes.objectOf(React.PropTypes.number),

  // 特定形式的对象
  optionalObjectWithShape: React.PropTypes.shape({
    color: React.PropTypes.string,
    fontSize: React.PropTypes.number
  }),

  // 可以为上面的声明后添加`isRequired`使得如果没有提供props会给出warning
  // You can chain any of the above with `isRequired` to make sure a warning
  // is shown if the prop isn't provided.
  requiredFunc: React.PropTypes.func.isRequired,

  // 任何值
  requiredAny: React.PropTypes.any.isRequired,

  // 可以声明自定义的验证器，如果验证失败返回Error对象。不要使用`console.warn`或者throw
  // 因为这不会在`oneOfType`类型的验证器中起作用。
  customProp: function(props, propName, componentName) {
    if (!/matchme/.test(props[propName])) {
      return new Error(
        'Invalid prop `' + propName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  },

  // 也可以声明`arrayOf`和`objectOf`类型的验证器，如果验证失败需要返回Error对象。
  // 会在数组或者对象的每一个元素上调用验证器。验证器的前两个参数分别是数组或者对象本身，
  // 以及当前元素的键值。
  customArrayProp: React.PropTypes.arrayOf(function(propValue, key, componentName, location, propFullName) {
    if (!/matchme/.test(propValue[key])) {
      return new Error(
        'Invalid prop `' + propFullName + '` supplied to' +
        ' `' + componentName + '`. Validation failed.'
      );
    }
  })
};
```

### Requiring Single Child

你可以使用`React.PropTypes.element`指定仅有一个单一子元素可以作为子节点传递给组件。

```javascript
class MyComponent extends React.Component {
  render() {
    // This must be exactly one element or it will warn.
    const children = this.props.children;
    return (
      <div>
        {children}
      </div>
    );
  }
}

MyComponent.propTypes = {
  children: React.PropTypes.element.isRequired
};
```

### 默认Prop值

通过赋值特殊的`defaultProps`属性，你可以为`props`定义默认值:

```javascript
class Greeting extends React.Component {
  render() {
    return (
      <h1>Hello, {this.props.name}</h1>
    );
  }
}

// Specifies the default values for props:
Greeting.defaultProps = {
  name: 'Stranger'
};

// Renders "Hello, Stranger":
ReactDOM.render(
  <Greeting />,
  document.getElementById('example')
);
```

如果父组件没有为`this.props.name`传值，`defaultProps`会给其一个默认值。`propTypes`的类型检测是在`defaultProps`解析之后发生的，因此也会对默认属性`defaultProps`进行类型检测。
