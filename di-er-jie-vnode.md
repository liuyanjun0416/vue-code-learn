---
description: VNODE
---

# 第二节 -- VNODE

vue 中构建了一个虚拟dom系统。每个dom由javascript中的数据去代替真实dom，这个数据就是对应的虚拟dom，每个虚拟dom的虚拟节点就是VNODE。

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
<div id="demo">
  <p class="text">hello</p>
</div>
```

转化为vdom:

```javascript
new VNode('div',{attrs:{id:'demo'}},
  new VNode('p',{class:'text'}),undefined,'hello')
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

### 生成一个新的VNode的方法

#### createEmptyVNode 创建一个空VNode节点

```javascript
/*创建一个空VNode节点*/
export const createEmptyVNode = () => {
  const node = new VNode()
  node.text = ''
  node.isComment = true
  return node
}
```



