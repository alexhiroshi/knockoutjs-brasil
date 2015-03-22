---
layout: documentation
title: Estendendo a sintaxe binding do Knockout usando pré-processamento
---

*Nota: Isso é uma técnica avançada, normalmente usada apenas ao criar bibliotecas com bindings reutilizáveis. Não é algo que você normalmente precisa fazer ao criar aplicativos com Knockout.*

Começando com Knockout 3.0, desenvolvedores podem definir sintaxes personalizadas, proporcionando callbacks que reescrevem nós DOM e strings binding durante o processo de binding.

## Pré-processamento binding strings

You can hook into Knockout's logic for interpreting `data-bind` attributes by providing a *binding preprocessor* for a specific binding handler (such as `click`, `visible`, or any [custom binding handler](custom-bindings.html)).

Para fazer isso, adicione uma função `preprocess` para o manipulador de binding:

    ko.bindingHandlers.yourBindingHandler.preprocess = function(stringFromMarkup) {
        // Return stringFromMarkup if you don't want to change anything, or return
        // some other string if you want Knockout to behave as if that was the
        // syntax provided in the original HTML
    }

Veja mais adiante nessa página uma referência da API.

### Exemplo 1: Definindo um valor padrão para um binding

If you leave off the value of a binding, it's bound to `undefined` by default. If you want to have a different default value for a binding, you can do so with a preprocessor. For example, you can allow `uniqueName` to be bound without a value by making its default value `true`:

    ko.bindingHandlers.uniqueName.preprocess = function(val) {
        return val || 'true';
    }

Agora você pode vinculá-lo assim:

    <input data-bind="value: someModelProperty, uniqueName" />
    
### Exemplo 2: Expressões binding para eventos

If you'd like to be able to bind expressions to `click` events (rather than a function reference as Knockout expects), you can set up a preprocessor for the `click` handler to support this syntax:

    ko.bindingHandlers.click.preprocess = function(val) {
        return 'function($data,$event){ ' + val + ' }';
    }

Agora você pode vincular o `click` assim:

    <button type="button" data-bind="click: myCount(myCount()+1)">Increment</button>

### Referência do pré-processador binding

  * `ko.bindingHandlers.<name>.preprocess(value, name, addBindingCallback)`

    If defined, this function will be called for each `<name>` binding before the binding is evaluated.

    **Parameters:**

      * `value`: the syntax associated with the binding value before Knockout attempts to parse it (e.g., for `yourBinding: 1 + 1`, the associated value is `"1 + 1"` as a string).

      * `name`: the name of the binding (e.g., for `yourBinding: 1 + 1`, the name is `"yourBinding"` as a string).

      * `addBinding`: a callback function you can optionally use to insert another binding on the current element. This requires two parameters, `name` and `value`. For example, inside your `preprocess` function, call `addBinding('visible', 'acceptsTerms()');` to make Knockout behave as if the element had a `visible: acceptsTerms()` binding on it.

    **Return value**:

    Your `preprocess` function must return the new string value to be parsed and passed to the binding, or return `undefined` to remove the binding.

    For example, if you return `'value + ".toUpperCase()"'` as a string, then `yourBinding: "Bert"` would be interpreted as if the markup contained `yourBinding: "Bert".toUpperCase()`. Knockout will parse the returned value in the normal way, so it has to be a legal JavaScript expression.

    Don't return non-string values. That wouldn't make sense, because markup is always a string.

## Preprocessing DOM nodes

You can hook into Knockout's logic for traversing the DOM by providing a *node preprocessor*. This is a function that Knockout will call once for each DOM node that it walks over, both when the UI is first bound, and later when any new DOM subtrees are injected (e.g., via a [`foreach` binding](foreach-binding.html)).

To do this, define a `preprocessNode` function on your binding provider:

    ko.bindingProvider.instance.preprocessNode = function(node) {
        // Use DOM APIs such as setAttribute to modify 'node' if you wish.
        // If you want to leave 'node' in the DOM, return null or have no 'return' statement.
        // If you want to replace 'node' with some other set of nodes,
        //    - Use DOM APIs such as insertChild to inject the new nodes
        //      immediately before 'node'
        //    - Use DOM APIs such as removeChild to remove 'node' if required
        //    - Return an array of any new nodes that you've just inserted
        //      so that Knockout can apply any bindings to them
    }

Veja mais adiante nessa página uma referência da API.

### Exemplo 3: Elementos virtuais de template

Se você geralmente inclui conteúdo de template usando elementos virtuais, a sintaxe normal pode ser um pouco detalhado. Usando pré-processamento, você pode adicionar um novo formato de template que usa o comentário único:

    ko.bindingProvider.instance.preprocessNode = function(node) {
        // Only react if this is a comment node of the form <!-- template: ... -->
        if (node.nodeType == 8) {
            var match = node.nodeValue.match(/^\s*(template\s*:[\s\S]+)/);
            if (match) {
                // Create a pair of comments to replace the single comment
                var c1 = document.createComment("ko " + match[1]),
                    c2 = document.createComment("/ko");
                node.parentNode.insertBefore(c1, node);
                node.parentNode.replaceChild(c2, node);

                // Tell Knockout about the new nodes so that it can apply bindings to them
                return [c1, c2];
            }
        }
    }

Agora você pode incluir um template em sua view assim:

    <!-- template: 'some-template' -->

### Referência de pré-processamento

  * `ko.bindingProvider.instance.preprocessNode(node)`

    If defined, this function will be called for each DOM node before bindings are processed. The function can modify, remove, or replace `node`. Any new nodes must be inserted immediately before `node`, and if any nodes were added or `node` was removed, the function must return an array of the new nodes that are now in the document in place of `node`.
