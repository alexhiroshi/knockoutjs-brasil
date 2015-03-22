---
layout: documentation
title: O binding "submit"
---

### Propósito
O binding `submit` adicona um evento para que uma função JavaScript seja chamada quando o elemento DOM é submetido. Normalmente, você irá querer usar isso apenas em elementos `form`.

Quando você usa o binding `submit` em um formulário, Knockout irá previnir que o navegador envie a ação padrão para esse formulário. Em outas palavras, o navegador irá chamar a sua função mas não irá enviar o formulário para o servidor. Isso é um padrão útil porque quando você usa o `submit`, normalmente é porque você está usando o formulário como uma interface para sua view model, não como um formulário HTML normal. Se você *quiser* deixar o envio do formulário normal, basta retornar `true` na sua função `submit`.

### Exemplo
    <form data-bind="submit: doSomething">
        ... form contents go here ...
        <button type="submit">Submit</button>
    </form>

    <script type="text/javascript">
        var viewModel = {
            doSomething : function(formElement) {
                // ... now do something
            }
        };
    </script>

Como foi ilustrado nesse exemplo, KO passa o elemento do formulário como um parâmetro para a sua função submit. Você pode ignorar esse parâmetro se você quiser, ou existem várias maneiras que você pode usá-lo, por exemplo:

 * Extraindo dados ou estados adicionais de um elemento form

 * Disparando validação em nível de interface usando uma biblioteca como [jQuery Validation](https://github.com/jzaefferer/jquery-validation), usando um código semelhante ao seguinte trecho: `if ($(formElement).valid()) { /* do something */ }`.

### Por que não adicionar um evento `click` no botão submit?

Em vez de usar `submit` no formulário, vocie pode usar o `click` no botão enviar. No entanto, o `submit` tem a vantagem de capturar formas alternativas de envio do formulário, como pressionar a tecla enter enquanto digita em um textbox.

### Parâmetros

  * Parâmetro principal

    A função que você deseja adicionar no evento `submit` do elemento.

    Você pode referenciar qualquer função JavaScript – ele não tem que ser uma função da sua view model. Você pode referenciar uma função em qualquer objeto escrevendo `submit: someObject.someFunction`.

    Funções em sua view model são um pouco especiais porque você pode referenciar pelo nome, ou seja, você pode escrever `submit: doSomething` e não precisa escrever `submit: viewModel.doSomething` (embora tecnicamente isso também é válido).

  * Parâmetros adicionais

     * Nenhum

### Notas

Para informações sobre como passar parâmetros adicionais para a sua função submit, ou como controlar o `this` ao chamar funções que nnao estão em sua view model, veja as notas relacionadas ao [binding click](click-binding.html). Todas as notas nessa página também se aplicam ao manipulador `submit`.

### Dependências

Nenhuma, exceto a biblioteca Knockout.