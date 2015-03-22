---
layout: documentation
title: O binding "component"
---

O binding `component` injeta um [componente](component-overview.html) especificado em um elemento e opcionalmente passa parâmetros a ele.

* [Table of contents injected here]
{:toc}

### Live example

<style type="text/css">
    .liveExample h4 { margin-bottom: 0.3em; }
    .liveExample h4:first-of-type { margin-top: 0; }
</style>

{% capture live_example_viewmodel %}
    ko.components.register('message-editor', {
        viewModel: function(params) {
            this.text = ko.observable(params && params.initialText || '');
        },
        template: 'Message: <input data-bind="value: text" /> '
                + '(length: <span data-bind="text: text().length"></span>)'
    });

    ko.applyBindings();
{% endcapture %}
{% capture live_example_view %}
    <h4>First instance, without parameters</h4>
    <div data-bind='component: "message-editor"'></div>

    <h4>Second instance, passing parameters</h4>
    <div data-bind='component: {
        name: "message-editor",
        params: { initialText: "Hello, world!" }
    }'></div>
{% endcapture %}
{% include live-example-minimal.html %}

Note: In more realistic cases, you would typically load component viewmodels and templates from external files, instead of hardcoding them into the registration. See [an example](component-overview.html#example-loading-the-likedislike-widget-from-external-files-on-demand) and [registration documentation](component-registration.html).

### API

Existem duas maneiras de usar o binding `component`:

  * **Sintaxe abreviada**
   
    Se você passar apenas uma string, ela é interpretada como um nome do componente. Então o componente chamado é injetado sem fornecer quaisquer parâmetros a ele. Exemplo:

        <div data-bind='component: "my-component"'></div>

    The shorthand value can also be observable. In this case, if it changes, the `component` binding will [dispose](#disposal-and-memory-management) the old component instance, and inject the newly-referenced component. Exemplo:

        <div data-bind='component: observableWhoseValueIsAComponentName'></div>

  * **Sintaxe completa**

    Para enviar parâmetros para o componente, passe um objeto com as seguintes propriedades:

      * `name` --- o nome do componente a ser injetado. Novamente, isso pode ser observable.
      * `params` --- um objeto que será passado para o componente. Geralmente esse é um objeto de chave-valor que contém vários parâmetros e é normalmente recebida pelo construtor viewmodel do componente.

    Exemplo:

        <div data-bind='component: {
            name: "shopping-cart",
            params: { mode: "detailed-list", items: productsList }
        }'></div>

Note que sempre que o componente é removido (ou porque o `name` observable é alterado, ou porque um delimitador de fluxo de controle remove todo o elemento), o componente removido fica [disposed](#disposal-and-memory-management)

### Ciclo de vida do component

Quando um binding `component` injeta um componente,

 1. **Your component loaders are asked to supply the viewmodel factory and template**

      * Multiple component loaders may be consulted, until the first one recognises the component name and supplies a viewmodel/template. This process only takes place **once per component type**, since Knockout caches the resulting definitions in memory.
      * The default component loader supplies viewmodels/templates based on [what you have registered](component-registration.html). If applicable, this is the phase where it requests any specified AMD modules from your AMD loader.

    Normally, this is an *asynchronous* process. It may involve requests to the server. For API consistency, Knockout by default ensures that the loading process completes as an asynchronous callback even if the component is already loaded and cached in memory. For more about this, and how to allow synchronous loading, see [Controlling synchronous/asynchronous loading](component-registration.html#controlling-synchronousasynchronous-loading).

 2. **The component template is cloned and injected into the container element**

    Any existing content is removed and discarded.

 3. **If the component has a viewmodel, it is instantiated**

    If the viewmodel is given as a constructor function, this means Knockout calls `new YourViewModel(params)`.

    If the viewmodel is given as a `createViewModel` factory function, Knockout calls `createViewModel(params, componentInfo)`, where `componentInfo.element` is the element into which the not-yet-bound template has already been injected.

    This phase always completes synchronously (constructors and factory functions are not allowed to be asynchronous), since it occurs *every time a component is instantiated* and performance would be unacceptable if it involved waiting for network requests.

 4. **The viewmodel is bound to the view**

    Or, if the component has no viewmodel, then the view is bound to any `params` you've supplied to the `component` binding.

 5. **The component is active**

    Now the component is operating, and can remain on-screen for as long as needed.

    If any of the parameters passed to the component is observable, then the component can of course observe any changes, or even write back modified values. This is how it can communicate cleanly with its parent, without tightly coupling the component code to any parent that uses it.

 6. **The component is torn down, and the viewmodel is disposed**

    If the `component` binding's `name` value changes observably, or if an enclosing control-flow binding causes the container element to be removed, then any `dispose` function on the viewmodel is called just before the container element is removed from the DOM. See also: [disposal and memory management](#disposal-and-memory-management).

    Note: If the user navigates to an entirely different web page, browsers do this without asking any code running in the page to clean up. So in this case no `dispose` functions will be invoked. This is OK because the browser will automatically release the memory used by all objects that were in use.

### Note: Template-only components

Components usually have viewmodels, but they don't necessarily have to. A component can specify just a template.

In this case, the object to which the component's view is bound is the `params` object that you passed to the `component` binding. Example:

    ko.components.register('special-offer', {
        template: '<div class="offer-box" data-bind="text: productName"></div>'
    });

... can be injected with:

    <div data-bind='component: {
         name: "special-offer-callout",
         params: { productName: someProduct.name }
    }'></div>

... or, more conveniently, as a [custom element](component-custom-elements.html):

    <special-offer params='productName: someProduct.name'></special-offer>

### Note: Usando `component` sem um elemento container

Em alguns casos, você pode querer injetar um componente em uma view sem utilizar um elemento container extra. Você pode fazer isso usando a sintaxe de *fluxo de controle sem container*, que é baseada em tags de comentários. Por exemplo:

    <!-- ko component: "message-editor" -->
    <!-- /ko -->

... ou passando parâmetros:

    <!-- ko component: {
        name: "message-editor",
        params: { initialText: "Hello, world!", otherParam: 123 }
    } -->
    <!-- /ko -->

Os comentários `<!-- ko -->` e `<!-- /ko -->` funcionarão como marcadores de início/fim, definindo um “elemento virtual” que contém a marcação. Knockout entende esta sintaxe de elemento virtual e liga como se você tivesse um conjunto de elementos reais.

### Note: Passing markup to components

The element you attach a `component` binding to may contain further markup. For example,

    <div data-bind="component: { name: 'my-special-list', params: { items: someArrayOfPeople } }">
        <!-- Look, here's some arbitrary markup. By default it gets stripped out
             and is replaced by the component output. -->
        The person <em data-bind="text: name"></em>
        is <em data-bind="text: age"></em> years old.
    </div>

Although the DOM nodes in this element will be stripped out and not bound by default, they are not lost. Instead, they are supplied to the component (in this case, `my-special-list`), which can include them in its output however it wishes.

This is useful if you want to build components that represent "container" UI elements, such as grids, lists, dialogs, or tab sets, which need to inject and bind arbitrary markup into a common structure. See [a complete example for custom elements](component-custom-elements.html#passing-markup-into-components), which also works without custom elements using the syntax shown above.

### Disposal and memory management

Optionally, your viewmodel class may have a `dispose` function. If implemented, Knockout will call this whenever the component is being torn down and removed from the DOM (e.g., because the corresponding item was removed from a `foreach`, or an `if` binding has become `false`).

You must use `dispose` to release any resources that aren't inherently garbage-collectable. Por exemplo:

 * `setInterval` callbacks will continue to fire until explicitly cleared.
   * Use `clearInterval(handle)` to stop them, otherwise your viewmodel might be held in memory.
 * `ko.computed` properties continue to receive notifications from their dependencies until explicitly disposed.
   * If a dependency is on an external object, then be sure to use `.dispose()` on the computed property, otherwise it (and possibly also your viewmodel) will be held in memory. Alternatively, consider using a [*pure* computed](computed-pure.html) to avoid the need for manual disposal.
 * **Subscriptions** to observables continue to fire until explicitly disposed.
   * If you have subscribed to an external observable, be sure to use `.dispose()` on the subscription, otherwise the callback (and possibly also your viewmodel) will be held in memory.
 * Manually-created **event handlers** on external DOM elements, if created inside a `createViewModel` function (or even inside a regular component viewmodel, although to fit the MVVM pattern you shouldn't) must be removed.
   * Of course, you don't have to worry about releasing any event handlers created by standard Knockout bindings in your view, as KO automatically unregisters them when the elements are removed.

Por exemplo:

    var someExternalObservable = ko.observable(123);

    function SomeComponentViewModel() {
        this.myComputed = ko.computed(function() {
            return someExternalObservable() + 1;
        }, this);

        this.myPureComputed = ko.pureComputed(function() {
            return someExternalObservable() + 2;
        }, this);

        this.mySubscription = someExternalObservable.subscribe(function(val) {
            console.log('The external observable changed to ' + val);
        }, this);

        this.myIntervalHandle = window.setInterval(function() {
            console.log('Another second passed, and the component is still alive.');
        }, 1000);
    }

    SomeComponentViewModel.prototype.dispose = function() {
        this.myComputed.dispose();
        this.mySubscription.dispose();
        window.clearInterval(this.myIntervalHandle);
        // this.myPureComputed doesn't need to be manually disposed.
    }

    ko.components.register('your-component-name', {
        viewModel: SomeComponentViewModel,
        template: 'some template'
    });

It isn't strictly necessary to dispose computeds and subscriptions that only depend on properties of the same viewmodel object, since this creates only a circular reference which JavaScript garbage collectors know how to release. However, to avoid having to remember which things need disposal, you may prefer to use `pureComputed` wherever possible, and explicitly dispose all other computeds/subscriptions whether technically necessary or not.
