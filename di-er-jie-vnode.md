---
description: VNODE
---

# 第二节 -- VNODE

vue 中构建了一个虚拟dom系统。每个dom由javascript中的数据去代替真实dom，这个数据就是对应的虚拟dom，每个虚拟dom的虚拟节点就是VNODE。正是因为有Virtual DOM是以Javascript为基础的，这使得跨平台变成变得更加容易，使用者只需要在操作层新增对应平台的api，即可实现各个平台的使用，如现在的Weex或可扩展pixi等3d引擎等。

首先看下VNODE的源码是怎样的：

{% code-tabs %}
{% code-tabs-item title="core/vdom/vnode.js" %}
```typescript
export default class VNode {
  tag: string | void;
  data: VNodeData | void;
  children: ?Array<VNode>;
  text: string | void;
  elm: Node | void;
  ns: string | void;
  context: Component | void; // rendered in this component's scope
  key: string | number | void;
  componentOptions: VNodeComponentOptions | void;
  componentInstance: Component | void; // component instance
  parent: VNode | void; // component placeholder node

  // strictly internal
  raw: boolean; // contains raw HTML? (server only)

  isStatic: boolean; // hoisted static node
  isRootInsert: boolean; // necessary for enter transition check
  isComment: boolean; // empty comment placeholder?
  isCloned: boolean; // is a cloned node?
  isOnce: boolean; // is a v-once node?
  asyncFactory: Function | void; // async component factory function
  asyncMeta: Object | void;
  isAsyncPlaceholder: boolean;
  ssrContext: Object | void;
  fnContext: Component | void; // real context vm for functional nodes
  fnOptions: ?ComponentOptions; // for SSR caching
  fnScopeId: ?string; // functional scope id support
  constructor (
    tag?: string,
    data?: VNodeData,
    children?: ?Array<VNode>,
    text?: string,
    elm?: Node,
    context?: Component,
    componentOptions?: VNodeComponentOptions,
    asyncFactory?: Function
  ) {
    this.tag = tag   //当前节点的标签
    this.data = data  //当前节点的对象，包含具体的一些数据
    this.children = children //当前节点的子节点，可以为数据
    this.text = text  //当前节点的文本
    this.elm = elm  //当前节点的真实dom节点
    this.ns = undefined  //当前节点的命名空间(namespace)
    this.context = context  //当前节点的上下文(编译作用域)
    this.fnContext = undefined  //函数组件作用域
    this.fnOptions = undefined
    this.fnScopeId = undefined
    this.key = data && data.key //节点的key属性，用于优化性能
    this.componentOptions = componentOptions //组件options选项
    this.componentInstance = undefined //当前节点对应的组件实例
    this.parent = undefined //当前节点的父节点
    this.raw = false   //是否为原生HTML或只是普通文本
    this.isStatic = false   //是否为静态节点
    this.isRootInsert = true  //是否根节点
    this.isComment = false //是否是注释节点
    this.isCloned = false //是否是克隆节点
    this.isOnce = false  //是否有v-once指令
    this.asyncFactory = asyncFactory  
    this.asyncMeta = undefined
    this.isAsyncPlaceholder = false
  }
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

因此可以简单的将一个dom，如：

```markup
<div id="demo" v-show="isShow">
  <p class="text">hello</p>
</div>
```

转化为vdom:

```javascript
new VNode('div',
  {
    attrs:{
      id:'demo'
    },
    directives: [
      {
        /* v-show指令 */
        rawName: 'v-show',
        expression: 'isShow',
        name: 'show',
        value: true
     }
   ]
  },
  new VNode('p',{staticClass:'text'}),undefined,'hello')
)
```

VNodeData的类型如下:

{% code-tabs %}
{% code-tabs-item title="types/vnode.d.ts" %}
```typescript
export interface VNodeData {
  key?: string | number;
  slot?: string;
  scopedSlots?: { [key: string]: ScopedSlot };
  ref?: string;
  tag?: string;
  staticClass?: string;
  class?: any;
  staticStyle?: { [key: string]: any };
  style?: object[] | object;
  props?: { [key: string]: any };
  attrs?: { [key: string]: any };
  domProps?: { [key: string]: any };
  hook?: { [key: string]: Function };
  on?: { [key: string]: Function | Function[] };
  nativeOn?: { [key: string]: Function | Function[] };
  transition?: object;
  show?: boolean;
  inlineTemplate?: {
    render: Function;
    staticRenderFns: Function[];
  };
  directives?: VNodeDirective[];
  keepAlive?: boolean;
}
```
{% endcode-tabs-item %}
{% endcode-tabs %}

接下来对Vnode进行封装对应的操作方法。

* 生成一个新的VNode的方法

createEmptyVNode 创建一个空VNode节点

```javascript
/*创建一个空VNode节点*/
export const createEmptyVNode = () => {
  const node = new VNode()
  node.text = ''
  return node
}
```

* 创建一个文本节点

```javascript
export const createTextNode = () => {
  return new VNode(undefined,undefined,undefined,String(val))
}
```

* 克隆一个VNode节点

```javascript
function cloneVNode (vnode) {
  var cloned = new VNode(
    vnode.tag,
    vnode.data,
    vnode.children,
    vnode.text,
    vnode.elm,
    vnode.context,
    vnode.componentOptions,
    vnode.asyncFactory
  );
  cloned.ns = vnode.ns;
  cloned.isStatic = vnode.isStatic;
  cloned.key = vnode.key;
  cloned.isComment = vnode.isComment;
  cloned.fnContext = vnode.fnContext;
  cloned.fnOptions = vnode.fnOptions;
  cloned.fnScopeId = vnode.fnScopeId;
  cloned.isCloned = true;
  return cloned
}
```

### 跨平台

上面说到，正是因为使用Virtual DOM的原因，使得vue拥有跨平台的能力，这里跨平台可以是跨到canvas，native等。这里通过一个适配层去适配对应的操作。

```javascript
export function createElement(tagName,vnode){
  let elm
  if(platform === "web"){
    elm = document.createElement(tagName);
  }else if(platform === "weex"){
    elm = nativeFunc.createElement(tagName); //nativeFunc为对应原生交互操作方法
  }
  return elm
}

export function createTextNode(text){
}

export function insertBefore(node,target,before){}

export function removeChild(node,child){}

export function appendChild(node,child){}

...
```

因此，在上面的适配层中，我们可以根据不同平台来执行不同的API方法。这样既可实现跨不同平台。

### 总结

Vue通过使用Virtual DOM可以实现跨平台，并且通过Virtual DOM进行Diff比较以及Patch机制来控制View层的变化，这样可以最大的减少对View层的变动，减少DOM操作，减少性能消耗。而VNODE则是Virtual DOM中的一个节点，这一个节点即是DOM节点的简化。

