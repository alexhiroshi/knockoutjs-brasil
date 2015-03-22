---
layout: documentation
title: O binding "click"
---

### Propósito
O binding `click` adiciona um manipulador de eventos para que a sua função JavaScript seja chamada quando o elemento associado for clicado. Isso é mais comumente usado com elementos `button`, `input` e `a`, mas funciona com qualquer elemneto DOM visível.

### Exemplo
    <div>
        You've clicked <span data-bind="text: numberOfClicks"></span> times
        <button data-bind="click: incrementClickCounter">Click me</button>
    </div>

    <script type="text/javascript">
        var viewModel = {
            numberOfClicks : ko.observable(0),
            incrementClickCounter : function() {
                var previousCount = this.numberOfClicks();
                this.numberOfClicks(previousCount + 1);
            }
        };
    </script>

Cada vez que o botão for clicado, será chamado `incrementClickCounter()` da sua view model, que por sua vez altera o estado da view model e atualiza a UI.

### Parâmetros

  * Parâmetro principal

    A função que você quer vincular ao evento `click` do elemento.

    Você pode referenciar qualquer função JavaScript - ele não tem que ser uma função da seu view model. Você pode referenciar uma função em qualquer objeto escrevendo `click: someObject.someFunction`.

  * Parâmetros adicionais

     * Nenhum

### Nota 1: Passando um "item atual" como parâmetro do seu manipulador de evento

Ao chamar seu manipulador, Knockout irá aplicar o valor atual do model como o primeiro parâmetro. Isso é útil se você estiver interpretando algumas UI para cada item em uma coleção e você precisa saber qual item da UI foi clicado. Por exemplo:

    <ul data-bind="foreach: places">
        <li>
            <span data-bind="text: $data"></span>
            <button data-bind="click: $parent.removePlace">Remove</button>
        </li>
    </ul>

     <script type="text/javascript">
         function MyViewModel() {
             var self = this;
             self.places = ko.observableArray(['London', 'Paris', 'Tokyo']);

             // The current item will be passed as the first parameter, so we know which place to remove
             self.removePlace = function(place) {
                 self.places.remove(place)
             }
         }
         ko.applyBindings(new MyViewModel());
    </script>

Dois pontos a serem observados nesse exemplo:

 * Se você estiver dentro de um [binding context](binding-context.html) aninhado, por exemplo, se você estiver dentro de um `foreach` ou um bloco `with`, mas a sua função está na raiz da viewmodel ou algum outro contexto pai, você vai precisar um prefixo como `$parent` ou `$root` para localizar a função.
 * Em sua viewmodel, muitas vezes é útil declarar `self` (ou alguma outra variável) como um apelido para `this`. Fazer isso evita problemas com `this` sendo redefinidos para significar algo mais em eventos manipuladores ou callbacks de chamadas Ajax.

### Nota 2: Acessando o objeto do evento, ou passando mais parâmetros

Em alguns casos, você vai precisar acessar o objeto do elemento associado com o seu evento click. Knockout irá passar o evento como segundo parâmetro para a sua função, como nesse exemplo:

    <button data-bind="click: myFunction">
        Click me
    </button>

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

Se você precisa passar mais parâmetros, uma maneira de fazê-lo é envolver seu manipulador em uma função literal que recebe um parâmetro, como nesse exemplo:

    <button data-bind="click: function(data, event) { myFunction('param1', 'param2', data, event) }">
        Click me
    </button>

Agora, KO irá passar os objetos data e event para a sua função literal que estão disponíveis para serem passados para o manipulador.

Outra opção, se você preferir evitar a função literal, é possível usar a função [bind](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Function/bind), que atribui valores de parâmetros específicos para uma função de referência:

    <button data-bind="click: myFunction.bind($data, 'param1', 'param2')">
        Click me
    </button>

### Nota 3: Permitindo a ação padrão do click

Por padrão, Knockout irá impedir o click a tomar qualquer ação padrão. Dessa forma, se você usar o binding `click` em uma tag `a` (um link), por exemplo, o navegador irá chamar apenas sua função e não irá navegar para o link do `href`. Este é um padrão útil, porque quando você usa o `click`, normalmente é porque você quer usar o link como parte de uma UI que manipula a sua view model, não como um hyperlink normal para outra página.

Contudo, se você *quer* deixar a ação padrão prosseguir, apenas retorne `true` na sua função `click`.

### Nota 4: Preventing the event from bubbling

Por padrão, Knockout irá permitir o evento click para continuar a bolha até quaisquer manipuladores de eventos de alto nível. Por exemplo, se o seu elemento e o pai desse elemento tiverem um manipulador de evento click, então o evento click de ambos os elementos serão acionados. Se necessário, você pode impedir que o evento de um não chame o outro incluindo um binding adicional chamado `clickBubble` e passando false para ele, como neste exemplo:

        <div data-bind="click: myDivHandler">
            <button data-bind="click: myButtonHandler, clickBubble: false">
                Click me
            </button>
        </div>

Normalmente, nesse caso o evento `myButtonHandler` seria chamado primeiro, em seguida, o evento `myDivHandler`. Contudo, o binding `clickBubble` que nós adicionamos com o valor `false`, impede que o evento seja passado depois de `myButtonHandler`.

### Dependências

Nenhuma, exceto a biblioteca Knockout.