---
layout: documentation
title: O binding "value"
---

### Propósito
O binding `value` vincula o valor do elemento DOM a uma propriedade do seu view model. Isso é normalmente usado com elementos de formulários como `<input>`, `<select>` e `<textarea>`.

Quando o usuário edita o value no controle do formulário associado, ele atualiza o valor do seu view model. Desta forma, quando você atualiza o valor em seu view model, o value no controle do formulário é atualizado na tela.

Nota: Se você estiver trabalhando com checkboxes ou radio buttons, use [o binding checked](checked-binding.html) para ler e escrever o estado checked dos seus elementos, não o binding `value`.

### Exemplo
    <p>Login name: <input data-bind="value: userName" /></p>
    <p>Password: <input type="password" data-bind="value: userPassword" /></p>

    <script type="text/javascript">
        var viewModel = {
            userName: ko.observable(""),        // Initially blank
            userPassword: ko.observable("abc"), // Prepopulate
        };
    </script>

### Parâmetros

  * Parâmetro principal

    KO define a propriedade `value` do elemento para o seu valor de parâmetro. Qualquer valor anterior será sobrescrito.

    Se o seu parâmetro referencia um valor observable, o binding irá atualizar o atributo sempre que o valor for alterado. Se o parâmetro não referencia um valor observable, ele irá apenas setar o atributo uma vez.

    Se você fornecer algo diferente de um número ou uma string (ex.: passar um objeto ou um array), o texto exibido será equivalente a `yourParameter.toString()` (que normalmente não é muito útil, por isso é melhor usar string ou valores numéricos).

    Sempre que o usuário edita o valor no controle associado, KO irá atualiar a propriedade em sua view model. KO sempre tentará atualizar a sua view model quando o valor for modificado e um usuário transferir o foco para outro nó do DOM (ou seja, no evento `change`), mas você pode desencadear atualizações baseado em outros eventos usando o parâmetro `valueUpdate` descrito abaixo.

  * Parâmetros adicionais

      * `valueUpdate`

        Se o seu binding também inclui um parâmetro chamado `valueUpdate`, isso definirá eventos adicionais no navegador que o KO deve usar para detectar mudanças além do evento `change`. Os seguintes valores de string são, geralmente, as escolhas mais úteis: 

          * `"input"` - atualiza a view model quando o valor de um elemento `<input>` ou `<textarea>` é alterado. Note que esse evento gerado apenas por navegadores razoavelmente modernos (ex.: IE 9+).
          * `"keyup"` - atualiza a view model quando o usuário solta a tecla
          * `"keypress"` - atualiza a view model quando o usuário digitou um caractere. Diferente de `keyup`, ele atualiza repetidamente enquando o usuário mantém a tecla pressionada
          * `"afterkeydown"` - atualiza a view model assim que o usuário começa a digitar um caractere. Isso funciona ao pegar o evento `keydown` do navegador e manipular o evento de forma assíncrona. Isso nnao funciona em alguns navegadores mobile.

      * `valueAllowUnset`

        Veja [Nota 2](#using-valueallowunset-with-select-elements) abaixo. Note that `valueAllowUnset` is only applicable when using `value` to control selection on a `<select>` element. On other elements it has no effect.

### Nota 1: Obtendo atualizações de valores instantaneamente a partir de inputs

If you are trying to bind an `<input type="text" />` or `<textarea>` to get instant updates to your viewmodel, use the [the `textInput` binding](textinput-binding.html). It has better support for browser edge cases than any combination of `valueUpdate` options.


### Nota 2: Trabalhando com drop-down (elementos `<select>`)

Knockout tem suporte especial para drop-down (elementos `<select>`). O binding `value` funciona em conjunto com o binding `options` para deixá-lo ler e escrever valores que são objectos JavaScript, não somente valores string. Isso é muito útil se você quiser deixar o usuário selecionar um conjunto de objetos do model. Como exemplo, veja [o binding `options`](options-binding.html) ou para manuseamento de multi-select, veja a documentação do [binding `selectedOptions`](selectedOptions-binding.html).

Você também pode usar o binding `value` com um elemento `<select>` que não usa o binding `options`. Nesse caso, você pode escolher especificar seu elemento `<option>` na marcação ou contruí-lo usando o binding `foreach` ou `template`. Você pode até mesmo aninhar options dentro de elementos `<optgroup>` e o Knockout irá definir o valor selecionado de forma adequada.

#### Usando `valueAllowUnset` com elementos `<select>`

Normalmente, quando você usa o binding `value` com um elemento `<select>`, isso significa que você deseja associar o valor do model para descrever qual item no `<select>` é selecionado. Mas o que acontece se você definir o valor do model para algo que não existe na lista? O comportamento padrão do Knockout é substituir o valor do model para redefini-lo para o que já está selecionado no dropdown, impedindo que o model e a interface fiquem fora de sincornia.

No entando, as vezes você pode não querer esse comportamento. Se você quer que o Knockout permita o seu model observable assumir valores que não existam no `<select>`, então especifique `valueAllowUnset: true`. Nesse caso, sempre que o seu valor não pode ser representado no `<select>`, então o `<select>` simplesmente não deixa um valor selecionado, que é representado visualmente por ele ser branco. Quando o usuário seleciona mais tarde uma opção no dropdown, este será gravado em seu model como de costume. Por exemplo:

    <p>
        Select a country:
        <select data-bind="options: countries,
                           optionsCaption: 'Choose one...',
                           value: selectedCountry,
                           valueAllowUnset: true"></select>
    </p>

    <script type="text/javascript">
        var viewModel = {
            countries: ['Japan', 'Bolivia', 'New Zealand'],
            selectedCountry: ko.observable('Latvia')
        };
    </script>

No exemplo acima, `selectedCountry` reterá o valor `'Latvia'` e o dropdown ficará branco, porque não há nenhuma opção correspondente.

If `valueAllowUnset` had not been enabled, then Knockout would have overwritten `selectedCountry` with `undefined`, so that it would match the value of the `'Choose one...'` caption entry.

### Nota 3: Atualizando valores de propriedades observáveis e não observáveis

If you use `value` to link a form element to an observable property, KO is able to set up a 2-way binding so that changes to either affect the other.

However, if you use `value` to link a form element to a *non*-observable property (e.g., a plain old string, or an arbitrary JavaScript expression), KO will do the following:

  * If you reference a *simple property*, i.e., it is just a regular property on your view model, KO will set the form element's initial state to the property value, and when the form element is edited, KO will write the changes back to your property. It cannot detect when the property changes (because it isn't observable), so this is only a 1-way binding.

  * If you reference something that is *not* a simple property, e.g., the result of a function call or comparison operation, KO will set the form element's initial state to that value, but it will not be able to write any changes back when the user edits the form element. In this case it's a one-time-only value setter, not an ongoing binding that reacts to changes.

Exemplo:

    <!-- Two-way binding. Populates textbox; syncs both ways. -->
    <p>First value: <input data-bind="value: firstValue" /></p>

    <!-- One-way binding. Populates textbox; syncs only from textbox to model. -->
    <p>Second value: <input data-bind="value: secondValue" /></p>

    <!-- No binding. Populates textbox, but doesn't react to any changes. -->
    <p>Third value: <input data-bind="value: secondValue.length > 8" /></p>

    <script type="text/javascript">
        var viewModel = {
            firstValue: ko.observable("hello"), // Observable
            secondValue: "hello, again"         // Not observable
        };
    </script>

### Nota 4: Usando o binding `value` com o binding `checked`

The [`checked`](checked-binding.html) binding should be used to bind a view model property against the value of a checkbox (`<input type='checkbox'>`) or radio button (`<input type='radio'>`). If you do include the `value` binding with the `checked` binding on one of these elements, then the `value` binding will simply act like the [`checkedValue`](checked-binding.html#checkedValue) option that can be used with the `checked` binding and will control the value that is used for updating your view model.

### Dependências

Nenhuma, exceto a biblioteca Knockout.