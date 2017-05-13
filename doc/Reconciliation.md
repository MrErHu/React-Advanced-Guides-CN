# 一致化处理(Reconciliation)

原文: [Reconciliation](https://facebook.github.io/react/docs/reconciliation.html)

翻译: [MrErHu(请叫我王磊同学)](https://github.com/MrErHu)

邮箱: [wanglei_cs@163.com](mailto:wanglei_cs@163.com)

React提供声明式API，因此在每次更新中你不需要关心具体的更改内容。这使得编写应用更加容易，但是这样使得你对React内部具体实现并不了解，这篇文章介绍了在React的"diffing"算法中我们所作出地决择，以使得组件的更新是可预测的并且可以适用于高性能应用。

## 动机

当你使用React的时候，在任何时刻，你可以认为`render()`函数的作用是创建React元素树。当state或者props更新的时候，`render()`函数将会返回一个不同的React元素树。接下来React将会找出如何高效地更新UI来匹配先前的React元素树。

目前存在大量通用的方法能够以最少的操作步骤将一个树转化成另外一棵树。然而，[state of the art algorithms](http://grfia.dlsi.ua.es/ml/algorithms/references/editsurvey_bille.pdf)的时间复杂度为O(n<sup>3</sup>)，其中n为树中的元素个数。

如果我们在React中展示1000个元素，那么每次更新都需要1百万次的比较，这样的代价过于昂贵。然而，React基于以下两个假设实现启发式算法，使得时间复杂度为O(n):

1. 不同的两个元素会产生不同的树。
2. 开发者通过`key`属性可以在不同的渲染中表示那些元素是相同的。

事实上，这些假设对于大部分实例都是有效的。

## Diffing 算法

当React比较(diffing)两棵树时，React首先比较两棵树的根元素。根据根元素的不同，行为也有所不同。

### 元素类型不相同

无论什么时候，当树的根节点类型不同时，React将会销毁原先的树并重头构建新的树。从`<a>`到`<img>`，从`<Article>`到`<Comment>`，从`<Button>`到`<div>`，这都导致重新构建。

当卸载原先的树时，之前的DOM节点将销毁。实例组件执行`componentWillUnmount()`。当构建新的一个树，新的DOM元素将会插入DOM中。组件将会执行`componentWillMount()`以及`componentDidMount()`。与之前旧的树相关的状态都会丢失。

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

### DOM元素类型相同

当React比较两个React DOM元素是相同类型时，React观察两者属性，保持底层DOM节点相同，并且仅更新已经改变的属性，例如:

```xml
<div className="before" title="stuff" />

<div className="after" title="stuff" />
```

通过比较两个元素，React会仅修改底层DOM节点的`className`属性。

当更新`style`属性，React也会仅仅只更新已经改变的属性，例如:

```xml
<div style={{color: 'red', fontWeight: 'bold'}} />

<div style={{color: 'green', fontWeight: 'bold'}} />
```

当React对两个元素进行转化的时候，仅会修改`color`，而不会修改`fontWeight`。

在处理完当前DOM节点后，React会递归处理子节点。

### 相同类型的组件

当一个组件更新的时候，组件实例保持不变，以便在渲染过程中保存state。React会更新组件实例的属性来匹配新的元素，并在元素实例上调用`componentWillReceiveProps()` and `componentWillUpdate()`。

接下来，`render()`方法会被调用并且`diff`算法对上一次的结果和新的结果进行递归。

### 子元素递归

当React递归DOM元素的子节点时，默认地在同一时刻迭代这两个子元素列表，并在有差异时生成一个变量。

例如，当给子元素末尾添加一个元素，在两棵树之间转化中性能就不错:

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

React会比较`<li>first</li>`树与`<li>second</li>`树，然后插入`<li>third</li>`树。

如果在开始处插入一个节点也是这样简单地实现，那么性能将会很差。例如，在下面两棵树的转化中性能就不佳。

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

如果React改变每一个元素，而不是知道应该保持`<li>Duke</li>`和`<li>Villanova</li>`子树，那么性能将是很大的问题。

### Keys

为了解决这个问题，React支持`key`属性。当组件拥有keys时，React用key比较原始树子节点与后续树子节点。如下所示，添加`key`属性给我们的低效率实例代码可以使得两个树的转化更加高效。

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

现在React知道拥有key属性为`2014`节点是新的。key为`2015`和`2016`的两个元素仅仅只是被移动而已。

事实上，查找一个key属性并不困难。你所将要展示的组件一般都有唯一的ID，因此你的数据可以作为key的来源。

```js
<li key={item.id}>{item.name}</li>
```

当情况不同时，你可以添加一个新的ID属性给你的model或者部分内容的hash值来作为key。key仅仅只需要在其兄弟节点中是唯一的，并非全局唯一。

作为最后一种方案，你可以将元素的下标作为key属性。如果元素永远不会被重新排序的情况下这样也是不错的，但是如果存在重新排序，性能将会很差。

## 折衷

需要记住的是一致化算法(reconciliation algorithm)仅仅只是一个实现细节。React会在每个操作上重新渲染整个应用，最终的结果可能是相同的。我们经常细化启发式算法，以便优化性能。

在最近的实现中，你能确定子树会在兄弟节点中移动，但你不能确定它可以移动到别的地方去。算法会重新渲染整个树。

因为React依赖于启发式算法，如果下面的假设没有实现，性能将会大大的损失。

1. 算法不会尝试匹配不同节点类型的子树。如果你发现在有输出类似,但两个节点类型不同，你可能需要将其转化成同种类型，事实上，我们没有在其中发现问题。

2. keys应该是稳定的、可预测的并且是唯一的。不稳定的key(类似于`Math.random()`函数的结果)可能会产生非常多的组件实例并且DOM节点也会非必要性的重新创建。这将会造成极大的性能损失和组件内state的丢失。
