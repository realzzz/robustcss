
> 原文地址 [8 simple rules for a robust, scalable CSS architecture](https://github.com/jareware/css-architecture) 本文仅仅是对原文的中文翻译, 所有权利归原作者 [Jarno Rantanen](https://github.com/jareware). 
> Original address [8 simple rules for a robust, scalable CSS architecture](https://github.com/jareware/css-architecture). This article is only the Chinese translation, all the rights still belong to original author [Jarno Rantanen](https://github.com/jareware). 


这些是我这些年web开发中从各种大型的复杂的web项目里学到的，已经有很多次别人问我这些知识，所以我觉得是时候把这些心得写成文档了。 

我会尽量保持解释的简洁，会涵盖以下几个必要的点：

1. [**尽可能使用类**](#1-优先使用类)
1. [**按类存放组件代码**](#2-按类存放组件代码)
1. [**保持类的命名空间一致性**](#3-保持类的命名空间一致性)
1. [**文件名和命名空间保持一致性**](#4-文件名和命名空间保持一致性)
1. [**防止样式泄漏到组件之外**](#5-防止样式泄漏到组件之外)
1. [**防止组件内的样式泄漏**](#6-防止组件内的样式泄漏)
1. [**尊重组件的边界**](#7-尊重组件的边界)
1. [**解耦外部的style依赖**](#8-解耦外部的style依赖)

## 简介

如果你从事前端工作的话，你早晚会遇到样式相关的工作. 虽然各种前端的技术日新月异，但CSS始终坚挺的是前端样式的唯一选择(早晚..各种本地应用也会使用的)。  总体来说，样式的工作方式有两种大类：

* CSS 预处理, 这些年一直使用这种方法 (比如 [SASS](http://sass-lang.com/), [LESS](http://lesscss.org/), 还有一些其他的)
* CSS嵌入JS代码模式, 这是最近才出来的 (例如 [free-style](https://github.com/blakeembrey/free-style), and [其他的一些选择](https://github.com/MicheleBertoli/css-in-js))

如何在以上的两种方式中做出选择是另一个话题了，但就像任何事物的两面性一样，这两种方式都有自己的优缺点，本文中我会更多的讨论第一种方式，如果你在使用第二种方式那么本文可能没那么有意思了.


## 主要目标


我们说的强健的，可扩展的CSS架构,  具体的说到底体现在什么地方呢？

* **面向组件** - 处理UI复杂性的最佳途径就是将UI拆分成小的组件.如果你已经在使用一个合理的框架，那么Javascript部分已经会是这样了.例如 [React](https://facebook.github.io/react/), 就鼓励高度的组件化和拆分化. 我们希望CSS也能够有这样的架构体系.
* **沙盒化** - 把UI拆分成各个不同的组件可能会引起各个组件之间样式的冲突问题. CSS 的基础特征例如 [层叠样式](https://developer.mozilla.org/en/docs/Web/Guide/CSS/Getting_started/Cascading_and_inheritance), 还有独立和全局的命名空间会让你出各种乱子. 如果你熟悉 [网页组件细则](https://developer.mozilla.org/en-US/docs/Web/Web_Components), 你可以把这当作 [影子DOM的样式独立](http://www.html5rocks.com/en/tutorials/webcomponents/shadowdom-201/) ，而且不用考虑浏览器支持 (无论这个细则是否得到支持). 
* **便捷性** - 我们需要这些美好的方法，但不是自己来实现. 我们不希望开发者因为使用了我们的架构而感到开发体验变差了. 我们需要尽一切可能让这体验更好。
* **确保安全** - 就像之前的观点提到的, 我们希望所有的事情都 *默认在本地*, 只有例外在全局. 我们工程师是一群懒骨头，总是会寻找最轻松的正确方式来完成任务。

## 详细的规则

### 1. 优先使用类

这只是在陈述一个显然的事实.

不要是用目标ID (例如 `#header`), 因为无论你有多确定只有一个实例, [在长远看来](https://twitter.com/stedwick/status/525777867146539009), 你都会被证明是错的. 我们曾经在一个大型应用上尝试找出所有数据绑定的问题，我们使用了2个UI，用同样的DOM, 同样的数据模型. 来确认所有的数据改动都能够被正确的在UI上显示，任何你以为唯一的组件（例如标题栏），都不在是唯一了, 这是个简单的例子能够证明假设唯一是不靠谱的。好吧，我有点偏题了，我只是想说，永远不会有情况是指向ID会被指向类更好的，永远！

同时你也不应该直接指向元素(例如 `p`).  通常情况下你可以直接指向属于某个组件的元素(详见下面), 但是对于组件本身而言，如果你这样做的化，你就早晚会需要去为不需要的组件[撤销样式](http://csswizardry.com/2012/11/code-smells-in-css/). 回到我们的主要目标上，这几乎违反了所有的目标( 非面向组件，未遵循样式层叠，成为了默认的值). 像字体，行高和颜色( 属于[继承属性](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance)) 这些body中的属性可以在你需要的时候成为例外，但是如果你严肃的对待组件隔离，这些也完全可以被放弃。(参考下面的 [解耦外部的style依赖](#8-integrate-external-styles-loosely))

所以除了极少的例外, 你的样式应该始终指向类.

### 2. 按类存放组件代码

当你开发或者使用一个组件的时候，假如所有关于这个组件的内容 - 它的Javascript, 样式，测试， 文档 - 都放在一起， 那会很有帮助。prominent

```
ui/
├── layout/
|   ├── Header.js              // 组件代码
|   ├── Header.scss            // 组件样式
|   ├── Header.spec.js         // 组件单元测试
|   └── Header.fixtures.json   // 测试代码的模拟数据 (如需要的话)
├── utils/
|   ├── Button.md              // 组件使用文档
|   ├── Button.js               // ..
|   └── Button.scss
```

当你使用这些代码的时候，只要打开项目的目录, 所有相关的组件文件都触手可得。 这些生成DOM的样式文件和Javascript有着很自然的联系, 所以有理由推断你不会只使用其中的某一个文件。 同理适用于组件和他的测试， 比如你可以把这个当作UI组件的[参考地点原则](https://en.wikipedia.org/wiki/Locality_of_reference).  我原来也一直一丝不苟地的维护着不同镜像下我的代码，在这些`styles/`, `tests/`, `docs/` 目录下， 直到我意识到这么做的唯一理由是我习惯那样. 

### 3. 保持类的命名空间一致性

CSS 为类名和其他标识(例如id, 动画名)准备的命名空间是单独的也是扁平的。 就像过去使用PHP的日子，开发社区已经适应了这个规则。通过使用更长的，结构化的名字来枚举名字空间(例子[BEM](http://getbem.com/))

例如， 我们使用的类名  `myapp-Header-link`，其中的3个部分都有具体的指代:

* `myapp` 首先区分于其他在相同DOM下的其他app
* `Header` 区分开了同个app下的其他组件组件
* `link`  为本地的样式指定了一个本地的名字(在组件的命名空间下)

作为一个特殊的例子，'Header'组件的根元素作为一个简单的组件可以直接被`myapp-Header` 类标记, 这些已经足够用了.

无论采用何种命名空间的方式， 请在项目中保持一致。 就像上面三个部分作为不同的*功能*，他们同样有明确的*意义*。只要看见这些类，你就知道他们属于哪里，命名空间本身就是一种对于项目的向导地图。

从这里开始我讲假定命名空间总是循序 '应用-组件-类' 的规则，我自己觉得这样很好，但你也可以有你自己的命名方式。 

### 4. 文件名和命名空间保持一致性

这仅仅是两个规则的合并(按类存放组件代码和保持类的命名空间一致性)， 所有的指定组件的样式都需要以这个组件命名，没有例外。

If you're working in the browser, and you spot a component that's misbehaving, you can right-\
如果你在浏览器里看到一个组件没有正确的显示，你可以右键点击它并查看， 例如:

```html
<div class="myapp-Header">...</div>
```

你知道组件名字后，就可以到编辑器里搜索了，“快速打开文件”, 输入名字"head", 你能看到

![quick-open-file.png](http://upload-images.jianshu.io/upload_images/1655500-720464c24cf3bd66.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

这样文件名严格的匹配组件名称非常有用，对于团队不熟悉架构的新人而言，TA可以轻松而自然的找到应该工作的代码。

一个很自然的（但不是立即）的推论: 一个单独的样式文件只能包含单独名字空间的样式. 为什么？比如我们有个登陆的表单，只用在`顶部`组件中. 在Javascript代码中被定义为小组件在`Header.js`中，而且不暴露在外。也许你会定义类名为`myapp-LoginForm`,同时把这个定义放入 `Header.js` 和 `Header.scss`. 但想象一下，如果一个新的项目成员加入后想登陆表单中的一个布局问题，他打开检查后会发现没有`LoginForm.js`和 `LoginForm.scss` ，那他只能通过`grep`来查找相关的代码。所以，login form需要不同的命名空间，那么就把他放到不同的控件中去。一致性在重大的项目中和金子一样重要。

### 5. 防止样式泄漏到组件之外

好了，我们建立了我们的命名空间规则，现在需要对UI控件沙盒化起来了。假如每个控件只使用他们命名空间开头的样式类，那么就可以保证样式不被泄漏到附近的控件中。 这是非常有效的(见下说明), 但不停的输入名字空间前缀也是一件繁琐的事情～

一个强健的，同时也机器简单的解决方案：把整个样式包裹到一个前缀区块内。 如下，我们只需要重复应用和组件名字一次:

```scss
.myapp-Header {
  background: black;
  color: white;

  &-link {
    color: blue;
  }

  &-signup {
    border: 1px solid gray;
  }
}
```

以上的例子来自SASS, 但是这个 `&`符号，也许让你惊讶，在所有其他的css预处理语言中都有一样的功能 ([SASS](http://sass-lang.com/), [PostCSS](https://github.com/postcss/postcss-nested), [LESS](http://lesscss.org/) and [Stylus](http://stylus-lang.com/))。完整的说，下面是SASS编译后的CSS:

```css
.myapp-Header {
  background: black;
  color: white;
}

.myapp-Header-link {
  color: blue;
}

.myapp-Header-signup {
  border: 1px solid gray;
}
```

所有的模式都能和这个契合，例如不同控件状态下的样式: (考虑 [Modifier in BEM terms](http://getbem.com/naming/))

```scss
.myapp-Header {

  &-signup {
    display: block;
  }

  &-isScrolledDown &-signup {
    display: none;
  }
}
```

会编译成:

```css
.myapp-Header-signup {
  display: block;
}

.myapp-Header-isScrolledDown .myapp-Header-signup {
  display: none;
}
```

甚至媒体的查询都能方便的工作，只要你的预编译支持冒泡(SASS, LESS, PostCSS 和 Stylus 都有可以):

```scss
.myapp-Header {

  &-signup {
    display: block;

    @media (max-width: 500px) {
      display: none;
    }
  }
}
```

编译为:

```css
.myapp-Header-signup {
  display: block;
}

@media (max-width: 500px) {
  .myapp-Header-signup {
    display: none;
  }
}
```

以上的模式使得使用唯一的长类名变得简单，不用一边又一边的重复。便利是必须的，不然的话人们便会偷懒走捷径。

### 快速的过下JS这边的情况


这篇文档是关于样式约定的，但样式并不是凭空存在的，JS这边的一样需要创造同样的类命名空间，一样的需要便利性.

无耻的插播一个广告 (**译者：对作者就是这么说的**), 我创建了一个简单的，0依赖的JS库来说明，叫做 [`css-ns`](https://github.com/jareware/css-ns). 当在其他框架中使用的时候,[例如. React](https://github.com/jareware/css-ns#use-with-react), 这允许你在指定文件中**强制**生成一个名字空间.

```js
// 创造一个名字空间绑定的React拷贝
var { React } = require('./config/css-ns')('Header');

// 创造元素:
<div className="signup">
  <div className="intro">...</div>
  <div className="link">...</div>
</div>
```

会这样绘制DOM:

```html
<div class="myapp-Header-signup">
  <div class="myapp-Header-intro">...</div>
  <div class="myapp-Header-link">...</div>
</div>
```

这非常方便，以上实现了JS部分的"默认本地";

我又一次跑题了，让我们回到CSS吧.

### 6. 防止组件内的样式泄漏


还记得我之前说的每个类名都需要增加组件前缀是一种很“方便”的沙盒化样式的方法？ 还记得我说过"说明"?


考虑以下的样式:

```scss
.myapp-Header {
  a {
    color: blue;
  }
}
```

和如下的组件层次:


```
+-------------------------+
| Header                  |
|                         |
| [home] [blog] [kittens] | <-- 这些是 <a> 元素
+-------------------------+
```

没问题吧? 只有在 `Header`中的 `<a>` 元素 inside `Header` 变 [蓝色](https://www.youtube.com/watch?v=axHe_BVY_9c) , 因为我们定义的规则是:

```css
.myapp-Header a { color: blue; }
```

加入这个布局变化为:

```
+-----------------------------------------+
| Header                    +-----------+ |
|                           | LoginForm | |
|                           |           | |
| [home] [blog] [kittens]   | [info]    | | <-- 这些是 <a> 元素
|                           +-----------+ |
+-----------------------------------------+
```

这是选择`.myapp-Header a` **同时符合了** 在`LoginForm`中的`<a>`元素，我们所谓的样式独立性就被破坏了。 所以说，把所有的样式都绑定在一个名字空间的区域内是一个有效的方式来实现组件邻居之间的样式独立性，**但对子组件不一定成立**。

以下两种方式可以修复这个问题:

1. 永远不要在样式中指定元素名字, 加入每个在`Header`中的`<a>`元素都是 `<a class="myapp-Header-link">` , 那我们就不会有这个问题了。可是，有的时候你已经有很多自定义（语义的）元素名字已经创建好了，而你并不想为他们额外增加类，那么你需要：
1. 通过 [ `>` 连接符 ] (https://developer.mozilla.org/en-US/docs/Web/CSS/Child_selectors) 指定命名空间内的子元素样式。(**译者：原文的outside感觉并不是作者本意**)

对第2种方式, 样式可以写作:

```scss
.myapp-Header {
  > a {
    color: blue;
  }
}
```

这样确保了在更深层的组件树不受当前的样式影响，因为生成的选择表达是：`.myapp-Header > a`. ( **译者:关于selector可以参考译者原创的文章 [JQuery Selector 入门](http://www.jianshu.com/p/acf10d02bdb9)**)

加入你还不确定，让我提个更加疯狂的方案，而且这也可行：

```scss
.myapp-Header {
  > nav > p > a {
    color: blue;
  }
}
```


这些年来我们一直被教导不要使用嵌套选择 (当然包括使用 '>') [例如这许多年的有用例子建议](http://lmgtfy.com/?q=css+nesting+harmful)， 为什么？三个理由：

1.  串联样式早晚会出事的。你越用嵌套选择，越有可能从某个不知名的角落出来一个元素刚好符合这个选择。 你能读到这里一定了解了我们在之前的建议都在尽力的避免这种可能(严格的命名空间前缀，需要时使用子选择)
1. 太多的指定破坏了重复使用的可能。样式目标例如 `nav p a`是没发在其他地方被使用的，除非有相同的结构。 但我们并不想这样，事实上我们应该禁止类似这样的重用，因为他违背了我们的原则：组件样式需要各自独立。
1. 太多的指定使得重构艰难。这可是有事实依据的，加入你只是用了 `.myapp-Header-link a`, 你可以随意的将`<a>` 在你的组件里移动，样式不会有问题。但对于`> nav > p > a`来说你需要更新选择来匹配新的位置。但我们说过了，组件需要小而独立，这样的工作显得毫无意义。 当然在重构的时候，你需要考虑应用整体的HTML&CSS，也许挺吓人的。但加入你只是操作若干只有几行的沙盒组件, 而且无需关心沙盒外的任何东西，那事情就变的简单多了。

这是一个理解规则的好例子，让你知道什么时候可以违反。在这个架构中，不要使用嵌套选择， 但有些时候确又是正确的事情，祝你好运. 

### 好奇的另一面: 防止泄漏样式*进入到*组件里 

我们已经能够做到组件的样式沙盒化，那这些组件是否能确保独立于整个其他页面了？ 让我们回忆下:

* 我们通过使用不同的空间名字空间前缀来避免样式泄漏**出组件**

        +-------+
        |       |
        |    -----X--->
        |       |
        +-------+

* 扩展的说，这同样意味着**组件之间**样式不泄漏。

        +-------+     +-------+
        |       |     |       |
        |    ------X------>   |
        |       |     |       |
        +-------+     +-------+

* 我们使用了子选择来防止样式泄漏**进入子组件**

        +---------------------+
        |           +-------+ |
        |           |       | |
        |    ----X------>   | |
        |           |       | |
        |           +-------+ |
        +---------------------+

* 但严重的是，**外部样式仍然可以泄漏进组件**

              +-------+
              |       |
        ---------->   |
              |       |
              +-------+

例如，我们有这样一个组件样式:

```scss
.myapp-Header {
  > a {
    color: blue;
  }
}
```

但当我们引入了一个有问题的第三方库，它包含了这样的CSS:

```css
a {
  font-family: "Comic Sans";
}
```

**There is no simple way to protect your components from such external abuse**, and this is where we often need to just:
**压根就没简单的办法来保护你的组件不受外部滥用样式的影响**， 这里你只能:

![give-up.gif](http://upload-images.jianshu.io/upload_images/1655500-548bfb402fe9b933.gif?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

放弃吧！

不幸中的万幸，你至少可以控制那些依赖你可以使用，找到那些正确又优雅的替代品吧。


不过，我只是说没有*简单*的方法来保护你的控件，但这并不意味这没有任何方法了。[哥们，有办法的](https://www.youtube.com/watch?v=20wUS_bbOHY) ，只是任何办法都有代价的.

* 使用强行重置: 加入你对每个组件的每个元素都引入[css重置](http://cssreset.com/what-is-a-css-reset/)， 然后放到一个总是能比第3方更优先的选择中，那么就可以了。 但除非你的应用非常的小，(例如3方应用的一个“分享”按钮)，这样的方式很快就会超出你的控制。所以这并不是一个好的主意，列在这里只是为了完整性.
* [`all: initial`](https://developer.mozilla.org/en/docs/Web/CSS/all)  是个不被人知的新CSS属性，它就是设计来解决这个问题的，它能够[阻挡继承的属性进入](https://jsfiddle.net/0d9htatc/),而且能够将本地的属性重置 [如果它能赢得协议的话](https://jsfiddle.net/e7rw4L8L/).  这个实现包含了一些[复杂因素](https://speakerdeck.com/csswizardry/refactoring-css-without-losing-your-mind?slide=39)，可能不是每个地方都[支持](http://caniuse.com/#feat=css-all) ， 但相信我， `all: initial` 可能成为样式独立非常有用的工具.
* Shadow DOM（影子DOM）已经提到了，这也是解决这个问题的很好方式，它允许定义JS和CSS清晰的边界. 除了[近期的一些希望](https://developer.apple.com/library/content/releasenotes/General/WhatsNewInSafari/Articles/Safari_10_0.html), 这网页组件的协议近几年来并没有更新, 除非你的目标浏览器是明确的，你没法正式的使用Shadow DOM.
* 最后的方法就是 `<iframe>`了，这提供了最强大的网页运行时独立环境(对JS和CSS都一样), 但同时也会带来启动和运行效率的惩罚。 但这样的代价有时候是值得的，那些最接触的网页嵌入（例如Facebook, Twitter, Disqus）事实上就是用iframes写的。但为了这篇文档的出发点 - 独立成千上万个小的组件会让你的开销成百倍的增加。

这次走题有点长了，回到CSS.

### 7. 尊重组件的边界

就像我们样式 `.myapp-Header > a`, 当我们嵌套一个组件时候，我们可能要对子组件设置一些样式, 考虑这布局:

```
+---------------------------------+
| Header           +------------+ |
|                  | LoginForm  | |
|                  |            | |
|                  | +--------+ | |
| +--------+       | | Button | | |
| | Button |       | +--------+ | |
| +--------+       +------------+ |
+---------------------------------+
```

我们立即可以看到样式 `.myapp-Header .myapp-Button`  不是个好主意，显然我们应该这样写 `.myapp-Header > .myapp-Button`。 但什么样式是我们可能会希望子组件也继承的呢？

注意到 `登陆表单`被锁定在`顶部栏`的右边，直觉上，下面的样式需要:

```scss
.myapp-LoginForm {
  float: right;
}
```

我们还没违反任何规定，但我们也让 `登陆表单`变的难以重用了, 如果我们下面的主要需要`登陆表单`, 但又不是右浮动的，那就没戏了。

实用的解决方法是部分的放松我们之前的规则，只写当前名字控件的组件样式。 我们可以这样做:

```scss
.myapp-Header {
  > .myapp-LoginForm {
    float: right;
  }
}
```

这其实很完美，只要我们不破坏子组件的沙盒性。

```scss
// COUNTER-EXAMPLE; DON'T DO THIS
.myapp-Header {
  > .myapp-LoginForm {
    color: blue;
    padding: 20px;
  }
}
```


看起来我们并不希望这样，因为我们失去了对本地样式的保护，全局样式可以影响了。 同时上面的代码中，`LoginForm.scss` 不再是唯一一个地方你能看到`LoginForm`组件的样式了，这听起来挺可怕的，那么，我们到底如何区分什么是可以的什么是不可以的呢？

We want to respect the sandbox *inside* each child component, as we don't want to rely on its implementation details. It's a black box to us. What's *outside* the child component, conversely, is the sandbox of the parent, where it reigns supreme. The distinction between inside and outside emerges quite naturally from one of the most fundamental concepts in CSS: [the box model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model).
我们需要尊重每个组件的内部沙盒，这并不应该依赖于他们自己的内部实现。每个组件都是一个嘿嘿。对于那些在*外*的组件, 相反的，是他们父辈节点的沙盒, 自我管理。内部和外部的区别在CSS的最重要的基础知识上能够体现出来: [the box model](https://developer.mozilla.org/en-US/docs/Web/CSS/CSS_Box_Model/Introduction_to_the_CSS_box_model).

![box-model.png](http://upload-images.jianshu.io/upload_images/1655500-05f9ecdd93c94b17.png?imageMogr2/auto-orient/strip%7CimageView2/2/w/1240)

我的推理并不好，但详细的说是这样的： 就像*在国家内*意味这在他的边界内，我们认为父节点只能影响它直接下一代的子节点组件边界外的属性. 这意味着和位置与像素相关的属性(例如`position`, `margin`, `display`, `width`, `float`, `z-index` 等), 但那些涉及到边界之内的属性(例如`border`本身，`padding`, `color`, `font`等)就不行。

综上，以下这种写法是不允许的:

```scss
// 反例; 请不要这样写
.myapp-Header {
  > .myapp-LoginForm {
    > a { // 依赖 LoginForm 的详细实现
      color: blue;
    }
  }
}
```

但也有一些有趣的边界情况，比如:

* `box-shadow` - 一种特定类型的阴影可以整合成为组件外观的一部分，所以它应该成为自有的样式。但同时，视觉上的感受它明显是在边界以外的, 从这个角度它又该属于父节点组件的一部分。 
* `color`, `font` 和其它 [继承的属性](https://developer.mozilla.org/en-US/docs/Web/CSS/inheritance) - `.myapp-Header > .myapp-LoginForm { color: red }`涉及到了内部的子组件, 但同时来说也相当于函数 `.myapp-Header { color: red; }`, 这也是符合我们的规则的.
* `display` -加入子组件用了 [Flexbox](https://css-tricks.com/snippets/css/a-guide-to-flexbox/)布局, 很有可能在根元素上设置了 `display: flex` 属性. 但是，父节点可以选择设置 `display: none` 来隐藏子节点。

对于这些边界条件下的情况，你需要清楚的认识到这没什么大不了的，只是一点点CSS串联进你样式。 比起其他让你烦恼的事情来说，你完全可以接受*适度*的串联。 例如这个例子[指定内容](https://developer.mozilla.org/en-US/docs/Web/CSS/Specificity), 就像你期望的那样展现: 当组件可见时,`.myapp-LoginForm { display: flex }`是指定的, 当需要隐藏时，`.myapp-Header-loginBoxHidden > .myapp-LoginBox { display: none }`成为了指定的样式。

### 8. 解耦外部的style依赖

为了避免重复造轮子，有时候你需要在组件间共享样式， 有时候你也需要使用其他人创建的样式。 这样的情况下，就需要避免对代码长生比不要的依赖影响.

一个很实在的例子, 我们来用一些来自 [Bootstrap](http://getbootstrap.com/css/)的样式, 作为那些整合起来让人头大的框架的一个完美例子. 考虑我们上面所提到的所有内容，说到把所有的样式放到一个全局的名字空间是个很坏的习惯，然而Bootstrap:

* 往全局名字空间里导出了非常多的选择器 (准确的说2481个，对于版本3.3.7), 不管你是不是要用。(一花费了*很多天*来调试这个问题..)
* 使用硬编码的类名字类如`.btn` 和 `.table`. 没法想象如果这些如果被其他开发者或者项目重用的话是多么恐怖的事情.

先不管上面这些，假定我们要使用Bootstrap作为我们`Button`组件的基础.

除了可以在HTML里这么写外：

```html
<button class="myapp-Button btn">
```

考虑[扩展](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#extend) 这个类进入你的样式:

```html
<button class="myapp-Button">
```

```scss
.myapp-Button {
  @extend .btn; // from Bootstrap
}
```

这样做的话，可没人能够知道你这个组件的样式是荒谬的依赖于`btn`类的. `Button` 原始的样式实现是完全没必要在外部展示的。作为一个结果，一旦你放弃bootstrap转用别的框架(或者自己写样式), 这样的改变从代码层面是很难体现出来的，但结果就是.. 哈哈 你的`Button`样子变了！

同样的原则你应该用自己的帮助类, 这里你也许会选择更合理的名字，例如:

```scss
.myapp-Button {
  @extend .myapp-utils-button; // 在项目其他地方定义
}
```

或者 [只是占位类](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#placeholder_selectors_) altogether ([主流预编译都支持](https://csspre.com/placeholder-selectors/)):

```scss
.myapp-Button {
  @extend %myapp-utils-button; // 在项目其他地方定义
}
```

最后，所有主流的css预编译工具都支持 [mixins](http://sass-lang.com/documentation/file.SASS_REFERENCE.html#mixins), 这同样十分的有用.

```scss
.myapp-Button {
  @include myapp-generateCoolButton($padding: 15px, $withExplosions: true);
}
```

It should be noted that when dealing with more civilized style frameworks (such as [Bourbon](http://bourbon.io/) or [Foundation](http://foundation.zurb.com/)), they'll in fact be doing just this: declaring a bunch of mixins for you to use where they're needed, and not emitting any styles on their own. [Neat](http://neat.bourbon.io/).

必须指出现在很多良好的框架(例如 [Bourbon](http://bourbon.io/) 或者 [Foundation](http://foundation.zurb.com/))，它们就是像上面这样做的，声明一些列的mixins给你按需来使用，才不会散发他们自己的样式. [优雅](http://neat.bourbon.io/)!

## 结束语

> 熟悉规则了，才能更好的知道什么时候打破规则

最后，就像上面提到的，你熟悉了解了你所依赖的这些规则(无论是陌生人告诉你的还是你从网上学的), 你才能合理的做一些例外的事情。 例如，如果你需要直接增加一个帮助类，你可以这么做

```html
<button class="myapp-Button myapp-utils-button">
```

这个添加的值可以，举例来说，帮助你的测试框架来自动识别这是一个可以被点击的按钮。

Or you might decide that it's OK to break component isolation when the breach is tiny, and the additional work from splitting components would be too great. While I'll want to remind you that it's a slippery slope and that consistency is king et cetera, as long as your team is in agreement, and you get stuff done, you're doing the right thing.

或者你也可以决定偶尔打破下组件隔离的规矩，小小的违背下, 因为如果要完全实现组件分割的代价会很大。 虽然我也许会提醒你这可能会有风险，最好还是保持一致，但只要你的团队ok,你又完成任务了，那也没问题的！

## 最后

喜欢请转发，谢谢

## License

[CC BY 4.0](https://creativecommons.org/licenses/by/4.0/)


> 声明： 之前的翻译中增加了译者本人的一些注释，其中部分曲解了作者的原意，在此向原作者标示歉意，同时删除了之前的注释。 

