# Uncontrolled Components

在大多数情况下，我们推荐使用[受控组件](https://facebook.github.io/react/docs/forms.html)来实现表单。在受控组件中，表单数据由React组件负责处理。另外一个选择是不受控组件，其表单数据由DOM元素本身处理。

不同于通过事件处理函数程序来处理状态更新，在不受控组件中，你可以使用[ref](https://facebook.github.io/react/docs/refs-and-the-dom.html)从DOM获得表单数据。

例如，在不受控组件中，以下代码可以输入名字:

```javascript{8,17}
class NameForm extends React.Component {
  constructor(props) {
    super(props);
    this.handleSubmit = this.handleSubmit.bind(this);
  }

  handleSubmit(event) {
    alert('A name was submitted: ' + this.input.value);
    event.preventDefault();
  }

  render() {
    return (
      <form onSubmit={this.handleSubmit}>
        <label>
          Name:
          <input type="text" ref={(input) => this.input = input} />
        </label>
        <input type="submit" value="Submit" />
      </form>
    );
  }
}
```

[可以尝试在CodePen中打开](https://codepen.io/gaearon/pen/WooRWa?editors=0010)

因为不受控组件的数据来源是DOM元素，当使用不受控组件时很容易实现React代码与非React代码的集成。如果你希望的是快速开发、不要求代码质量，不受控组件可以一定程度上减少代码量。否则。你应该使用受控组件。

如果你对在特定的场景下你需要使用哪种组件感到疑惑的话，[ this article on controlled versus uncontrolled inputs](http://goshakkk.name/controlled-vs-uncontrolled-inputs-react/)这篇文章会对你有所帮助。

### 默认值

在React渲染的生命周期中，表单元素的value值将会覆盖DOM中的value值。在不受控组件中，你可能希望React有初始值，但保留后续更新不受控制。在这种情况下，你需要使用`defaultValue`属性而不是`value`属性。

```javascript{7}
render() {
  return (
    <form onSubmit={this.handleSubmit}>
      <label>
        Name:
        <input
          defaultValue="Bob"
          type="text"
          ref={(input) => this.input = input} />
      </label>
      <input type="submit" value="Submit" />
    </form>
  );
}
```

同样，`<input type="checkbox">`和`<input type="radio">`支持`defaultChecked`属性，`<select>`支持`defaultValue`。