## Base Widget

This document aims to explain the Base Widget class and its methods, in terms of how they are used to create an Analytics Widget.

### props
```js
BaseWidget.propTypes = {
  hSize: PropTypes.string,
  vSize: PropTypes.string,
  name: PropTypes.string.isRequired,
  title: PropTypes.string.isRequired,
  resources: PropTypes.arrayOf(WidgetResource).isRequired
};
```
### state

```js
this.state = {
  isLoading: false,
  data: {},
  updatedAt: ''
};
```

When the base widget is initialised the first method to run after ```constructor``` is ```componentDidMount```. When this runs React has already updated the DOM, which is when we will begin fetching data.

```js
componentDidMount() {
  this.fetchData();
}
```

In ```fetchData``` we create an array of Promises, returned by the ```fetchData``` method of the resource class. Resource class instances are passed as props to _BaseWidget_. When all these promises resolve, the responses are passed to ```processFetchedData```.

```js
fetchData() {
  this.setState({isLoading: true});
  let promises = this.props.resources.map(
    resource => resource.fetchData(this.props.name, this.props, this.state)
  );
  Promise.all(promises)
    .then((responses) => {
      this.setState({isLoading: false}, () => {
        this.processFetchedData(responses);
      });
    })
    .catch((err) => {
      logger(
        'error',
        `error fetching data for widget: ${this.props.name}`,
        err
      );
    });
}
```

```appendToRootElementCSSClasses``` method is used by child classes or components that inherit _BaseWidget_.


```js
appendToRootElementCSSClasses(classes) {
  let _cls = utils.component.addClasses(this.props, classes);
  let identifierCls = ['widget', `widget__${this.props.name}`];
  let sizeCls = this._getSize();
  return `${_cls} ${identifierCls.join(' ')} ${sizeCls}`;
}
```

```processFetchedData``` is another method that is also overridden by child classes or components that inherit _BaseWidget_. Here it logs the responses it receives as arguments.

```js
processFetchedData(responses) {
  logger('info', `responses from resources in widget: ${this.props.name}`, responses);
}
```
It is in ```updateData``` method that the processed data is set to state variable, _data_. In addition _updatedAt_ is updated with the time at which state was set. ```setState``` method can take in a callback function as a second argument. Over here we pass Promise.resolve as the callback function. This means when the state is updated successfully the Promise returned by ```updateData``` resolves.

```js
updateData(data) {
  return new Promise((resolve, reject) => {
    if (!data) {
      reject();
    }
    this.setState({data, updatedAt: moment().format('D/M/YYYY-h:mm:ss')}, resolve);
  });
}
```

```loader``` method relates to UI logic as it conditionally renders a _Loader_ component.

```js
loader() {
  return this.state.isLoading && <Loader info={'Loading data...'}/>;
}
```
