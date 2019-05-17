## Widget Resource

This document aims to explain the Widget Resource class and its methods.

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
```get propKeys``` and ```get stateKeys``` are overridden to return keys that will form the query params for the api call.
