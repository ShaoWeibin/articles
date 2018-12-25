# styled-components 入门

### 引入
去年看博客的时候了解到了 `styled-components`，但一直没有仔细研究，也没有在项目开发中实践过，最近项目中尝试着用了一下，用着挺爽，推荐给大家。
styled-components 是一个比较常用的 css in js 类库，是实现在 js 中编写 css 的解决方案之一。
> styled-components utilises tagged template literals to style your components.  
> It removes the mapping between components and styles. This means that when you're defining your styles, you're actually creating a normal React component, that has your styles attached to it.

### 入门
#### 如何开始
```
npm install styled-components
yarn add styled-components
```

#### 用法

##### react 函数式定义组件的方式
```javascript
// js 部分
const Button = (props) => {
  return (
    <div className="button">{props.children}</div>
  )
}

// css 部分
.button {
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
}
```

##### styled-components 定义组件的方式
```javascript
// 引用 styled-components 官方示例
const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: palevioletred;
  margin: 0 1em;
  padding: 0.25em 1em;
`;

const Container = styled.div`
  display: flex; 
  align-items: center;
  justify-content: space-between;
  padding: 0 20px;
`;

// 如何使用
render(
  <Container>
    <Button>Normal Button</Button>
  </Container>
);

```

#### 如何使用 props 装配组件
在 styled-components 组件中如何使用 props，下面举例说明如何实现 primary button。通过将 Button 的 primary 属性，可控制 Button 的background 和 color。
```javascript
const Button = styled.button`
  background: transparent;
  border-radius: 3px;
  border: 2px solid palevioletred;
  color: ${props => props.color || 'palevioletred'}; // 可自定义颜色
  margin: 0 1em;
  padding: 0.25em 1em;

  // primary button style
  ${props => props.primary && css`
    background: palevioletred;
    color: white;
  `}
`;

// 如何使用
render(
  <Container>
    <Button primary>Primary Button</Button>
    <Button color="red">Primary Button</Button>
  </Container>
);
```

#### 组件继承
如何使一个组件继承自另一个组件，实现组件复用，只需要将父组件包裹在 `styled()` 构造函数中。
```javascript
// 引用 styled-components 官方示例
// TomatoButton 组件继承自 Button, 有一些覆盖样式
const TomatoButton = styled(Button)`
  color: tomato;
  border-color: tomato;
`;
```
值得一提的是，组件继承不单适用于 styled-components 定义的组件，也适用于任何三方组件（如 普通 react 组件）。
```javascript
const Link = ({ className, children }) => (
  <a className={className}>
    {children}
  </a>
);

const StyledLink = styled(Link)`
  color: palevioletred;
  font-weight: bold;
`;
```

#### 组件附加属性
除了给渲染组件添加普通的 props，也可以通过 `.attrs` 构造函数为组件添加额外的 `props( or attributes)`。
```javascript
const Input = styled.input.attrs({
  // 定义静态属性
  type: "password",

  // 定义动态属性
  margin: props => props.size || "1em",
  padding: props => props.size || "1em"
})`
  color: palevioletred;
  font-size: 1em;
  border: 2px solid palevioletred;
  border-radius: 3px;

  // 使用上面动态计算的属性
  margin: ${props => props.margin};
  padding: ${props => props.padding};
`;

// 如何使用
render(
  <div>
    <Input placeholder="A small text input" size="1em" />
    <Input placeholder="A bigger text input" size="2em" />
  </div>
);
```

#### css 支持情况
`styled-components` 支持 css 伪类，less 嵌套语法，keyframes 动画等，此处不一一展开了，实际开发过程中有问题参考官网示例即可。

### 进阶
#### 组件的实质
`styled.button` 是一个标签模板函数，返回的是一个 `React Component`，`styled-components` 为这个 `React Component` 添加了一个随机命名的 `class`，`styled.button` 模板字符串参数的值实际是 dom 对应的 css 样式，如图所示。

![css 组件实质](https://github.com/ShaoWeibin/images/blob/master/%E4%B8%8B%E8%BD%BD.png?raw=true)

### 高级
#### 定制主题

### 总结
* 开箱即用，学习成本极低。
* 很优雅的方式定义组件。
* 支持组件继承，提高代码复用。
* 不需要关注 css class 的命名问题。
* 很简单的方式实现组件的动态样式定义。
* 不需要配置 webpack 及 babel 转码等问题。
* vscode 支持语法高亮，对应插件为 `vscode-styled-components`。

### 参考资料
https://www.styled-components.com/
https://zhuanlan.zhihu.com/p/28876652