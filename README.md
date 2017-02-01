# @stugotech/handler-decorator

Sort of like overloading, but more _dynamic_.
Allows you to create a single handler function from an instance of a class, marked up to handle different situations.
TypeScript definitions are built in.

## Installation

    $ npm install --save handler-decorator

## Example

A good use of this could be to handle [Redux](http://redux.js.org/) actions.

```js
import HandlerBuilder, { handler, defaultHandler } from 'handler-decorator';

let reducerBuilder = new HandlerBuilder<string>(
  // how to decide what a call is for
  (state, action) => action.type,
  // what to do in the default case
  (state, action) => state
);

// define a class with handlers in it
class TodoReducers {
  @handler('ADD_TODO')
  addTodo(state, action) {
    return [...state, {id: action.id, text: action.text, completed: action.completed}];
  }

  @handler('REMOVE_TODO')
  removeTodo(state, action) {
    return state.filter((todo) => todo.id !== action.id);
  }
}

// build a function which will call the correct method on TodoReducers
let todoReducer = reducerBuilder.build(new TodoReducers());
let newState = todoReducer(state, action);
```


## API

### `HandlerBuilder` class

This is the main class.

#### `constructor(selector, defaultValue)`

Constructor.  The `selector` parameter is a function which takes the same arguments as the resulting handler function, and returns a
value that descriminates which handler to call.

The `defaultParameter` value says what to do in the event that a matching handler is not found.  If no value was provided, and there is
no method marked with `defaultHandler`, then an exception is thrown.  If a value is provided, that value is returned.  If a function is
provided, then the function is called with the arguments supplied to the resulting handler function, and the value returned is what is
returned to the caller.

#### `build(target)`

Returns a handler function which decides which handler method on the target to call, calls it, and returns the result.

### Decorators

#### `@handler(key)`

Marks the method as handling the specified descriminator value.  I.e., when the handler function is called, it calls the `selector`
function passed to the constructor, and if the value returned is the same as `key`, then this method is called with the same arguments.

#### `@defaultHandler`

Marks the method as handling the case where no other matching handler is found.  This will take precedence over any `defaultValue` passed
to the constructor.
