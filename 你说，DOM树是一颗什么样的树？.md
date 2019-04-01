# 你说，DOM树是一颗什么样的树？

> 引子：讲起浏览器的渲染就会提到DOM树的生成，讲到DOM树，或许你脑海里会浮现一颗以html元素为根的树。但DOM树究竟是个怎样的存在？DOM树的构成过程是怎样的？了解DOM树的构造过程会带来怎样的启发？

## 1. HTML5标准

假设你是一个浏览器，给你一个 [HTML MIME 类型](https://www.w3.org/TR/html5/infrastructure.html#html-mime-type)的文档，你会怎么去理解这个文档，并且提取出你要的信息呢？私以为编程语言也是语言，就像你读一篇英语文章，想要从中提取信息，就得先知道英语的词法、语法等的约定。w3c就给HTML定义了所谓的[词法、语法](https://www.w3.org/TR/html5/syntax.html#writing-html-documents)，并且告诉用户代理[解析的规则](https://www.w3.org/TR/html5/syntax.html#parsing-html-documents)。

## 2. DOM树是个啥

你说长在你脑海里的那颗树上有啥信息？节点+关系。

节点信息可以存在一个对象里，关系就是指针指来指去。

## 3. 浏览器如何生成一颗DOM树

不同的浏览器内核针对标准规定的解析规则，有着不同的实现。以chrome为例子，[这是一篇参考好文](https://www.cnblogs.com/yincheng/p/build-dom.html)。

解析的流程按照HTML标准走：

![解析模型](https://haitao.nos.netease.com/0aecc65c-3665-43ec-af3d-6ba0520b932c_736_906.jpg)

关键步骤：生成tokens->通过构造器根据tokens构建节点及其关系信息。

1）生成tokens，是一个根据标准遍历拿信息的过程，最终会生成这样子的信息：

![token信息](https://haitao.nos.netease.com/495202a9-e293-43d6-af8f-f7ab443e44e5_1418_866.png)

2）构建节点

- 针对不同的类型的节点，使用不同的构造器实例化。构造器内通过指针指向来确定父子兄弟关系。
- 构造器间通过继承和组合实现代码的复用。

3）关系的确认

- 利用栈建立父子关系，容错机制
- 借助上一次的lastChild建立起兄弟关系

## 4. Vue的虚拟dom 

存在的意义，减少dom解析流程中script execution造成dom树重构的频次。

1）create

- 使用html语法规则解析模版，parse成AST ；做一些优化，标记静态节点和根，减少patch的dom数量；codegen生成可执行代码，将AST转化成虚拟dom


- Vue的VNode用来映射真实dom的渲染，去除了包含操作dom的方法，在src/core/vdom/vnode.js中定义，相对真实的dom元素轻量了许多。用createElement方法根据tag类型使用不同的构造器创建VNode、用createTextVNode创建文本VNode、用createEmptyVNode创建空的VNode，.etc，最终形成一个js对象

2）diff

详细过程：[解析vue2.0的diff算法](https://github.com/aooy/blog/issues/2)

- 比较只会在同层级进行, 不会跨层级比较。
- 最大化程度的节点复用，key
- 两节点值得比较时，如果两节点都有子节点，并且不一样时，从新老节点的首尾开始遍历。是因为首尾是dom变更较多的地方？

## 5. 启发

虚拟dom的生成和DOM树的生成，思想上类似。了解DOM树的构造过程，一是可以借鉴其思想，比如构造器的抽象方式，父子关系的确认用到了栈，就很妙；二是根据其过程，可以知道优化的方向，虚拟dom的存在减少dom解析流程中script execution造成dom树重构的频次。



#### 参考：

https://www.w3.org/TR/html5/syntax.html

https://www.cnblogs.com/yincheng/p/build-dom.html

https://www.w3cplus.com/javascript/dom-tree-and-traversals.html

https://github.com/aooy/blog/issues/2

