# Reconciliation

原文: [Reconciliation](https://facebook.github.io/react/docs/reconciliation.html)

翻译: [MrErHu(请叫我王磊同学)](https://github.com/MrErHu)

邮箱: [wanglei_cs@163.com](mailto:wanglei_cs@163.com)

React提供声明式API，因此在每次更新中你不需要关心具体的更改内容。这使得编写应用更加容易，但是这样使得你对React内部具体实现并不了解，这篇文章介绍了在React的"diffing"算法中我们所作出地决择，以使得组件的更新是可预测的并且可以适用于高性能应用。

## 动机

当你使用React的时候，在任何一个单点时刻你可以认为`render()`函数的作用是创建React元素树。当state或者props更新的时候，`render()`函数将会返回一个不同的React元素树。接下来React将会找出如何高效地更新UI来匹配最近时刻的React元素树。

目前存在大量通用的方法能够以最少的操作步骤将一个树转化成另外一棵树。然而，[state of the art algorithms](http://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf)的时间复杂度为O(n<sup>3</sup>)，其中n为树中的元素个数。

如果你在React中展示1000个元素，那么每次更新都需要1百万次的比较，这样的代价过于昂贵。然而，React基于以下两个假设实现了时间复杂度为O(n)的算法:

1. 不同的两个元素会产生不同的树。
2. 开发者通过`key`属性可以在不同的渲染中那些元素是相同的。

事实上，这些假设对于大部分实例都是有效的。

## Diffing 算法

当React比较(diffing)两棵树时，React首先比较两棵树的根元素。根据根元素的不同，行为也有所不同。

### Elements Of Different Types

无论什么时候，当树的根节点类型不同时，React将会销毁原先的树并重写构建新的树。从`<a>`到`<img>`，从`<Article>`到`<Comment>`，从`<Button>`到`<div>`，这都导致重新构建。

当销毁原先的树时，之前的DOM节点将销毁。实例组件执行`componentWillUnmount()`。当构建新的一个树，新的DOM元素将会插入DOM中。组件将会执行`componentWillMount()`以及`componentDidMount()`。与之前旧的树相关的状态都会丢失。

树的根节点以下的任何组件都会被卸载(unmount)，其状态(state)都会丢失。例如，当比较:

```xml
<div>
  <Counter />
</div>

<span>
  <Counter />
</span>
```

`Counter`将会被销毁，重新赋予新的实例。

### DOM Elements Of The Same Type

When comparing two React DOM elements of the same type, React looks at the attributes of both, keeps the same underlying DOM node, and only updates the changed attributes. For example:

```xml
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

By comparing these two elements, React knows to only modify the `className` on the underlying DOM node.

When updating `style`, React also knows to update only the properties that changed. For example:

```xml
<div style={{'{{'}}color: 'red', fontWeight: 'bold'}} />

<div style={{'{{'}}color: 'green', fontWeight: 'bold'}} />
```

When converting between these two elements, React knows to only modify the `color` style, not the `fontWeight`.

After handling the DOM node, React then recurses on the children.

### Component Elements Of The Same Type

When a component updates, the instance stays the same, so that state is maintained across renders. React updates the props of the underlying component instance to match the new element, and calls `componentWillReceiveProps()` and `componentWillUpdate()` on the underlying instance.

Next, the `render()` method is called and the diff algorithm recurses on the previous result and the new result.

### Recursing On Children

By default, when recursing on the children of a DOM node, React just iterates over both lists of children at the same time and generates a mutation whenever there's a difference.

For example, when adding an element at the end of the children, converting between these two trees works well:

```xml
<ul>
  <li>first</li>
  <li>second</li>
</ul>

<ul>
  <li>first</li>
  <li>second</li>
  <li>third</li>
</ul>
```

React will match the two `<li>first</li>` trees, match the two `<li>second</li>` trees, and then insert the `<li>third</li>` tree.

If you implement it naively, inserting an element at the beginning has worse performance. For example, converting between these two trees works poorly:

```xml
<ul>
  <li>Duke</li>
  <li>Villanova</li>
</ul>

<ul>
  <li>Connecticut</li>
  <li>Duke</li>
  <li>Villanova</li>
</ul>
```

React will mutate every child instead of realizing it can keep the `<li>Duke</li>` and `<li>Villanova</li>` subtrees intact. This inefficiency can be a problem.

### Keys

In order to solve this issue, React supports a `key` attribute. When children have keys, React uses the key to match children in the original tree with children in the subsequent tree. For example, adding a `key` to our inefficient example above can make the tree conversion efficient:

```xml
<ul>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>

<ul>
  <li key="2014">Connecticut</li>
  <li key="2015">Duke</li>
  <li key="2016">Villanova</li>
</ul>
```

Now React knows that the element with key `'2014'` is the new one, and the elements with the keys `'2015'` and `'2016'` have just moved.

In practice, finding a key is usually not hard. The element you are going to display may already have a unique ID, so the key can just come from your data:

```js
<li key={item.id}>{item.name}</li>
```

When that's not the case, you can add a new ID property to your model or hash some parts of the content to generate a key. The key only has to be unique among its siblings, not globally unique.

As a last resort, you can pass item's index in the array as a key. This can work well if the items are never reordered, but reorders will be slow.

## Tradeoffs

It is important to remember that the reconciliation algorithm is an implementation detail. React could rerender the whole app on every action; the end result would be the same. We are regularly refining the heuristics in order to make common use cases faster.

In the current implementation, you can express the fact that a subtree has been moved amongst its siblings, but you cannot tell that it has moved somewhere else. The algorithm will rerender that full subtree.

Because React relies on heuristics, if the assumptions behind them are not met, performance will suffer.

1. The algorithm will not try to match subtrees of different component types. If you see yourself alternating between two component types with very similar output, you may want to make it the same type. In practice, we haven't found this to be an issue.

2. Keys should be stable, predictable, and unique. Unstable keys (like those produced by `Math.random()`) will cause many component instances and DOM nodes to be unnecessarily recreated, which can cause performance degradation and lost state in child components.
