# React与其他库的集成

React可以在任何web应用中使用。React可以嵌入其他的应用中，也可以将其他的应用嵌入React中，不过需要多加小心。本篇教程将介绍部分常见的使用场景，主要包括集成jQuery和Backbone,但是同样的思想可以用来集成组件到其他任何现有的代码。

## 与DOM操作插件的集成

React 无法感知到React之外的DOM变化。这决定了更新只能基于React内部的表示，如果相同的DOM节点被其他库所操作，React会对此产生疑惑并无法恢复。

这并不意味着很难或者无法将React于其他影响DOM的方式相结合，你需要更加注意两者各自的行为。

避免冲突最简单的方式就是阻止React的更新。你可以通过渲染React无法更新的元素来实现，例如空的`<div />`。

### 如何处理这个问题

为了展示这个问题，我们来为通用的jQuery插件绘制一个包装器(wrapper)。

我们给根DOM元素添加[ref](https://facebook.github.io/react/docs/refs-and-the-dom.html)。在`componentDidMount`中，我们将获得引用(`reference`)，因此将其传递给jQuery插件。

为了防止React在mount之后处理DOM元素，我们将在`render`方法中返回空的`<div />`。<div>元素没有属性或者子元素，因此React不会更新它，使得jQuery插件可以自由地管理这部分的DOM节点。


```js{3,4,8,12}
class SomePlugin extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.somePlugin();
  }

  componentWillUnmount() {
    this.$el.somePlugin('destroy');
  }

  render() {
    return <div ref={el => this.el = el} />;
  }
}
```

注意，我们定义了`componentDidMount`和`componentWillUnmount`[生命周期函数](https://facebook.github.io/react/docs/react-component.html#the-component-lifecycle)。很多jQuery插件为DOM元素添加了监听器(listener),因此在`componentWillUnmount`中退订监听器是非常重要的。如果插件本身不提供清除(cleanup)的方法，你可能需要自己提供，牢记一定要移除注册在插件中的事件监听者(event listener)以防止内存泄露。

### 与jQuery的Chosen插件集成

为了更具体地描述这个概念，让我们写一个最小化的[Chosen](https://harvesthq.github.io/chosen/)插件的包装器(wrapper)，其中插件Chosen接受`<select>`元素的输入。

>**注意:**
>
> 仅仅不能因为有实现的可能性，就意味着这对React应用来讲是最佳实践。我们鼓励在可能的情况下使用React组件。React组件很容易在React应用中重用(reuse)并且能更好控制其行为与外观。

首先，我们来看看Chosen对DOM的行为。

如果你在`<select>`DOM节点上调用Chosen，其会读取原始DOM节点的属性，以内联样式方式隐藏，并在内部的虚拟表达中添加单独的DOM节点。随后触发jQuery事件通知事件改变。

让我们了解一下我们所设计的React组件包装器(wrapper)`<Chosen>`的API:

```javascript
function Example() {
  return (
    <Chosen onChange={value => console.log(value)}>
      <option>vanilla</option>
      <option>chocolate</option>
      <option>strawberry</option>
    </Chosen>
  );
}
```

为了简单起见，我们将其实现为一个[不受控组件](https://facebook.github.io/react/docs/uncontrolled-components.html)。

首先我们先创建一个空的组件，其中包含`render()`方法，其返回一个由`<div>`包裹的`<select>`:

```javascript
class Chosen extends React.Component {
  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

需要注意为什么我们为`<select>`包裹一个额外的`<div>`。这是必须的，因为Chosen插件会为我们传入`<select>`节点后添加另一个DOM节点。。然而，就React而言，`<div>`总是只有一个子节点。这就是我们如何来确保React更新不会与Chosen插件所添加的额外DOM节点相冲突。值得注意的是，如果在React流之外修改了DOM节点，必须确保React无论如何都不会再接触DOM节点。

接下来，我们实现生命周周期函数。我们在`componentDidMount`中对引用的`<select>`节点初始化Chosen插件，并在`componentWillUnmount`中清除:

```js{2,3,7}
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();
}

componentWillUnmount() {
  this.$el.chosen('destroy');
}
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/qmqeQx?editors=0010)

注意,React对`this.el`变量并没有特殊的含义，生效的原因仅仅是我们先前在`render()`方法中`ref`对其进行了赋值:

```js
<select className="Chosen-select" ref={el => this.el = el}>
```

上述对于组件的渲染已经足够，但是我们也想要获得值改变的通知(notifies about the value changes),为了实现这个目的，我们在`<select>`节点上订阅(subscribe)jQuery Chosen插件的`change`事件。

我们不直接给Chosen传递`this.props.onChange`，因为包括事件处理程序在内的组件的属性可能会发生改变。相反，我们声明`handleChange`方法，它会调用`this.props.onChange`方法，并订阅jQuery的`change`事件:

```javascript
componentDidMount() {
  this.$el = $(this.el);
  this.$el.chosen();

  this.handleChange = this.handleChange.bind(this);
  this.$el.on('change', this.handleChange);
}

componentWillUnmount() {
  this.$el.off('change', this.handleChange);
  this.$el.chosen('destroy');
}

handleChange(e) {
  this.props.onChange(e.target.value);
}
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/bWgbeE?editors=0010)

最后，还剩一件事做。在React里props会随时间而改变。例如，如果父组件state改变，`<Chosen>`组件可能会得到不同的children。这意味着集成的要点是我们必须手动地更新DOM节点来响应props的更新，因为我们已经不能让React再管理DOM节点。

Chosen的文档建议我们使用jQuery的`trigger()`API来通知原始DOM节点的更新。我们使用React去关注`<select>`标签内的子节点`this.props.children`的更新，我们会在生命周期函数`componentDidUpdate`中向Chosen通知子元素的改变。

```javascript
componentDidUpdate(prevProps) {
  if (prevProps.children !== this.props.children) {
    this.$el.trigger("chosen:updated");
  }
}
```

这样一来，当React导致`<select>`子元素的改变时，Chosen将会所感知到DOM节点的更新。

`Chosen`组件的完整实现如下所示:

```javascript
class Chosen extends React.Component {
  componentDidMount() {
    this.$el = $(this.el);
    this.$el.chosen();

    this.handleChange = this.handleChange.bind(this);
    this.$el.on('change', this.handleChange);
  }

  componentDidUpdate(prevProps) {
    if (prevProps.children !== this.props.children) {
      this.$el.trigger("chosen:updated");
    }
  }

  componentWillUnmount() {
    this.$el.off('change', this.handleChange);
    this.$el.chosen('destroy');
  }

  handleChange(e) {
    this.props.onChange(e.target.value);
  }

  render() {
    return (
      <div>
        <select className="Chosen-select" ref={el => this.el = el}>
          {this.props.children}
        </select>
      </div>
    );
  }
}
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/xdgKOz?editors=0010)

## 与其他的视图库集成

感谢极具灵活性的方法[`ReactDOM.render()`](https://facebook.github.io/react/docs/react-dom.html#render)，使得React可以嵌入其他的应用中。

虽然React通常在启动时将单个根节点的React组件加载进DOM节点，但`ReactDOM.render()`也可以被多次调用来生成独立的部分UI，小到一个按钮，大到一个应用。

事实上，在Facebook中React就是这么用的。这使得我们可以一步一步地使用React编写程序，并与我们现存的服务器生成的模板与其他客户端代码相结合。

### 用React替换字符串渲染

在之前的web应用中，一种常见的模式是将DOM块作为字符串描述，并将其插入DOM节点，例如: `$el.html(htmlString)`。这种代码是非常适合引入React的，仅仅需要将渲染的字符串重写为React组件。

因此下面的jQuery实现:

```js
$('#container').html('<button id="btn">Say Hello</button>');
$('#btn').click(function() {
  alert('Hello!');
});
```

可以用React组件重写成:

```js
function Button() {
  return <button id="btn">Say Hello</button>;
}

ReactDOM.render(
  <Button />,
  document.getElementById('container'),
  function() {
    $('#btn').click(function() {
      alert('Hello!');
    });
  }
);
```

从这里开始，你就可以将更多的逻辑移动进组件中，并采用更常见的React实践。例如，在组件，最好不要依赖id值，因为相同的组件可能被多次渲染。相反，我们可以使用[React事件系统](https://facebook.github.io/react/docs/handling-events.html),直接在React的`<button>`元素上注册点击事件处理函数。

```javascript
function Button(props) {
  return <button onClick={props.onClick}>Say Hello</button>;
}

function HelloButton() {
  function handleClick() {
    alert('Hello!');
  }
  return <Button onClick={handleClick} />;
}

ReactDOM.render(
  <HelloButton />,
  document.getElementById('container')
);
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/RVKbvW?editors=1010)

你可以按照你的想法创建多个独立的组件，并使用`ReactDOM.render()`将它们渲染进不同的DOM容器中。在随着你逐渐地将你的应用转成React应用的过程中，你会将组件合并成更大的组件，将将部分的`ReactDOM.render()`调用改为React的层次结构。

### 在Backbone视图中集成React

[Backbone](http://backbonejs.org/)视图(View)是典型的使用字符串或者字符串产生函数来生成DOM元素的内容。这个过程可以通过渲染React组件来替代。

下面我们将创建一个名为`ParagraphView`的Backbone视图，它将用来覆盖Backbone的`render`函数，渲染React的`<Paragraph>`组件到Backbone提供的DOM节点。这里我们也使用`ReactDOM.render()`:

```javascript
function Paragraph(props) {
  return <p>{props.text}</p>;
}

const ParagraphView = Backbone.View.extend({
  render() {
    const text = this.model.get('text');
    ReactDOM.render(<Paragraph text={text} />, this.el);
    return this;
  },
  remove() {
    ReactDOM.unmountComponentAtNode(this.el);
    Backbone.View.prototype.remove.call(this);
  }
});
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/gWgOYL?editors=0010)

在`remove`方法中我们必须调用`ReactDOM.unmountComponentAtNode()`，使得React注销与组件树销毁时相关的事件处理函数和其他相关资源。

当组件从组件树中删除时，清除将自动执行，但是因为我们手动地移除整个组件树，我们必须调用这个方法。

## React与Model层集成

尽管我们推荐使用单向数据流例如:[React state](http://facebook.github.io/react/react/docs/lifting-state-up.html)、[Flux](http://facebook.github.io/flux/)或者[Redux](http://redux.js.org/)，但React组件仍然可以使用其他框架和库的model层。

### 在React组件中使用Backbone Models

对React组件而言，使用Backbone models最简单的方式就是监听不同的change事件并手动强制刷新。

负责渲染model的React组件必须监听`'change'`事件，负责渲染collections的React组件必须监听`'add'`和`'remove'`事件。在这些场景下，调用[`this.forceUpdate()`](http://facebook.github.io/react/docs/react-component.html#forceupdate)用新的数据重新渲染组件。

在下面的例子中，`List`组件渲染Backbone的collection，而`Item`组件负责渲染单个items。

```javascript
class Item extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.model.on('change', this.handleChange);
  }

  componentWillUnmount() {
    this.props.model.off('change', this.handleChange);
  }

  render() {
    return <li>{this.props.model.get('text')}</li>;
  }
}

class List extends React.Component {
  constructor(props) {
    super(props);
    this.handleChange = this.handleChange.bind(this);
  }

  handleChange() {
    this.forceUpdate();
  }

  componentDidMount() {
    this.props.collection.on('add', 'remove', this.handleChange);
  }

  componentWillUnmount() {
    this.props.collection.off('add', 'remove', this.handleChange);
  }

  render() {
    return (
      <ul>
        {this.props.collection.map(model => (
          <Item key={model.cid} model={model} />
        ))}
      </ul>
    );
  }
}
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/GmrREm?editors=0010)

### 从Backbone Models中提取数据

上述方法需要你的React组件了解Backbone的模型和集合。如果你随后计划迁移到另一个数据管理方案，你可能希望将Backbone的概念集中在尽可能少的代码中。

一个解决这个问题的方案是，每当模型的数据改变时将模型的属性提取成一个纯数据，并将逻辑保存在一个单一的位置。接下来的[高阶组件](https://facebook.github.io/react/docs/higher-order-components.html)提取Backbone中的属性作为state,并将数据传递给被包裹的组件。

这样，仅有高阶组件需要了解Backbone内部的model,应用中大多数的组件可以与Backbone保持独立。

在下面的例子中，我们将复制model的属性来形成最初的状态。我们订阅`change`事件(以及在卸载时的`unsubscribe`事件），当`change`事件发生时，我们使用model的当前属性来更新state。最终，我们确定，如果`model`属性本身改变时，我们不要忘记退订之前的model,订阅新的model。

请注意，下面的例子并不意味着涵盖与Backbone集成使用的方方面面，但是它提供一种通用的思路来解决上述的问题:

```javascript
function connectToBackboneModel(WrappedComponent) {
  return class BackboneComponent extends React.Component {
    constructor(props) {
      super(props);
      this.state = Object.assign({}, props.model.attributes);
      this.handleChange = this.handleChange.bind(this);
    }

    componentDidMount() {
      this.props.model.on('change', this.handleChange);
    }

    componentWillReceiveProps(nextProps) {
      this.setState(Object.assign({}, nextProps.model.attributes));
      if (nextProps.model !== this.props.model) {
        this.props.model.off('change', this.handleChange);
        nextProps.model.on('change', this.handleChange);
      }
    }

    componentWillUnmount() {
      this.props.model.off('change', this.handleChange);
    }

    handleChange(model) {
      this.setState(model.changedAttributes());
    }

    render() {
      const propsExceptModel = Object.assign({}, this.props);
      delete propsExceptModel.model;
      return <WrappedComponent {...propsExceptModel} {...this.state} />;
    }
  }
}
```

为了演示如何使用这个例子，我们将`NameInput`React组件连接到Backbone的model,并在每次输入改变时更新其`firstName`属性。

```javascript
function NameInput(props) {
  return (
    <p>
      <input value={props.firstName} onChange={props.handleChange} />
      <br />
      My name is {props.firstName}.
    </p>
  );
}

const BackboneNameInput = connectToBackboneModel(NameInput);

function Example(props) {
  function handleChange(e) {
    model.set('firstName', e.target.value);
  }

  return (
    <BackboneNameInput
      model={props.model}
      handleChange={handleChange}
    />
  );
}

const model = new Backbone.Model({ firstName: 'Frodo' });
ReactDOM.render(
  <Example model={model} />,
  document.getElementById('root')
);
```

[在CodePen中尝试](http://codepen.io/gaearon/pen/PmWwwa?editors=0010)

这个技术并不局限于Backbone。你可以通过在生命周期函数中订阅model改变并且可选地将model中的数据复制进React内部的state的方式，实现React与其他model库集成使用。
