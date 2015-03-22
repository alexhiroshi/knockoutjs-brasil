---
layout: documentation
title: O binding "if"
---

### Propósito
O binding `if` faz com que uma seção da marcação apareça em seu documento (e tenha o atributo `data-bind` aplicada) somente se uma expressão especificada for `true` (ou um valor equivalente a true, como um objeto não nulo ou uma string não vazia).

`if` desempenha um papel semelhante do [binding `visible`](visible-binding.html). TA diferença é que, com `visible`, a marcação permanece no DOM e o atributo `data-bind` é sempre aplicado – o binding `visible` usa apenas CSS para alternar a visibilidade do elemento. O binding `if`, no entanto, fisicamente adiciona ou remove a marcação contida no DOM e só se aplica para bindings descendentes se a expressão for `true`.

### Exemplo 1

Esse exemplo mostra que o binding `if` pode adicionar e remover dinamicamente seções de marcação conforme o valor observable é trocado.

{% capture live_example_view %}
<label><input type="checkbox" data-bind="checked: displayMessage" /> Display message</label>

<div data-bind="if: displayMessage">Here is a message. Astonishing.</div>
{% endcapture %}

{% capture live_example_viewmodel %}
ko.applyBindings({
    displayMessage: ko.observable(false)
});
{% endcapture %}

{% include live-example-minimal.html %}

### Exemplo 2

No seguinte exemplo, o elemento `<div>` estará vazio para "Mercury", mas preenchida para "Earth". Isso porque Earth tem uma propriedade não nula para `capital`, enquanto "Mercury" tem a propriedade como `null`.

    <ul data-bind="foreach: planets">
        <li>
            Planet: <b data-bind="text: name"> </b>
            <div data-bind="if: capital">
                Capital: <b data-bind="text: capital.cityName"> </b>
            </div>
        </li>
    </ul>


    <script>
        ko.applyBindings({
            planets: [
                { name: 'Mercury', capital: null }, 
                { name: 'Earth', capital: { cityName: 'Barnsley' } }        
            ]
        });
    </script>

It's important to understand that the `if` binding really is vital to make this code work properly. Without it, there would be an error when trying to evaluate `capital.cityName` in the context of "Mercury" where `capital` is `null`. In JavaScript, you're not allowed to evaluate subproperties of `null` or `undefined` values.

### Parâmetros

  * Parâmetro principal
 
    A expressão que você deseja avaliar. Se for `true` (ou valores que representem true), a marcação estará presente no documento, e qualquer atributo `data-bind` que estiver sobre ele será aplicada. Se sua expressão for `false`, a marcação será removida do seu documento sem antes aplicar qualquer binding.

    Se a sua expressão envolve quaisquer valores observable, a expressão será reavaliada sempre que algum deles mudar. Do mesmo modo, a marcação dentro do seu bloco `if` pode ser adicionada ou removida dinamicamente como resultado das mudanças da expressão. Atributos `data-bind` serão aplicadas a **uma nova cópia da marcação** sempre que ele for adicionado novamente.
     
  * Parâmetros adicionais

     * Nenhum

### Nota: Usando “if” sem um elemento container

Em alguns casos, você pode querer controlar para que uma seção apareça ou não sem precisar que qualquer elemento contenha um binding `if`. Por exemplo, você pode querer controlar um determinado elemento `<li>` estando ao lado de um outro elemento que deve sempre aparecer:

    <ul>
        <li>This item always appears</li>
        <li>I want to make this item present/absent dynamically</li>
    </ul>

Neste caso, você não pode colocar if no `<ul>` (porque isso afetaria a primeira `<li>` também), e você não pode colocar qualquer outra tag em torno da segunda `<li>` (porque o HTML não permitir outras tags dentro de `<ul>`).

Para lidar com isso, você pode usar a sintaxe de fluxo de controle containerless, que é baseado em tags de comentários. Por exemplo:

    <ul>
        <li>This item always appears</li>
        <!-- ko if: someExpressionGoesHere -->
            <li>I want to make this item present/absent dynamically</li>
        <!-- /ko -->
    </ul>

Os comentários `<!-- ko -->` e `<!-- /ko -->` funcionarão como marcadores de início/fim, definindo um “elemento virtual” que contém a marcação. Knockout entende esta sintaxe de elemento virtual e liga como se você tivesse um conjunto de elementos reais.

### Dependências

Nenhuma, exceto a biblioteca Knockout.