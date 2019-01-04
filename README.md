<p align="center">
  <img src="http://svgshare.com/i/3tk.svg"><br/>
  <h2 align="center">React HotKeys</h2>
</p>

[![npm](https://img.shields.io/npm/dm/react-hotkeys.svg)]()
[![Build Status](https://travis-ci.org/greena13/react-hotkeys.svg)](https://travis-ci.org/greena13/react-hotkeys)
[![GitHub license](https://img.shields.io/github/license/greena13/react-hotkeys.svg)](https://github.com/greena13/react-hotkeys/blob/master/LICENSE)
[![Gitter](https://img.shields.io/gitter/room/chrisui/react-hotkeys.svg)](https://gitter.im/chrisui/react-hotkeys)

A declarative library for handling hotkeys and focus areas in React applications.

## Feature Overview

- Offers a minimal declarative [JSX](#HotKeys-component-API) and [HoC](#withHotKeys-HoC-API) APIs
- Supports [browser key names](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) and [Mousetrap syntax](https://github.com/ccampbell/mousetrap)
- Allows you to define [global](#GlobalHotKeys-component) and [in-focus](#HotKeys-component) hot keys
- Works with React's Synthetic KeyboardEvents and event delegation and provides [predictable and expected behaviour](#Interaction-with-React) to anyone  familiar with React
- It's customizable through a simple [configuration API](#Configuration)
- [Optimized for larger applications](#Optimizations), with many hot keys active at once
- Depends only on `prop-types` and a peer dependency of `react`
- Uses rollup, Uglify and strips out comments and logging for a small production build
- Has more than [1800 automated tests](/tests)

## Basic Usage

#### Define a key map

```javascript
import {HotKeys} from 'react-hotkeys';
import MyNode from './MyNode';

const keyMap = {
  SNAP_LEFT: 'command+left',
  DELETE_NODE: ['del', 'backspace']
};

const App = React.createClass({
  render() {
    return (
      <HotKeys keyMap={keyMap}>
        <div>
          <MyNode />
          <MyNode />
        </div>
      </HotKeys>
    );
  }
});

export default App;
```

#### Define handlers

```javascript
import {HotKeys} from 'react-hotkeys';

const MyNode = React.createClass({
  render() {
    const handlers = {
      DELETE_NODE: this.deleteNode
    };

    return (
      <HotKeys handlers={handlers}>
        Node contents
      </HotKeys>
    );
  }
});

export default MyNode;
```

## Install

### CommonJS & ES6 Modules

`react-hotkeys` is available as a CommonJS or a ES6 Modules through npm or yarn. It uses `NODE_ENV` to determine whether to export the development or production build in your library or application.

It is expected you will use a bundling tool like Webpack or Uglify to remove the version of the bundle you are not using with each version of your application's code, to keep the library size to a minimum.

#### npm

```
npm install react-hotkeys --save
```

#### yarn

```
yarn add react-hotkeys
```

### UMD

`react-hotkeys` as a UMD module is available on your CDN of choice.

Change `1.0.1` for the version that you would like to use.

#### Development build

```
<script crossorigin src="https://cdn.jsdelivr.net/npm/react-hotkeys@1.0.1/umd/react-hotkeys.js"></script>
```

```
<script crossorigin src="https://unpkg.com/react-hotkeys@1.0.1/umd/react-hotkeys.js"></script>
```

#### Minified production build

```
<script crossorigin src="https://cdn.jsdelivr.net/npm/react-hotkeys@1.0.1/umd/react-hotkeys.min.js"></script>
```

```
<script crossorigin src="https://unpkg.com/react-hotkeys@1.0.1/umd/react-hotkeys.min.js"></script>
```

### Bower

Bower support was removed in `v1.0.0`, but those who already rely on earlier versions of `react-hotkeys` through Bower can continue to do so using the following command:

```
bower install react-hotkeys@0.10.0
```

The Bower version of the package will **not** be supported going forward (including fixing any outstanding issues).

## Defining key maps

`react-hotkeys` uses key maps to decouple defining keyboard shortcuts from the functions they call. This allows hot keys and handler functions to be defined and maintained independent of one another.

> When a user presses the corresponding combination or sequence of keys, it is said they *match* the hot keys, which causes an action to be *triggered*. `react-hotkeys` may then resolve an appropriate handler function to *handle* the action.

Key maps are Plain Old JavaScript Objects, where the keys are the action names and the values are usually a [Mousetrap-supported](https://craig.is/killing/mice) or [Browser Key Values](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) sequence string (but can also be an [array](#Alternative-Hotkeys) or an [object](#Key-Combination-vs-Sequences)) that must be matched in order to trigger the action.

```javascript
const keyMap = {
  'deleteNode': 'del',
  'moveUp': 'up'
};
```

#### Key Combinations vs Sequences

Every hotkey or sequence string is parsed and treated as a sequence of key combinations. The simplest case is a sequence of 1 key combination, consisting of 1 key: e.g. `'a'` or `'shift'`.

```
// Key sequence with a combination of a single key
'4'

// Special single key sequence (ie. shift is handled automagically)
'?'

// Sequence of a single combination with multiple keys (keys must be pressed at the same time)
'command+shift+k'

// Sequence of multiple combinations (keys must be pressed and released one after another)
'up down left right'
```

#### Full Reference

Please refer to [Mousetrap's documentation](https://craig.is/killing/mice) or [Browser Key Values](https://developer.mozilla.org/en-US/docs/Web/API/KeyboardEvent/key/Key_Values) for an exhaustive list of supported shortcuts and sequences.

#### Alternative Hotkeys

You can specify multiple *alternative* key sequences (they will trigger the same action) using arrays:

```javascript
const keyMap = {
  DELETE_NODE: ['del', 'backspace'],
  MOVE_UP: ['up', 'w']
};
```
#### Specifying key events (keydown, keypress, keyup)

By default, `react-hotkeys` will match hotkey sequences on the `keypress` event (or, more precisely: on the `keypress` event of the last key to complete the last combination in a sequence).

If you want to trigger a single action on a different key event, you can use the object syntax and the `action` attribute to explicitly set which key event you wish to bind to:

```javascript
const keyMap = {
  CONTRACT: 'alt+down',
  COMMAND_DOWN: {sequence: 'command', action: 'keydown'},
};
```

If you want to change the default key event for all hotkeys, you can use the `defaultKeyEvent` option of the [configuration API](#Configuration).

The full list of valid key events is: `keypress`, `keydown`, and `keyup`.

#### Deciding which key map syntax to use

As a general rule, you should use the syntax that is the most brief, but still allows you to express the configuration you want.

| Question | Yes | No |
| :--- | :--- | :--- |
| **Need to define alternative key sequences to trigger the same action?** | Use an array of strings or objects. | Use a string or object. |
| **Need to explicitly define the key event to bind to (or some other additional option)?** | Use an object. | Use a string. |

## Defining Handlers

Key maps trigger actions when they match a key sequence. Handlers are the functions that `react-hotkeys` calls to handle those actions.

Handlers may be defined in the same `<HotKeys />` component as the key map:

```javascript
import {HotKeys} from 'react-hotkeys';

const keyMap = {
    MOVE_UP: 'up',
}

const handlers = {
  MOVE_UP: (event) => console.log('Move up hotkey called!')
};

<HotKeys keyMap={keyMap} handlers={handlers}>
  <input />
</HotKeys>
```

Or they may be defined in any *descendant* of the `<HotKeys />` component that defines the key map:

 ```javascript
 import {HotKeys} from 'react-hotkeys';

 const keyMap = {
     MOVE_UP: 'up',
 }

 const handlers = {
   MOVE_UP: (event) => console.log('Move up hotkey called!')
 };

 <HotKeys keyMap={keyMap}>
   <div>
    <HotKeys handlers={handlers}>
      <input />
    </HotKeys>
   </div>

   <div>
    <input />
   </div>
 </HotKeys>
 ```

#### DEPRECATED: Hard Sequence Handlers

Handlers associated to actions with names that are valid key sequence strings implicitly define actions that are matched by the corresponding key sequence. This means you do not have to define the key maps in order for these handlers to "just work".

This functionality is not advised and exists mainly for backwards compatibility. It is generally advisable to explicitly define an action in a key map rather than rely on this behaviour.

```javascript
/**
 * If no named 'up' action has been defined in a key map and it is a valid
 * key sequence, react-hotkeys assumes it's a hard sequence handler and
 * implicitly defines an action for it
 */

const handlers = {
  'up': (event) => console.log('up key called')
};
```

## Interaction with React

Rather than re-invent the wheel, `react-hotkeys` piggy-backs of the React SyntheticEvent and event propagation, so all of the normal React behaviour that you expect still applies.

- Key events propagate up from a source or target towards the root of the application.
- If an event has `stopPropagation()` called on it, it will not be seen by components higher up in the render tree.

## HotKeys components

`<HotKeys>` components listen only to key events that happen when one of their DOM-mounted descendents are in focus (`<div/>`, `<span/>`, `<input/>`, etc). This emulates (and re-uses) the behaviour of the browser and React's SyntheticEvent propagation.

This is the default type of `<HotKeys />` component, and should normally be your first choice for efficiency and clarity (the user generally expects keyboard input to affect the focused element in the browser).

Each time a new element is focused, the history of the keys that have already been pressed are reset.

### How action handlers are resolved

> If one of the DOM-mounted descendents of an `<HotKeys>` component are in focus (and it is listening to key events) AND those key events match a hot key in the component's key map, then the corresponding action is triggered.

`react-hotkeys` starts at the `<HotKeys />` component closest to the event's target (the element that was in focus when the key was pressed) and works its way up through the component tree of focused `<HotKeys />` components, looking for a matching handler for the action. The handler closest to the event target AND a descendant of the `<HotKeys />` component that defines the action (or the component itself), is the one that is called.

That is:

- Unless one of the DOM-mounted descendents of a `<HotKeys>` component is in focus, the component's actions are not matched
- Unless a `<HotKeys>` component is nested within the `<HotKeys />` component that defines the action (or is the same `<HotKeys />` component), its handler is not called
- If a `<HotKeys />` component closer to the event target has defined a handler for the same action, a `<HotKeys />` component's handler won't be called (the closer component's handler will)

A more exhaustive enumeration of `react-hotkeys` behaviour can be found by reviewing the [test suite](/test).

### Managing focus in the browser

#### Focusable elements

HTML5 allows any element with a `tabindex` attribute to receive focus.

If you wish to support HTML4 you are limited to the following focusable elements:

* `<a>`
* `<area>`
* `<button>`
* `<input>`
* `<object>`
* `<select>`
* `<textarea>`


#### Tab Index

If no elements have a `tabindex` in a HTML document, the browser will tab between [focusable elements](#Focusable-elements) in the order that they appear in the DOM.

If there are elements with `tabindex` values greater than zero, they are iterated over first, according their `tabindex` value (from smallest to largest). Then the browser tabs over the focusable elements with a `0` or unspecified `tabindex` in the order that they appear in the DOM.

If any element is given a negative `tabindex`, it will be skipped when a user tabs through the document. However, a user may still click or touch on that element and it can be focused programmatically (see below).

> By default, `<HotKeys>` render its children inside an element with a `tabindex` of `-1`. You can change this by passing a `tabIndex` prop to `<HotKeys>` or you can change the default `tabindex` value for all <HotKeys>` components using the `defaultTabIndex` option for the [Configuration API](#Configuration).

#### Autofocus

HTML5 supports a boolean `autofocus` attribute on the following input elements:

* `<button>`
* `<input>`
* `<select>`
* `<textarea>`

It can be used to automatically focus parts of your React application, without the need to [programmatically manage focus](#Programmatically-manage-focus).

Only one element in the document should have this attribute at any one time (the last element to mount with the attribute will take effect).

#### Programmatically manage focus

To programmatically focus a DOM element, it must meet two requirements:

* It must be a [focusable elements](#Focusable-element)
* You must have a reference to it

You can get a reference to an element using React's `ref` property:


```javascript
class MyComponent extends Component {

    componentDidUpdate(prevProps) {

        if(!prevProps.isFocused && this.props.isFocused) {
            this._container.focus();
        }

    }

    render() {
        return (
            <div ref={ (c) => this._container = c } >
                My focusable content
            </div>
        )
    }

}
```

#### Get the element currently in focus

You can retrieve the element that is currently focused using the following:

```javascript
document.activeElement
```

## HotKeys component API

The HotKeys component provides a declarative and native JSX syntax that is best for succinctly declaring hotkeys in a way that best maintains separation and encapsulation with regards to the rest of your code base.

However, it [does require that its children be wrapped in a DOM-mounted node](#Hotkeys-is-rendering-a-div-that-is-breaking-my-styling), which can break styling and add extra levels to your render tree.

```javascript

<HotKeys
  /**
   * An object that defines actions as keys and key sequences as values
   * (using either a string, array or object).
   *
   * Actions defined in one HotKeys component are available to be handled
   * in an descendent HotKeys component.
   *
   * Optional.
   */
  keyMap={ {} }

  /**
   * An object that defines handler functions as values, and the actions
   * that they handle as keys.
   *
   * Optional.
   */
  handlers={ {} }

  /**
   * The type of DOM-mountable component that should be used to wrap
   * the component's children.
   */
  component={ 'div' }

  /**
   * tabindex value to pass to DOM-mountable component wrapping children
   */
  tabIndex={-1}

  /**
   * Whether the keyMap or handlers are permitted to change after the
   * component mounts. If false, changes to the keyMap and handlers
   * props will be ignored
   *
   * Optional.
   */
  allowChanges={false}
  >

  /**
   * Wraps all children in a DOM-mountable component
   */
   { children }

</HotKeys>
```

## withHotKeys HoC API

The HotKeys component API is generally recommended, but if wrapping your component in a DOM-mountable node is not acceptable, or you need more control over how the `react-hotkeys` props are applied, then the `withHotKeys()` HoC is available.

### Simple use-case

The simplest use-case of `withHotKeys()` is to simply pass it your component class as the first argument. What is returned is a new component that will accept all of the same props as a `<HotKey>` component, so you can specify key maps and handlers at render time, for example.

> The component you wrap **must** take responsibility for passing the `hotKeys` props to a DOM-mountable element. If you fail to do this, key events will not be detected when a descendant of the component is in focus.

```javascript
import {withHotKeys} from 'react-hotkeys';

class MyComponent extends Component {
  render() {
    /**
     * Must unwrap hotKeys prop and pass its values to a DOM-mountable
     * element (like the div below).
     */
    const {hotKeys, ...remainingProps} = this.props;

    return (
      <div { ... { ...hotKeys, ...remainingProps } } >
        <span>My HotKeys are effective here</span>

       { this.props.children }
      </div>
    )
  }
}

const MyHotKeysComponent = withHotKeys(MyComponent);

const keyMap = {
    TEST: 't'
};

const handlers = {
    TEST: ()=> console.log('Test')
};

<MyHotKeysComponent keyMap={ keyMap } handlers={ handlers }>
  <div>
    You can press 't' to log to the console.
  </div>
</MyHotKeysComponent>
```

### Pre-defining default prop values

You can use the second argument of `withHotKeys` to specify default values for any props you would normally pass to `<HotKeys />`. This means you do not have to specify them at render-time.

> If you do provide prop values when you render the component, these will be merged with (and override) those defined in the second argument of `withHotKeys`.

```javascript
import {withHotKeys} from 'react-hotkeys';

class MyComponent extends Component {
  render() {
    /**
     * Must unwrap hotKeys prop and pass its values to a DOM-mountable
     * element (like the div below).
     */
    const {hotKeys, ...remainingProps} = this.props;

    return (
      <div { ... { ...hotKeys, ...remainingProps } } >
        <span>My HotKeys are effective here</span>

       { this.props.children }
      </div>
    )
  }
}

const keyMap = {
    TEST: 't'
};

const handlers = {
    TEST: ()=> console.log('Test')
};

const MyHotKeysComponent = withHotKeys(MyComponent, {keyMap, handlers});

/**
 * Render without having to specify prop values
 */
<MyHotKeysComponent>
  <div>
    You can press 't' to log to the console.
  </div>
</MyHotKeysComponent>
```

## GlobalHotKeys component

`<GlobalHotKeys>` components match key events that occur anywhere in the document (even when no part of your React application is in focus).

```javascript
const keyMap = { SHOW_ALL_HOTKEYS: 'shift+?' };
const handlers = { SHOW_ALL_HOTKEYS: this.showHotKeysDialog };

<GlobalHotKeys keyMap={ keyMap } handlers={ handlers } />
```

`<GlobalHotKeys>` generally have no need for children, so should use a self-closing tag (as shown above). The only exception is when you are nesting other `<GlobalHotKeys>` components somewhere in the descendents (these are mounted before their parents, and so are generally matched first).

### How actions and handlers are resolved

Regardless of where `<GlobalHotKeys>` components appear in the render tree, they are matched with key events after the event has finished propagating through the React app (if the event originated in the React at all). This means if your React app is in focus and it handles a key event, it will be ignored by the `<GlobalHotKeys>` components.

The order used for resolving actions and handlers amongst `<GlobalHotKeys>` components, is the order in which they mounted (those mounted first, are given the chance to handle an action first). When a `<GlobalHotKeys>` component is unmounted, it is removed from consideration. This can get less deterministic over the course of a long session using a React app as components mount and unmount, so it is best to define actions and handlers that are globally unique.

It is recommended to use `<HotKeys>` components whenever possible for better performance and reliability.

> You can use the [autofocus attributes](#Autofocus) or [programmatically manage focus](#Programmatically-manage-focus) to automatically focus your React app so the user doesn't have to select it in order for hot keys to take effect. It is common practice to place a `<HotKeys>` component towards the top of your application to match hot keys across your entire React application.

## GlobalHotKeys component API

The GlobalHotKeys component provides a declarative and native JSX syntax for defining hotkeys that are applicable beyond you React application.

```javascript

<GlobalHotKeys
  /**
   * An object that defines actions as keys and key sequences as values
   * (using either a string, array or object).
   *
   * Actions defined in one HotKeys component are available to be handled
   * in an descendent HotKeys component.
   *
   * Optional.
   */
  keyMap={ {} }

  /**
   * An object that defines handler functions as values, and the actions
   * that they handle as keys.
   *
   * Optional.
   */
  handlers={ {} }

  /**
   * Whether the keyMap or handlers are permitted to change after the
   * component mounts. If false, changes to the keyMap and handlers
   * props will be ignored
   *
   * Optional.
   */
  allowChanges={false}
  >

  /**
   * Wraps all children in a DOM-mountable component
   */
   { children }

</GlobalHotKeys>
```

## Ignoring events

By default, all key events that originate from `<input>`, `<select>` or `<textarea>`, or have a `isContentEditable` attribute of `true` are ignored by `react-hotkeys`.

If this is not what you want for your application, you can modify the list of tags using the `ignoreTags` [configuration option](#Configuration) or if you need additional control, you can specify a brand new function using the `ignoreEventsCondition` [configuration option](#Configuration).

## Allowing hotkeys and handlers props to change

For performance reasons, by default `react-hotkeys` takes the `keyMap` and `handlers` prop values when `<HotKeys>` components are focused and when `<GlobalHotKeys>` components are mounted. It ignores all subsequent updates
to their values when these props change.

If you need the ability to change them while `<HotKeys>` are still in focus, or while `<GlobalHotKeys>` are still mounted, then you can passe the `allowChanges` prop, permitting this behaviour for the particular component.

If you need to do this for all your `<HotKeys>` and `<GlobalHotKeys>` components, you can use the `ignoreKeymapAndHandlerChangesByDefault` option for the [Configuration API](#Configuration). This should normally never be done, as it can have significant performance implications.

## Configuration

The default behaviour across all `<HotKeys>` components is configured using the `HotKeys.configure` method.

> HotKeys.configure() should be called as your app is initialising and before the first time you mount a `<HotKeys>` component anywhere your app.

The following options are available (**default values are shown**):

```javascript
import {HotKeys} from 'react-hotkeys';

HotKeys.configure({
  /**
   * The level of logging of its own behaviour React HotKeys should perform.
   */
  logLevel: 'warn',

  /**
   * Default key event key maps are bound to (keydown|keypress|keyup)
   */
  defaultKeyEvent: 'keypress',

  /**
   * The default component type to wrap HotKey components' children in, to provide
   * the required focus and keyboard event listening for HotKeys to function
   */
  defaultComponent: 'div',

  /**
   * The default tabIndex value passed to the wrapping component used to contain
   * HotKey components' children. -1 skips focusing the element when tabbing through
   * the DOM, but allows focusing programmatically.
   */
  defaultTabIndex: '-1',

  /**
   * The HTML tags that React HotKeys should ignore key events from. This only works
   * if you are using the default ignoreEventsCondition function.
   * @type {String[]}
   */
  ignoreTags: ['input', 'select', 'textarea'],

  /**
   * The function used to determine whether a key event should be ignored by React
   * Hotkeys. By default, keyboard events originating elements with a tag name in
   * ignoreTags, or a isContentEditable property of true, are ignored.
   *
   * @type {Function<KeyboardEvent>}
   */
  ignoreEventsCondition: function,

  /**
   * Whether to ignore changes to keyMap and handlers props by default
   * (this reduces a significant amount of unnecessarily resetting
   * internal state)
   * @type {Boolean}
   */
  ignoreKeymapAndHandlerChangesByDefault: true,

  /**
   * Whether React HotKeys should simulate keypress events for the keys that do not
   * natively emit them.
   * @type {Boolean}
   */
  simulateMissingKeyPressEvents: true
});
```


## Troubleshooting & Gotchas

#### Hotkeys is wrapping my components in a div that is breaking my styling

You have 3 options:

1. Use the [`component` prop](#HotKeys-component-API) to specify a `span` or some other alternative DOM-mountable component to wrap your component in, each time you render a component you don't want to wrap in a div element.
1. Use the [`defaultComponent` configuration option](#Configuration) to specify a `span` or some other alternative DOM-mountable component to wrap *all* `<HotKeys>` children in.
1. Use the [withHotKeys HoC API](#withHotKeys-HoC-API) to avoid rendering a wrapping component at all.

#### Actions aren't being triggered when using withHotKeys

Check that you are [correctly passing the hotKeys props to a DOM-mountable component](#Pre-defining-default-prop-values).

#### Actions aren't being triggered for HotKeys

Make sure you are focusing a descendant of the `<HotKeys>` component before you press the keys.

Check that the `<HotKeys>` component that defines the handler is also an ancestor of the focused component, and is above (or *is*) the component that defines the `handlers`.

Also make sure your React application is not calling `stopPropagation()` on the key events before they reach the `<HotKeys>` component that defines the `keyMap`.

Finally, make sure your key event are not coming from one of the [tags ignored by react-hotkeys](#Ignoring-events).

#### Sequence is not being matched when moving between components

For efficiency, every time the focused element changes, the key event history is reset. HotKeys does not currently support key sequences that involve shifting focus mid-way through.

#### Blue border appears around children of HotKeys

`react-hotkeys` adds a `<div />` around its children with a `tabindex="-1"` to allow them to be programmatically focused. This can result in browsers rendering a blue outline around them to visually indicate that they are the elements in the document that is currently in focus.

This can be disabled using CSS similar to the following:

```css
div[tabindex="-1"]:focus {
    outline: 0;
}
```

## Logging

`react-hotkeys` provides comprehensive logging of all of its internal behaviour and allows setting one of 6 log levels.

The default level is `warn`, which provides warnings and errors only, and is generally sufficient for most usage. However, if you are troubleshooting an issue or reporting a bug, you should increase the log level to `debug` or `verbose` to see what is going on, and be able to communicate it concisely.

You can set the logging level using the `logLevel` [configuration option](#Configuration).

For performance reasons, only some of the log levels are available in the production build. You will need to use the development build to get the full log output.

| Log Level | Severity | Description | Available in Dev | Available in Prod |
| :-- | :-- | :-- | :-- | :-- |
| verbose | (highest) | `debug` + internal data representations | Yes | No |
| debug | | `info` + event propagation info | Yes | No |
| info | | `warn` + general info | Yes | No |
| warn | (default) | `error` + warnings | Yes | Yes |
| error | | Errors only (ignore warnings) | Yes | Yes |
| none | (lowest) | Log nothing | Yes | Yes |

Logs appear in the developer console of the browser.

Each line is prefixed with (where applicable):

- The focus tree id
- The component id
- The event id

Each id is also given a coloured emoticon, to make it easy to visually trace the propagation of particular events through multiple components.

Below is an example (verbose) log output:

```
HotKeys (GLOBAL-C0🔺): Bound handler handleGlobalKeydown() to document.onkeydown()
HotKeys (GLOBAL-C0🔺): Bound handler handleGlobalKeypress() to document.onkeypress()
HotKeys (GLOBAL-C0🔺): Bound handler handleGlobalKeyup() to document.onkeyup()
HotKeys (GLOBAL-C0🔺): Mounted.
HotKeys (GLOBAL-C0🔺): Component options:
 {
    "actions": {
        "KONAMI": [
            {
                "prefix": "ArrowUp ArrowUp ArrowDown ArrowDown ArrowLeft ArrowRight ArrowLeft ArrowRight b a",
                "actionName": "KONAMI",
                "sequenceLength": 11,
                "id": "Enter",
                "keyDictionary": {
                    "Enter": true
                },
                "eventBitmapIndex": 1,
                "size": 1
            }
        ],
        "LOG_DOWN": [
            {
                "prefix": "",
                "actionName": "LOG_DOWN",
                "sequenceLength": 1,
                "id": "Meta",
                "keyDictionary": {
                    "Meta": true
                },
                "eventBitmapIndex": 0,
                "size": 1
            }
        ],
        "LOG_UP": [
            {
                "prefix": "",
                "actionName": "LOG_UP",
                "sequenceLength": 1,
                "id": "Meta",
                "keyDictionary": {
                    "Meta": true
                },
                "eventBitmapIndex": 2,
                "size": 1
            }
        ]
    },
    "handlers": {
        "KONAMI": "function () { [native code] }",
        "LOG_DOWN": "function logCommandKeyDown() {\n      console.log('command down');\n    }",
        "LOG_UP": "function logCommandKeyUp() {\n      console.log('command up');\n    }"
    },
    "componentId": 0,
    "options": {
        "defaultKeyEvent": "keypress"
    }
}
HotKeys (FT0📕-E0❤️-C0🔺): Focused.

HotKeys (FT0📕-E0❤️-C0🔺): Component options:
 {
    "actions": {},
    "handlers": {
        "MOVE_UP": "function () { [native code] }",
        "MOVE_DOWN": "function () { [native code] }",
        "MOVE_LEFT": "function () { [native code] }",
        "MOVE_RIGHT": "function () { [native code] }",
        "DELETE": "function () { [native code] }",
        "EXPAND": "function () { [native code] }",
        "CONTRACT": "function () { [native code] }"
    },
    "componentId": 0,
    "options": {
        "defaultKeyEvent": "keypress"
    }
}
HotKeys (FT0📕-E0❤️-C1⭐️): Focused.

HotKeys (FT0📕-E0❤️-C1⭐️): Component options:
 {
    "actions": {
        "DELETE": [
            {
                "prefix": "",
                "actionName": "DELETE",
                "sequenceLength": 1,
                "id": "Backspace",
                "keyDictionary": {
                    "Backspace": true
                },
                "eventBitmapIndex": 2,
                "size": 1
            }
        ],
        "EXPAND": [
            {
                "prefix": "",
                "actionName": "EXPAND",
                "sequenceLength": 1,
                "id": "Alt+ArrowUp",
                "keyDictionary": {
                    "Alt": true,
                    "ArrowUp": true
                },
                "eventBitmapIndex": 1,
                "size": 2
            }
        ],
        "CONTRACT": [
            {
                "prefix": "",
                "actionName": "CONTRACT",
                "sequenceLength": 1,
                "id": "Alt+ArrowDown",
                "keyDictionary": {
                    "Alt": true,
                    "ArrowDown": true
                },
                "eventBitmapIndex": 1,
                "size": 2
            }
        ],
        "MOVE_UP": [
            {
                "prefix": "",
                "actionName": "MOVE_UP",
                "sequenceLength": 1,
                "id": "ArrowUp",
                "keyDictionary": {
                    "ArrowUp": true
                },
                "eventBitmapIndex": 1,
                "size": 1
            }
        ],
        "MOVE_DOWN": [
            {
                "prefix": "",
                "actionName": "MOVE_DOWN",
                "sequenceLength": 1,
                "id": "ArrowDown",
                "keyDictionary": {
                    "ArrowDown": true
                },
                "eventBitmapIndex": 1,
                "size": 1
            }
        ],
        "MOVE_LEFT": [
            {
                "prefix": "",
                "actionName": "MOVE_LEFT",
                "sequenceLength": 1,
                "id": "ArrowLeft",
                "keyDictionary": {
                    "ArrowLeft": true
                },
                "eventBitmapIndex": 1,
                "size": 1
            }
        ],
        "MOVE_RIGHT": [
            {
                "prefix": "",
                "actionName": "MOVE_RIGHT",
                "sequenceLength": 1,
                "id": "ArrowRight",
                "keyDictionary": {
                    "ArrowRight": true
                },
                "eventBitmapIndex": 1,
                "size": 1
            }
        ]
    },
    "handlers": {},
    "componentId": 1,
    "options": {
        "defaultKeyEvent": "keypress"
    }
}
HotKeys (FT0📕-E1💚-C0🔺): New 'ArrowDown' keydown event.
HotKeys (FT0📕-E1💚-C0🔺): Added 'ArrowDown' to current combination: ArrowDown.
HotKeys (FT0📕-E1💚-C0🔺): Ignored 'ArrowDown' keydown because it doesn't have any keydown handlers.
HotKeys (FT0📕-E1💚-C1⭐️): Ignored 'ArrowDown' keydown because it doesn't have any keydown handlers.
HotKeys (FT0📕-E1💚-C0🔺): Simulating 'ArrowDown' keypress event because 'ArrowDown' doesn't natively have one.
HotKeys (FT0📕-E2💙-C0🔺): New 'ArrowDown' keypress event.
HotKeys (FT0📕-E2💙-C0🔺): Attempting to find action matching 'ArrowDown' keypress . . .
HotKeys (FT0📕-E2💙-C0🔺): Internal key mapping:
 {
    "sequences": {
        "": {
            "combinations": {
                "Backspace": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "Backspace",
                    "keyDictionary": {
                        "Backspace": true
                    },
                    "size": 1,
                    "events": {
                        "2": {
                            "actionName": "DELETE",
                            "eventBitmapIndex": 2,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "Alt+ArrowUp": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "Alt+ArrowUp",
                    "keyDictionary": {
                        "Alt": true,
                        "ArrowUp": true
                    },
                    "size": 2,
                    "events": {
                        "1": {
                            "actionName": "EXPAND",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "Alt+ArrowDown": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "Alt+ArrowDown",
                    "keyDictionary": {
                        "Alt": true,
                        "ArrowDown": true
                    },
                    "size": 2,
                    "events": {
                        "1": {
                            "actionName": "CONTRACT",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "ArrowUp": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "ArrowUp",
                    "keyDictionary": {
                        "ArrowUp": true
                    },
                    "size": 1,
                    "events": {
                        "1": {
                            "actionName": "MOVE_UP",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "ArrowDown": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "ArrowDown",
                    "keyDictionary": {
                        "ArrowDown": true
                    },
                    "size": 1,
                    "events": {
                        "1": {
                            "actionName": "MOVE_DOWN",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "ArrowLeft": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "ArrowLeft",
                    "keyDictionary": {
                        "ArrowLeft": true
                    },
                    "size": 1,
                    "events": {
                        "1": {
                            "actionName": "MOVE_LEFT",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                },
                "ArrowRight": {
                    "prefix": "",
                    "sequenceLength": 1,
                    "id": "ArrowRight",
                    "keyDictionary": {
                        "ArrowRight": true
                    },
                    "size": 1,
                    "events": {
                        "1": {
                            "actionName": "MOVE_RIGHT",
                            "eventBitmapIndex": 1,
                            "handler": "function () { [native code] }"
                        }
                    }
                }
            }
        }
    },
    "eventBitmap": [
        false,
        true,
        true
    ],
    "longestSequence": 1
}
HotKeys (FT0📕-E2💙-C0🔺): Found action that matches 'ArrowDown': MOVE_DOWN. Calling handler . . .
HotKeys (FT0📕-E2💙-C1⭐️): Simulating 'ArrowDown' keypress event because 'ArrowDown' doesn't natively have one.
HotKeys (FT0📕-E2💙-C1⭐️): Ignored 'ArrowDown' keypress as it has already been handled.
```

## Optimizations

`react-hotkeys` uses a lot of optimizations to help keep it as performant as possible (both in terms of time and memory). It can be helpful to be aware of some of these measures if you are seeing unexpected behaviour:

### Code optimizations

- If an event is handled by an earlier handler, it is ignored by an further components (this is really a design decision, rather than an optimization, but it helps).
- Events are ignored unless an action exists that is bound to that particular event type (keydown, keypress, keyup)
- Events are processed at each level, as they propagate up the React render tree. If a action is triggered by a leaf node, `react-hotkeys` stops there (and does not build the full application's mappings of key sequences and handlers)
- Changes to keyMaps and handlers are ignored unless you explicitly opt-in to the behaviour of resetting them each time their prop value changes.
- Key histories longer than the longest registered key sequence are discarded.
- The mapping between an action's key sequences and handlers is built "on-the-fly", so unless a particular action is triggered, `react-hotkeys` doesn't do the work of finding its corresponding handler.
- Global event listeners are only bound to `document` when a global hotkey is defined (and are removed when the last one is unmounted).

### Production optimizations

- The production build strips out all comments and logging statements below a level of warning, before undergoing minification using Uglify.
- An es6 version is also available, that allows for tree-shaking in compatible build setups.

## Support

Please use [Gitter](https://gitter.im/Chrisui/react-hotkeys) to ask any questions you may have regarding how to use `react-hotkeys`.

If you believe you have found a bug or have a feature request, please [open an issue](https://github.com/greena13/react-hotkeys/issues).

## Stability & Maintenance

`react-hotkeys` is considered stable and already being widely used (most notably Lystable and Whatsapp).


## Contribute, please!

If you're interested in helping out with the maintenance of `react-hotkeys`, make yourself known on [Gitter](https://gitter.im/Chrisui/react-hotkeys), [open an issue](https://github.com/greena13/react-hotkeys/issues) or create a pull request.

All contributions are welcome and greatly appreciated - from contributors of all levels of experience.

Collaboration is loosely being coordinated across [Gitter](https://gitter.im/Chrisui/react-hotkeys) and [Projects](https://github.com/greena13/react-hotkeys/projects).

### Using GitHub Issues

* Use the search feature to check for an existing issue
* Include as much information as possible and provide any relevant resources (Eg. screenshots)
* For bug reports ensure you have a reproducible test case
    * A pull request with a breaking test would be super preferable here but isn't required

### Submitting a Pull Request

- Squash commits
- Lint your code with eslint (config provided)
- Include relevant test updates/additions

## Authorship

All credit, and many thanks, goes to [Chris Pearce](https://github.com/Chrisui) for the inception of `react-hotkeys` and all versions before `1.0.0`.
