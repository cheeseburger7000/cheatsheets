# CSS Modules

[TOC]

可以保证某个组件的样式，不会影响到其他组件。

局部作用域

模块依赖



一个标签需要使用到多个样式的时候要怎么处理？

1. 直接用es6的 `${}` 链接就好了 `${style.title1} ${style.title2}`

2. className={[style.title1, style.title2, style.title3].join(" ")}

3. 三方库：classnames

   

参考文章

- [CSS Modules 用法教程 - 阮一峰](https://www.ruanyifeng.com/blog/2016/06/css_modules.html)
- [CSS Modules - Github](https://github.com/css-modules/css-modules)



CSS样式处理

- height: calc(100% - 56px);

```css
/* width: 100% - 56px; */
width: calc(100% - 56px);
```

https://stackoverflow.com/questions/899107/how-can-i-do-width-100-100px-in-css

- Viewport 可视窗口 浏览器

- https://developer.mozilla.org/en-US/docs/Web/CSS/::after
- overflow
- Less nest feature
- &:hover .moreIcon {
- &.selectItem {
- max-width min-width max-height min-heigth width height
- 使用 global 覆盖UI组件库的样式

https://stackoverflow.com/questions/43613619/what-does-global-colon-global-do

```less
.newSwitch {
  // 新增的样式
  position: relative;
  display: inline-block;
  :global {
    .ant-switch {
      // 要覆盖组件库的样式
    }
  }
}
```

- 来自组件库的一种奇怪的写法

```less
.ant-select-dropdown-menu-item:hover:not(.ant-select-dropdown-menu-item-disabled) {
  //background-color: transparent;
}
```

- `!important` 样式覆盖

- `@import`

```less
@import '../../../src/style/variables.less';
```

- []

```less
.ant-input-number[disabled] {
  color: #BFBFBF;
  background-color: #F2F2F2;
}
```

- .A > span { ... } 这个选择器经常使用！！！要特别注意
- 看明白这个文件表达什么意思

```less
@import '../../../src/style/variables.less';

.headerSearch {
  :global(.anticon-search) {
    font-size: 16px;
    cursor: pointer;
  }
  .input {
    width: 0;
    background: transparent;
    border-radius: 0;
    transition: width 0.3s, margin-left 0.3s;
    :global(.ant-select-selection) {
      background: transparent;
    }
    input {
      padding-right: 0;
      padding-left: 0;
      border: 0;
      box-shadow: none !important;
    }
    &,
    &:hover,
    &:focus {
      border-bottom: 1px solid @border-color-base;
    }
    &.show {
      width: 210px;
      margin-left: 8px;
    }
  }
}
```



