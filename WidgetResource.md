## Widget Resource

This document aims to explain the Widget Resource class and its methods.

```js
class WidgetResource {
  constructor(url) {
    this._resourceUrl = `analytics/${url}`;
    this._apiService = getInstance('api');
  }
}
```
Passing a url to the constructor initialises the class attributes ```_resourceUrl``` and ```_apiService```.

```js
get propKeys() {
  return [];
}

get stateKeys() {
  return [];
}
```

```js
_formParamsFromPropsAndState(props, state) {
  return Object.assign({}, pick(props, this.propKeys), pick(state, this.stateKeys));
}
```

```get propKeys``` and ```get stateKeys``` are called in ```_formParamsFromPropsAndState``` method. Although they return empty arrays here, they are overridden in child classes to return specific props or state.

```js
fetchData(widgetName, props, state) {
  return this._apiService.getItems(this._resourceUrl, this._formParamsFromPropsAndState(props,state))
    .then(response => response.data)
    .catch(err => {
      logger('error', `failed to fetch data for widget: ${widgetName}`, err);
      throw err;
    });
}
```

```fetchData``` returns a Promise that resolves to the response returned from the api call.
