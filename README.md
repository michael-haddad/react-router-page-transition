# React Router Page Transition
Highly customizable page transition component for your React Router

# Introduction

**React Router** is awesome, but doing transition between pages is hard, especially for complex ones.

No worries **react-router-page-transition** is here to help. You can use css to define your own transition
effect with custom data and callbacks, so now you can apply cool technique like [FLIP your animation](https://aerotwist.com/blog/flip-your-animations/)
and implement cool transitions like this:

**Live demo**: https://trungdq88.github.io/react-router-page-transition/

|Simple|Material|Reveal|
|------|--------|------|
|![simple](https://cloud.githubusercontent.com/assets/4214509/16784316/6dc99028-48b2-11e6-8f03-230e1178761b.gif)|![material](https://cloud.githubusercontent.com/assets/4214509/16781947/aa83ca34-48a7-11e6-8c93-dfdd794d7a28.gif) | ![reveal](https://cloud.githubusercontent.com/assets/4214509/16783423/1c58b880-48ae-11e6-97fb-5e92a7da1b40.gif)|

# Installation

    npm install react-router-page-transition --save

# How it work?
TODO

**Pros:**
 - Keep page structure clean.
 - FLIP

**Cons:**
 - Requires extra setup for page components

# Examples

## Example 1: Simple zoom trasition

This is a simple transition: detail page zoom out from the middle.

### Router
```js
ReactDOM.render(
  <Router history={hashHistory}>
    <Route path="/" component={Home}>
      <IndexRoute component={ListPage} />
      <Route path="/detail/:itemId" component={ItemDetailPage} />
    </Route>
  </Router>, document.getElementById('app'));
```

### ListPage component

```js
export default class ListPage extends React.Component {

  ...
  
  render() {
    return (
      <div className="transition-item list-page">
        {this.state.items.map(item => (
          <Link
            key={item.id}
            className="list-item"
            to={`/detail/${item.id}`}
          >
            <Item {...item} />
          </Link>
        ))}
      </div>
    );
  }

```

### Home component
```js
import React from 'react';
import PageTransition from 'react-router-page-transition';

export default (props) => (
  <div>
    <PageTransition>
      {props.children}
    </PageTransition>
  </div>
);
```

### DetailPage component
Add class `transition-item` to your root element, we will use this to animate the page
when route change.
```jsx
export default class ItemDetailPage extends React.Component {
  render() {
    return (
      <div className="transition-item detail-page">
        <Link to="/">Back</Link>
        <h1>
          Detail {this.props.params.itemId}
        </h1>
      </div>
    );
  }
}
```

### CSS:
Define animation using CSS

```less
.trasition-wrapper {
  position: relative;
  z-index: 1;
  .transition-item {
    position: fixed;
    top: 0;
    left: 0;
    right: 0;
  }
}

.detail-page {
  padding: 20px;
  background-color: #03a9f4;
  transition: transform 0.5s;
  height: 100vh;
  box-sizing: border-box;

  &.transition-appear {
    transform: scale(0);
  }

  &.transition-appear.transition-appear-active {
    transform: scale(1);
  }
}
```

**See demo:** https://trungdq88.github.io/react-router-page-transition/simple/

## Example 2: Material design transition
From a cool idea of [Material Motion](https://material.google.com/motion/material-motion.html#material-motion-how-does-material-move) that provide meaningful transition between pages.

![material-sample](https://cloud.githubusercontent.com/assets/4214509/16789846/0ebb37f2-48db-11e6-8755-f106e5ae1488.gif)

In order to implement this, we need to pass custom data into the detail page, which requires a data flow management for the app. In this example, I'll use **RxJS** to pass data between ListPage component, Home component and ItemDetalPage component, but you can use another library to handle the data (like Flux or Redux).


### ListPage component
See the `onClick` method, we send the clicked item's data (color and position) to the action.
```js
  render() {
    return (
      <div className="transition-item list-page">
        {this.state.items.map(item => (
          <Link
            key={item.id}
            className="list-item"
            onClick={e => action.onNext({
              name: 'CLICKED_ITEM_DATA',
              data: {
                position: e.target.getBoundingClientRect(),
                color: item.color,
              },
            })}
            to={`/detail/${item.id}`}
          >
            <Item {...item} />
          </Link>
        ))}
      </div>
    );
  }
```

### Home component
Home component receives the data and pass it to the `PageTransition` component.

```js
export default class Home extends React.Component {
  constructor(...args) {
    super(...args);
    this.state = {
      clickedItemData: null,
    };
  }

  componentDidMount() {
    // Receive data from the clicked item
    this.obsClickedItemData = action
    .filter(a => a.name === 'CLICKED_ITEM_DATA')
    .map(a => a.data)
    .subscribe(clickedItemData => this.setState({ clickedItemData }));
  }

  componentWillUnmount() {
    this.obsClickedItemData.dispose();
  }

  render() {
    return (
      <div>
        <PageTransition
          data={{ clickedItemData: this.state.clickedItemData }}
        >
          {this.props.children}
        </PageTransition>
      </div>
    );
  }
}
```

## ItemDetailPage component
This component will receive the callbacks with data from `PageTransition` component, then we can use tihs to animate our page as we want.

```js
export default class ItemDetailPage extends React.Component {

  constructor(...args) {
    super(...args);
    this.state = {
      doTransform: false,
      position: null,
      color: null,
    };
  }

  onTransitionWillStart(data) {
    // Start position of the page
    this.setState({
      doTransform: true,
      position: data.clickedItemData.position,
      color: data.clickedItemData.color,
    });
  }

  onTransitionDidEnd() {
    // Transition is done, do your stuff here
  }

  transitionManuallyStart() {
    // When this method exsits, `transition-appear-active` will not be added to the dom
    // we will define our animation manually.
    this.setState({
      position: {
        top: 0,
        height: '100%',
        left: 0,
        right: 0,
      },
      doTransform: true,
    });
  }

  transitionManuallyStop() {
    // When this method exsits, `transition-appear-active` will not be removed
    this.setState({
      doTransform: false,
    });
  }

  render() {
    return (
      <div
        style={{
          transform: this.state.doTransform ?
            `translate3d(0, ${this.state.position.top}px, 0)` :
              undefined,
          height: this.state.doTransform ?
            this.state.position.height : null,
          left: this.state.doTransform ?
            this.state.position.left : null,
          right: this.state.doTransform ?
            this.state.position.left : null,
          backgroundColor: this.state.color,
        }}
        className="transition-item detail-page"
      >
        <Link to="/">
          Item {this.props.params.itemId}
        </Link>
        <h1>
          Detail page here
        </h1>
        <Link to="/">
          Back
        </Link>
      </div>
    );
  }
```

**See demo:** https://trungdq88.github.io/react-router-page-transition/material/

## Example 3: Reveal effect

Similar to the material, we use `border-radius` to animate the circle.

**See demo:** https://trungdq88.github.io/react-router-page-transition/reveal/

# API

`PageTransition`'s properties:
- `timeout`: transition duration in milisecond, this must be the same with the `transition-duration` in your CSS.
   
   Example:
    ```jsx
        <PageTransition timeout={500}>
          {props.children}
        </PageTransition>
    ```
    
- `data`: custom data to send to the page component via `onTransitionWillStart`, `onTransitionDidEnd`, `transitionManuallyStart`, `transitionManuallyEnd`.

    Example:
    ```jsx
        <PageTransition
          data={{ clickedItemData: this.state.clickedItemData }}
        >
          {this.props.children}
        </PageTransition>
    ```

- `onLoad`: this callback will be call after the new page is finished replaced and animated.

    Example:
    ```jsx
        <PageTransition
          onLoad={() => this.refs.scrollArea.scrollTop = 0}
        >
          {this.props.children}
        </PageTransition>
    ```
