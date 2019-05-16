# Biz Dashboard Guide: "Creating Analytics Widgets"

## 1. The React Components

### Analytics Widget

```bash
src/views/analytics/transaction/widgets
```

This is where the main widget exists, let’s call this Analytics Widget. This widget is a React Component that inherits another ReactComponent. Sometimes they are composed of other components.

```js
class TransactionSuccessPercentage extends NumberWidget {
 constructor(props) {
   super(props);
 }

 processFetchedData(responses) {
   super.processFetchedData(responses);
   this.updateData({
     value: parseFloat(responses[0].successPercentage)
   });
 }
}
```

_TransactionSuccessPercentage_ waits for data and processes it by overriding the super class method ```processFetchedData```. This method calls ```updateData```  where the processed data is set to the component's state.

### Shared Widget Component
>extended by Analytics Widget

```bash
src/components/widgets
```

```js
class NumberWidget extends BaseWidget {
  constructor(props) {
    super(props);
  }

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
}
```
_NumberWidget_ is one of the shared Base Widget
_NumberWidget_ which is inherited by _TransactionSuccessPercentage_ has the sole responsibility of rendering the component. No other classes related to the Analytics Widget has a ```render``` method in them. The exceptions are _TopCustomer_ and _TopFailureReason_ analytics widgets, as they do not extend

### BaseWidget
>base class extended by Shared Widgets and sometimes Analytics Widget

```bash
src/components/widgets/base
```

```js
class BaseWidget extends React.Component {
  constructor(props) {
    super(props);
    this.state = {
      isLoading: false,
      data: {},
      updatedAt: ''
    };
  }
}
```

_BaseWidget_ extends [React.Component](https://reactjs.org/docs/components-and-props.html) so it contains [Lifecycle Methods](https://reactjs.org/docs/state-and-lifecycle.html) and the State. In addition, it contains the methods that handle fetching of data from the API and some UI logic.

## 2. The Resource Classes

## Widget Resource
```bash
src/common/resources/WidgetResource
```

```js
class WidgetResource {
  constructor(url) {
    this._resourceUrl = `analytics/${url}`;
    this._apiService = getInstance('api');
  }

  get propKeys() {
    return [];
  }

  get stateKeys() {
    return [];
  }

  fetchData(widgetName, props, state) {
    return this._apiService.getItems(this._resourceUrl, this._formParamsFromPropsAndState(props,state))
      .then(response => response.data)
      .catch(err => {
        logger('error', `failed to fetch data for widget: ${widgetName}`, err);
        throw err;
      });
  }

  _formParamsFromPropsAndState(props, state) {
    return Object.assign({}, pick(props, this.propKeys), pick(state, this.stateKeys));
  }
}
```

_WidgetResource_ is responsible for fetching data from the url provided to the constructor.

## TransactionResource
```bash
src/views/analytics/transaction/resources/TransactionResource
```

```js
class TransactionResource extends WidgetResource {
  get propKeys() {
    return ['startDate', 'endDate'];
  }

  get stateKeys() {
    return ['groupBy'];
  }
}
```

_TransactionResource_ extends _WidgetResource_ to override get methods ```propKeys``` and ```stateKeys```. This means it takes responsibility of passing values that form the params for the api calls.
