# JSX In Depth

从根本上讲，JSX只是`React.createElement(component, props, ...children)`函数的语法糖。JSX代码:

```js
<MyButton color="blue" shadowSize={2}>
  Click Me
</MyButton>
```

会被编译为:

```js
React.createElement(
  MyButton,
  {color: 'blue', shadowSize: 2},
  'Click Me'
)
```

如果不存在子节点，你可以使用自闭合格式的标签。例如:

```js
<div className="sidebar" />
```

会被编译为:

```js
React.createElement(
  'div',
  {className: 'sidebar'},
  null
)
```
如果你想要了解JSX是如何编译为JavaScript，可以尝试[在线Babel编译器](https://babeljs.io/repl/#?babili=false&evaluate=true&lineWrap=false&presets=es2015%2Creact%2Cstage-0&code=function%20hello()%20%7B%0A%20%20return%20%3Cdiv%3EHello%20world!%3C%2Fdiv%3E%3B%0A%7D).

## 指定React元素类型

JSX标签的开始部分决定了React元素的类型。首字母大写的标签指示JSX标签是一个React组件。这些标签会被编译成命名变量的直接引用。所以如果你使用JSX的`<Foo />`表达式，`Foo`必须在作用域中。

### React必须在作用域中存在
因为JSX被编译为`React.createElement`，所以`React`库必须在代码的作用域中。例如，在下面的代码中，虽然`React`和`CustomButton`并没有在JavaScript代码中直接使用，但是必须二者都需要在代码中引用。

```js{1,2,5}
import React from 'react';
import CustomButton from './CustomButton';

function WarningButton() {
  // return React.createElement(CustomButton, {color: 'red'}, null);
  return <CustomButton color="red" />;
}
```
如果你没有打包JavaScript而是在script标签中添加了React，那么全局中已经存在`React`。

### 对JSX类型使用点表示法

在JSX中，你可以通过点表示法引用React组件。如果仅有一个module但其中却对外提供多个React组件时，点表示法就非常的方便。

```js{10}
import React from 'react';

const MyComponents = {
  DatePicker: function DatePicker(props) {
    return <div>Imagine a {props.color} datepicker here.</div>;
  }
}

function BlueDatePicker() {
  return <MyComponents.DatePicker color="blue" />;
}
```

### 自定义组件必须以大写字母开头

对于以小写字母开头的元素类型，其指示类似于`<div>`或者`<span>`的内置组件，会给`React.createElement`方法传递字符串`div`或者`span`。以大写字母开头的类型，类似于`<Foo />`，将会被编译成`React.createElement(Foo)`，对应于自定义组件或者在JavaScript文件中引入的组件。

我们建议给组件以大写字母开头的方式命名。如果你已经有以小写字母开头的组件，需要在JSX中使用前将其赋值给以大写字母开头的变量。例如下面代码无法按照预期运行:

```js{3,4,10,11}
import React from 'react';

// Wrong! This is a component and should have been capitalized:
function hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Wrong! React thinks <hello /> is an HTML tag because it's not capitalized:
  return <hello toWhat="World" />;
}
```

为了修复这个问题，我们将`hello`重命名为`Hello`，然后在引用时使用`<Hello />`

```js{3,4,10,11}
import React from 'react';

// Correct! This is a component and should be capitalized:
function Hello(props) {
  // Correct! This use of <div> is legitimate because div is a valid HTML tag:
  return <div>Hello {props.toWhat}</div>;
}

function HelloWorld() {
  // Correct! React knows <Hello /> is a component because it's capitalized.
  return <Hello toWhat="World" />;
}
```

### 运行时决定React类型

不能使用普通的表达式作为React元素类型。如果你想使用通用的表达式来表示元素类型，首先你需要将其赋值给大写的变量。这通常会出现在出现在根据不同的props渲染不同的组件：

```js{10,11}
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Wrong! JSX type can't be an expression.
  return <components[props.storyType] story={props.story} />;
}
```

为了解决这个问题，首先需要将其赋值给一个以大写字母开头的变量。

```js{9-11}
import React from 'react';
import { PhotoStory, VideoStory } from './stories';

const components = {
  photo: PhotoStory,
  video: VideoStory
};

function Story(props) {
  // Correct! JSX type can be a capitalized variable.
  const SpecificStory = components[props.storyType];
  return <SpecificStory story={props.story} />;
}
```

## JSX中的props

在JSX中有下面几种不同的方式赋值props。

### JavaScript 表达式

你可以给props传递一个用`{}`包裹的JavaScript表达式，例如：

```js
<MyComponent foo={1 + 2 + 3 + 4} />
```

对`MyComponent`对讲，`props.foo`的值为`10`，因为表达式`1 + 2 + 3 + 4`会被计算。

对于JavaScript，`if`语句和`for`循环不是表达式，因此不能在JSX中直接使用。但你可以将其写入代码块中，例如:

```js{3-7}
function NumberDescriber(props) {
  let description;
  if (props.number % 2 == 0) {
    description = <strong>even</strong>;
  } else {
    description = <i>odd</i>;
  }
  return <div>{props.number} is an {description} number</div>;
}
```

### 字符串

你可以给prop传入字符串，下面两种JSX表达式是等价的：

```js
<MyComponent message="hello world" />

<MyComponent message={'hello world'} />
```

当给props传递字符串时，其值是未转义的HTML。下面两种JSX表达式是等价的：

```js
<MyComponent message="&lt;3" />

<MyComponent message={'<3'} />
```

这种行为通常是不相关的，这里所提到的只是完整性方面。

### Props Default to "True"

如果prop没有传入值，默认为`true`。下面两种JSX表达式是等价的：

```js
<MyTextBox autocomplete />

<MyTextBox autocomplete={true} />
```

通常情况下，我们不建议使用这种类型，因为这会与[ES6中的对象shorthand](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Operators/Object_initializer#New_notations_in_ECMAScript_2015)混淆。ES6 shorthand中`{foo}`指的是`{foo: foo}`而不是`{foo: true}`。

### 属性展开

如果存在一个object的`props`并且相传入JSX,你可以使用展开符`...`传入整个props对象。下面两种组件是等价的：

```js{7}
function App1() {
  return <Greeting firstName="Ben" lastName="Hector" />;
}

function App2() {
  const props = {firstName: 'Ben', lastName: 'Hector'};
  return <Greeting {...props} />;
}
```

当你构建一个container时，属性展开非常有用。然而，这可能会使得你的代码非常混乱，因为这使得非常多不相关的props传递给组件，但组件并不需要。因此我们建议谨慎使用该语法。

## JSX中的Children

在开标签和闭合标签中包含的JSX表达式会被传递给一个特殊的prop:`props.children`。下面有好几种方式传递`children`

### 字符串

您可以在开标签和闭合标签中写入字符串，这对于内置HTML元素非常有用，例如:

```js
<MyComponent>Hello world!</MyComponent>
```
这是有效的JSX，`MyComponent`的`props.children`属性值为`"Hello world!"`。HTML是非转义的,因此你可以像写HTML一样来写JSX。

```html
<div>This is valid HTML &amp; JSX at the same time.</div>
```

JSX会删除每行开头和结尾的空格，并且也会删除空行和邻接标签的新行，字符串之间的空格会被压缩成一个空格，因此下面的渲染效果都是相同的：

```js
<div>Hello World</div>

<div>
  Hello World
</div>

<div>
  Hello
  World
</div>

<div>

  Hello World
</div>
```

### JSX Children

你可以传递多个JSX元素作为子元素，这对显示嵌套组件非常有用：

```js
<MyContainer>
  <MyFirstComponent />
  <MySecondComponent />
</MyContainer>
```

你可以混合不同类型的子元素，因此你可以混用字符串和JSX子元素。这是JSX与HTML另一点相似的地方，因此下面是有效的HTML和有效的JSX:

```html
<div>
  Here is a list:
  <ul>
    <li>Item 1</li>
    <li>Item 2</li>
  </ul>
</div>
```
React组件不能多个React元素，但是单个JSX表达式可以返回多个子元素，因此如果你想要渲染多个元素，你可以像上面一样，将其包裹在`div`中

### JavaScript 表达式

你可以将任何的JavaScript元素通过用`{}`包裹而作为子元素传递，下面表达式是等价的:

```js
<MyComponent>foo</MyComponent>

<MyComponent>{'foo'}</MyComponent>
```

这对于渲染长度不定的JSX表达式列表非常有用，例如，下面会渲染HTML list:

```js{2,9}
function Item(props) {
  return <li>{props.message}</li>;
}

function TodoList() {
  const todos = ['finish doc', 'submit pr', 'nag dan to review'];
  return (
    <ul>
      {todos.map((message) => <Item key={message} message={message} />)}
    </ul>
  );
}
```

JavaScript表达式可以和其他类型的子元素混用，这对于字符串模板非常有用：

```js{2}
function Hello(props) {
  return <div>Hello {props.addressee}!</div>;
}
```

### Function类型的子元素

通常情况下，嵌入JSX中的JavaScript表达式会被认为是字符串、React元素或者是这些内容的列表。然而，`props.children`类似于其他的props，可以被传入任何数据，而不是仅仅只是React可以渲染的数据。例如，如果有自定义组件，其`props.children`的值可以是回调函数：

```js{4,13}
// Calls the children callback numTimes to produce a repeated component
function Repeat(props) {
  let items = [];
  for (let i = 0; i < props.numTimes; i++) {
    items.push(props.children(i));
  }
  return <div>{items}</div>;
}

function ListOfTenThings() {
  return (
    <Repeat numTimes={10}>
      {(index) => <div key={index}>This is item {index} in the list</div>}
    </Repeat>
  );
}
```
自定义组件的子元素可以是任何类型，只要在渲染之前组件可以将其转化为React能够处理的东西即可。这种用法并不常见，但是如果你需要扩展JSX的话，则会非常有用。

### Booleans, Null和Undefined都会被忽略

`false`, `null`, `undefined`, 和 `true`都是有效的子类型。但是并不会被渲染，下面的JSX表达式渲染效果是相同的：

```js
<div />

<div></div>

<div>{false}</div>

<div>{null}</div>

<div>{undefined}</div>

<div>{true}</div>
```

在有条件性渲染React元素时非常有用。如果`showHeader`为`true`时，`<Header />`会被渲染：

```js{2}
<div>
  {showHeader && <Header />}
  <Content />
</div>
```

需要注意的是，React仍然提供了["falsy"值](https://developer.mozilla.org/en-US/docs/Glossary/Falsy),例如数值`0`，仍然会被React渲染。例如：你可能会认为当`props.messages`为空数组时，将会渲染`0`，但实际并不是这样：

```js{2}
<div>
  {props.messages.length &&
    <MessageList messages={props.messages} />
  }
</div>
```
为了解决这个问题，你需要使得`&&`前的表达式是boolean类型：

```js{2}
<div>
  {props.messages.length > 0 &&
    <MessageList messages={props.messages} />
  }
</div>
```
反过来，如果在输出中想要渲染`false`、`true`、`null`或者`undefined`，
你必须先将其[转化为字符串](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/String#String_conversion) first:

```js{2}
<div>
  My JavaScript variable is {String(myVariable)}.
</div>
```
