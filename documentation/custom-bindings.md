---
layout: documentation
title: Criando bindings customizados
---

Você não está limitado a usar os bindings integrados como `click`, `value` etc. – você pode criar os seus. Isto é como controlar como observable interage com os elementos DOM, e te dá muita flexibilidade para encapsular comportamentos sofisticados em uma maneira fácil de reutilizar.

Por exemplo, você pode criar componentes interativos como grids, tabsets, e etc. na forma de bindings customizados (veja o [exemplo grid](http://knockoutjs.com/examples/grid.html)).

### Registrando seu binding

Para registrar um binding, adicione-o como uma subpropriedade de um `ko.bindingHandlers`:

    ko.bindingHandlers.yourBindingName = {
        init: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
            // This will be called when the binding is first applied to an element
            // Set up any initial state, event handlers, etc. here
        },
        update: function(element, valueAccessor, allBindings, viewModel, bindingContext) {
            // This will be called once when the binding is first applied to an element,
            // and again whenever any observables/computeds that are accessed change
            // Update the DOM element based on the supplied values here.
        }
    };

... e então você pode usar em inúmeros elementos DOM:

    <div data-bind="yourBindingName: someValue"> </div>

Nota: você na verdade não precisa providenciar ambos os callback `init` e `update` – você pode só providenciar um ou outro se é tudo o que você precisa.

### O callback "update"

Knockout chamará o callback `update` inicialmente quando o binding é aplicado a um elemento e localizar qualquer dependência (observables/computeds) que você acessar. Quando qualquer uma dessas dependências serem alteradas, o callback `update` será chamado novamente. Os seguintes parâmetros são passados para ele:

 * `element` --- O elemento DOM envolvido nesse binding
 * `valueAccessor` --- Uma função JavaScript que você pode chamar para obter a propriedade model atual que está envolvida nesse binding. Chame sem passar qualquer parâmetros (em outras palavras, chame `valueAccessor()`) para obter o valor da propriedade model. Para aceitar valores simples e observable, chame `ko.unwrap` no valor retornado.
 * `allBindings` --- Um objeto JavaScript que pode usar para acessar todos os valores do model ligados ao elemento DOM. Chame `allBindings.get('name')` para adquirir o valor do binding `name` (retorna `undefined` se o binding não existir); ou `allBindings.has('name')` para determinar se o binding `name` está presente no elemento.
 * `viewModel` --- Este parâmetro está deprecado na versão 3.x do Knockout. Use `bindingContext.$data` ou `bindingContext.$rawData` para acessar a view model.
 * `bindingContext` --- Um objeto que guarda o [binding context](http://knockoutjs.com/documentation/binding-context.html) disponível neste binding do elemento. Este objeto inclui propriedades especiais incluindo `$parent`, `$parents`, e `$root` que pode ser usado para acessar dados que estão ligados aos ancestrais deste contexto.

exemplo, você pode ter estado controlando a visibilidade de um elemento usando ob binding `visible`, mas agora você quer dar um passo a mais e animar a transição. Você quer os elementos deslizem de acordo com o valor de um observable. Você pode fazer isto escrevendo um binding customizado que chama as funções `slideUp`/`slideDown` do jQuery:

    ko.bindingHandlers.slideVisible = {
        update: function(element, valueAccessor, allBindings) {
            // First get the latest data that we're bound to
            var value = valueAccessor();

            // Next, whether or not the supplied model property is observable, get its current value
            var valueUnwrapped = ko.unwrap(value);

            // Grab some more data from another binding property
            var duration = allBindings.get('slideDuration') || 400; // 400ms is default duration unless otherwise specified

            // Now manipulate the DOM element
            if (valueUnwrapped == true)
                $(element).slideDown(duration); // Make the element visible
            else
                $(element).slideUp(duration);   // Make the element invisible
        }
    };

Agora você pode usar esse binding da seguinta maneira:

    <div data-bind="slideVisible: giftWrap, slideDuration:600">You have selected the option</div>
    <label><input type="checkbox" data-bind="checked: giftWrap" /> Gift wrap</label>

    <script type="text/javascript">
        var viewModel = {
            giftWrap: ko.observable(true)
        };
        ko.applyBindings(viewModel);
    </script>

Claro, é um monte de código a primeira vista, mas depois que você criou seu binding customizado, eles podem ser re-usados facilmente em diversos lugares.

### O callback "init"

Knockout chamará a sua função `init` uma vez para cada elemento DOM que você usar no binding. Há duas formas de uso para `init`:

 * Para estabelecer qualquer estado inicial para o elemento DOM
 * Para registrar qualquer event handlers para que, por exemplo, quando usuário clicar ou modificar o elemento DOM, você pode mudar o estado do observable associado

KO passará exatamente o mesmo conjunto de parâmetro que ele passa ao [callback de `update`](#the-update-callback).

Continuando o exemplo anterior, você pode querer que `slideVisible` estabeleça que o elemento seja instantaneamente visível ou invisível quando a página aparece pelo primeira vez (sem nenhum slide animado), para que a animação apenas seja executada quando o usuário alterar o estado do model. Você pode fazer isto desta forma:

    ko.bindingHandlers.slideVisible = {
        init: function(element, valueAccessor) {
            var value = ko.unwrap(valueAccessor()); // Get the current value of the current property we're bound to
            $(element).toggle(value); // jQuery will hide/show the element depending on whether "value" or true or false
        },
        update: function(element, valueAccessor, allBindings) {
            // Leave as before
        }
    };

Isto significa que se `giftWrap` foi definido com o estado inicial `false` (em outras palavras, `giftWrap: ko.observable(false)`) então a DIV associado estaria inicialmente escondida, e depois deslizaria na view quando o usuário marcar o checkbox.

### Modificando observables depois dos eventos DOM

Você já viu como usar `update` para, quando um observable ser alterado, você pode atualizar o elemento DOM associado. Mas e os eventos? Quando o usuário realizar alguma ação em um elemento DOM, você pode querer atualizar o observable associado.

Você pode usar o callback `init` como um lugar para registrar um event handler que irá causar alteração ao observable associado. Por exemplo:

    ko.bindingHandlers.hasFocus = {
        init: function(element, valueAccessor) {
            $(element).focus(function() {
                var value = valueAccessor();
                value(true);
            });
            $(element).blur(function() {
                var value = valueAccessor();
                value(false);
            });
        },
        update: function(element, valueAccessor) {
            var value = valueAccessor();
            if (ko.unwrap(value))
                element.focus();
            else
                element.blur();
        }
    };

Now you can both read and write the "focusedness" of an element by binding it to an observable:

    <p>Name: <input data-bind="hasFocus: editingName" /></p>

    <!-- Showing that we can both read and write the focus state -->
    <div data-bind="visible: editingName">You're editing the name</div>
    <button data-bind="enable: !editingName(), click:function() { editingName(true) }">Edit name</button>

    <script type="text/javascript">
        var viewModel = {
            editingName: ko.observable()
        };
        ko.applyBindings(viewModel);
    </script>

### Nota: Suporte a elementos virtuais

Se você quer um binding customizado para usar com a sintaxe de *elementos virtuais* do Knockout, ex.:

    <!-- ko mybinding: somedata --> ... <!-- /ko -->

... então veja [a documentação para elementos virtuais](custom-bindings-for-virtual-elements.html).