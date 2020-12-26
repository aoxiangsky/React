# 使用 hooks 实现一个 TodoList

```js
import React, { useEffect, useState, memo, useRef, useCallback } from "react";
import "./index.css";

let idSeq = Date.now();

const Control = memo(function Control(props) {
  const { addTodo } = props;
  const inputRef = useRef();
  const onSubmit = e => {
    e.preventDefault();
    const newText = inputRef.current.value.trim();
    if (newText.length === 0) {
      return;
    }
    addTodo({
      id: ++idSeq,
      text: newText,
      complete: false
    });
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
    toggleTodo,
    removeTodo
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
  const { todos, toggleTodo, removeTodo } = props;
  return (
    <ul>
      {todos.map(todo => {
        return (
          <TodoItem
            key={todo.id}
            todo={todo}
            toggleTodo={toggleTodo}
            removeTodo={removeTodo}
          />
        );
      })}
    </ul>
  );
});

const LS_KEY = "$todos";

function TodoList() {
  const [todos, setTodos] = useState([]);
  const addTodo = useCallback(todo => {
    setTodos(todos => [...todos, todo]);
  }, []);
  const removeTodo = useCallback(id => {
    setTodos(todos =>
      todos.filter(todo => {
        return todo.id !== id;
      })
    );
  }, []);
  const toggleTodo = useCallback(id => {
    setTodos(todos =>
      todos.map(todo => {
        return todo.id === id
          ? {
              ...todo,
              complete: !todo.complete
            }
          : todo;
      })
    );
  }, []);
  useEffect(() => {
    const todos = JSON.parse(localStorage.getItem(LS_KEY));
    setTodos(todos);
  }, []);
  useEffect(() => {
    localStorage.setItem(LS_KEY, JSON.stringify(todos));
  }, [todos]);
  return (
    <div className="todo_list">
      <Control addTodo={addTodo} />
      <Todos removeTodo={removeTodo} toggleTodo={toggleTodo} todos={todos} />
    </div>
  );
}

export default TodoList;
```

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
