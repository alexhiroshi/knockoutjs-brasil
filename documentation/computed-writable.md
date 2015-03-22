---
layout: documentation
title: Writable computed observables
---
*Iniciantes podem pular essa seção - observables writable computed são bem avançados e não são necessários na maioria da situações*

Normalente, observables computed tem um valor que é computado de outros observables e são, portanto, *read-only*. O que pode parecer surpreendente, é que é possível fazer os computed observables *writable*. Você só precisa fornecer sua própria função calback que faz alguma coisa com os valores escritos.

Você pode usar um writable computed observable exatamente como um observable normal, com sua própria lógica customizada intercepitando todas as leituras e escritas. Assim como observables, você pode escrever valores para múltiplas propriedade de um observable or computed observable em um objeto model usando *chaining syntax*. Por exemplo, `myViewModel.fullName('Joe Smith').age(50)`.

Computed observables editáveis são características poderosas com uma ampla possibilidade de usos.

### Example 1: Decompondo entrada do usuário

Voltado ao clássico exemplo "primeiro nome + último nome = nome completo", voc6e pode tornas as coisas de trás para frente: faça o computed observable `fullName` editável, para que o usuário possa editar diretamente o nome completo, e o valor fornecido pelo mesmo seja passado e mapeado de volta aos observables `firstName` e `lastName`. Nesse exemplo, o callback `write`controla os valores de entrada separando o texto em componentes "firstName" e "lastName", e escrevendo esses valores de volta ao observables.


{% capture live_example_id %}decompose-input{% endcapture %}
{% capture live_example_viewmodel %}
    function MyViewModel() {
        this.firstName = ko.observable('Planet');
        this.lastName = ko.observable('Earth');

        this.fullName = ko.pureComputed({
            read: function () {
                return this.firstName() + " " + this.lastName();
            },
            write: function (value) {
                var lastSpacePos = value.lastIndexOf(" ");
                if (lastSpacePos > 0) { // Ignore values with no space character
                    this.firstName(value.substring(0, lastSpacePos)); // Update "firstName"
                    this.lastName(value.substring(lastSpacePos + 1)); // Update "lastName"
                }
            },
            owner: this
        });
    }

    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% capture live_example_view %}
    <div>First name: <span data-bind="text: firstName"></span></div>
    <div>Last name: <span data-bind="text: lastName"></span></div>
    <div class="heading">Hello, <input data-bind="textInput: fullName"/></div>
{% endcapture %}
{% include live-example-minimal.html %}

Isso é exatamente o contrário do exemplo [Hello World](http://knockoutjs.com/examples/helloWorld.html), nele os primeiros e últimos nomes não são editáveis, mas o nome completo é editável.

O código do view model a seguir demonstra o *single parameter syntax* para inicializar os computed observables. Veja a [ referência computed observable](computed-reference.html) para a lista completa de opções disponíveis.

### Example 2: Selecionando/desmarcando todos os itens

Quando apresentando ao usuário uma lista de itens selecionáveis, é útil incluir um método para selecionar ou desmarcar todos os itens. Isso pode ser representado bem intuitivamente com um valore booleano que representa se todos os itens estão selecionados. Quando o valor for `true` todos os itens estarão selecionados, e quando o valor for `false` os itens estarão desmarcados.

<style type="text/css">
    #select-all-items label { display: block; }
    #select-all-items .heading { border-bottom: 1px solid black; }
</style>

{% capture live_example_id %}select-all-items{% endcapture %}
{% capture live_example_viewmodel %}
    function MyViewModel() {
        this.produce = [ 'Apple', 'Banana', 'Celery', 'Corn', 'Orange', 'Spinach' ];
        this.selectedProduce = ko.observableArray([ 'Corn', 'Orange' ]);
        this.selectedAllProduce = ko.pureComputed({
            read: function () {
                // Comparing length is quick and is accurate if only items from the
                // main array are added to the selected array.
                return this.selectedProduce().length === this.produce.length;
            },
            write: function (value) {
                this.selectedProduce(value ? this.produce.slice(0) : []);
            },
            owner: this
        });
    }
    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% capture live_example_view %}
    <div class="heading">
        <input type="checkbox" data-bind="checked: selectedAllProduce" title="Select all/none"/> Produce
    </div>
    <div data-bind="foreach: produce">
        <label>
            <input type="checkbox" data-bind="checkedValue: $data, checked: $parent.selectedProduce"/>
            <span data-bind="text: $data"></span>
        </label>
    </div>
{% endcapture %}
{% include live-example-minimal.html %}

### Example 3: Um conversor de valor

Sometimes you might want to represent a data point on the screen in a different format than its underlying storage. For example, you might want to store a price as a raw float value, but let the user edit it with a currency symbol and fixed number of decimal places. You can use a writable computed observable to represent the formatted price, mapping incoming values back to the underlying float value:

{% capture live_example_id %}value-converter{% endcapture %}
{% capture live_example_viewmodel %}
    function MyViewModel() {
        this.price = ko.observable(25.99);

        this.formattedPrice = ko.pureComputed({
            read: function () {
                return '$' + this.price().toFixed(2);
            },
            write: function (value) {
                // Strip out unwanted characters, parse as float, then write the 
                // raw data back to the underlying "price" observable
                value = parseFloat(value.replace(/[^\.\d]/g, ""));
                this.price(isNaN(value) ? 0 : value); // Write to underlying storage
            },
            owner: this
        });
    }

    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% capture live_example_view %}
    <div>Enter bid price: <input data-bind="textInput: formattedPrice"/></div>
    <div>(Raw value: <span data-bind="text: price"></span>)</div>
{% endcapture %}
{% include live-example-minimal.html %}

Agora, sempre que o usuário inserir um novo preço, a caixa de texto imediatamente atualizará para mostra-lo formatado com o símbolo da moeda e duas casas decimais, não importando qual formato eles insiram. Isso fornece uma ótima experiência de usuário, porque o usuário vê como o software entende a entrada de dado como um preço. Eles sabem que não podem entrar com mais de duas casas decimais, porque se eles tentarem, as casas decimais adicionais serão removidas imediatamente. Similarmente, eles não podem entrar com valores negativos, porque o callback `write` remove qualquer sinal de menos.

### Exemplo 4: Filtrando e validando entrada do usuário

Exemplo 1 mostrou como um computed observable editável pode efetivamente *filtrar* seus dados de entrada escolhendo não escrever certos valores de volta aos observables se eles não atenderem alguns critérios. Ele ignorou valores de nome completo que não tinha um espaço.

Levando isso ao um passo a frente, você podria também ativar um flag `isValid` dependendo se a entrada mais recente é satisfatória, e exibir a mensagem na UI conformemente. Há um jeito mais fácil de fazer a validação (explicado abaixo), mas primeiro considere o seguinte exemplo, no qual demonstra o mecanismo:

<style type="text/css">
    #validate-input .error { color: #A71500; font-weight: bold;  }
</style>

{% capture live_example_id %}validate-input{% endcapture %}
{% capture live_example_viewmodel %}
    function MyViewModel() {
        this.acceptedNumericValue = ko.observable(123);
        this.lastInputWasValid = ko.observable(true);

        this.attemptedValue = ko.pureComputed({
            read: this.acceptedNumericValue,
            write: function (value) {
                if (isNaN(value))
                    this.lastInputWasValid(false);
                else {
                    this.lastInputWasValid(true);
                    this.acceptedNumericValue(value); // Write to underlying storage
                }
            },
            owner: this
        });
    }

    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% capture live_example_view %}
    <div>Enter a numeric value: <input data-bind="textInput: attemptedValue"/></div>
    <div class="error" data-bind="visible: !lastInputWasValid()">That's not a number!</div>
    <div>(Accepted value: <span data-bind="text: acceptedNumericValue"></span>)</div>
{% endcapture %}
{% include live-example-minimal.html %}

Agora, `acceptedNumericValue` irá sempre conter apenas valores numéricos, e qualquer outro valor inserido irá ativar a mensagem de validação em vez de atualizar o `acceptedNumericValue`.

**Nota:** Para tal requisito como validar se uma entrada é numérica, esse técnica é um exagero. Seria mais bem mais fácil apenas usar validação por jQuery e sua classe `number` no elemento `<input>`. Validação por Knockout e jQuery trabalham bem juntos, como demonstrado no exemplo [grid editor](http://knockoutjs.com/examples/gridEditor.html). No entanto, o exemplo atenrior demonstra um mecanimos geral para filtrar e validar com uma lógica customizada para controlar qual tipo de feedback aparecerá, o que pode ser útil se seu cenário é mais complexo que uma validação por jQuery poderia controlar nativamente.