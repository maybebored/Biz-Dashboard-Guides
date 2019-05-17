## Shared Widget

This document aims to explain the Shared Widget classes and its methods, in terms of how they are used to create an Analytics Widget.

### props & state
>Inherits from _BaseWidget_

Any shared widget's purpose is to separate rendering logic from the Analytics Widget. It does this by inheriting _BaseWidget_. Over here it gets the necessary data from _state.data_ and renders the value using HTML elements. Notice that ```appendToRootElementCSSClasses``` is called here.

```js
render() {
  let {value} = this.state.data;
  value = millify(parseFloat(value), 2);
  return (
    <div className={this.appendToRootElementCSSClasses('widget__number')}>
      {this.loader()}
      <h1 className="widget__title">{this.props.title}</h1>
      <h2 className="widget__value">
        {value ? value : '---'}
        {this.props.metric && <small>{this.props.metric}</small>}
      </h2>
    </div>
  );
}
```
