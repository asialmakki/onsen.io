### Creating bindings

Since Onsen UI is implemented on top of Web Components (standard technology), it can be used with just pure JavaScript or potencially along with any JS framework. This is straightforward for libraries and frameworks that simply add a layer of usability to the development such as jQuery. However, with more complex frameworks that also manage the DOM itself or need to compile it, some extra logic is needed to tell Onsen UI how to create and load new elements according to these frameworks. This logic is what we call "bindings".

In most of the cases they are simple wrappers that fix timing or DOM manipulation issues. Sometimes the bindings can implement some optional extra logic to support cool features of the framework. For example, in AngularJS it is common to write attributes for event handlers that can run code from controller scopes. This can be done in the bindings by simply defining these attributes (directives) and adding an event listener behind the scenes that runs a scoped handler. Thanks to this it is possible to tailor the code style in order to make it feel like a continuation of the used framework.

Onsen UI already have bindings support for the most common and possibly complex frameworks such as React, AngularJS, Angular 2 and Vue. Other libraries or frameworks like Knockout.js or jQuery are simple enough and do not require wrappers.

For those frameworks that do not have bindings yet, this guide explains how to extend Onsen UI to make it work properly.

#### Page loader for routing components

This is the main point that bindings usually need to modify. When creating new pages in the routing components (navigator, tabbar, splitter), Onsen UI core reads the content of `ons-templates` or makes a request to an external file and gets the content in string format. Afterwards, this string is converted to HTML and appended to the DOM. All of this is done through `ons.pageLoader` and can be modified to fit the needs of the framework in question.

Every routing component has a `pageLoader` property that can be overwritten with a new one. The `pageLoader` constructor needs two paramenter, a `loader` and an `unloader` functions:

```
const loader = ({page, parent, params}, done) => {
  myCustomBindings.createPageElementFrom(page)
    .then((pageElement) => {
      parent.appendChild(pageElement);
      done(pageElement);
    })
};

const unloader = (pageElement) => {
  myCustomBindings.cleanAllRelatedTo(pageElement);
  pageElement.remove();
}

myNavigator.pageLoader = new ons.pageLoader(loader, unloader);
```



The `loader` function gets two parameters. The first one is an object containing three properties: `page`, `parent` and `params`. The second one is a `done` function that must be called with the final result. `parent` is the HTMLElement where the final result should be attached to (`ons-navigator`, `ons-splitter-content`, etc.). `page` is the parameter provided in the API of the bindings. For example, in the core library `ons-navigator` expects a string as the page parameter: `myNavigator.pushPage('page.html', options)`. This parameter will be passed to the `pageLoader` as is. Therefore, the `pageLoader` decides what to do with that parameter. For the core it uses the string as a template ID or as a URL to get the real HTML content of the page. For Angular 2, this parameter is already an Angular 2 component so it just needs to be properly linked and loaded. Lastly, `params` is an object containing the extra options passed to the router component.

The `unloader` function is much simpler. It only receives `pageElement` as a parameter and must clean everything related to the element. In case your bindings store information outside the element itself and need a way to access it, JavaScript's [`WeakMap`](https://developer.mozilla.org/en/docs/Web/JavaScript/Reference/Global_Objects/WeakMap) might be a useful structure for this. It is possible to set a link to the elment's information in `loader` function using the element itself as a key and retrieve it in `unloader` function.

#### Wrappers for content components

Most of the components are simple HTML + CSS with some enhanced behavior that should work without issue. However, due to peculiarities and features of some frameworks, there could be timing issues or other minor problems. Usully this is fixed by creating a wrapper around the original component and using it instead. This wrapper can implement event handlers or modify the component if necessary. For example, `ons-input` in AngularJS needs to parse its content with Angular's `$parse` in order to make it work with `ng-model` attribute.

#### Examples

For real world examples, please have a look at the [`bindings`](https://github.com/OnsenUI/OnsenUI/blob/master/bindings) already implemented in Onsen UI. We are happy to accept Pull Requests with new bindings or improvements for the existing ones in this directory.

Another interesting example is [Knockout.js + Onsen UI](https://tutorial.onsen.io/index.html?external=https://tutorial.onsen.io/tutorial/other/knockout_bindings.html), where a simple event listener is enough to add Knockout functionality to Onsen UI pages.
