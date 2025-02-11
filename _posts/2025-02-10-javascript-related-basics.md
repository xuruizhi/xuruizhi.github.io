---
layout: post
title:  "JavaScript相关基础"
date:   2025-02-10 10:00:00 +0800
categories: "software_development"
---

## HTML和 CSS

### HyperText Markup Language, 超文本标记语言

HTML 负责内容和结构：HTML 用于创建网页的基本骨架，通过各种标签来定义网页的不同部分，如标题、段落、列表、链接、图片等。它决定了网页中有什么内容以及这些内容的组织方式。

### Cascading Style Sheets, 层叠样式表

CSS 负责样式和布局：CSS 则专注于网页的外观呈现，它可以控制 HTML 元素的颜色、字体、大小、间距、背景、位置等样式属性，以及实现页面的布局，如浮动、定位、弹性布局和网格布局等。通过 CSS，能够让网页更加美观、易读和吸引人。

### 结合方式

- 内部样式表：在 HTML 文件的 head 标签中使用 style 标签来定义 CSS 样式。这种方式适用于对当前页面的多个元素应用样式。

  ```HTML
  <!DOCTYPE html>
  <html>
      <head>
          <title>选择器示例</title>
          <style>
              /* 标签选择器，选择所有 p 元素 */
              p {
                  color: red;
              }
              /* 类选择器，选择所有 class 为 highlight 的元素 */
              .highlight {
                  background-color: yellow;
              }
              /* ID 选择器，选择 id 为 special 的元素 */
              #special {
                  font-weight: bold;
              }
          </style>
      </head>
      <body>
          <p>这是一个普通段落。</p>
          <p class="highlight">这是一个高亮段落。</p>
          <div id="special">这是一个特殊的 div 元素。</div>
      </body>
  </html>
  ```

没有CSS：![image](https://github.com/user-attachments/assets/46a3d6ec-b3b2-4475-ab39-a85b1916a494)

有CSS：![image](https://github.com/user-attachments/assets/3d2d43ab-ecf1-44e4-829c-0e5634384443)

- 外部样式表：将 CSS 代码单独保存为一个 .css 文件，然后在 HTML 文件中使用 <link> 标签引入该文件。这种方式可以实现样式的复用，提高代码的可维护性。

  ```HTML
  <!DOCTYPE html>
  <html>
  <head>
      <title>外部样式表示例</title>
      <link rel="stylesheet" href="styles.css">
  </head>
  <body>
      <h1>欢迎使用外部样式表</h1>
      <p>这是一个段落。</p>
  </body>
  </html>
  ```

  ```CSS
  h1 {
      color: purple;
  }
  p {
      font-size: 16px;
  }
  ```

- 内联样式：直接在 HTML 元素的 style 属性中添加 CSS 样式。这种方式适用于对单个元素应用简单的样式。例如：
  ```HTML
  <p style="color: green; font-size: 20px;">这是一个使用内联样式的段落。</p>
  ```

## JavaScript与Web编程接口（DOM/BOM）基础概念

- JavaScript：是一门编程语言，ECMAScript是JavaScript的语法标准，包括变量、表达式、运算符、函数、语句等。
- DOM: Document Object Model（文档对象模型），是面向web文档（html/xml）的一组编程接口，它将web文档解析为一个由节点和对象（包含属性和方法）组成的结构集合，使得程序可以动态地访问、修改、删除和添加web文档的内容、结构和样式。
- BOM: Browser Object Model（浏览器对象模型），它提供了一系列用于操作浏览器窗口和相关功能的对象和方法，允许开发者与浏览器窗口进行交互，实现诸如页面跳转、弹窗显示、获取浏览器信息等操作。
- JavaScript和BOM、DOM的关系
  - BOM、DOM提供web页面接口，JavaScript利用这些接口来操作浏览器页面，实现复杂的交互效果。
- DOM和BOM的关系：
  - 包含关系：BOM 是一个更宽泛的概念，它包含了 DOM。DOM 主要关注文档的结构和内容，而 BOM 则涵盖了整个浏览器窗口以及与之相关的各种功能。可以将 BOM 看作是一个大的框架，DOM 是其中用于处理 HTML 或 XML 文档的一部分。window 对象是 BOM 的核心，而 document 对象是 window 对象的一个属性，document 对象正是 DOM 的入口，通过它可以访问和操作整个文档树。
  - 功能侧重点不同：
    - DOM：主要用于操作 HTML 或 XML 文档的内容、结构和样式。开发者可以通过 DOM API 查找元素、修改元素的属性和内容、添加或删除元素等。
    - BOM：侧重于与浏览器窗口进行交互，处理浏览器的各种功能和行为。例如，控制浏览器的导航、管理浏览器窗口的大小和位置、显示弹窗等。

- 相互协作：在实际的 Web 开发中，JavaScript 通常会同时利用 BOM 和 DOM 来实现复杂的交互效果。例如，当用户点击页面上的某个按钮时，可能需要通过 DOM 事件监听器捕获点击事件，然后根据用户的操作使用 BOM 进行页面跳转或显示提示信息。

  ```HTML
  <!DOCTYPE html>
  <html>

  <body>
      <button id="jumpButton">点击跳转</button>
      <script>
          const button = document.getElementById('jumpButton');
          button.addEventListener('click', function () {
              // 通过 DOM 事件触发 BOM 的页面跳转操作
              window.location.href = 'https://www.example.com';
          });
      </script>
  </body>

  </html>
  ```

> JavaScript 是在 Web 开发中最常用的用于操作 BOM 和 DOM 的语言，但并非只有 JavaScript 能够使用 BOM 和 DOM；
> Python 和 Java 都可以通过特定框架操作 BOM 和 DOM。

### DOM简介和DOM操作

### BOM简介和BOM操作

### 如何确保JavaScript代码执行时所有的DOM对象加载完毕

```JavaScript
<!DOCTYPE html>
<html>
  <head>
    <meta charset="UTF-8" />
    <title></title>
    <script type="text/javascript">
      // 【方式一: 先加载，后执行】这段 js 代码是写在 <head> 标签里的，所以建议放在 window.onload 里面。

      window.onload = function() {
        // 获取id为btn的按钮
        var btn = document.getElementById("btn");
        // 为按钮绑定点击事件
        btn.onclick = function() {
          alert("hello");
        };
      };
    </script>
  </head>
  <body>
    <button id="btn">点我一下</button>
    <script type="text/javascript">
      // 【方式二: 后加载，后执行】这段 js 代码是写在 <body> 标签里的，代码的位置是处在页面的下方。这么做，也可以确保: 在页面加载完毕后，再执行 js 代码。

      // 获取id为btn的按钮
      var btn = document.getElementById("btn");
      // 为按钮绑定点击事件
      btn.onclick = function() {
        alert("hello");
      };
    </script>
  </body>
</html>
```

## Node.js

Node.js 是JavaScript在服务器端的运行环境：

- 功能特点：Node.js 基于 Chrome 的 V8 JavaScript 引擎构建，使 JavaScript 可以在服务器端运行。它采用单线程、事件驱动和非阻塞 I/O 模型，非常适合处理大量并发请求，具有高效、轻量级的特点。通过 Node.js，开发者能够使用 JavaScript 编写服务器端代码，操作文件系统、进行网络通信等。
- 提供基础支撑：Node.js 提供了一系列核心模块，如 http、fs、net 等，这些模块为后端开发提供了基础的功能支持。但直接使用这些核心模块进行复杂的后端开发会比较繁琐，因此催生了许多基于 Node.js 的后端框架，如：Express、Nest.js。

## 同样使用JavaScript，前后端编程的差异

1. 运行环境
- 前端：前端 JavaScript 代码运行在浏览器环境中，依赖于浏览器提供的 DOM（文档对象模型）和 BOM（浏览器对象模型）API 来与网页进行交互，操作网页的内容、结构和样式。不同的浏览器可能在 JavaScript 实现和支持的 API 上存在一些差异，需要进行兼容性处理。
- 后端：后端 JavaScript 代码运行在 Node.js 环境中，Node.js 提供了一系列与服务器端操作相关的 API，如文件系统操作、网络通信、进程管理等。后端代码可以直接访问服务器的硬件资源和操作系统功能。

2. 技术栈框架
- 前端：常见的前端框架有 Vue.js、React.js、Angular 等，这些框架主要用于构建用户界面，提供了组件化开发、虚拟 DOM、状态管理等功能，帮助开发者高效地开发交互式的 Web 应用。
- 后端：后端常用的框架有 Express、Koa、Hapi 等，这些框架主要用于构建服务器端应用，处理 HTTP 请求、路由管理、中间件处理等任务，帮助开发者快速搭建 Web 服务器和 API 接口。

---

## Vue&React和Electron有什么关系？

1. Vue 和 React

- 定位：都是用于构建用户界面的前端框架，支持组件化开发，适合开发 Web 应用。
- 区别：
  - Vue 是渐进式框架，更注重开发体验和易用性；
  - React 强调函数式编程和灵活性，生态更庞大（如 Redux、Next.js）。

2. Electron

- 定位：基于 Chromium 和 Node.js 的跨平台桌面应用开发框架，允许用 Web 技术（HTML/CSS/JS）开发桌面软件（如 VS Code）。
- 核心能力：将网页“套壳”为桌面应用，同时支持调用系统级功能（如文件读写）。

3. 三者如何结合

- Electron + Vue/React：
  - Vue 或 React 负责构建应用界面和交互逻辑；
  - Electron 负责将 Web 应用打包为桌面程序，并提供 Node.js 能力（如访问本地文件）。
- 典型场景：
  - 用 Vue/React 开发界面 → 通过 Electron 封装为桌面应用。
  - 例如：VS Code（Electron + 类 React 技术）、部分管理工具（Electron + Vue）。

### 总结

- Vue/React：专注 Web 端用户界面开发。
- Electron：作为“桥梁”，将 Vue/React 的 Web 应用转化为桌面应用，并扩展系统功能。
- 协作模式：三者可组合使用，但 Vue 和 React 是同级替代关系，而 Electron 是跨端封装工具。

如果需要具体整合方法，可参考 Electron 官方文档或社区模板（如  electron-vue 、 electron-react-boilerplate ）

## Node.js和Electron有什么联系？

1. 底层依赖：
Electron的核心由Chromium和Node.js共同构成，其中Node.js为Electron提供了访问操作系统底层资源的能力（如文件读写、进程通信等）。
2. 技术融合：
Electron将Node.js与Chromium整合到同一个运行时环境中，使得开发者既可以用Web技术（HTML/CSS/JS）构建界面，又能通过Node.js实现桌面应用特有的系统级功能。
3. 应用形态：
Electron应用本质上是一个增强版的Node.js应用：主进程使用Node.js管理窗口和系统交互，渲染进程则通过Chromium展示界面，两者通过IPC机制通信。
4. 开发特性：
虽然Electron应用入口是JavaScript脚本（类似Node.js），但它扩展了Node.js的能力边界，使其从服务端开发延伸到跨平台桌面应用开发领域。

简单来说，Node.js为Electron提供了后端能力支撑，而Electron通过整合浏览器引擎，让Node.js具备了构建图形界面应用的能力，形成"Web前端界面+Node.js系统操作"的桌面开发方案。

![image](https://github.com/user-attachments/assets/50803805-b92b-4780-88ce-59ab5c1601e3)

## References

