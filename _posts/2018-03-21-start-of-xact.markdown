---
layout: post
title:  "从零开始写一个类React框架"
date:   2018-03-21 12:54:05 +0800
categories: Tech blog
---


> 千里之行始于足下

# 前言
React是当前比较火的几个前端框架之一。React引入了很多的新概念，比如Virtual DOM, JSX, Patial Update等等。我们可以通过查看React官方文档，教程和源代码等方式学习React。本文试图从另外一个全新的角度去学习React的工作原理，即模仿React实现类React前端框架xact。

# 正文
本文是“从零开始写一个类React框架”第一篇，主要总结如何实现xact.createElement方法和xactDOM.render方法。本文最终实现了xact渲染JSX树。

### 相关概念
*JSX*: 首先我们需要先了解什么是JSX。JSX其实是一个语法糖，能够减少前端开发者和UI设计者之间的沟通难度。[Introducing JSX](https://reactjs.org/docs/introducing-jsx.html)简单介绍了JSX是什么及其使用方法。JSX在外观上像HTML,通过Babel转译后会变成如下形式：

```
const element = (
  <h1 className="greeting">
    Hello, world!
  </h1>
);
```
```javascript
const element = React.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
在上面的代码所示中，常量element最终就是React.createElement方法的一个返回值。babel默认将JSX转换成React.createElement()的返回值。我们可以通过设置babel文件，使其返回为：
```
const element = xact.createElement(
  'h1',
  {className: 'greeting'},
  'Hello, world!'
);
```
那么，我们需要实现xact的createElement去返回一个xact element.
xact element是一个object。它被用于描述DOM树，亦即虚拟DOM树。通过createElement返回的object示例如下：

```javascript
const element = (
  <div>
    Hello world
    <img src="/public/images/hello.jpeg" alt="hello" cv-data="hello"/> 
    <span>
      lalal
      <div>fjfjf</div>
    </span>
    Something Behind
  </div>
) 
```

```javascript
{"type":"div","props":{"children":[{"type":"TEXT_NODE","props":{"nodeValue":"Hello world"}},{"type":"img","props":{"src":"hello.png","alt":"hello","cv-data":"hello"}},{"type":"span","props":{"children":[{"type":"TEXT_NODE","props":{"nodeValue":"lalal"}},{"type":"div","props":{"children":[{"type":"TEXT_NODE","props":{"nodeValue":"fjfjf"}}]}}]}},{"type":"TEXT_NODE","props":{"nodeValue":"Something Behind"}}]}}
```

createElement方法逻辑如下： 
```javascript
function createElement(tag, attributes, ...args) {
  let element = {};
  let props = {};
  if(args.length > 0) {
    args.forEach((item, index) => {
      let node = null;
      if(typeof item === "string"){
        node = {
          type: "TEXT_NODE",
          props: {
            nodeValue: item
          }
        }
      } else {
        node = item;
      }

      props.children?props.children.push(node):props.children=[node];
    })
  }
  if(attributes !== null) {
    Object.keys(attributes).forEach(key=>{
      props[key] = attributes[key];
    })
  }
  element.type = tag;
  element.props = props;
  return element;
}
```

### 渲染Virtual DOM
现在，我们通过createElement返回了Virtual DOM，接下来我们需要渲染所得到的Virtual DOM。本文是本系列的第一篇，所以xactDOM.render方法只是负责把Virtual DOM渲染至HTML上，并不会比较Virtual DOM并做部分渲染。

render的内部逻辑就是根据type创建dom object, 然后在根据props里的property判断其为attributes, event handler还是子object。然后递归地将Virtual DOM append上我们的root dom object。

本文的render方法逻辑如下：

```javascript
function render(element, domContainer) {
  let children = [];
  let parentNode = domContainer || document.querySelector("body");
  let currentNode = null;
  if(element.type === "TEXT_NODE") {
    currentNode = document.createTextNode(element.props.nodeValue);
  } else {
    currentNode = document.createElement(element.type);
    let props = element.props || {};

    Object.keys(props).forEach(key => {
      if (/on/.test(key)) {
        const event = key.substring(2);
        currentNode.addEventListener(event, props[key])
      } else if(key !== "children") {
        currentNode.setAttribute(key, props[key]);
      } else {
        if(element.props.children.length > 0) {
          children = element.props.children;
          children.forEach(child => {
            render(child, currentNode);
          })
        }
      }
    });

  }
  domContainer.appendChild(currentNode);
}
```

# 结论
通过本文的createElement和render方法，可以将React官网上[Rendering Elements](https://reactjs.org/docs/rendering-elements.html)介绍的示例成功渲染至HTML。 本文的code可以在我的[gitbut](https://github.com/jcdby/xact)查看。

# 未来工作
实现虚拟树比较方法， 状态更新通知方法，组件周期等更多高级算法。