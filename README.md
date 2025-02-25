# testcafe-react-selectors

This plugin provides selector extensions that make it easier to test ReactJS components with [TestCafe](https://github.com/DevExpress/testcafe). These extensions allow you to select page elements in a way that is native to React.

## Install

`$ npm install testcafe-react-selectors`

## Usage

### Wait for application to be ready to run tests

To wait until the React's component tree is loaded, add the `waitForReact` method to fixture's `beforeEach` hook.

```js
import { waitForReact } from 'testcafe-react-selectors';

fixture `App tests`
    .page('http://react-app-url')
    .beforeEach(async () => {
        await waitForReact();
    });
```

The default timeout for `waitForReact` is `10000` ms. You can specify a custom timeout value:

```js
await waitForReact(5000);
```

If you need to call a selector from a Node.js callback, pass the current test controller as the second argument in the `waitForReact` function:

```js
import { waitForReact } from 'testcafe-react-selectors';

fixture `App tests`
    .page('http://react-app-url')
    .beforeEach(async t => {
        await waitForReact(5000, t);
    });
``` 

The test controller will be assigned to the [boundTestRun](https://devexpress.github.io/testcafe/documentation/test-api/obtaining-data-from-the-client/#optionsboundtestrun) function's option. Otherwise, TestCafe would throw the following error: `ClientFunction cannot implicitly resolve the test run in context of which it should be executed`. See the [TestCafe documentation](https://devexpress.github.io/testcafe/documentation/test-api/obtaining-data-from-the-client/#calling-client-functions-from-nodejs-callbacks) for further details.

### Creating selectors for ReactJS components

`ReactSelector` allows you to select page elements by the name of the component class or the nested component element.

Suppose you have the following JSX.

```xml
<TodoApp className="todo-app">
    <TodoInput />
    <TodoList>
        <TodoItem priority="High" key="HighPriority">Item 1</TodoItem>
        <TodoItem priority="Low" key="LowPriority">Item 2</TodoItem>
    </TodoList>

    <div className="items-count">Items count: <span>{this.state.itemCount}</span></div>
</TodoApp>
```

#### Selecting elements by the component name

To get a root DOM element for a component, pass the component name to the `ReactSelector` constructor.

```js
import { ReactSelector } from 'testcafe-react-selectors';

const todoInput = ReactSelector('TodoInput');
```

#### Selecting nested components

To obtain a nested component or DOM element, you can use a combined selector or add DOM element's tag name.

```js
import { ReactSelector } from 'testcafe-react-selectors';

const TodoList         = ReactSelector('TodoApp TodoList');
const itemsCountStatus = ReactSelector('TodoApp div');
const itemsCount       = ReactSelector('TodoApp div span');
```

Warning: if you specify a DOM element's tag name, React selectors search for the element among the component's children without looking into nested components. For instance, for the JSX above the `ReactSelector('TodoApp div')` selector will be equal to `Selector('.todo-app > div')`.

#### Selecting components by the component key

To obtain a component by its key, use the `withKey` method. 

```js
import { ReactSelector } from 'testcafe-react-selectors';

const item = ReactSelector('TodoItem').withKey('HighPriority');
```

#### Selecting components by property values

React selectors allow you to select elements that have a specific property value. To do this, use the `withProps` method. You can pass the property and its value as two parameters or an object.

```js
import { ReactSelector } from 'testcafe-react-selectors';

const item1 = ReactSelector('TodoApp').withProps('priority', 'High');
const item2 = ReactSelector('TodoApp').withProps({ priority: 'Low' });
```

You can also search for elements by multiple properties.

```js
import { ReactSelector } from 'testcafe-react-selectors';

const element = ReactSelector('componentName').withProps({
    propName: 'value',
    anotherPropName: 'differentValue'
});
```

##### Properties whose values are objects

React selectors allow you to filter components by properties whose values are objects.

When the `withProps` function filters properties, it determines whether the objects (property values) are strictly or partially equal.

The following example illustrates strict and partial equality.

```js
object1 = {
    field1: 1
}
object2 = {
    field1: 1
}
object3 = {
    field1: 1
    field2: 2
}
object4 = {
    field1: 3
    field2: 2
}
```

* `object1` strictly equals `object2`
* `object2` partially equals `object3`
* `object2` does not equal `object4`
* `object3` does not equal `object4`

Prior to version 3.0.0, `withProps` checked if objects are strictly equal when comparing them. Since 3.0.0, `withProps` checks for partial equality. To test objects for strict equality, specify the `exactObjectMatch` option.

The following example returns the `componentName` component because the `objProp` property values are strictly equal and `exactObjectMatch` is set to true.

```js
// props = {
//   simpleProp: 'value',
//   objProp: {
//       field1: 'value',
//       field2: 'value'
//   }
// }

const element = ReactSelector('componentName').withProps({
    simpleProp: 'value',
    objProp: {
        field1: 'value',
        field2: 'value'
    }
}, { exactObjectMatch: true })
```

Note that the partial equality check works for objects of any depth.

```js
// props = {
//     simpleProp: 'value',
//     objProp: {
//         field1: 'value',
//         field2: 'value',
//         nested1: {
//             someField: 'someValue',
//             nested2: {
//                 someField: 'someValue',
//                 nested3: {
//                     field: 'value',
//                     someField: 'someValue'
//                 }
//             }
//         }
//     }
// }


const element = ReactSelector('componentName').withProps({
    simpleProp: 'value',
    objProp: {
        field1: 'value',
        nested1: {
            nested2: {
                nested3: {
                    field: 'value'
                }
            }
        }
    }
}, { exactObjectMatch: false })
```

#### Searching for nested components

You can search for a desired subcomponent or DOM element among the component's children using the `.findReact(element)` method. The method takes the subcomponent name or tag name as a parameter.

Suppose you have the following JSX.

```xml
<TodoApp className="todo-app">
    <div>
        <TodoList>
            <TodoItem priority="High">Item 1</TodoItem>
            <TodoItem priority="Low">Item 2</TodoItem>
        </TodoList>
    </div>
</TodoApp>
```

The following sample demonstrates how to obtain the `TodoItem` subcomponent.

```js
import { ReactSelector } from 'testcafe-react-selectors';

const component    = ReactSelector('TodoApp');
const div          = component.findReact('div');
const subComponent = div.findReact('TodoItem');
```

You can call the `.findReact` method in a chain, for example:

```js
import { ReactSelector } from 'testcafe-react-selectors';

const subComponent = ReactSelector('TodoApp').findReact('div').findReact('TodoItem');
```

You can also combine `.findReact` with regular selectors and [other](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#functional-style-selectors)) methods like [.find](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#find) or [.withText](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#withtext), for example:

```js
import { ReactSelector } from 'testcafe-react-selectors';

const subComponent = ReactSelector('TodoApp').find('div').findReact('TodoItem');
```

#### Combining with regular TestCafe selectors

Selectors returned by the `ReactSelector` constructor are recognized as TestCafe selectors. You can combine them with regular selectors and filter with [.withText](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#withtext), [.nth](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#nth), [.find](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#find) and [other](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/selectors.html#functional-style-selectors) functions. To search for elements within a component, you can use the following combined approach.

```js
import { ReactSelector } from 'testcafe-react-selectors';

var itemsCount = ReactSelector('TodoApp').find('.items-count span');
```

**Example**

Let's use the API described above to add a task to a Todo list and check that the number of items changed.

```js
import { ReactSelector } from 'testcafe-react-selectors';

fixture `TODO list test`
	.page('http://localhost:1337');

test('Add new task', async t => {
    const todoTextInput = ReactSelector('TodoInput');
    const todoItem      = ReactSelector('TodoList TodoItem');

    await t
        .typeText(todoTextInput, 'My Item')
        .pressKey('enter')
        .expect(todoItem.count).eql(3);
});
```

### Obtaining component's props and state

As an alternative to [testcafe snapshot properties](http://devexpress.github.io/testcafe/documentation/test-api/selecting-page-elements/dom-node-state.html), you can obtain `state`, `props` or `key` of a ReactJS component.

To obtain component's properties, state and key, use the React selector's `.getReact()` method.

The `.getReact()` method returns a [client function](https://devexpress.github.io/testcafe/documentation/test-api/obtaining-data-from-the-client.html). This function resolves to an object that contains component's properties (excluding properties of its `children`), state and key.

```js
const reactComponent      = ReactSelector('MyComponent');
const reactComponentState = await reactComponent.getReact();

// >> reactComponentState
//
// {
//     props:    <component_props>,
//     state:    <component_state>,
//     key:      <component_key>
// }
```

The returned client function can be passed to assertions activating the [Smart Assertion Query mechanism](https://devexpress.github.io/testcafe/documentation/test-api/assertions/#smart-assertion-query-mechanism).

**Example**

```js
import { ReactSelector } from 'testcafe-react-selectors';

fixture `TODO list test`
	.page('http://localhost:1337');

test('Check list item', async t => {
    const el         = ReactSelector('TodoList');
    const component  = await el.getReact();

    await t.expect(component.props.priority).eql('High');
    await t.expect(component.state.isActive).eql(false);
    await t.expect(component.key).eql('componentID');
});
```

As an alternative, the `.getReact()` method can take a function that returns the required property, state or key. This function acts as a filter. Its argument is an object returned by `.getReact()`, i.e. `{ props: ..., state: ..., key: ...}`.

```js
ReactSelector('Component').getReact(({ props, state, key }) => {...})
```

**Example**

```js
import { ReactSelector } from 'testcafe-react-selectors';

fixture `TODO list test`
    .page('http://localhost:1337');

test('Check list item', async t => {
    const el = ReactSelector('TodoList');

    await t
        .expect(el.getReact(({ props }) => props.priority)).eql('High')
        .expect(el.getReact(({ state }) => state.isActive)).eql(false)
        .expect(el.getReact(({ key }) => key)).eql('componentID');
});
```

The `.getReact()` method can be called for the `ReactSelector` or the snapshot this selector returns.

### TypeScript Generic Selector

Use the generic `ReactComponent` type to create scalable selectors in TypeScript.

Pass the `props` object as the type argument to `ReactComponent` to introduce a type for a specific component.

```ts
type TodoItem = ReactComponent<{ id: string }>;
```

You can then pass the created `TodoItem` type to the `withProps` and `getReact` generic methods.

```ts
const component  = ReactSelector('TodoItem');
type TodoItem    = ReactComponent<{ id: string }>;

const item1  = component.withProps<TodoItem>('id', 'tdi-1');
const itemId = component.getReact<TodoItem>(({ props }) => props.id);
```

**Example**

``` ts
import { ReactSelector, ReactComponent } from 'testcafe-react-selectors';

fixture`typescript support`
    .page('http://react-page-example.com')

test('ReactComponent', async t => {
    const todoList         = ReactSelector('TodoList');
    type TodoListComponent = ReactComponent<{ id: string }>;

    const todoListId = todoList.getReact<TodoListComponent>(({ props }) => props.id);

    await t.expect(todoListId).eql('ul-item');
});
```

#### Composite Types in Props and State

If a component's props and state include other composite types, you can create your own type definitions for them. Then pass these definitions to `ReactComponent` as type arguments.

The following example shows custom `Props` and `State` type definitions. The `State` type uses another composite type - `Option`.

``` ts
import { ReactComponent } from 'testcafe-react-selectors';

interface Props {
    id: string;
    text: string;
}

interface Option {
    id: number;
    title: string;
    description: string;
}

interface State {
    optionsCount: number;
    options: Option[];
}

export type OptionReactComponent = ReactComponent<Props, State>;
```

### Limitations

* `testcafe-react-selectors` support ReactJS starting with version 15. To check if a component can be found, use the [react-dev-tools](https://chrome.google.com/webstore/detail/react-developer-tools/fmkadmapgofadopljbjfkapdkoienihi) extension.
* Search for a component starts from the root React component, so selectors like `ReactSelector('body MyComponent')` will return `null`.
* ReactSelectors need class names to select components on the page. Code minification usually does not keep the original class names. So you should either use non-minified code or configure the minificator to keep class names.

  For `babel-minify`, add the following options to the configuration:

  ```js
  { keepClassName: true, keepFnName: true }
  ```

  In UglifyJS, use the following configuration:

   ```js
   {
       compress: {
           keep_fnames: true
       },

       mangle: {
           keep_fnames: true
       }
   }
   ```
