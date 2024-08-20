# Web Component

使用原生 DOM 操作实现的组件

特点：

1. 无关框架，任意环境都可以使用

## 影子 DOM

可以和 iframe 进行类比，相当于一个封闭的容器，它会渲染在页面上，但是其内容对于 js 以及 css 是隐藏的，只有设置 mode 为 open 时，才可以通过宿主的 shadowRoot 进行访问

创建：`element.attachShadow()`

## 使用

### 基本流程

1. 构造自定义元素类
2. 在 `connectedCallback` 回调中，创建 shadow 节点，以及构造各种元素内容，通过 DOM 操作进行组合
3. 通过 `customElements.define()` 注册自定义元素
4. 在页面中进行使用

### 创建

一个自定义元素就是一个类，可以继承 HTMLElement 或者其他 HTML 元素来构造

```js
class WordCount extends HTMLParagraphElement {
  constructor() {
    super();
  }
  // 此处编写元素功能
}
```

元素会有生命周期回调

- `connectedCallback()`：每当元素添加到文档中时调用。规范建议开发人员尽可能在此回调中实现自定义元素的设定，而不是在构造函数中实现。
- `disconnectedCallback()`：每当元素从文档中移除时调用。
- `adoptedCallback()`：每当元素被移动到新文档中时调用。
- `attributeChangedCallback()`：在属性更改、添加、移除或替换时调用。

对于一个组件的创建，基本都在 `connectedCallback()` 回调中，通过 DOM 操作实现

**常用操作**

`getElementById`，`setAttributes`，`getAttributes`，`appendChild`

### 注册和使用自定义元素

通过 `customElements.define( tagName, element )` 注册，使用时与原生 HTML 标签相同方法使用

### 示例

创建一个 my-card 组件

```html
<script>
    class Card extends HTMLElement {
        constructor(){
            super()
        }
        connectedCallback(){
            const shadow = this.attachShadow({mode: "open"})

            const container = document.createElement('div')
            container.setAttribute('class', 'container')

            const topbar = document.createElement('span')

            const topbarText = this.getAttribute('title')
            topbar.innerText = topbarText
            console.log(topbarText)
            
            const style = document.createElement('style')
            style.textContent = `
                .topbar{
                    color: "red";
                }
                .container{
                    border: 1px solid black;
                    padding: 1rem;
                    border-radius: 1rem;
                    width: fit-content;
                }
            `
            container.appendChild(topbar)
            shadow.appendChild(container)
            shadow.appendChild(style)
        }
    }
    customElements.define('my-card', Card)
</script>

<body>
    <my-card title="Title">
    </my-card>
</body>
```

![image-20240814162431763](C:\Users\zhengdaxu\AppData\Roaming\Typora\typora-user-images\image-20240814162431763.png)