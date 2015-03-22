---
layout: documentation
title: O binding "event"
---

### Propósito
O binding `event` permite que você adicione um manipulador de evento para um evento específico para que a sua função JavaScript escolhida seja chamada quando esse evento for acionado no elemento DOM associado. Isso pode ser usado para qualquer evento, como `keypress`, `mouseover` ou `mouseout`.

### Exemplo
    <div>
        <div data-bind="event: { mouseover: enableDetails, mouseout: disableDetails }">
            Mouse over me
        </div>
        <div data-bind="visible: detailsEnabled">
            Details
        </div>
    </div>

    <script type="text/javascript">
        var viewModel = {
            detailsEnabled: ko.observable(false),
            enableDetails: function() {
                this.detailsEnabled(true);
            },
            disableDetails: function() {
                this.detailsEnabled(false);
            }
        };
        ko.applyBindings(viewModel);
    </script>

Agora, movendo o ponteiro do mouse sobre ou fora do primeiro elemento, será chamado métodos da view model para alternar o observable `detailsEnabled`. O segundo elemento reage às mudanças do valor de `detailsEnabled` mostrando ou escondendo.

### Parâmetros

  * Parâmetro principal

    Você deve passar um objeto JavaScript em que os nomes das propriedades conrrespondam aos nomes dos eventos e os valores correspondam à função que você deseja ligar ao evento.

    Você pode referenciar qualquer função JavaScript - ele não deve ser uma função da sua view model. Você pode referenciar uma função em qualquer objeto escrevendo `event { mouseover: someObject.someFunction }`.

  * Parâmetros adicionais

      * Nenhum

### Nota 1: Passando um "item autal" como um parâmetro para a sua função

Ao chamar o seu manipulador, Knockout irá fornecer o valor do model atual como o primeiro parâmetro. Isso é particularmente útil se você estiver renderizando algumas UI`s para cada item em uma coleção e você precisa saber qual o item que o evento se refere. Por exemplo:

    <ul data-bind="foreach: places">
        <li data-bind="text: $data, event: { mouseover: $parent.logMouseOver }"> </li>
    </ul>
    <p>You seem to be interested in: <span data-bind="text: lastInterest"> </span></p>

     <script type="text/javascript">
         function MyViewModel() {
             var self = this;
             self.lastInterest = ko.observable();
             self.places = ko.observableArray(['London', 'Paris', 'Tokyo']);

             // The current item will be passed as the first parameter, so we know which place was hovered over
             self.logMouseOver = function(place) {
                 self.lastInterest(place);
             }
         }
         ko.applyBindings(new MyViewModel());
    </script>

Dois pontos a serem observados nesse exemplo:

 * Se você está dentro de um [binding context](binding-context.html) aninhado, por exemplo, se você estiver dentro de um `foreach` ou um bloco `with`, mas a sua função está na raiz da viewmodel ou algum outro contexto pai, você vai precisar usar um prefixo como `$parent` ou `$root` para localizar a função.
 * Em sua viewmodel, muitas vezes é útil declar `self` (ou alguma outra variável) como um apelido para `this`. Fazer isso evita muitos problemas com `this` sendo redefinido para significar outra coisa no manipulador ou no callback de um Ajax.

### Nota 2: Acessando o objeto do evento ou passando mais parâmetros

Em algumas situações, pode ser necessário acessar o objeto do evento DOM associado com seu evento. Knockout irá passar o evento como o segundo parâmetro para a sua função, como nesse exemplo:

    <div data-bind="event: { mouseover: myFunction }">
        Mouse over me
    </div>

     <script type="text/javascript">
        var viewModel = {
            myFunction: function(data, event) {
                if (event.shiftKey) {
                    //do something different when user has shift key down
                } else {
                    //do normal action
                }
            }
        };
        ko.applyBindings(viewModel);
    </script>

Se você precisa passar mais parâmetros, uma maneira é envolver o seu manipulador em uma função literal que recebe um parâmetro, como nesse exemplo:

    <div data-bind="event: { mouseover: function(data, event) { myFunction('param1', 'param2', data, event) } }">
        Mouse over me
    </div>

Agora, KO irá passar o evento para a sua função literal, que é disponível para ser passado para o seu manipulador.

Alternativamente, se você preferir evitar a função literal em sua view, você pode usar a função [bind](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind), que atribui valores de parâmetros específicos para uma função de referência:

    <button data-bind="event: { mouseover: myFunction.bind($data, 'param1', 'param2') }">
        Click me
    </button>

### Nota 3: Permitindo a ação padrão

Por padrão, Knockout irá impedir o evento de tomar qualquer ação padrão. Por exemplo, se você usar o binding `event` para capturar o evento `keypress` de uma tag `input`, o navegador só irá chamar a sua função e *não* irá adicionar o valor da tecla para o valor do elemento `input`. Um exemplo mais comum é usar [o binding click](click-binding.html), que internamente usa esse binding, onde a sua função será chamada, mas o navegador *não* irá nevegar para o link do `href`. Isso é um padrão útil porque quando você usa o binding `click`, é normalmente porque você vai usar o link como parte de uma interface que manipula a sua view model, não como um hyperlink normal para outra página.

No entando, se você *quer* deixar a ação padrão continuar, apenas retorne `true` para a sua função `event`.

### Nota 4: Preventing the event from bubbling

Por padrão, Knockout irá permitir o evento para continuar a bolha até quaisquer manipuladores de eventos de alto nível. Por exemplo, se o seu elemento lida com um evento `mouseover` e o pai desse elemento também lida com esse mesmo evento, então o evento de ambos os elementos serão acionados. Se necessário, você pode impedir que o evento de um não chame o outro incluindo um binding adicional chamado `youreventBubble` e passando false para ele, como nese exemplo:

        <div data-bind="event: { mouseover: myDivHandler }">
            <button data-bind="event: { mouseover: myButtonHandler }, mouseoverBubble: false">
                Click me
            </button>
        </div>

Normalmente, nesse caso `myButtonHandler` seria chamado primeiro, em seguida, o evento `myDivHandler`. Contudo, o binding `mouseoverBubble` que nós adicionamos com o valor `false`, impede que o evento seja passado depois de `myButtonHandler`.

### Dependências

Nenhuma, exceto a biblioteca Knockout.