# react 基础知识学习

## Context

[Context 官方中文文档](https://react.docschina.org/docs/context.html)
[Context 官方英文文档](https://reactjs.org/docs/context.html)

`Context`提供了一种方式，能让数据在组件树中传递，而不必一层一层手动传递。`Context`的不恰当使用将会影响组件的独立性。以下为`Context`的简单使用。
需注意的是，实际项目中并不会像如下 Demo 使用，下列操作只是熟悉下用法练手而写的。

```js
import React, { Component } from "react";
const NumberContext = React.createContext();
const colorContext = React.createContext();

class NumberShow extends Component {
  static contextType = colorContext;
  render() {
    const color = this.context;
    return (
      <div>
        <NumberContext.Consumer>
          {number => <h1 style={{ color: color }}>Number:{number}</h1>}
        </NumberContext.Consumer>
      </div>
    );
  }
}

class MiddleLevel extends Component {
  static contextType = colorContext;
  render() {
    const color = this.context;
    return (
      <>
        <h2 style={{ color: color }}>颜色{color}</h2>
        <NumberShow />
      </>
    );
  }
}

class ContextDemo extends Component {
  state = {
    numberText: 99,
    color: "red"
  };

  render() {
    return (
      <div>
        <NumberContext.Provider value={this.state.numberText}>
          <colorContext.Provider value={this.state.color}>
            <button
              type="button"
              onClick={() =>
                this.setState({ numberText: this.state.numberText - 1 })
              }
            >
              press
            </button>
            <MiddleLevel />
          </colorContext.Provider>
        </NumberContext.Provider>
      </div>
    );
  }
}

export default ContextDemo;
```

## Lazy 与 Suspense

这两个配合使用以达到 react 组件懒加载,实例代码如下，实际操作中，若组件加载失败，需在错误边界捕捉错误，然后返回替代。`Suspense`中的`fallback`需传入一`jsx`实例

```js
import React, { Component, lazy, Suspense } from "react";

// 为了真切感觉延迟加载，套上一个Promise
const slowImport = function(value, ms = 1000) {
  return new Promise(resolve => {
    setTimeout(() => {
      resolve(value);
    }, ms);
  });
};

// ErrorBoundary
// componentDidCatch

// const Lazy = lazy(() =>
//   slowImport(import(/* webpackChunkName:"lazyComponent" */ "./lazy"), 2000)
// );

const Lazy = lazy(() =>
  import(/* webpackChunkName:"lazyComponent" */ "./lazy")
);

class LazySuspence extends Component {
  state = {
    hasError: false
  };
  //   componentDidCatch() {
  //     this.setState({
  //       hasError: true
  //     });
  //   }
  static getDerivedStateFromError() {
    return {
      hasError: true
    };
  }
  render() {
    if (this.state.hasError) {
      return <div>Component Error</div>;
    }
    return (
      <div>
        <Suspense fallback={<div>Loading</div>}>
          <Lazy />
        </Suspense>
      </div>
    );
  }
}

export default LazySuspence;
```

## Memo,shouldComponentUpdate,PureComponent

当组件复杂化，拥有状态变多时，我们需要设计一种模式，对所有的 props 变量和 state 变量，做一个“shallow comparison(浅比较）”，这样会使得 SCU 函数冗杂化，为了解决该问题，React 给了我们提供了生命周期方法`ShouldComponentUpdate`，该方法需返回比对后的`bool`值以及另一个继承方法`React.PureComponent`,该方法只能比较变量，而对于对象无效，解决方法是组件赋值生成一个新的对象

```js
import React, { Component, PureComponent, memo } from "react";

const MemoFunc = memo(function MemoFunc(props) {
  return <h1>{props.num}</h1>;
});

class MemoShow extends Component {
  //   shouldComponentUpdate(nextProps, nextState) {
  //     //   console.log(nextProps , this.props)
  //     if (nextProps.num === this.props.num) {
  //       return false;
  //     }
  //     return true;
  //   }
  render() {
    console.log("render");
    return (
      <>
        {/* <h1>{this.props.person.age}</h1> */}
        {/* <h2>{this.props.num}</h2> */}
      </>
    );
  }
}

class MemoDemo extends Component {
  state = {
    num: 99,
    person: {
      age: 1
    }
  };

  callback = () => {};

  render() {
    const person = this.state.person;
    return (
      <>
        <button
          type="button"
          onClick={() => {
            person.age++;
            this.setState({ num: this.state.num });
          }}
          //   onClick={() => {
          //     person.age++;
          //     this.setState({ person });
          //   }}
        >
          press
        </button>
        <MemoShow
          // num={this.state.num}
          //   person={person}
          cb={this.callback}
        />
        <MemoFunc num={this.state.num} />
      </>
    );
  }
}

export default MemoDemo;
```

## React Hooks

- 类组件的不足

  - 状态逻辑难以复用

  > 缺少复用机制
  > 渲染属性和高阶组件导致层级冗余

  - 趋向复杂难以维护

  > 生命周期函数混杂不相干逻辑
  > 相关逻辑分布在不同的声明周期

  - this 指向困扰

  > 内联函数过度创建新句柄
  > 类成员函数不能保证 this

- Hooks 的优势

> 优化类组件的三大问题

1. 函数组件无 this 问题
2. 自定义 Hook 方便复用状态逻辑
3. 副作用的关注点分离

### useState()

更新 API，使用 demo 如下

```js
import React, { Component, useState } from "react";

class HooksDemo extends Component {
  state = {
    count: 0
  };
  render() {
    const { count } = this.state;
    return (
      <>
        <button
          type="button"
          onClick={() => {
            this.setState({ count: count + 1 });
          }}
        >
          Click({count})
        </button>
        <StateDemo />
      </>
    );
  }
}

function StateDemo(props) {
  // const [count, setCount] = useState(0);
  // useState默认值也可传入一个函数,但也只在组件初始渲染时赋值一次
  const [count, setCount] = useState(() => {
    return props.defualtCount || 0;
  });

  return (
    <button
      type="button"
      onClick={() => {
        setCount(count + 1);
      }}
    >
      Click({count})
    </button>
  );
}

export default HooksDemo;
```

### useEffect()

```js
import React, { useState, useEffect } from "react";

function HooksDemo() {
  const [count, setCount] = useState(0);
  const [size, setSize] = useState({
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight
  });

  const onResize = () => {
    setSize({
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight
    });
  };

  useEffect(() => {
    document.title = count;
  });

  useEffect(() => {
    window.addEventListener("resize", onResize, false);
    // 回调函数在视图被销毁之前触发
    return () => {
      window.removeEventListener("resize", this.onResize, false);
    };
    // 第二个参数为一个数组，只有数组中的每一项都没有发生变化时，useEffect才不会执行
  }, []);

  useEffect(() => {
    document.querySelector("#size").addEventListener("click", onClick);
    return () => {
      document.querySelector("#size").removeEventListener("click", onClick);
    };
  });

  const onClick = () => {
    console.log("click");
  };

  return (
    <>
      <button
        type="button"
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click({count})
      </button>
      {count % 2 ? (
        <span id="size">
          size:({size.width}-{size.height})
        </span>
      ) : (
        <p id="size">
          size:({size.width}-{size.height})
        </p>
      )}
    </>
  );
}

export default HooksDemo;
```

### useContext

```js
import React, { useState, createContext, useContext } from "react";

const CountContext = createContext();

function Counter() {
  const count = useContext(CountContext);
  return <h1>{count}</h1>;
}

function HooksDemo() {
  const [count, setCount] = useState(0);
  return (
    <>
      <button
        type="button"
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click({count})
      </button>
      <CountContext.Provider value={count}>
        <Counter />
      </CountContext.Provider>
    </>
  );
}

export default HooksDemo;
```

### useMemo/useCallback

> useMemo(() => fn) 等价于 useCallback(fn)
> useMemo()的返回值如果是函数的话，就可以简写成 useCallback()形式
> useCallback 解决的是传入子组件的函数不过多变化，而导致子组件重新渲染的问题

```js
import React, { useState, useMemo, memo, useCallback } from "react";

const Counter = memo(function Counter(props) {
  console.log("render");
  return <h1 onClick={props.onClick}>'----'{props.count}</h1>;
});

function HooksDemo() {
  const [count, setCount] = useState(0);
  const [clickCount, setClickCount] = useState(0);

  // const double = useMemo(() => {
  //   return count * 2;
  // }, [count]);

  // const onClick = useMemo(() => {
  //   return () => {
  //     console.log("Click");
  //   };
  // }, []);

  const onClick = useCallback(() => {
    console.log("Click", clickCount);
    setClickCount(clickCount + 1);
  }, [clickCount]);

  return (
    <>
      <button
        type="button"
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click({count})
      </button>
      <Counter
        // double={double}
        onClick={onClick}
      />
    </>
  );
}

export default HooksDemo;
```

### Ref Hooks

> 获取子组件或者 DOM 节点的句柄
> 渲染周期之间共享数据的存储

```js
import React, {
  useState,
  useMemo,
  useRef,
  useCallback,
  useEffect,
  PureComponent
} from "react";

class Counter extends PureComponent {
  render() {
    const { props } = this;
    return <h1 onClick={props.onClick}>'----'{props.count}</h1>;
  }
}

function HooksDemo() {
  const [count, setCount] = useState(0);
  const [clickCount, setClickCount] = useState(0);
  const counterRef = useRef();
  let it = useRef();

  const double = useMemo(() => {
    return count * 2;
  }, [count]);

  useEffect(() => {
    it.current = setInterval(() => {
      setCount(count => count + 1);
    }, 1000);
  }, []);

  useEffect(() => {
    if (count >= 10) {
      clearInterval(it.current);
    }
  });

  const onClick = useCallback(() => {
    console.log(counterRef);
    setClickCount(clickCount => clickCount + 1);
  }, [counterRef]);

  return (
    <>
      <button
        type="button"
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click({count})
      </button>
      <Counter
        ref={counterRef}
        // double={double}
        onClick={onClick}
      />
    </>
  );
}

export default HooksDemo;
```

### 自定义 Hooks

```js
import React, { useState, useRef, useEffect, useCallback } from "react";

function useCounter(count) {
  const size = useSize();
  return (
    <h1>
      {count}
      {size.width}*{size.height}
    </h1>
  );
}

function useCount(defaultCount) {
  const [count, setCount] = useState(defaultCount);
  let it = useRef();

  useEffect(() => {
    it.current = setInterval(() => {
      setCount(count => count + 1);
    }, 1000);
  }, []);

  useEffect(() => {
    if (count >= 10) {
      clearInterval(it.current);
    }
  });
  return [count, setCount];
}

function useSize() {
  const [size, setSize] = useState({
    width: document.documentElement.clientWidth,
    height: document.documentElement.clientHeight
  });

  const onResize = useCallback(() => {
    setSize({
      width: document.documentElement.clientWidth,
      height: document.documentElement.clientHeight
    });
  }, []);

  useEffect(() => {
    window.addEventListener("resize", onResize, false);
    return () => {
      window.removeEventListener("resize", onResize, false);
    };
  }, [onResize]);

  return size;
}

function HooksDemo() {
  const [count, setCount] = useCount(0);
  const Counter = useCounter(count);
  const size = useSize();
  return (
    <>
      <button
        type="button"
        onClick={() => {
          setCount(count + 1);
        }}
      >
        Click({count})
      </button>
      <h2>{Counter}</h2>
      <h4>
        {size.width}*{size.height}
      </h4>
    </>
  );
}

export default HooksDemo;
```

## HOOKS 的规则

[HOOK 的规则]('https://react.docschina.org/docs/hooks-rules.html')

## HOOKS 的常见问题

1. 生命周期函数如何映射到 Hooks？
2. 类实例成员变量如何映射到 Hooks?
   `const it = useRef(0)` 不可传入函数
3. Hooks 中如何获取历史 props 和 state?
   通过`Ref`进行存储
4. 如何强制更新一个 Hooks 组件?
   主动创建一个不参与实际渲染的 state，然后强制更新 state 而使 hooks 渲染
