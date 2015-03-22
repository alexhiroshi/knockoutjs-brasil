---
layout: documentation
title: O binding "text"
---

### Propósito
O binding `text` faz com que o elemento DOM associado exiba o valor do seu parâmetro.

Geralmente isso é útil com elementos como `<span>` ou `<em>` que normalmente exibe texto, mas tecnicamente você pode usar com qualquer elemento.

### Exemplo
    Today's message is: <span data-bind="text: myMessage"></span>

    <script type="text/javascript">
        var viewModel = {
            myMessage: ko.observable() // Initially blank
        };
        viewModel.myMessage("Hello, world!"); // Text appears
    </script>

### Parâmetros

  * Parâmetro principal

    Knockout seta o conteúdo do elemento a um nó de texto com o seu valor de parâmetro. Qualquer conteúdo anterior será sobrescrito.

    Se o seu parâmetro referencia um valor observable, o binding irá atualizar o atributo sempre que o valor for alterado. Se o parâmetro não referencia um valor observable, ele irá apenas setar o atributo uma vez.

    Se você fornecer algo diferente de um número ou uma string (ex.: passar um objeto ou um array), o texto exibido será equivalente a `yourParameter.toString()`

  * Parâmetros adicionais

      * Nenhum

### Nota 1: Usando funções e expressões para setar valores de texto

Se você quiser setar um texto programaticamente, uma opção é criar um [computed observable](computedObservables.html), e use its evaluator function as a place for your code that works out what text to display.

Por exemplo

    The item is <span data-bind="text: priceRating"></span> today.

    <script type="text/javascript">
        var viewModel = {
            price: ko.observable(24.95)
        };
        viewModel.priceRating = ko.pureComputed(function() {
            return this.price() > 50 ? "expensive" : "affordable";
        }, viewModel);
    </script>

Agora, o texto irá alternar entre “expensive” e “affordable” conforme necessário, sempre que houver alterações de `price`.

Opcionalmente, você não precisa criar uma computed observable se você estiver fazendo algo simples como isso. Você pode passar uma expressão JavaScript qualquer para o binding `text`. Por exemplo:

    The item is <span data-bind="text: price() > 50 ? 'expensive' : 'affordable'"></span> today.

Isso é exatamente o mesmo resultado, sem precisar do computed observable `priceRating`.

### Nota 2: Sobre HTML encoding

Desde que esse binding seta o seu valor usando um nó de texto, é seguro setar qualquer valor sem prejudicar o HTML ou injeção de script. Por exemplo, se você escreveu:

    viewModel.myMessage("<i>Hello, world!</i>");

... isso não mostraria o texto em itálico, mostraria um texto literal com os colchetes visíveis.

Se você precisa setar o conteúdo HTML desse jeito, veja [o binding html](html-binding.html).

### Nota 3: Usando “text” sem um elemento container

Às vezes você pode querer setar o texto usando Knockout sem incluir um elemento extra. Por exemplo, você não está autorizado a incluir outros elementos dentro de um elemento, então o seguinte não irá funcionar.

    <select data-bind="foreach: items">
        <option>Item <span data-bind="text: name"></span></option>
    </select>

Para lidar com isso, você pode usar a sintaxe *containerless*, que é baseado em tags de comentário.

    <select data-bind="foreach: items">
        <option>Item <!--ko text: name--><!--/ko--></option>
    </select>

Os comentários `<!--ko-->` e `<!--/ko-->` funcionarão como marcadores de início/fim, definindo um “elemento virtual” que contém a marcação. Knockout entende esta sintaxe de elemento virtual e liga como se você tivesse um conjunto de elementos reais.

### Nota 4: Sobre uma peculiaridade no IE 6

O IE 6 tem uma estranha peculiaridade que, às vezes, ele ignora espaços em branco que segue logo depois de um span vazio. Isso não tem a ver diretamente com o Knockout, mas se você quiser escrever:

    Welcome, <span data-bind="text: userName"></span> to our web site.

... e o IE 6 não criar um espaço em branco antes das palavras `to our web site`, você pode resolver o problema adicionando qualquer caractere dentro do `<span>`, exemplo:

    Welcome, <span data-bind="text: userName">&nbsp;</span> to our web site.

Outros navegadores e versões mais recentes do IE, não tem essa peculiaridade.

### Dependências

Nenhuma, exceto a biblioteca Knockout.