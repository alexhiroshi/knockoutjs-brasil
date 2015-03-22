---
layout: documentation
title: O binding "with"
---

### Propósito
O binding `with` cria um novo [binding context](binding-context.html), de modo que os elementos descendentes estão vinculados no contexto de um objecto especificado.

Claro, você pode arbitrairamente aninhar bindings `with` junto com outros bindings de fluxos de controle tais como [`if`](if-binding.html) e [`foreach`](foreach-binding.html).

### Exemplo 1

Esse é um exemplo muito básico de alterar o contexto de um objeto filho. Observe que nos atributos `data-bind`, não é necessário o prefixo de latitude` ou `longitude` com `coords` porque o contexto é alterado para `coords`.

    <h1 data-bind="text: city"> </h1>
    <p data-bind="with: coords">
        Latitude: <span data-bind="text: latitude"> </span>,
        Longitude: <span data-bind="text: longitude"> </span>
    </p>

    <script type="text/javascript">
        ko.applyBindings({
            city: "London",
            coords: {
                latitude:  51.5001524,
                longitude: -0.1262362
            }
        });
    </script>

### Exemplo 2

Esse exemplo interativo demonstra que:

 * O binding `with` vai dinamicamente adicionar ou remover elementos descendentes dependendo se o valor associado for `null`/`undefined` ou não
 * Se você quer acessar as data/functions do contexto pai, você pode usar [propriedade especial de contexto como $parent e $root](binding-context.html).

Experimente:

{% capture live_example_view %}
<form data-bind="submit: getTweets">
    Twitter account:
    <input data-bind="value: twitterName" />
    <button type="submit">Get tweets</button>
</form>

<div data-bind="with: resultData">
    <h3>Recent tweets fetched at <span data-bind="text: retrievalDate"> </span></h3>
    <ol data-bind="foreach: topTweets">
        <li data-bind="text: text"></li>
    </ol>

    <button data-bind="click: $parent.clearResults">Clear tweets</button>
</div>
{% endcapture %}

{% capture live_example_viewmodel %}
function AppViewModel() {
    var self = this;
    self.twitterName = ko.observable('@example');
    self.resultData = ko.observable(); // No initial value

    self.getTweets = function() {
        var name = self.twitterName(),
            simulatedResults = [
                { text: name + ' What a nice day.' },
                { text: name + ' Building some cool apps.' },
                { text: name + ' Just saw a famous celebrity eating lard. Yum.' }
            ];

        self.resultData({ retrievalDate: new Date(), topTweets: simulatedResults });
    }

    self.clearResults = function() {
        self.resultData(undefined);
    }
}

ko.applyBindings(new AppViewModel());
{% endcapture %}

{% include live-example-minimal.html %}

### Parâmetros

  * Parâmetro principal

    O objeto que você quer para usar como o contexto para binding de elementos descendentes.

    If the expression you supply evaluates to `null` or `undefined`, descendant elements will *not* be bound at all, but will instead be removed from the document.

    Se a sua expressão envolve quaisquer valores observable, a expressão será reavaliada sempre que algum deles mudar. 
    If the expression you supply involves any observable values, the expression will be re-evaluated whenever any of those observables change. Em seguida, os elementos descendentes serão apagados, e **uma nova cópia da marcação** será adicionada no seu documento e ligado ao novo contexto do resultado.

  * Parâmetros adicionais

     * Nenhum

### Nota 1: Usando "with" sem um elemento container

Assim como outros fluxos de controles, como [`if`](if-binding.html) e [`foreach`](foreach-binding.html), você pode usar `with` sem um elemento container. Isso é útil se você precisa usar o `with` em um lugar onde não seria válido adicionar um novo elemento para segurá-lo. Veja a documentação de [`if`](if-binding.html) ou [`foreach`](foreach-binding.html) para mais detalhes.

Exemplo:

    <ul>
        <li>Header element</li>
        <!-- ko with: outboundFlight -->
            ...
        <!-- /ko -->
        <!-- ko with: inboundFlight -->
            ...
        <!-- /ko -->
    </ul>

Os comentários `<!-- ko -->` e `<!-- /ko -->` funcionarão como marcadores de início/fim, definindo um “elemento virtual” que contém a marcação. Knockout entende esta sintaxe de elemento virtual e liga como se você tivesse um conjunto de elementos reais.

### Dependências

Nenhuma, exceto a biblioteca Knockout.