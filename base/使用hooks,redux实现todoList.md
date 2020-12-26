# 使用 hooks,redux 实现 todoList

拆分成 4 个文件

- `index.js`
- `reducers.js`
- `actions.js`
- `index.css`

## `index.js`

```js
import React, { useEffect, useState, memo, useRef, useCallback } from "react";
import { createSet, createAdd, createRemove, createToggle } from "./actions";
import reducer from "./reducers";
import "./index.css";

function bindActionCreators(actionCreators, dispatch) {
  const ret = {};
  for (let key in actionCreators) {
    ret[key] = function(...args) {
      const actionCreator = actionCreators[key];
      const action = actionCreator(...args);
      dispatch(action);
    };
  }
  return ret;
}

const Control = memo(function Control(props) {
  const { addTodo } = props;
  const inputRef = useRef();
  const onSubmit = e => {
    e.preventDefault();
    const newText = inputRef.current.value.trim();
    if (newText.length === 0) {
      return;
    }
    addTodo(newText);
    inputRef.current.value = "";
  };
  return (
    <div className="control">
      <h1>todos</h1>
      <form onSubmit={onSubmit}>
        <input
          type="text"
          ref={inputRef}
          className="new-todo"
          placeholder="what needs to be done"
        />
      </form>
    </div>
  );
});

const TodoItem = memo(function TodoItem(props) {
  const {
    todo: { id, text, complete },
    removeTodo,
    toggleTodo
  } = props;
  const onChange = () => {
    toggleTodo(id);
  };
  const onRemove = () => {
    removeTodo(id);
  };
  return (
    <li className="todo_item">
      <input type="checkbox" onChange={onChange} checked={complete} />
      <label className={complete ? "complete" : ""}>{text}</label>
      <button onClick={onRemove}>X</button>
    </li>
  );
});

const Todos = memo(function Todos(props) {
  const { todos, removeTodo, toggleTodo } = props;
  return (
    <ul>
      {todos.map(todo => {
        return (
          <TodoItem
            key={todo.id}
            todo={todo}
            removeTodo={removeTodo}
            toggleTodo={toggleTodo}
          />
        );
      })}
    </ul>
  );
});

const LS_KEY = "$todos";
let store = {
  todos: [],
  incrementCount: 0
};

function TodoList() {
  const [todos, setTodos] = useState([]);
  const [incrementCount, setIncrementCount] = useState(0);

  // 使用store确保reducer和aciton函数执行时拿到的是最新的state
  useEffect(() => {
    Object.assign(store, {
      todos,
      incrementCount
    });
  }, [todos, incrementCount]);

  const dispatch = useCallback(action => {
    const setters = {
      todos: setTodos,
      incrementCount: setIncrementCount
    };
    if ("function" === typeof action) {
      action(dispatch, () => store);
      return;
    }
    const newState = reducer(store, action);
    for (let key in newState) {
      setters[key](newState[key]);
    }
  }, []);

  useEffect(() => {
    const todos = JSON.parse(localStorage.getItem(LS_KEY));
    dispatch(createSet(todos));
  }, [dispatch]);

  useEffect(() => {
    localStorage.setItem(LS_KEY, JSON.stringify(todos));
  }, [todos]);

  return (
    <div className="todo_list">
      <Control
        {...bindActionCreators(
          {
            addTodo: createAdd
          },
          dispatch
        )}
      />
      <Todos
        {...bindActionCreators(
          {
            removeTodo: createRemove,
            toggleTodo: createToggle
          },
          dispatch
        )}
        todos={todos}
      />
    </div>
  );
}

export default TodoList;
```

## `reducers.js`

```js
function combineReducers(reducers) {
  return function reducer(state, action) {
    const changed = {};
    for (let key in reducers) {
      changed[key] = reducers[key](state[key], action);
    }
    return {
      ...state,
      ...changed
    };
  };
}

const reducers = {
  todos(state, action) {
    const { type, payload } = action;
    switch (type) {
      case "set":
        return payload;
      case "add":
        return [...state, payload];
      case "remove":
        return state.filter(todo => {
          return todo.id !== payload;
        });
      case "toggle":
        return state.map(todo => {
          return todo.id === payload
            ? {
                ...todo,
                complete: !todo.complete
              }
            : todo;
        });
      default:
    }
    return state;
  },
  incrementCount(state, action) {
    const { type } = action;
    switch (type) {
      case "set":
      case "add":
        return state + 1;
      default:
    }
    return state;
  }
};

export default combineReducers(reducers);
```

### `actions.js`

```js
export function createSet(payload) {
  return {
    type: "set",
    payload
  };
}

let idSeq = Date.now();

export function createAdd(text) {
  return (dispatch, getState) => {
    setTimeout(() => {
      const { todos } = getState();
      if (!todos.find(todo => todo.text === text)) {
        dispatch({
          type: "add",
          payload: {
            id: ++idSeq,
            text,
            complete: false
          }
        });
      }
    }, 1000);
  };
}

export function createRemove(payload) {
  return {
    type: "remove",
    payload
  };
}

export function createToggle(payload) {
  return {
    type: "toggle",
    payload
  };
}
```

## index.css

```css
.todo_list {
  width: 550px;
  margin: 130px auto;
  background: #fff;
  box-shadow: 0px 0px 50px 0px rgba(0, 0, 0, 0.5);
}

.todo_list ul {
  padding: 0;
}

.control h1 {
  width: 100%;
  font-size: 100px;
  text-align: center;
  margin: 0;
  color: rgba(175, 47, 47, 0.15);
}

.control .new-todo {
  padding: 16px 16px 16px 60px;
  border: 0;
  outline: none;
  font-size: 24px;
  box-sizing: border-box;
  width: 100%;
  line-height: 14rm;
  box-shadow: inset 0 -2px 1px rgba(0, 0, 0, 0.3);
}

.todos {
  margin: 0;
  padding: 0;
  list-style: none;
  font-size: 24px;
  display: flex;
  align-items: center;
}

.todo_item {
  display: flex;
  align-items: center;
}

.todo_item input {
  display: block;
  width: 20px;
  height: 20px;
  margin: 0 20px;
}

.todo_item label {
  flex: 1;
  padding: 15px 15px 15px 0;
  line-height: 1.2em;
  display: block;
}

.todo_item label.complete {
  text-decoration: line-through;
}

.todo_item button {
  border: 0;
  outline: 0;
  display: block;
  width: 40px;
  text-align: center;
  font-size: 30px;
  color: #cc9a9a;
  background: #fff;
}
```
