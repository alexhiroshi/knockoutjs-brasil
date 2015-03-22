---
layout: documentation
title: Elementos customizados
---

Elementos customizados oferecem uma maneira conveniente de injetar [components](component-overview.html) em suas views.

* [Table of contents injected here]
{:toc}

### Introdução

Elementos customizados são uma alternativa sintática para o [binding `component`](component-binding.html) (na verdade, elementos customizados usam um binding component por trás).

Por exemplo, em vez de escrever isso:

    <div data-bind='component: { name: "flight-deals", params: { from: "lhr", to: "sfo" } }'></div>

... você pode escrever:

    <flight-deals params='from: "lhr", to: "sfo"'></flight-deals>

This allows for a very modern, [WebComponents](http://www.w3.org/TR/components-intro/)-like way to organize your code, while retaining support for even very old browsers (see [custom elements and IE 6 to 8](#note-custom-elements-and-internet-explorer-6-to-8)).

### Exemplo

Esse exemplo declara um componente e então adicionar duas instancia dele na view. Veja o código fonte abaixo.

<style type="text/css">
    .liveExample h4 { margin-bottom: 0.3em; }
    .liveExample h4:first-of-type { margin-top: 0; }
</style>

{% capture live_example_viewmodel %}
    ko.components.register('message-editor', {
        viewModel: function(params) {
            this.text = ko.observable(params.initialText || '');
        },
        template: 'Message: <input data-bind="value: text" /> '
                + '(length: <span data-bind="text: text().length"></span>)'
    });

    ko.applyBindings();
{% endcapture %}
{% capture live_example_view %}
    <h4>First instance, without parameters</h4>
    <message-editor></message-editor>

    <h4>Second instance, passing parameters</h4>
    <message-editor params='initialText: "Hello, world!"'></message-editor>
{% endcapture %}
{% include live-example-minimal.html %}

Nota: Em casos mais reais, você normalmente carregaria o componente viewmodel e templates de um arquivo externo, em vez de codificá-los no registro. Veja [um exemplo](component-overview.html#example-loading-the-likedislike-widget-from-external-files-on-demand) e a [documentação para registrar componentes](component-registration.html).

### Passando parâmetros

As you have seen in the examples above, you can use a `params` attribute to supply parameters to the component viewmodel. The contents of the `params` attribute are interpreted like a JavaScript object literal (just like a `data-bind` attribute), so you can pass arbitrary values of any type. Example:

    <unrealistic-component
        params='stringValue: "hello",
                numericValue: 123,
                boolValue: true,
                objectValue: { a: 1, b: 2 },
                dateValue: new Date(),
                someModelProperty: myModelValue,
                observableSubproperty: someObservable().subprop'>
    </unrealistic-component>

#### Communication between parent and child components

If you refer to model properties in a `params` attribute, then you are of course referring to the properties on the viewmodel outside the component (the 'parent' or 'host' viewmodel), since the component itself is not instantiated yet. In the above example, `myModelValue` would be a property on the parent viewmodel, and would be received by the child component viewmodel's constructor as `params.someModelProperty`.

This is how you can pass properties from a parent viewmodel to a child component. If the properties themselves are observable, then the parent viewmodel will be able to observe and react to any new values inserted into them by the child component.

#### Passando expressões observables

No seguinte exemplo:

    <some-component
        params='simpleExpression: 1 + 1,
                simpleObservable: myObservable,
                observableExpression: myObservable() + 1'>
    </some-component>

... o parâmetro `params` do do viewmodel terá três valores:

  * `simpleExpression`
      * This will be the numeric value `2`. It will not be an observable or computed value, since there are no observables involved.

        In general, if a parameter's evaluation does not involve evaluating an observable (in this case, the value did not involve observables at all), then the value is passed literally. If the value was an object, then the child component could mutate it, but since it's not observable the parent would not know the child had done so.

  * `simpleObservable`
      * This will be the [`ko.observable`](observables.html) instance declared on the parent viewmodel as `myObservable`. It is not a wrapper --- it's the actual same instance as referenced by the parent. So if the child viewmodel writes to this observable, the parent viewmodel will receive that change.

        In general, if a parameter's evaluation does not involve evaluating an observable (in this case, the observable was simply passed without evaluating it), then the value is passed literally.

  * `observableExpression`
      * This one is trickier. The expression itself, when evaluated, reads an observable. That observable's value could change over time, so the expression result could change over time.

        To ensure that the child component can react to changes in the expression value, Knockout **automatically upgrades this parameter to a computed property**. So, the child component will be able to read `params.observableExpression()` to get the current value, or use `params.observableExpression.subscribe(...)`, etc.

        In general, with custom elements, if a parameter's evaluation involves evaluating an observable, then Knockout automatically constructs a `ko.computed` value to give the expression's result, and supplies that to the component.

In summary, the general rule is:

  1. If a parameter's evaluation **does not** involve evaluating an observable/computed, it is passed literally.
  2. If a parameter's evaluation **does** involve evaluating one or more observables/computeds, it is passed as a computed property so that you can react to changes in the parameter value.

### Passing markup into components

Sometimes you may want to create a component that receives markup and uses it as part of its output. For example, you may want to build a "container" UI element such as a grid, list, dialog, or tab set that can receive and bind arbitrary markup inside itself.

Consider a special list component that can be invoked as follows:

    <my-special-list params="items: someArrayOfPeople">
        <!-- Look, I'm putting markup inside a custom element -->
        The person <em data-bind="text: name"></em>
        is <em data-bind="text: age"></em> years old.
    </my-special-list>

By default, the DOM nodes inside `<my-special-list>` will be stripped out (without being bound to any viewmodel) and replaced by the component's output. However, those DOM nodes aren't lost: they are remembered, and are supplied to the component in two ways:

 * As an array, `$componentTemplateNodes`, available to any binding expression in the component's template (i.e., as a [binding context](binding-context.html) property). Usually this is the most convenient way to use the supplied markup. See the example below.
 * As an array, `componentInfo.templateNodes`, passed to its [`createViewModel` function](component-registration.html#a-createviewmodel-factory-function)

The component can then choose to use the supplied DOM nodes as part of its output however it wishes, such as by using `template: { nodes: $componentTemplateNodes }` on any element in the component's template.

For example, the `my-special-list` component's template can reference `$componentTemplateNodes` so that its output includes the supplied markup. Here's the complete working example:

{% capture live_example_id %}component-pass-markup{% endcapture %}
{% capture live_example_viewmodel %}
    ko.components.register('my-special-list', {
        template: { element: 'my-special-list-template' },
        viewModel: function(params) {
            this.myItems = params.items;
        }
    });

    ko.applyBindings({
        someArrayOfPeople: ko.observableArray([
            { name: 'Lewis', age: 56 },
            { name: 'Hathaway', age: 34 }
        ])
    });
{% endcapture %}
{% capture live_example_view %}
    <!-- This could be in a separate file -->
    <template id="my-special-list-template">
        <h3>Here is a special list</h3>

        <ul data-bind="foreach: { data: myItems, as: 'myItem' }">
            <li>
                <h4>Here is another one of my special items</h4>
                <!-- ko template: { nodes: $componentTemplateNodes, data: myItem } --><!-- /ko -->
            </li>
        </ul>
    </template>

    <my-special-list params="items: someArrayOfPeople">
        <!-- Look, I'm putting markup inside a custom element -->
        The person <em data-bind="text: name"></em>
        is <em data-bind="text: age"></em> years old.
    </my-special-list>
{% endcapture %}
{% include live-example-minimal.html %}

This "special list" example does nothing more than insert a heading above each list item. But the same technique can be used to create sophisticated grids, dialogs, tab sets, and so on, since all that is needed for such UI elements is common UI markup (e.g., to define the grid or dialog's heading and borders) wrapped around arbitrary supplied markup.

This technique is also possible when using components *without* custom elements, i.e., [passing markup when using the `component` binding directly](component-binding.html#note-passing-markup-to-components).

### Controlando nome de tags de elementos customizados

Por padrão, Knockout assume que suas tags de elementos customizados correspondem exatamente aos nomes dos componentes registrados usando `ko.components.register`. Essa estratégia de converção sobre configuração é a ideia de muitas aplicações.

Se você precisa ter nomes de tags diferentes, você pode dar um override no `getComponentNameForNode` para controlar isso. Por exemplo:

    ko.components.getComponentNameForNode = function(node) {
        var tagNameLower = node.tagName && node.tagName.toLowerCase();

        if (ko.components.isRegistered(tagNameLower)) {
            // If the element's name exactly matches a preregistered
            // component, use that component
            return tagNameLower;
        } else if (tagNameLower === "special-element") {
            // For the element <special-element>, use the component
            // "MySpecialComponent" (whether or not it was preregistered)
            return "MySpecialComponent";
        } else {
            // Treat anything else as not representing a component
            return null;
        }
    }

You can use this technique if, for example, you want to control which subset of registered components may be used as custom elements.

### Registrando elementos customizados {#registering-custom-elements}

If you are using the default component loader, and hence are registering your components using `ko.components.register`, then there is nothing extra you need to do. Components registered this way are immediately available for use as custom elements.

If you have implemented a [custom component loader](component-loaders.html), and are not using `ko.components.register`, then you need to tell Knockout about any element names you wish to use as custom elements. To do this, simply call `ko.components.register` - you don't need to specify any configuration, since your custom component loader won't be using the configuration anyway. For example,

    ko.components.register('my-custom-element', { /* No config needed */ });

Alternatively, you can [override `getComponentNameForNode`](#controlling-custom-element-tag-names) to control dynamically which elements map to which component names, independently of preregistration.

### Note: Combining custom elements with regular bindings

A custom element can have a regular `data-bind` attribute (in addition to any `params` attribute) if needed. For example,

    <products-list params='category: chosenCategory'
                   data-bind='visible: shouldShowProducts'>
    </products-list>

However, it does not make sense to use bindings that would modify the element's contents, such as the [`text`](text-binding.html) or [`template`](template-binding.html) bindings, since they would overwrite the template injected by your component.

Knockout will prevent the use of any bindings that use [`controlsDescendantBindings`](custom-bindings-controlling-descendant-bindings.html), because this also would clash with the component when trying to bind its viewmodel to the injected template. Therefore if you want to use a control flow binding such as `if` or `foreach`, then you must wrap it around your custom element rather than using it directly on the custom element, e.g.,:

    <!-- ko if: someCondition -->
        <products-list></products-list>
    <!-- /ko -->

ou:

    <ul data-bind='foreach: allProducts'>
        <product-details params='product: $data'></product-details>
    </ul>

### Nota: Elementos customizados não podem se auto fechar

Você deve escrever `<my-custom-element></my-custom-element>`, e **não** `<my-custom-element />`. Caso contrário, seu elemento customizado não estará fechado e os elementos subsequentes serão analisados como elementos filhos.

Isso é uma limitação das especificações HTML e está fora do escopo do que o Knockout pode controlar. Analisadores HTML, seguindo as especificações HTML, [ignoram qualquer barra de fechamento automático](http://dev.w3.org/html5/spec-author-view/syntax.html#syntax-start-tag) (com exceção de um pequeno número de "elementos estrangeiros" especiais, que são condificados no analisador). HTML não é como XML.

### Nota: Elementos customizados e Internet Explorer 6 a 8

Knockout se esforça para poupar a dor dos desenvolvedores de lidar com questões de compatibilidade de cross-browser, especialmente aquelas relacionadas aos navegadores antigos! Embora os elementos customizados proporcionam um estilo muito moderno de desenvolvimento web, eles ainda funcionam em todos os navegadores comumente encontrados:

 * HTML5-era browsers, which includes **Internet Explorer 9** and later, automatically allow for custom elements with no difficulties.
 * **Internet Explorer 6 to 8** also supports custom elements, *but only if they are registered before the HTML parser encounters any of those elements*.

IE 6-8's HTML parser will discard any unrecognized elements. To ensure it doesn't throw out your custom elements, you must do one of the following:

 * Ensure you call `ko.components.register('your-component')` *before* the HTML parser sees any `<your-component>` elements
 * Or, at least call `document.createElement('your-component')` *before* the HTML parser sees any `<your-component>` elements. You can ignore the result of the `createElement` call --- all that matters is that you have called it.

Por exemplo, se você estruturar a sua página como esta, então tudo estará OK:

    <!DOCTYPE html>
    <html>
        <body>
            <script src='some-script-that-registers-components.js'></script>

            <my-custom-element></my-custom-element>
        </body>
    </html>

Se você estiver trabalhando com AMD, então você deve preferir uma estrutura como esta:

    <!DOCTYPE html>
    <html>
        <body>
            <script>
                // Since the components aren't registered until the AMD module
                // loads, which is asynchronous, the following prevents IE6-8's
                // parser from discarding the custom element
                document.createElement('my-custom-element');
            </script>

            <script src='require.js' data-main='app/startup'></script>

            <my-custom-element></my-custom-element>
        </body>
    </html>

Or if you really don't like the hackiness of the `document.createElement` call, then you could use a [`component` binding](component-binding.html) for your top-level component instead of a custom element. As long as all other components are registered before your `ko.applyBindings` call, they can be used as custom elements on IE6-8 without futher trouble:

    <!DOCTYPE html>
    <html>
        <body>
            <!-- The startup module registers all other KO components before calling
                 ko.applyBindings(), so they are OK as custom elements on IE6-8 -->
            <script src='require.js' data-main='app/startup'></script>

            <div data-bind='component: "my-custom-element"'></div>
        </body>
    </html>

### Avançado: Acessando o parâmetro `$raw`

Considere o seguinte caso incomum, em que `useObservable1`, `observable1` e `observable2` são todos observables:

    <some-component
        params='myExpr: useObservable1() ? observable1 : observable2'>
    </some-component>

Since evaluating `myExpr` involves reading an observable (`useObservable1`), KO will supply the parameter to the component as a computed property.

However, the value of the computed property is itself an observable. This would seem to lead to an awkward scenario, where reading its current value would involve double-unwrapping (i.e., `params.myExpr()()`, where the first parentheses give the value of the expression, and the second give the value of the resulting observable instance).

This double-unwrapping would be ugly, inconvenient, and unexpected, so Knockout automatically sets up the generated computed property (`params.myExpr`) to unwrap its value for you. That is, the component can read `params.myExpr()` to get the value of whichever observable has been selected (`observable1` or `observable2`), without the need for double-unwrapping.

In the unlikely event that you *don't* want the automatic unwrapping, because you want to access the `observable1`/`observable2` instances directly, you can read values from `params.$raw`. For example,

    function MyComponentViewModel(params) {
        var currentObservableInstance = params.$raw.myExpr();
        
        // Now currentObservableInstance is either observable1 or observable2
        // and you would read its value with "currentObservableInstance()"
    }

Esse deve ser um cenário muito incomum, deste modo, normalmente você não irá precisar trabalhar com `$raw`.