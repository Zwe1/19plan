## Provider

Provider 是 react-redux 的核心 api 之一，结合 react context api，用于向组件传递状态集。

##### 使用案例

```js
<Provider store={store}>
  <APP />
</Provider>
```

可见 Provider 也是一个常规的 React 组件，我们向这个组件传入了 store 属性，它的值即为调用 redux 中   creatStore 方法返回的 store。

#### ./Context

```js
import React from "react";

export const ReactReduxContext = React.createContext(null);

export default ReactReduxContext;
```

#### Provider 源码

```js
import React, { Component } from "react";
import PropTypes from "prop-types";
// react context, 可以跨组件层级传递状态。
import { ReactReduxContext } from "./Context";

class Provider extends Component {
  // Provider的使用如上例
  constructor(props) {
    super(props);

    const { store } = props;

    this.state = {
      // storeState为redux创建的状态集合，呈🌲树结构的数据对象
      storeState: store.getState(),
      store
    };
  }

  componentDidMount() {
    this._isMounted = true;
    this.subscribe();
  }

  componentWillUnmount() {
    if (this.unsubscribe) this.unsubscribe();

    this._isMounted = false;
  }

  componentDidUpdate(prevProps) {
    if (this.props.store !== prevProps.store) {
      // 组件更新时，有可能传入了新的store。
      // Provider中订阅了redux的状态变化。
      if (this.unsubscribe) this.unsubscribe();

      this.subscribe();
    }
  }

  subscribe() {
    // 订阅props中store的变化。
    const { store } = this.props;
    // 使用redux中的subscibe方法订阅状态变化。
    this.unsubscribe = store.subscribe(() => {
      const newStoreState = store.getState();

      if (!this._isMounted) {
        // 对于provider中数据的更新需要保证组件已经挂载结束。
        return;
      }

      this.setState(providerState => {
        // 如果前后两次状态未发生变化，则无需进行更新。
        if (providerState.storeState === newStoreState) {
          return null;
        }

        return { storeState: newStoreState };
      });
    });

    // 有可能在reder和挂载两个时间点之间dispatch action，需要考虑此种情况
    const postMountStoreState = store.getState();
    if (postMountStoreState !== this.state.storeState) {
      this.setState({ storeState: postMountStoreState });
    }
  }

  render() {
    // context可以传入，默认使用react context
    const Context = this.props.context || ReactReduxContext;
    // provider是整个应用的状态提供者
    return (
      <Context.Provider value={this.state}>
        {this.props.children}
      </Context.Provider>
    );
  }
}

Provider.propTypes = {
  store: PropTypes.shape({
    subscribe: PropTypes.func.isRequired,
    // disptach在其他api中有用
    dispatch: PropTypes.func.isRequired,
    getState: PropTypes.func.isRequired
  }),
  context: PropTypes.object,
  children: PropTypes.any
};

export default Provider;
```

#### 总结

provider api 通过使用 react context 技术，为应用提供通过 redux 生产的状态集合 store，并在 store 发生更新时，及时更新内部状态。

**_注意：_**context api 需要 react 特定版本的支持。⚠️
