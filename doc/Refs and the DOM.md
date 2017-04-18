# Refs and the DOM

在常规的React数据流中，[props](https://facebook.github.io/react/docs/components-and-props.html)是父组件与子组件交互的唯一方式。为了修改子元素，你需要用新的props去重新渲染子元素。然而，在少数情况下，你需要在常规数据流外强制修改子元素。被修改的子元素可以是React组件实例，也可以是DOM元素。在这种情况下，React提供了解决办法。

### 何时使用Refs

下面有一些恰当地使用refs的场景:

* 处理focus、文本选择或者媒体播放
* 触发强制动画
* 集成第三方DOM库

如果可以通过声明式实现，就尽量避免使用refs。

例如，相比于在`Dialog`组件中暴露`open()`和`close()`方法，最好传递`isOpen`属性。

### 为DOM元素添加Ref

React支持给任何组件添加特殊属性。`ref`属性接受回调函数，当组件安装(`mounted`)或者卸载(`unmounted`)之后，回调函数会立即执行。

当给HTML元素添加`ref`属性时，回调函数接受底层的DOM元素作为参数。例如，下面的代码使用`ref`函数来存储DOM节点的引用。

```javascript{8,9,19}
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focus = this.focus.bind(this);
  }

  focus() {
    // 通过使用原生API，显式地聚焦text输入框
    this.textInput.focus();
  }

  render() {
    // 在实例中通过使用`ref`回调函数来存储text输入框的DOM元素引用(例如:this.textInput)
    return (
      <div>
        <input
          type="text"
          ref={(input) => { this.textInput = input; }} />
        <input
          type="button"
          value="Focus the text input"
          onClick={this.focus}
        />
      </div>
    );
  }
}
```

React将会在组件安装(`mount`)时，用DOM元素作为参数回调`ref`函数，在组件卸载(`unmounts`)时，使用`null`作为参数回调函数。

对class设置`ref`回调函数是访问DOM元素的一种常见方法。首选的方式就是像上面的代码一样，对`ref`设置回调函数。还有更简洁的方式:`ref={input => this.textInput = input}`。

### 为类(Class)组件添加Ref

为用类(class)声明的自定义组件设置`ref`属性时，`ref`回调函数收到的参数是安装(`mounted`)的组件实例。例如，如果我们想包装`CustomTextInput`组件，实现组件在`mounted`后立即点击的效果:

```javascript{3,9}
class AutoFocusTextInput extends React.Component {
  componentDidMount() {
    this.textInput.focus();
  }

  render() {
    return (
      <CustomTextInput
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```
需要注意的是，这种方法仅对以类(class)声明的`CustomTextInput`有效。

```js{1}
class CustomTextInput extends React.Component {
  // ...
}
```

### Refs与函数式Components

**你不能在函数式组件上使用`ref`**因为函数式组件不会创建实例:

```javascript{1,7}
function MyFunctionalComponent() {
  return <input />;
}

class Parent extends React.Component {
  render() {
    // 无效!
    return (
      <MyFunctionalComponent
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

如果你需要使用`ref`，你需要将组件转化成class组件，就像需要生命周期函数或者state那样。

然而你可以在函数式组件内部使用`ref`来引用一个DOM元素或者class组件。

```javascript{2,3,6,13}
function CustomTextInput(props) {
  // textInput must be declared here so the ref callback can refer to it
  let textInput = null;

  function handleClick() {
    textInput.focus();
  }

  return (
    <div>
      <input
        type="text"
        ref={(input) => { textInput = input; }} />
      <input
        type="button"
        value="Focus the text input"
        onClick={handleClick}
      />
    </div>
  );
}
```

### 不要滥用Refs

在你应用中，你可能会倾向于使用`ref`使得"事件发生"。如果这种情况下，花一点时间仔细考虑一下在组件层次结构中哪些地方需要state。通常情况下，应该在层次结构中较高级别中拥有state。有关这方面的实例参阅[提升state](https://facebook.github.io/react/docs/lifting-state-up.html)。

### 旧版API: String类型的Refs

如果你之前使用过React，你可能了解过之前的API:string类型的`ref`属性。类似于`textInput`，可以通过`this.refs.textInput`访问DOM节点。我们不建议使用，因为string类型的`ref`存在[问题](https://github.com/facebook/react/pull/8333#issuecomment-271648615)。已经过时了，**可能会在未来的版本是移除**。如果你目前还在使用`this.refs.textInput`这种方式访问`refs`，我们建议用回调函数的方式代替。

### 注意

如果`ref`以内联函数的方式定义，在update期间会被调用两次，第一次参数是`null`,之后参数是DOM元素。这是因为在每次渲染中都会创建一个新的函数实例。因此，React需要清理旧的ref并且设置新的。通过将`ref`的回调函数定义成类的绑定函数的方式可以避免上述问题，但是在大多数例子中这都不是很重要。
