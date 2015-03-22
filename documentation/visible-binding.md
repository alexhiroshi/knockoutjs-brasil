---
layout: documentation
title: O binding "visible"
---

### Propósito
O binding `visible` faz com que o elemento DOM associado fique oculto ou visível dependendo do valor que você informar.

### Exemplo
    <div data-bind="visible: shouldShowMessage">
	    You will see this message only when "shouldShowMessage" holds a true value.
    </div>

    <script type="text/javascript">
	    var viewModel = {
			shouldShowMessage: ko.observable(true) // Message initially visible
	    };
	    viewModel.shouldShowMessage(false); // ... now it's hidden
	    viewModel.shouldShowMessage(true); // ... now it's visible again
    </script>

### Parâmetros

  * Parâmetro principal

      * Quando o parâmetro é um valor do **tipo false** (ex. o valor boolean `false`, ou o valor numérico `0`, ou `null`, ou `undefined`), o binding define `none` para `yourElement.style.display`, fazendo com que o elemento fique oculto. Este tem prioridade sobre qualquer estilo display que você definiu utilizando CSS.

      * Quando o parâmetro é um valor do **tipo true** (ex. o valor boolean `true`, ou o um objeto não `null` ou um array), o binding remove o valor de `yourElement.style.display`, fazendo com que o elemento fique visível.

        Note que qualquer estilo display que você configurou usando CSS será aplicada (assim, regras de CSS como `display:table-row` vão funcionar bem em conjunto com esse binding).

    Se esse parâmetro é um valor observable, o binding irá atualizar a visibilidade do elemento sempre que o valor mudar. Se o parâmetro não é observable, ele irá definir a visibilidade do elemento apenas uma vez sem alterá-lo depois.

  * Parâmetros adicionais

      * Nenhum

### Nota: Usando funções e expressões para controlar a visibilidade do elemento

Você também pode usar uma função JavaScript ou um expressão JavaScript qualquer como valor do parâmetro. Se você usar, o KO irá executar a sua função ou sua expressão e usar o resultado para determinar se deve ocultar o elemento.

Exemplo:

    <div data-bind="visible: myValues().length > 0">
	    You will see this message only when 'myValues' has at least one member.
    </div>

    <script type="text/javascript">
	    var viewModel = {
			myValues: ko.observableArray([]) // Initially empty, so message hidden
	    };
	    viewModel.myValues.push("some value"); // Now visible
    </script>

### Dependências

Nenhuma, exceto a biblioteca Knockout.