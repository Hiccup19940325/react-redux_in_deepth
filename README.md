# react-localize-redux

[![Build Status](https://travis-ci.org/ryandrewjohnson/react-localize-redux.svg?branch=master)](https://travis-ci.org/ryandrewjohnson/react-localize-redux)

A collection of helpers for managing localized content in your React/Redux application.

* Dynamically load translation data from json files
* Supports rendering of HTML in translation strings
* Supports dynamic variable replacement in translation strings

## Table of Contents

- [Installation](#installation)
- [Getting Started](#getting-started)
- [Bundle translations with Webpack and react-router](#localize-translationid--wrappedcomponent-)
- [API](#api)
- [Introduction to react-localize-redux](https://medium.com/@ryandrewjohnson/adding-multi-language-support-to-your-react-redux-app-cf6e64250050#.jmxxptvpu)

## Installation

```
npm install react-localize-redux --save
```

## Getting Started

### 1. Add the `localeReducer` to your app's redux store.

```javascript
...
import { localeReducer } from 'react-localize-redux';

const store = createStore(localeReducer);

const App = props => {
  return (
    <Provider store={ store }>
      ...
    </Provider>
  );
};
```

### 2. Set the languages your app supports

Dispatch `setLanguages` action creator and pass in the languages for your app.

```javascript
import { setLanguages } from 'react-localize-redux';

const languages = ['en', 'fr', 'es'];
store.dispatch(setLanguages(languages));
```

### 3. Set your app's current language

Dispatch `setActiveLanguage` action creator and pass the language you want to use.

```javascript
import { setActiveLanguage } from 'react-localize-redux';

store.dispatch(setActiveLanguage('en'));
```

### 4. Add your app's localized translation data


### Set global translations

Translations that are shared between components are called `global` translations.
Assuming you have global transaltions stored in a file called `global.locale.json` you can add them
to your store by dispatching the `setGlobalTranslations` action creator.

> NOTE: The following assumes you are using [webpack](https://webpack.github.io/) with [json-loader](https://github.com/webpack/json-loader)

```javascript
import { setGlobalTranslations } from 'react-localize-redux';

const json = require("global.locale.json");
store.dispatch(setGlobalTranslations(json));
```

As mentioned above translation content is stored in json files. Each json file requires that there be
a property for each supported language, where the property name would match the language key passed to [updateLanguage](#updatelanguagelanguagecode).

```json
{
  "en": {
    "greeting": "Hello ${ name }",
    "farwell": "Goodbye"
  },
  "fr": {
    "greeting": "Bonjour ${ name }",
    "farwell": "Au revoir"
  },
  "es": {
    "greeting": "Hola ${ name }",
    "farwell": "Adiós"
  }
}
```
To add translations to a component you will need to decorate it with the [localize](#localize-translationid--wrappedcomponent-) function. By default
all components decorated with `localize` have access to `global` translations. It will not modify your
component class, but return a new localized component with additional props `translate` and `currentLanguage`.

The `translate` prop is a function that takes the unique id from the transaltion file as a param,
and an optional `data` param for variable substitutions. The function will return the localized string based on `currentLanguage`.

```javascript
import React from 'react';
import { localize } from 'react-localize-redux';

const Greeting = ({ translate, currentLanguage }) => (
  <div>
    <h1>{ translate('greeting', { name: 'Ryan' }) }</h1>
    <p>The current language is { `${ currentLanguage }` }</p>
    <button>{ translate('farwell') }</button>
  </div>
);

// decorate your component with localize
export default localize()(Greeting);
```

### Set local translations

In addtion to `global` translations you can also include translations specific to your component called `local` translations.
Similar to global translations `local` transaltions are added using an action creator `setLocalTranslations`.

Assuming we have a component called `WelcomeView` with translations specific to it stored in a file named `welcome.locale.json`.

```json
{
  "en": {
    "welcome-body": "Here is some <strong>bold</strong> text."
  },
  "fr": {
    "welcome-body": "Voici un texte en <strong>gras</strong>"
  },
  "es": {
    "welcome-body": "Aquí le damos algunos texto en <strong>negrita</strong>"
  }
}
```

First you will load the local json data passing in a `translationId`, in this case `welcome`, followed by the json data.

```javascript
import { setLocalTranslations } from 'react-localize-redux';

const json = require("welcome.locale.json");
store.dispatch(setLocalTranslations('welcome', json));
```

To access `local` translations in your component you still use the `localize` function, 
but this time passing in the unique id that was used in `setLocalTranslations`.

> NOTE: In addition to the `local` translations you will still have access `global` translations as well.

```javascript
import React from 'react';
import { localize } from 'react-localize-redux';

const WelcomeView = ({ translate }) => (
  <div>
    <h1>{ translate('greeting') }</h1>
    <p>{ translate('welcome-body') }</p>
    <button>{ translate('farwell') }</button>
  </div>
);

// pass in the unique id for the local content you would like to add
export default localize('welcome')(Greeting);
```

## Bundle translations with Webpack and react-router

When used with Webpack's [json-loader](https://github.com/webpack/json-loader), and react-router [Dynamic Routing](https://github.com/ReactTraining/react-router/blob/master/docs/guides/DynamicRouting.md) you can leverage code splitting to bundle components with their transaltion data. 

See the `dynamic-routes` folder in the examples forward on a working example of how to implement this setup.

## API

### localize( [translationId] )( WrappedComponent )

A HoC factory method that returns an enhanced version of the WrappedComponent with the additional props for 
adding localized content to your component. 

By calling `localize` with no params your WrappedComponent will only have access to `global` translations.

```javascript
const MyComponent = ({ translate }) => <div>{ translate('greeting') }</div>;
export default localize()(MyComponent);
```

By default all components decorated with `localize` will have access to `global` transaltions. To add additional transaltion
data that was added by [setLocalTranslations(translationId, json)](#setlocaltranslationsid-json) you will need pass the `translationId` as a param to `localize`.

```javascript
const MyComponent = ({ translate }) => <div>{ translate('title') }</div>;
export default localize('translationId')(MyComponent);
```

The following additional props are provided to localized components: 

#### currentLanguage

The current language set in your application. See [updateLanguage]() on how to update current language.

#### translate( id, data ) 

The translate will be used to insert translated copy in your component. The `id` param will need to match the property of the string you wish
to retrieve from your json translaion data.

The `data` param is optional and can be used to insert dynamic data into your translations. The syntax follows the Javascript template
literal format.

```javascript
// Here is a string where I want to dynamically insert the user's name, and country
{ "greet": "Hi here is my ${ name } and ${ country }" }

// With translate you'd do the following
translate('greet', { name: 'Ryan', country: 'Canada' })
```

For example if the below json file was added using either [setGlobalTranslations](#setglobaltranslationsjson) or [setLocalTranslations](#setlocaltranslationsid-json).

```json
{
  "en": {
    "title": "My Title",
    "desc": "My Description"
  },
  "fr": {
    "title": "My Title French",
    "desc": "My Description French"
  }
}
```

and this component had been decorated with [localize](#localize-translationid--wrappedcomponent-) you would access the json content like so...

```html
<h1>{ translate('title') }</h1>
<p>{ translate('desc') }</p>
```

>NOTE: The json content that `translate` has access to will depend on the `translationId` passed to the `localize` method.

### Redux Action Creators

#### updateLanguage(languageCode)

This will set the current language for your application, where `languageCode` should match the `languageCode` prop used in your translation json data.

#### setGlobalTranslations(json)

The global json should contain any localized content that will be shared by multiple components. 
By default all components created by [localize](#localize-translationid--wrappedcomponent-) will have access to transaltion from this global json.

#### setLocalTranslations(translationId, json)

The local json should contain localized content specific to a component. This is especially useful when used 
in combination with react-router dynamic routing, and webpack code splitting features.
