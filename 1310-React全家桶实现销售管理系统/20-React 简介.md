---
show: step
version: 1.0
enable_checker: true
---

# React 简介

## 一、实验介绍

#### 1.1 实验内容

在本节中，我们将会介绍 React、React-router、Redux 的基础知识，通过对基础知识的学习，后续的实验中我们会更加的得心应手。

#### 1.2 实验知识点

+ React
+ React-router
+ Redux

#### 1.3 实验环境

+ `Theia`:  一款前后端分离的、基于 web 的 云 IDE。
+ `Mini Browser`：浏览器，右键点击目标 html 文件，选择 open with 下的 Mini Browser 即可在 IDE 中打开浏览器。

#### 1.4 适合人群

本课程难度一般，适合具有 React 基础的用户，熟悉 React 的基础知识，通过学习 React 的全家桶的使用以及 express 框架和 mongoDB 的简单使用，完成一个完整的 React 项目。

## 二、实验原理

下面我们来讲解基础知识。

### 2.1 React

React 是一个用来构建用户界面的 JavaScript 库，准确的来说，它不是一个框架，它起源于 Facebook 的一个内部项目，在 2013 年 5 月开源，目前在 github 上已经有超过 100000 的 stars。React 最出彩的地方之一就是 components，你可以创建各种各样的组件，来快速有效的构建用户页面；另外，React 还通过使用 state 和 prop 简化了数据的存储和处理，方便了数据的处理。

+ 安装配置

React 安装方式有多种，在本项目中，我们使用的是 create-react-app 脚手架的方式来构建一个基础的 React 项目。在 React 的学习中，我们推荐使用静态 CDN 的方式引入，这种方式和引入 jQuery 的方式相同，通过在头文件内加入：

```html
<script src="https://unpkg.com/react@16/umd/react.development.js"></script>
<script src="https://unpkg.com/react-dom@16/umd/react-dom.development.js"></script>
<script src="https://unpkg.com/babel-standalone@6.15.0/babel.min.js"></script>
```

+ JSX

JSX 是 React 组件内部构建标签的类 XML 语法。它并不是一门语言，只是语法糖，必须借助 React 才能运行。

*基础语法*

1. JSX 命名组件时首字母需要大写，原因是 React 会以首字母的大小写来区别组件和 html 元素。
2. JSX 的 return 中只能返回一个顶层标签，不能返回多个。
3. 解析器对 JSX 是这样工作的：遇到 < ,JSX 就当 HTML 解析，遇到 { ， JSX 就当 JavaScript 解析。
4. 我们可以使用 {} 引用用户输入的值而不必担心 xss 攻击，因为 {} 中的内容会被渲染为字符串。

+ props & state

在 react 中，父子组件的通讯方式是通过父组件给子组件传 props 实现的，props 是一个从外部传进组件内部的参数，主要作用就是为父组件到子组件的数据传递提供参数通道；state 同样也是对数据进行管理，只不过 props 侧重于传输这个过程，侧重于组件与组件之间的传递，state 侧重于一个组件内部的数据状态。在组件中，props 是只读的，state 是被组件维护，可以任意修改的。

+ 生命周期

![图片描述](https://doc.shiyanlou.com/courses/uid920932-20190426-1556268341197)

### 2.2 React-router

React Router 是一个基于 React 之上的强大路由库，它可以让你向应用中快速地添加视图和数据流，同时保持页面与 URL 间的同步。React Router 知道如何为我们搭建嵌套的 UI，因此我们不用手动找出需要渲染哪些 <Child> 组件。

以下是一个简单的路由配置的例子：

```js
import React from 'react'
import { Router, Route, Link } from 'react-router'

const App = React.createClass({
  render() {
    return (
      <div>
        <h1>App</h1>
        <ul>
          <li><Link to="/about">About</Link></li>
          <li><Link to="/inbox">Inbox</Link></li>
        </ul>
        {this.props.children}
      </div>
    )
  }
})

const About = React.createClass({
  render() {
    return <h3>About</h3>
  }
})

const Inbox = React.createClass({
  render() {
    return (
      <div>
        <h2>Inbox</h2>
        {this.props.children || "Welcome to your Inbox"}
      </div>
    )
  }
})

const Message = React.createClass({
  render() {
    return <h3>Message {this.props.params.id}</h3>
  }
})

React.render((
  <Router>
    <Route path="/" component={App}>
      <Route path="about" component={About} />
      <Route path="inbox" component={Inbox}>
        <Route path="messages/:id" component={Message} />
      </Route>
    </Route>
  </Router>
), document.body)
```

通过上面的路由配置，这个 app 知道如何渲染下面四个 url：

| URL | Components |
|----------|:-------------:|
| / | App |
| /about | App -> About |
| /inbox | App -> Inbox |
| /inbox/messages/:id | App -> Inbox -> Message |


使用 React-router 的时候，我们只需要在渲染的时候，本来是渲染的组件，现在渲染 `<Router />` 组件即可，而这个组件的使用需要传入的 props 包含路径和指向的组件。

### 2.3 Redux

Redux 是 JavaScript 状态容器，提供可预测化的状态管理，它是 React 全家桶的重要一员。

在 React 中，数据的传递主要使用 state 和 props，React 的数据流是单一的，由顶层至底层的，props 是由父组件传递下来的，而 state 是组件自己维护的状态，组件只能在自身中维护或者将 state 作为 props 传递给子组件，也就是说，React 的数据只能单向向下分发，无法进行数据的回溯。组件间进行通信的唯一方法就是嵌套，父组件可以将 state 作为 props 传递给子组件，子组件可以触发回调函数来改变父组件的状态，但是如果没有这种嵌套关系，React 中的数据交互似乎就显得捉襟见肘了，该如何解决呢？Redux 就应运而生了。Redux 的核心思想就是将所有的数据集中到顶层组件中，一层一层的往下分给各个子组件，子组件既可以调用这些数据，也可以通过调用方法来改变这些数据。Redux 使 React 的状态变得更加可预测，更容易管理。

我们来简单介绍一下 redux 的三大原则和基本原理：

1. 单一数据源
  
    整个应用的 state 被储存在一棵 object tree 中，并且这个 object tree 只存在于唯一一个 store 中。对 state 的读取和修改都是通过这棵唯一的 object tree 中，这样保证了数据源的单一性，也使我们的数据流更加清晰。

2. state 只读性

    在 redux 中，改变 state 只能通过触发 action 来改变，action 是一个用于描述已发生事件的普通对象。通过 action，我们可以实现 state 的集中化处理，不用担心竞态条件的出现。

3. 对于 state 的修改，使用纯函数进行。

    为了描述 action 如何改变 state tree，我们需要编写 reducers。Reducers 里包含只有一些纯函数，这些函数接收先前状态的 state 和 action，然后返回一个新的 state，注意，这里不会对先前的 state 造成任何的影响。

以上就是 redux 的三大原则，这三大原则既是 redux 的工作原理，也是 redux 能保证 state 更好的被管理的必要条件。下面，我们分别对 store、action、reducer 作详细讲解。

+ action

    action 是把数据从 app(数据的来源有很多种，可能是来自服务器的响应，可能是来自用户的输入等等) 传到 store 的介质，且它是 store 数据的唯一来源，一般情况下，我们会通过使用 `store.dispatch()` 把 action 传到 store。

    ```js
    const TO_DO = 'TO_DO';

    {
        type:TODO,
        text:'redux test'
    }
    ```

    上面是一个 action 的简单示例，实际的 action 写法与上述代码有些差距，action 的本质是 js 普通对象。我们约定，action 内必须使用一个字符串类型的 type 来表示这个 action 将要执行的动作。在项目比较大的时候，我们一般选择将 action 分模块存放，本项目就是采取这种方式。React-redux 中创建的 action 我们可以通过 dispatch()来调用，除此之外，我们还可以通过 react-redux 提供的 connect() 帮助器来调用。

+ reducers

    Reducers 指定了应用状态的变化如何响应 action 并将其发送到 store 的，action 只是描述了有事情发生，和变化的数据，并没有指出如何操作 state，对于 state 的操作，是在 reducers 中进行的。

    还记得我们在三大原则中提到 reducer 做了什么工作吗？reducer 接收先前的状态和 action，返回一个新的 state，返回的新的 state 我们需要通过 `Object.assign()` 来实现，并且不能把传入的 state 作为第一个参数，如：

    ```js
    function fn(state,action){
    //其它一些语句   
        return Object.assign(state,action.changed);//不能这样返回新的 state，state 不可以被改变。
    }
    ```

    我们需要使用以下的方式来返回一个新的 state：

    ```js
    return Object.assign({},state,{changed:action.changed});
    ```

    我们通过使用 `Object.assign()`，返回一个新的 state 对象，这个新的对象是通过旧的 state 和触发的 action 计算出来的。

    *注*：默认情况下或是未知的 action 触发时，我们需要返回旧的 state。

+ store

    store 把 reducers 和 action 联系到一起，它维护了一棵应用的 state tree，提供了方法可以对 state 进行读和更新，Redux 应用只有一个 store。action 定义好事情的发生和 state，store 触发这些传入的 action，然后调用 reducers 更新 state，这就是我们 redux 的工作流程。


## 三、总结

在本节中，我们简单学习了 React 全家桶的基础知识：

+ React
+ React-router
+ Redux

通过对基础知识的学习，后续的实验会更加的得心应手。如果想要深入的学习 React 全家桶的知识，请通过官方文档进行学习。