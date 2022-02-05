---
title: 在React和TypeScript中使用emotion
date: 2021-01-13 16:17:44
tags:
  - CSS in JS
  - emotion
---

> 本文介绍如何在React项目中使用CSS in JS方案：emotion。

![image](https://raw.githubusercontent.com/emotion-js/emotion/master/emotion.png)

<!--more-->

### emotion

emotion是一种高性能且灵活的CSS-in-JS库。它本身与框架无关，你可以在vue或者react中搭配使用。目前我们使用的是react，已经有多个项目在生产环境稳定运行。emotion11是对emotion10的略微改进，主要侧重于开发者的体验，TS的类型改进，以及使用新版本的解析器：Stylis。
本文主要介绍在React和TypeScript中如何集成emotion11。

### 变化

#### 包重命名
emotion11最重要的变化之一是大部分面向用户的package都已经重命名。

重命名包的列表：

- @emotion/core → @emotion/react
- emotion → @emotion/css
- emotion-theming → moved into @emotion/react
- emotion-server → @emotion/server
- create-emotion → @emotion/css/create-instance
- create-emotion-server → @emotion/server/create-instance
- babel-plugin-emotion → @emotion/babel-plugin
- eslint-plugin-emotion → @emotion/eslint-plugin
jest-emotion → @emotion/jest

#### Hooks

在内部使用钩子以优化包的大小并在React DevTools中展示更好的DOM树。

#### TypeScript

##### TypeScript 类型已经被完全重写。

1. 减少使用emotion时的构建时间，尤其是在大型项目中。
2. 在许多情况下，不再需要为emotion组件手动指定通用参数
3. 作为props的联合类型得到了更好的支持，应该正确推断
4. 限制了css函数以防止传递无效的类型
5. styled 的通用参数已更改，如果您指定ComponentType，则需要删除该通用参数
6. styled不再需要第二个参数ExtraProps，代替地将其移动至styled调用之后。因此，styled<typeof MyComponent, ExtraProps>(MyComponent)应该被改写为styled(MyComponent)<ExtraProps>({})


#####  Theme类型

现在，为主题提供类型更加容易。您可以像这样创建内置的Theme接口，而不是像以前那样创建自定义实例：


``` typescript
import '@emotion/react'

declare module '@emotion/react' {
  export interface Theme {
    primaryColor: string
    secondaryColor: string
  }
}
```

##### css prop 类型

基于使用的不同JSX运行时，emotion11为css prop提供TypeScript支持的方式已经更改，可以仅对支持className prop的组件添加对应css prop的支持（因为emotion的JSX工厂函数采用提供的css prop，对其解析并将生成的className传递给渲染的组件）。

#### Stylis V4

emotion使用的css解析器Stylis得到了升级，它修复了一些长期存在的解析边缘情况，同时变得更小，更快。

#### Emotion的缓存

创建高速缓存的自定义实例时，现在需要key选项。请确保它是唯一的（并且不等于“ css”），因为它用于将样式链接到缓存。如果多个缓存共享同一个键，它们可能会为彼此的样式元素“争斗”。 新的prepend选项可以使Emotion在指定DOM容器的开头而不是结尾处添加样式标签。

### 使用

#### 安装


``` bash
yarn add @emotion/react
```

#### 使用


```
// this comment tells babel to convert jsx to calls to a function called jsx instead of React.createElement
/** @jsx jsx */
import { jsx, css } from '@emotion/react'

const style = css`
  color: hotpink;
`

const SomeComponent = ({ children }) => (
  <div css={style}>
    Some hotpink text.
    {children}
  </div>
)

const anotherStyle = css({
  textDecoration: 'underline'
})

const AnotherComponent = () => (
  <div css={anotherStyle}>Some text with an underline.</div>
)
render(
  <SomeComponent>
    <AnotherComponent />
  </SomeComponent>
)
```

#### css prop

emotion提供主要的书写style的方式是使用css prop，它提供了一个简洁灵活的API来对组件进行样式设定。

有两种方式使用css prop：

##### Babel Preset

Babel预设可在使用classic JSX运行时时自动启用Emotion的css prop。如果要使用新的JSX运行时，请不要使用此预设，而应使用@emotion/babel-plugin

- 安装

```
yarn add @emotion/babel-preset-css-prop
```
- 使用

.babelrc
```
{
 "presets": [
   [
     "@emotion/babel-preset-css-prop",
     {
       "autoLabel": "dev-only",
       "labelFormat": "[local]"
     }
   ]
 ],
}
```
如果您使用兼容的React版本（>=16.14.0），则可以通过以下配置选择使用新的JSX运行时：

.babelrc
```
{
  "presets": [
    [
      "@babel/preset-react",
      { "runtime": "automatic", "importSource": "@emotion/react" }
    ]
  ],
  "plugins": ["@emotion/babel-plugin"]
}
```

#### JSX Pragma

在使用CSS prop的源文件顶部设置jsx编译指示。此选项最适合测试css prop功能或在无法配置babel配置的项目（create-react-app，codesandbox等）中。


```
/** @jsx jsx */
```

与包含linter配置的注释类似，此配置将jsx babel插件配置为使用jsx函数而不是React.createElement。 

如果您正在使用零配置工具来自动检测应该使用哪个运行时（classic还是automatic），并且您已经在使用具有新JSX运行时的React版本（因此运行时为您自动配置了runtime: 'automatic'），例如Create React App 4，然后
```
/ ** @jsx jsx * /
```
编译指示可能不起作用，您应该使用
```
/** @jsx jsx */
import { jsx } from '@emotion/react'
```
。

#### 使用css prop


```
/** @jsx jsx */
import { jsx } from '@emotion/react'

render(
  <div
    css={{
      backgroundColor: 'hotpink',
      '&:hover': {
        color: 'lightgreen'
      }
    }}
  >
    This has a hotpink background.
  </div>
)
```
### 主题

Theme包含在@ emotion / react包中。

将ThemeProvider添加到应用程序的顶层，并在样式化组件中使用props.theme来访问主题，或者提供一个接受该主题作为css prop的函数。

#### 使用

##### css function

```
import { ThemeProvider } from '@emotion/react'

const theme = {
  colors: {
    primary: 'hotpink'
  }
}

render(
  <ThemeProvider theme={theme}>
    <div css={theme => ({ color: theme.colors.primary })}>
      some other text
    </div>
  </ThemeProvider>
)
```
##### useTheme hook


```
import { ThemeProvider, useTheme } from '@emotion/react'

const theme = {
  colors: {
    primary: 'hotpink'
  }
}

function SomeText (props) {
  const theme = useTheme()
  return (
    <div
      css={{ color: theme.colors.primary }}
      {...props}
    />
  )
}

render(
  <ThemeProvider theme={theme}>
    <SomeText>some text</SomeText>
  </ThemeProvider>
)
```



#### 类型

主题的类型需要在类型文件中声明，否则TS类型会报错。

types/emotion.d.ts

```
import "@emotion/react";

declare module "@emotion/react" {
  export interface Theme {
    colors: {
      layoutBodyBackground: string;
      headingColor: string;
      textColorSecondary: string;
      success: string;
      warning: string;
      error: string;
      primary: string;
      textColor: string;
    };
    fontSizes: {
      base: number;
    };
  }
}

```

tsconfig.json


```
{
  "compilerOptions": {
        "paths": {
          "*": [
            "types/*"
          ],
        },
    }
}
```

### 总结

CSS in JS的方案使用下来相比传统的less、sass等优点的确不少，而emotion的类似方案也有很多，对我感受最深的一点是，打包速度提升很多，没有冗余的CSS出现，但是对于习惯了css写法的人来说还是需要时间熟悉，总之，不妨一试。


