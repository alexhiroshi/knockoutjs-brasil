---
layout: documentation
title: Criando bindigns customizados que controlam bindings descendentes
---

*Nota: Isso é uma técnica avançada, normalmente usada apenas ao criar bibliotecas com bindings reutilizáveis. Não é algo que você normalmente precisa fazer ao criar aplicativos com Knockout.*

Por padrão, bindings apenas afetam o elemento no qual eles estão aplicado. Mas e se você quiser afetar todos os elementos decendentes também? Isto é possível. Seu binding pode dizer ao Knockout para não ligar aos decendentes, e então seu binding customizado pode fazer o que preferir para ligar eles de uma maneira diferente.

Para fazer isso, basta retornar `{ controlsDescendantBindings: true }` da função `init` do seu binding.

### Exemplo: Controlando se os bindings descendentes são complicados ou não

For a very simple example, here's a custom binding called `allowBindings` that allows descendant bindings to be applied only if its value is `true`. If the value is `false`, then `allowBindings` tells Knockout that it is responsible for descendant bindings so they won't be bound as usual.

    ko.bindingHandlers.allowBindings = {
        init: function(elem, valueAccessor) {
            // Let bindings proceed as normal *only if* my value is false
            var shouldAllowBindings = ko.unwrap(valueAccessor());
            return { controlsDescendantBindings: !shouldAllowBindings };
        }
    };

Para ver esse efeito funcionando, aqui está um exemplo:

    <div data-bind="allowBindings: true">
        <!-- This will display Replacement, because bindings are applied -->
        <div data-bind="text: 'Replacement'">Original</div>
    </div>

    <div data-bind="allowBindings: false">
        <!-- This will display Original, because bindings are not applied -->
        <div data-bind="text: 'Replacement'">Original</div>
    </div>

### Exemplo: Fornecendo valores adicionais aos bindings decendentes

Normalmente, bindins que usam `controlsDescendantBindings` também irão chamar `ko.applyBindingsToDescendants(someBindingContext`, `element`) para aplicar os bindings decendentes contra alguns [binding context](binding-context.html) modificados. Por exemplo, você poderia ter um binding chamado `withProperties` agrega propriedades extra ao binding context que irá, então, ser disponível à todos os bindings decendentes:

    ko.bindingHandlers.withProperties = {
        init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
            // Make a modified binding context, with a extra properties, and apply it to descendant elements
            var innerBindingContext = bindingContext.extend(valueAccessor);
            ko.applyBindingsToDescendants(innerBindingContext, element);

            // Also tell KO *not* to bind the descendants itself, otherwise they will be bound twice
            return { controlsDescendantBindings: true };
        }
    };

Como você pode, binding contexts tem uma função `extend` que produz um clone com propriedades extras. A função `extend` aceita tanto um objeto com propriedades para copiar ou uma função que retorna um objeto como tal. A sintaxe da função é preferida para que alterações futuras ao valor do binding sejam sempre atualizadas no binding context. Este processo não afeta o binding context original, então não há perigo de afetar elementos filhos – irá afetar apenas os decendentes.

Aqui está um exemplo do uso do binding customizado acima:

    <div data-bind="withProperties: { emotion: 'happy' }">
        Today I feel <span data-bind="text: emotion"></span>. <!-- Displays: happy -->
    </div>
    <div data-bind="withProperties: { emotion: 'whimsical' }">
        Today I feel <span data-bind="text: emotion"></span>. <!-- Displays: whimsical -->
    </div>

### Exemplo: Adicionando níveis extras à hierarquia do binding context

Bindings como [`with`](with-binding.html) e [`foreach`](foreach-binding.html) criam níveis extras na hierarquia do binding context. Isto significa que seus decendentes podem acessar dodos no níveis exteriores usando `$parent`, `$parents`, `$root`, ou `$parentContext`.

Se você quer fazer isto com bindings customizados, então em vez de usar `bindingContext.extend()`, use `bindingContext.createChildContext(someData)`. Isto retorna um novo binding context no qual o viewmodel é `someData` e no qual `$parentContext` é `bindingContext`. Se você quiser, você pode então extender o child context com propriedades extras usando `ko.utils.extend`. Por exemplo:

    ko.bindingHandlers.withProperties = {
        init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
            // Make a modified binding context, with a extra properties, and apply it to descendant elements
            var childBindingContext = bindingContext.createChildContext(
                bindingContext.$rawData, 
                null, // Optionally, pass a string here as an alias for the data item in descendant contexts
                function(context) {
                    ko.utils.extend(context, valueAccessor());
                });
            ko.applyBindingsToDescendants(childBindingContext, element);

            // Also tell KO *not* to bind the descendants itself, otherwise they will be bound twice
            return { controlsDescendantBindings: true };
        }
    };

Este binding `withProperties` agora atualizado, poderia ser usado numa maneira aninhada, como cada nível do ninho podendo acessar o nível pai via `$parentContext`:

    <div data-bind="withProperties: { displayMode: 'twoColumn' }">
        The outer display mode is <span data-bind="text: displayMode"></span>.
        <div data-bind="withProperties: { displayMode: 'doubleWidth' }">
            The inner display mode is <span data-bind="text: displayMode"></span>, but I haven't forgotten
            that the outer display mode is <span data-bind="text: $parentContext.displayMode"></span>.
        </div>
    </div>

Modificando os binding contexts e controlando os bindings decendentes, você tem uma ferramenta poderosa e avançada para criar mecanismos de binding customizados.