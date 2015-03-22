---
layout: documentation
title: O binding "checked"
---

### Propósito
O binding `checked` liga um controle de formulário checável. &mdash; ex.: um checkbox (`<input type='checkbox'>`) ou um radio button (`<input type='radio'>`) &mdash; com uma propriedade em sua view model.

Quando o usuário marca o controle do formulário associado, este atualiza o valor na sua view model. Da mesma forma, quando você atualiza o valor da sua view model, ele marca ou desmarca o controle do formulário em sua tela.

Nota: Para text boxes, drop-down e todos os controles de formulário que não são checáveis, use [o binding `value`](value-binding.html) para ler e escrever o valor dos elementos, não o binding `checked`.

### Exemplo com checkbox
    <p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>

    <script type="text/javascript">
	    var viewModel = {
			wantsSpam: ko.observable(true) // Initially checked
	    };

	    // ... then later ...
	    viewModel.wantsSpam(false); // The checkbox becomes unchecked
    </script>

### Exemplo adicionando checkbox vinculados a um array
    <p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>
    <div data-bind="visible: wantsSpam">
    	Preferred flavors of spam:
    	<div><input type="checkbox" value="cherry" data-bind="checked: spamFlavors" /> Cherry</div>
    	<div><input type="checkbox" value="almond" data-bind="checked: spamFlavors" /> Almond</div>
    	<div><input type="checkbox" value="msg" data-bind="checked: spamFlavors" /> Monosodium Glutamate</div>
    </div>

    <script type="text/javascript">
	    var viewModel = {
			wantsSpam: ko.observable(true),
			spamFlavors: ko.observableArray(["cherry","almond"]) // Initially checks the Cherry and Almond checkboxes
	    };

	    // ... then later ...
	    viewModel.spamFlavors.push("msg"); // Now additionally checks the Monosodium Glutamate checkbox
    </script>

### Exemplo adicionando radio buttons
    <p>Send me spam: <input type="checkbox" data-bind="checked: wantsSpam" /></p>
    <div data-bind="visible: wantsSpam">
    	Preferred flavor of spam:
    	<div><input type="radio" name="flavorGroup" value="cherry" data-bind="checked: spamFlavor" /> Cherry</div>
    	<div><input type="radio" name="flavorGroup" value="almond" data-bind="checked: spamFlavor" /> Almond</div>
    	<div><input type="radio" name="flavorGroup" value="msg" data-bind="checked: spamFlavor" /> Monosodium Glutamate</div>
    </div>

    <script type="text/javascript">
	    var viewModel = {
			wantsSpam: ko.observable(true),
			spamFlavor: ko.observable("almond") // Initially selects only the Almond radio button
	    };

	    // ... then later ...
	    viewModel.spamFlavor("msg"); // Now only Monosodium Glutamate is checked
    </script>

### Parâmetros

  * Parâmetro principal

    KO define o estado do elemento marcado para corresponder com o seu valor do parâmetro. Qualquer estado marcado anteriormente serão substituídos. A forma como o seu parâmetro é interpretada, depende de que tipo de elemento você está fazendo o binding:

      * Para **checkboxes**, KO irá definir o elemento como *checked* quando o valor do parâmetro for `true`, e *unchecked* quando `false`. Se você der um valor que não é um boolean, ele será interpretado vagamente. Isso significa que números diferentes de zero, objetos não `null` e strings não vazias serão interpretadas como `true`, enquanto zero, `null`, `undefined` e strings vazias serão interpretadas como `false`.

        Quando o usuário marca ou desmarca o checkbox, KO irá definir a propriedade do seu model para `true` ou `false`, respectivamente.

        Uma atenção especial é dada se o seu parâmetro for um `array`. Nesse caso, KO irá definir o elemento para *checked* se o valor corresponde a um item do array e *unchecked* se não existir no array.

        Quando o usuário marca ou desmarca o checkbox, KO irá adicionar ou remover o valor do array. 

      * Para **radio buttons**, KO irá definir o elemento a ser *checado* se, e apenas se, o valor do parâmetro for igual ao atributo `value` do radio button especificado pelo parâmetro `checkedValue`. No exemplo anterior, o radio button com `value="almond"` foi checado apenas quando a propriedade da view model `spamFlavor` era igual a `"almond"`.

        When the user changes which radio button is selected, KO will set your model property to equal the value of the selected radio button. In the preceding example, clicking on the radio button with `value="cherry"` would set `viewModel.spamFlavor` to be `"cherry"`.

        Of course, this is most useful when you have multiple radio button elements bound to a single model property. To ensure that only *one* of those radio buttons can be checked at any one time, you should set all of their `name` attributes to an arbitrary common value (e.g., the value `flavorGroup` in the preceding example) - doing this puts them into a group where only one can be selected.

     Se o seu parâmetro é um valor observable, o binding atualizará o valor do elemento selecionado sempre que o valor for trocado. Se o parâmetro não for observable, ele irá definir o elemento selecionado apenas uma vez e não atualizará depois.

  * Parâmetros adicionais

      * <code id="checkedValue">checkedValue</code>

        Se o seu binding também inclui `checkedValue`, isso define o valor usado pelo binding `checked` em vez do atributo de elemento `value`. Isso é útil se você quiser que o valor seja alguma coisa diferente de uma string (como um integer ou objeto), ou quiser que o valor seja definido dinamicamente.

        No seguinte exemplo, o próprio item de objetos (não as string `itemName`) serão incluidos no array `chosenItems` quando o checkebox correspondente for marcado.

            <!-- ko foreach: items -->
                <input type="checkbox" data-bind="checkedValue: $data, checked: $root.chosenItems" />
                <span data-bind="text: itemName"></span>
            <!-- /ko -->

            <script type="text/javascript">
                var viewModel = {
                    items: ko.observableArray([
                        { itemName: 'Choice 1' },
                        { itemName: 'Choice 2' }
                    ]),
                    chosenItems: ko.observableArray()
                };
            </script>

        Se o seu parâmetro `checkedValue` é um valor observable, sempre que o valor for trocado e o elemento está marcado, o binding irá atualizar a propriedade checked do model. Para `checkboxes`, ele irá remover o valor antigo do array e adicionará o novo valor. Para radio buttons, ele irá somente atualizar o valor do model.

### Dependências

Nenhuma, exceto a biblioteca Knockout.