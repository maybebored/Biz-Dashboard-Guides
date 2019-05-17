# Biz Dashboard Guide: "Creating Analytics Widgets"

The widgets that display analytics on Biz Dashboard app, are complex react components that are composed of or inherit from other components. This is done to separate fetching data, processing data and then rendering the result. So a widget in Biz Dashboard is made up of **React Components** and **Resource Classes**.

## Prerequisites
Read these before to understand this guide better.
1. [React Concepts](https://reactjs.org/docs/hello-world.html)
2. [Component Lifecycle](http://projects.wojtekmaj.pl/react-lifecycle-methods-diagram)
2. [Biz Dashboard Development Guideline](https://bitbucket.org/DigitalPlumbing/apg-biz-dashboard/src/f2068d1aebfac044f5c13788d8a024a637227065/project-wiki/development.md)

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

### [Shared Widget Component](SharedWidget.md)
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

### [BaseWidget](BaseWidget.md)
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

## [Widget Resource](WidgetResource.md)
```bash
src/common/resources/WidgetResource
```

_WidgetResource_ is responsible for fetching data from the url provided to the constructor.

## [Transaction Resource](TransactionResource.md)
```bash
src/views/analytics/transaction/resources/TransactionResource
```

_TransactionResource_ extends _WidgetResource_ to override get methods ```propKeys``` and ```stateKeys```. This means it takes responsibility of passing values that form the params for the api calls.

## 3. The Analytics View
```bash
src/views/analytics/index.js
```

```js
this.transactionStatResource = new transactionResources.TransactionResource(
  'transactions/stat/all'
);

<transactionWidgets.TransactionTotalSuccess
            startDate={'2018-09-03'}
            name={'transaction-total-success'}
            title={'Total Successful Transactions'}
            resources={[this.transactionStatResource]}/>
```

This is where the first two parts come together to form a working analytics widget.
