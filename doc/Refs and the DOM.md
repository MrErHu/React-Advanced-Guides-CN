# Refs and the DOM

在常规的React数据流中，[prop](https://facebook.github.io/react/docs/components-and-props.html)是父组件与子组件交互的唯一方式。为了修改子元素，你需要用新的prop去重新渲染。然而，在少数情况下，你需要在常规数据流外强制修改子元素。被修改的子元素可以是React组件实例，也可以是DOM元素。在这种情况下，React提供了解决办法。

### 何时使用Refs

下面有一些使用refs的场景:

* 处理focus、文本选择或者媒体播放
* 触发强制动画
* 集成第三方DOM库

如果可以通过声明式实现，就尽量避免使用refs。

例如，相比于在`Dialog`组件中暴露`open()`和`close()`方法，最好传递`isOpen`属性。

### 为DOM元素添加Ref

React支持给任何组件添加特殊属性。`ref`属性接受回调函数，当组件`mounted`或者`unmounted`之后，回调函数会立即执行。

当给HTML元素添加`ref`属性时，回调函数接受底层的DOM元素作为参数。例如，下面的代码使用`ref`函数来存储DOM节点的引用。

```javascript{8,9,19}
class CustomTextInput extends React.Component {
  constructor(props) {
    super(props);
    this.focus = this.focus.bind(this);
  }

  focus() {
    // Explicitly focus the text input using the raw DOM API
    this.textInput.focus();
  }

  render() {
    // Use the `ref` callback to store a reference to the text input DOM
    // element in an instance field (for example, this.textInput).
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

React将会在组件`mount`时，用DOM元素作为参数回调`ref`函数，在组件`unmounts`时，使用`null`作为参数回调函数。

对class设置`ref`回调函数是访问DOM元素的一种常见方法。首选的方式就是像上面的代码一样，对`ref`设置回调函数。还有更简洁的方式:`ref={input => this.textInput = input}`。

### 为Class组件添加Ref

为用class声明的自定义组件设置`ref`属性时，`ref`回调函数收到的参数是`mounted`的组件实例。例如，如果我们想包装`CustomTextInput`组件，实现组件在`mounted`后立即点击的效果:

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
需要注意的是，这种方法仅对以class声明的`CustomTextInput`有效。

```js{1}
class CustomTextInput extends React.Component {
  // ...
}
```

### Refs and Functional Components

**You may not use the `ref` attribute on functional components** because they don't have instances:

```javascript{1,7}
function MyFunctionalComponent() {
  return <input />;
}

class Parent extends React.Component {
  render() {
    // This will *not* work!
    return (
      <MyFunctionalComponent
        ref={(input) => { this.textInput = input; }} />
    );
  }
}
```

You should convert the component to a class if you need a ref to it, just like you do when you need lifecycle methods or state.

You can, however, **use the `ref` attribute inside a functional component** as long as you refer to a DOM element or a class component:

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

### Don't Overuse Refs

Your first inclination may be to use refs to "make things happen" in your app. If this is the case, take a moment and think more critically about where state should be owned in the component hierarchy. Often, it becomes clear that the proper place to "own" that state is at a higher level in the hierarchy. See the [Lifting State Up](/react/docs/lifting-state-up.html) guide for examples of this.

### Legacy API: String Refs

If you worked with React before, you might be familiar with an older API where the `ref` attribute is a string, like `"textInput"`, and the DOM node is accessed as `this.refs.textInput`. We advise against it because string refs have [some issues](https://github.com/facebook/react/pull/8333#issuecomment-271648615), are considered legacy, and **are likely to be removed in one of the future releases**. If you're currently using `this.refs.textInput` to access refs, we recommend the callback pattern instead.

### Caveats

If the `ref` callback is defined as an inline function, it will get called twice during updates, first with `null` and then again with the DOM element. This is because a new instance of the function is created with each render, so React needs to clear the old ref and set up the new one. You can avoid this by defining the `ref` callback as a bound method on the class, but note that it shouldn't matter in most cases.
