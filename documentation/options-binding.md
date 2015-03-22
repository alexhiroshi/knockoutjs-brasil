---
layout: documentation
title: O binding "options"
---

### Propósito
O binding `options` controla que opções devem aparecer em um drop-down list (ou seja, em um elemento `<select>`) ou em um multi-select (ex. `<select size='6'>`). Esse binding não pode ser usado com qualquer outro elemento que não seja um elemento `<select>`.

O valor que você atribui deve ser um array (ou array observable). O elemento `<select>` irá exibir um item para cada item do seu array.

Nota: Para um multi-select, para definir ou ler qual das opções estão selecionadas, use [o binding `selectedOptions`](selectedOptions-binding.html). Para um simples select, você também pode ler e escrever a opção selecionada usando [o binding `value`](value-binding.html).

### Exemplo 1: Lista drop-down
    <p>
        Destination country:
        <select data-bind="options: availableCountries"></select>
    </p>

    <script type="text/javascript">
        var viewModel = {
            // These are the initial options
            availableCountries: ko.observableArray(['France', 'Germany', 'Spain'])
        };

        // ... then later ...
        viewModel.availableCountries.push('China'); // Adds another option
    </script>

### Exemplo 2: Lista multi-select
    <p>
        Choose some countries you would like to visit:
        <select data-bind="options: availableCountries" size="5" multiple="true"></select>
    </p>

    <script type="text/javascript">
        var viewModel = {
            availableCountries: ko.observableArray(['France', 'Germany', 'Spain'])
        };
    </script>

### Exemplo 3: Lista drop-down representando objetos JavaScripts, não apenas strings
    <p>
        Your country:
        <select data-bind="options: availableCountries,
                           optionsText: 'countryName',
                           value: selectedCountry,
                           optionsCaption: 'Choose...'"></select>
    </p>

    <div data-bind="visible: selectedCountry"> <!-- Appears when you select something -->
        You have chosen a country with population
        <span data-bind="text: selectedCountry() ? selectedCountry().countryPopulation : 'unknown'"></span>.
    </div>

    <script type="text/javascript">
        // Constructor for an object with two properties
        var Country = function(name, population) {
            this.countryName = name;
            this.countryPopulation = population;
        };

        var viewModel = {
            availableCountries : ko.observableArray([
                new Country("UK", 65000000),
                new Country("USA", 320000000),
                new Country("Sweden", 29000000)
            ]),
            selectedCountry : ko.observable() // Nothing selected by default
        };
    </script>

### Exemplo 4: Lista drop-down representando objetos JavaScript, com o texto exibido calculado como uma função do item representado

    <!-- Same as example 3, except the <select> box expressed as follows: -->
    <select data-bind="options: availableCountries,
                       optionsText: function(item) {
                           return item.countryName + ' (pop: ' + item.countryPopulation + ')'
                       },
                       value: selectedCountry,
                       optionsCaption: 'Choose...'"></select>

Note que a única diferença entre o exemplo 3 e 4 é o valor de `optionsText`.

### Parâmetros

  * Parâmetro principal

    Você deve fornecer um array (ou um observable array). Para cada item, KO irá adicionar um `<option>` para o nó `<select>` associado. Todas as options anteriores serão removidas.

    Se o valor do seu parâmetro é um array de strings, você não precisa fornecer quaisquer outros parâmetros. O elemento `<select>` exibirá um option para cada valor de string. No entando, se você quiser deixar o usuário escolher de um array de *objetos JavaScript* (não simplesmente strings), veja os parâmetro `optionsText` e `optionsValue` abaixo.

    Se esse parâmetro é um valor observable, o binding irá atualizar os options disponíveis do elemento sempre que o valor trocar. Se o parâmetro não é observable, ele irá definir os options disponíveis apenas uma vez.

  * Parâmetros adicionais

      * `optionsCaption`

        Às vezes, você pode não querer selecionar qualquer option particular por padrão. Mas uma lista single-select drop-down geralmente começa com algum item selecionado, assim, como você pode evitar a pré-seleção? A solução comum é adicionar uma option na lista com uma option falsa que apenas mostra "Selecione um item", "Por favor, selecione uma opção" ou similar e ter ele selecionado por padrão.

        Isso é fácil de fazer: apenas adicione um parâmetro adicional com o nome `optionsCaption`, com o valor de string para exibir. Por exemplo:

        `<select data-bind='options: myOptions, optionsCaption: "Select an item...", value: myChosenValue'></select>`

        KO irá adicionar o item no começo da lista com o texto "Select an item..." e com o value `undefined`. Assim, se `myChosenValue` pegar o value `undefined` (que observables fazem por padrão), então a opção falsa será selecionada. Se o parâmetro `optionsCaption` é um observable, então o texto inicial será atualizada com o valor alterado do observable.

      * `optionsText`

        Veja o exemplo 3 acima para ver como você pode ligar `options` com um array de objetos JavaScript - não apenas strings. Nesse caso, você precisa escolher qual das propriedades dos objetos devem ser exibidas como o texto na lista drop-down ou multi-select. O exemplo 3 mostra como você pode especificar o nome da propriedade por meio de um parâmetro adicional chamado `optionsText`.

        Se você não quiser mostrar apenas um valor de propriedade simples como o texto para cada item no dropdown, você pode passar uma função JavaScript para a opção `optionsText` e fornecer a sua própria lógica para mostrar o texto em termos de objeto representado. Veja o exemplo 4 acima, que mostra como você pode gerar o texto exibido pela concatenação junto com vários valores de propriedades.

      * `optionsValue`

        Similar ao `optionsValue`, você também pode passar um parâmetro adicional chamado `optionsValue` para especificar que as propriedades do objeto devem ser usadas para definir o atributo `value` no elemento `<option>` que o KO gera. Você também pode especificar uma função JavaScript para determinar esse valor. Essa função irá receber o item selecionado como seu único argumento e deve retornar uma string para usar no atributo value do elemento `<option>`.

        Typically you'd only want to use `optionsValue` as a way of ensuring that KO can correctly retain selection when you update the set of available options. For example, if you're repeatedly getting a list of "car" objects via Ajax calls and want to ensure that the selected car is preserved, you might need to set `optionsValue` to `"carId"` or whatever unique identifier each "car" object has, otherwise KO won't necessarily know which of the previous "car" objects corresponds to which of the new ones.

      * `optionsIncludeDestroyed`

        Sometimes you may want to mark an array entry as deleted, but without actually losing record of its existence. This is known as a non-destructive delete. For details of how to do this, see [the destroy function on `observableArray`](observableArrays.html#destroy-and-destroyall).

        By default, the options binding will skip over (i.e., hide) any array entries that are marked as destroyed. If you want to show destroyed entries, then specify this additional parameter like:

        `<select data-bind='options: myOptions, optionsIncludeDestroyed: true'></select>`

      * `optionsAfterRender`

        Se você precisa executar mais alguma lógica personalizada nos elementos `option` gerados, você pode usar o callback `optionsAfterRender`. Veja a nota 2 abaixo.

      * `selectedOptions`

        Para uma lista multi-select, você pode ler e escrever o estado selecionado usando `selectedOptions`. Tecnicamente isso é um binding separado, por isso tem [sua própria documentação](selectedOptions-binding.html).

      * `valueAllowUnset`

        If you want Knockout to allow your model property to take values that have no corresponding entry in your `<select>` element (and display this by making the `<select>` element blank), then see [documentation for `valueAllowUnset`](value-binding.html#using-valueallowunset-with-select-elements).

### Nota 1: A seleção é preservada quando os options são criados/alterados

Quando o binding `options` altera os options do seu elemento `<select>`, KO irá deixar a seleção do usuário inalterada sempre que possível. Assim, para uma lista drop-down de seleção única, o valor selecionado previamente será mantido selecionado e para uma lista de seleção múltipla, todos as opções selecionadas previamente ainda estarão selecionadas (a não ser, é claro, que você remova um ou mais dessas opções).

That's because the `options` binding tries to be independent of the `value` binding (which controls selection for a single-select list) and the `selectedOptions` binding (which controls selection for a multi-select list).

### Nota 2: Pós-processamento dos options gerados

Se você precisa executar alguma lógica customizada sobre os elementos `option` gerado, você pode usar o callback `optionsAfterRender`. A função callback é invocada cada vez que um elemento `option` é inserido na lista, com os seguintes parâmetros:

1. O elemento `option` inserido
2. O item de dados em relação ao elemento ele está ligado, ou `undefined` para o elemento caption

Aqui está um exemplo que usa `optionsAfterRender` para adicionar um binding `disable` em cada option.

    <select size=3 data-bind="
        options: myItems,
        optionsText: 'name',
        optionsValue: 'id',
        optionsAfterRender: setOptionDisable">
    </select>

    <script type="text/javascript">
        var vm = {
            myItems: [
                { name: 'Item 1', id: 1, disable: ko.observable(false)},
                { name: 'Item 3', id: 3, disable: ko.observable(true)},
                { name: 'Item 4', id: 4, disable: ko.observable(false)}
            ],
            setOptionDisable: function(option, item) {
                ko.applyBindingsToNode(option, {disable: item.disable}, item);
            }
        };
        ko.applyBindings(vm);
    </script>

### Dependências

Nenhuma, exceto a biblioteca Knockout.