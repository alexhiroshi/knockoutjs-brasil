---
layout: documentation
title: O binding "html"
---

### Propósito
O binding `html` faz com que o elemento DOM associado exiba o HTML especificado do seu parâmetro.

Geralmente isso é útil quando os valores em sua view model são marcações HTML que você deseja exibir.

### Exemplo
    <div data-bind="html: details"></div>

    <script type="text/javascript">
	    var viewModel = {
			details: ko.observable() // Initially blank
	    };
	    viewModel.details("<em>For further details, view the report <a href='report.html'>here</a>.</em>"); // HTML content appears
    </script>

### Parâmetros

  * Parâmetro principal

    KO limpa o conteúdo anterior e então define o conteúdo do elemento para o seu valor do parâmetro usando a função `html` do jQuery, or by parsing the string into HTML nodes and appending each node as a child of the element, if jQuery is not available.

    Se o seu parâmetro referencia um valor observable, o binding irá atualizar o atributo sempre que o valor for alterado. Se o parâmetro não referencia um valor observable, ele irá apenas setar o atributo uma vez.

    Se você fornecer algo diferente de um número ou uma string (ex., você passar um objecto ou um array), o `innerHTML` será equivalente a `yourParameter.toString()`

  * Parâmetros adicionais

      * Nenhum

### Nota: Sobre HTML encoding

Como esse binding define o conteúdo do seu elemento usando `innerHTML`, você deve ser cuidadoso para não usar isso com valores do model não confiáveis, porque isso pode abrir a possibilidade de ataques de injeção de script. Se você não pode garantir que o conteúdo é seguro para mostrar (por exemplo, se for com base na entrada de diferentes usuários que estavam armazenados em seu banco de dados), então você pode usar [o binding text](text-binding.html), que irá setar o texto do elemento usando `innerText` ou `textContent`.

### Dependências

Nenhuma, exceto a biblioteca Knockout.
