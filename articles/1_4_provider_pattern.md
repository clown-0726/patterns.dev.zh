## 提供者模式

> 使数据可用于多个子组件

在很多情况下，我们希望为应用程序中的一些（如果不是全部）组件提供可用数据。虽然我们可以使用 `props` 将数据传递给组件，但如果应用程序中几乎所有的组件都需要访问 props 的值，那么这可能很难做到。

当我们将 `props` 传递到组件树的下方时，我们经常会用到一种叫做 `prop drilling` 的东西。重构依赖于 `props` 的代码几乎是不可能的，而且很难知道某些数据来自哪里。

假设我们有一个包含特定数据的 `App` 组件。在组件树的最底层，我们有一个 `ListItem`、`Header` 和 `Text` 组件，它们都需要这些数据。为了将这些数据传输到这些组件，我们必须将其通过很多层组件。

![](../pic_bed/1_4_pic_1.gif)

在我们的代码库中，这将类似于以下内容：

```javascript
function App() {
  const data = { ... }

  return (
    <div>
      <SideBar data={data} />
      <Content data={data} />
    </div>
  )
}

const SideBar = ({ data }) => <List data={data} />
const List = ({ data }) => <ListItem data={data} />
const ListItem = ({ data }) => <span>{data.listItem}</span>

const Content = ({ data }) => (
  <div>
    <Header data={data} />
    <Block data={data} />
  </div>
)
const Header = ({ data }) => <div>{data.title}</div>
const Block = ({ data }) => <Text data={data} />
const Text = ({ data }) => <h1>{data.text}</h1>
```

这种方式传递 `props` 会变的很混乱。如果我们想在将来重命名数据 `prop`，那么我们必须在所有组件中重命名它。当应用程序变得越大，`prop drilling` 就就会变的越复杂。

如果我们可以跳过不需要使用这些数据的所有组件层，那这将会是最佳选择。我们需要通过一些方式，让需要访问数据值的组件直接访问数据，而不依赖于 `prop drilling`。

这就是提供者模式可以帮助我们的地方！通过提供程序模式，我们可以将数据提供给多个组件。我们可以将所有组件包装在一个提供者中，而不是通过 `props` 将数据传递到每一层。提供者是 `Context` 对象所提供的高阶组件。我们可以使用 React 提供的 `createContext` 方法创建一个 `Context`  对象。

提供者接收一个 `prop` 参数，其中包含我们想要传递的数据。包装在此提供者中的所有组件都可以访问 `prop` 的值。

```javascript
const DataContext = React.createContext()

function App() {
  const data = { ... }

  return (
    <div>
      <DataContext.Provider value={data}>
        <SideBar />
        <Content />
      </DataContext.Provider>
    </div>
  )
}
```

我们不再需要手动将数据传递给每个组件了！那么，`ListItem`、`Header` 和 `Text` 组件如何访问数据的值呢？

每个组件都可以通过 `useContext` 钩子函数来访问数据。这个钩子函数接收数据引用的上下文（context），在本例中是 `DataContext`。`useContext` 钩子函数允许我们对 context 对象进行读写数据。

```javascript
const DataContext = React.createContext();

function App() {
  const data = { ... }

  return (
    <div>
      <SideBar />
      <Content />
    </div>
  )
}

const SideBar = () => <List />
const List = () => <ListItem />
const Content = () => <div><Header /><Block /></div>


function ListItem() {
  const { data } = React.useContext(DataContext);
  return <span>{data.listItem}</span>;
}

function Text() {
  const { data } = React.useContext(DataContext);
  return <h1>{data.text}</h1>;
}

function Header() {
  const { data } = React.useContext(DataContext);
  return <div>{data.title}</div>;
}
```

不使用数据的组件根本不需要进行数据的处理。我们也不再需要担心通过层层组件的 `props` 向下传递数据，即便这些组件并不需要这些数据，这同时使得重构更加容易。

![](../pic_bed/1_4_pic_2.gif)

------

提供者模式对于共享全局数据非常有用。提供者模式的一个常见用例是在许多组件中共享主题 UI 的状态。

假设我们有一个显示列表的简单应用程序。

> ------
>
> > 打开 https://codesandbox.io/embed/busy-oskar-ifz3w 查看示例代码
>
> ------

我们希望用户能够通过切换开关在 lightmode 和 darkmode 两个主题之间互相切换。当用户从 darkmode 模式切换到 lightmode 模式时，背景颜色和文本颜色应该也随之改变，反之亦然！我们可以将组件包装在 `ThemeProvider` 中，并将当前主题颜色传递给提供者，而不是将当前主题状态传递给每个组件。

```javascript
export const ThemeContext = React.createContext();

const themes = {
  light: {
    background: "#fff",
    color: "#000"
  },
  dark: {
    background: "#171717",
    color: "#fff"
  }
};

export default function App() {
  const [theme, setTheme] = useState("dark");

  function toggleTheme() {
    setTheme(theme === "light" ? "dark" : "light");
  }

  const providerValue = {
    theme: themes[theme],
    toggleTheme
  };

  return (
    <div className={`App theme-${theme}`}>
      <ThemeContext.Provider value={providerValue}>
        <Toggle />
        <List />
      </ThemeContext.Provider>
    </div>
  );
}
```

由于 `Toggle` 和 `List` 组件都包装在 `ThemeContext` 提供者程序中，因此我们可以访问 `theme` 的值和 `toggleTheme`，而这都是作为提供者的值传递进来的。

在 `Toggle` 组件中，我们可以使用 `toggleTheme` 函数相应地更新主题。

```javascript
import React, { useContext } from "react";
import { ThemeContext } from "./App";

export default function Toggle() {
  const theme = useContext(ThemeContext);

  return (
    <label className="switch">
      <input type="checkbox" onClick={theme.toggleTheme} />
      <span className="slider round" />
    </label>
  );
}
```

`List` 组件本身并不关心主题的当前值。但是，`ListItem` 组件关心这个值！我们可以在 `ListItem` 组件中直接使用主题上下文（theme context）。

```javascript
import React, { useContext } from "react";
import { ThemeContext } from "./App";

export default function TextBox() {
  const theme = useContext(ThemeContext);

  return <li style={theme.theme}>...</li>;
}
```

太棒了！我们不必将主题当前值传递给任何不需要关心这个值的组件了。



> ------
>
> > 打开 https://codesandbox.io/embed/quirky-sun-9djpl 查看示例代码
>
> ------

#### 钩子函数（Hooks）



Not ending.

------

#### 参考文档

- [Proxy](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Proxy) - MDN
- [JavaScript Proxy](https://davidwalsh.name/javascript-proxy) - David Walsh
- [Awesome ES2015 Proxy](https://github.com/mikaelbr/awesome-es2015-proxy) - GitHub @mikaelbr
- [Thoughts on ES6 Proxies Performance](http://thecodebarbarian.com/thoughts-on-es6-proxies-performance) - Valeri Karpov