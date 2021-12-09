# props drilling

## 是什么

props drilling 是将数据以 props 的形式从 React 组件树中的一部分传递到另一部分, 只是传递的组件层级过深, 而中间层组件并不需要这些 props, 只是做一个向下转发, 这种情况就叫做 props drilling。

类比一下, 我们买东西的时候,经历了卖家发货、跨省、市、区一系列的流程最后到我们手上, 中间的这些都只是负责转发到某一个地方(买家), 它们并不关心这个东西是什么, 最终目的就是负责将快递派发到买家手上。

而 props drilling 也是如此:

1. `App` 首先把`prop`传递给`Dashboard`
2. `Dashboard` 传递给 `DashboardContent`(并不需要 props, 只是做一层转发)
3. `DashboardContent` 传递给 `WelcomeMessage`(并不需要 props, 只是做一层转发)
4. `WelcomeMessage` 接收`user props` 进行渲染(终于到达需要 props 得地方了)

看一下例子:

```jsx
function Dashboard({ user }) {
  return (
    <div>
      <h2>Dashboard</h2>;
      <DashboardContent user={user} />
    </div>
  );
}
function DashboardContent({ user }) {
  return (
    <div>
      <h3>DashboardContent</h3>;
      <WelcomeMessage user={user} />
    </div>
  );
}
function WelcomeMessage({ user }) {
  return (
    <div>
      <p>Welcome {user.name}</p>
    </div>
  );
}
export default function App() {
  const user = { name: 'itsuki' };
  return (
    <div className='App'>
      <Dashboard user={user} />
    </div>
  );
}
```

`user props` 从`App`组件一直传递到了`WelcomeMessage`组件, 然后只有`WelcomeMessage`组件才是真正消耗 props 的地方。

## props drilling 有什么问题?

### 传递

你肯定也碰到过这样子的情况, 孙子组件需要访问爷爷组件的 props, 中间的爸爸组件做一层传递, 但是只有三层传递, 所以问题就不是很明显, 但是如果组件树过于复杂, 假如有十层, 二十层, 完蛋, 至少有 20 个组件需要接收一个不需要的 props, 就是为了某一个组件需要使用这个 props, 那么如果有 n 个 props 需要这么传递, 不敢想象, 这就是噩梦。

### 黑盒

看上面的代码我们在`<App>`组件中使用一个组件`<FirstComponent/>` , what ? 很厉害, 一个组件就渲染了那么多东西, 但是它就像一个黑盒, 我只知道需要传下去`content props`就可以了, 它就能成功渲染想要的内容, 我们无法知道和组织它的内部结构, 更不要说去扩展了, 因为它已经"固定"住了, 当来一个新的功能需要扩展时, 完蛋, 得重新传递 props, 重新调整布局, 代码看起来多么脆弱。

## 怎么解决

我们想要解决的问题, 就如同上面的例子那样, `NeedPropsComponent` 的 props 不想要通过一层层传递或者说不想要在跨过 n 层组件传递读取想要的 props。

那么这个时候怎么去解决呢?

### 组合

我们不妨试试组合, 可以看看[官方文档](https://zh-hans.reactjs.org/docs/composition-vs-inheritance.html)如何介绍以及使用的。

看看使用组合改进后的代码。

```jsx
function Dashboard({ children }) {
  return (
    <div>
      <h2>Dashboard</h2>
      {children}
    </div>
  );
}

function DashboardContent({ children }) {
  return (
    <div>
      <h3>DashboardContent</h3>;{children}
    </div>
  );
}

function WelcomeMessage({ user }) {
  return (
    <div>
      <p>Welcome {user.name}</p>
    </div>
  );
}

export default function App() {
  return (
    <div className='App'>
      <Dashboard>
        <DashboardContent>
          <WelcomeMessage user={{ itsuki: 'user' }} />
        </DashboardContent>
      </Dashboard>
    </div>
  );
}
```

通过这样子我们解决来 props drilling 的问题, 你可能会想, 它不会影响我们扩展, 比如说, 我可能还有其他组件需要相同或者不同的 props? 使用组合的方式你会发现得心应手, 在无法保证你的组件写死的情况下, 请保持一定的抽象!!!

```jsx
export default function App() {
  const user = { name: 'itsuki' };
  return (
    <div className='App'>
      <Dashboard>
        <DashboardContent>
          <WelcomeMessage user={user} />
        </DashboardContent>
        <DashboardSidebar>
          <Avatar user={user} />
        </DashboardSidebar>
      </Dashboard>
    </div>
  );
}
```

**使用组合时不局限于 children prop, 你可以尝试 left 、 right 或者其他 prop name 去进行抽象, 这取决于你自己。**

```jsx
function Layout({ left, right, children }) {
  return (
    <div>
      <div className='left'>{props.left}</div>
      <main>{children}</main>
      <div className='right'>{props.left}</div>
    </div>
  );
}

function App({ children }) {
  return (
    <Layout left={<nav>left</nav>} right={<aside>right</aside>}>
      {children}
    </Layout>
  );
}
```

### context

还有一种是使用 Context, 有了 Hooks 的出现使用 Context 更加简洁。

```jsx
const AuthContext = createContext({});

function App() {
  const user = { name: 'itsuki' };
  return (
    <AuthContext.Provider value={user}>
      <App />
    </AuthContext.Provider>
  );
}

function WelcomeMessage() {
  const { user } = useContext(AuthContext);
  return (
    <div>
      <p>Welcome {user.name}</p>
    </div>
  );
}
```

正如官方文档所说, Context 主要应用场景在于很多不同层级的组件需要访问同样一些的数据, 比如说用户信息、主题或者语言。

如果需要使用 Context, 请注意它让组件复用更加困难, 换句话来说: 无法脱离"Context"去复用组件。

## 后记

最后在看 React 如何设计的更好, 然后结合在自己的博客代码中, 发现了这些知识点, 还挺有意思的, 突然觉得 React 太厉害了, 设计出一个好的 React 应用一定有一个好的抽象。
