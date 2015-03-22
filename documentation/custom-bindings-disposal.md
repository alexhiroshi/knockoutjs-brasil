---
layout: documentation
title: Custom disposal logic
---

Em uma típica aplicação Knockout, elementos DOM são dinamicamente adicionados e removidos, por exemplo, usando o [`template`](template-binding.html) ou via fluxo de controle ([`if`](if-binding.html), [`ifnot`](ifnot-binding.html), [`with`](with-binding.html), and [`foreach`](foreach-binding.html)). Quando criando um binding customizado, é normalmente desejável colocar uma lógica clean-up que é executada quando um elemento associado com seu binding customizado é removido pelo Knockout.

### Registrando um callback no descarte de um elemento

Para registrar uma função para executar quando um nó é removido, você precisa chamar `ko.utils.domNodeDisposal.addDisposeCallback(node, callback)`. Como exemplo, suponha que você crie um binding customizado para instanciar um widget. Quando o elemento com o binding é removido, você pode querer chamar o método `destroy` do widget:

    ko.bindingHandlers.myWidget = {
        init: function(element, valueAccessor) {
            var options = ko.unwrap(valueAccessor()),
                $el = $(element);

            $el.myWidget(options);

            ko.utils.domNodeDisposal.addDisposeCallback(element, function() {
                // This will be called when the element is removed by Knockout or
                // if some other part of your code calls ko.removeNode(element)
                $el.myWidget("destroy");
            });
        }
    };

### Overriding the clean-up of external data

Quando um elemento é removido, Knockout executa a lógica para limpar qualquer dado associado com o elemento. Como parte dessa lógica, Knockout chama o método jQuery `cleanData` se o jQuery estiver carregado em sua página. Em cenários avançados, você pode querer evitar ou customizar a forma como esses dados são removidos na sua aplicação. Knockout exibe um função, `ko.utils.domNodeDisposal.cleanExternalData(node)`, que pode ser sobrescrita para suportar lógicas customizadas. Por exemplo, para prevenir `cleanData` de ser chamado, uma função fazia poderia ser usada para substituir a implementação padrão `cleanExternalData`:

    ko.utils.domNodeDisposal.cleanExternalData = function () {
        // Do nothing. Now any jQuery data associated with elements will
        // not be cleaned up when the elements are removed from the DOM.
    };
