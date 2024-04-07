container布局容器在不同的框架和库中有不同的含义，但核心思想都是提供一种用于**组织和控制内容布局**的容器组件。

### 一、Bootstrap中的.container与.container-fluid

---

- `.container`: Bootstrap框架中的`.container`类是用来创建一个具有固定宽度并且居中显示的容器。它会根据屏幕大小自动调整宽度，但在较小和较大屏幕尺寸下都有固定的左右边距（即响应式布局），目的是为了确保内容在不同屏幕尺寸下保持合理的间距和美观。
- `.container-fluid`: 与此相反，`.container-fluid`类创建一个宽度为100%的容器，填满浏览器窗口的整个宽度。这意味着内容将扩展到视口边界，不会有任何固定边距，通常用于全屏背景或者需要内容铺满整个可视区域的布局。



### 二、Element UI中的el-container

---

Element UI是Vue.js的一个UI组件库，在这个库中，`el-container`是一种基于Flex布局的组件，用于构建整体页面布局。它包含一系列子组件如`el-header`、`el-main`、`el-footer`和`el-aside`，用于分别表示页面的头部、主体、底部和侧边栏。这些组件结合使用可以灵活地搭建出各种布局样式，如顶部导航+侧边栏+主要内容区的常见布局模式。

`el-container`组件主要用于构建页面的整体布局，它可以方便地实现水平布局或垂直布局。



### 三、Flutter中的Container

---

在Flutter框架中，Container Widget 是一个非常基础且常用的布局组件，它不仅可以用来包裹和定位子组件，还可以添加背景颜色、边框、阴影等多种视觉效果。Container不强制内容的排列方式，可以通过设置child、alignment、constraints等属性来控制其内部内容的布局。