---
layout: documentation
title: O binding "attr"
---

### Propósito
O binding `attr` fornece uma maneira genérica para setar o valor de qualquer atributo para o elemento DOM associado. Isso é útil, por exemplo, quando você precisa setar o atributo `title` de um elemento, o `src` de uma tag `img` ou o `href` de um link com base no seu view model, com o valor do atributo sendo atualizado automaticamente sempre que a propriedade correspondente do model for alterado.

### Exemplo
    <a data-bind="attr: { href: url, title: details }">
        Report
    </a>
    
    <script type="text/javascript">
        var viewModel = {
            url: ko.observable("year-end.html"),
            details: ko.observable("Report including final year-end statistics")
        };
    </script>

Isso irá setar o atributo `href` do elemento para `year-end.html` e o atributo `title` para o elemento `Report including final year-end statistics`.

### Parâmetros

  * Parâmetro principal
   
    Você deve passar um objeto JavaScript em que os nomes de propriedade correspondem aos nomes de atributo e os valores correspondem aos valores de atributo que você quer aplicar.
 
    Se o seu parâmetro referencia um valor observable, o binding irá atualizar o atributo sempre que o valor for alterado. Se o parâmetro não referencia um valor observable, ele irá apenas setar o atributo uma vez.
   
  * Parâmetros adicionais

     * Nenhum
   
### Nota: Aplicando atributos cujos nomes não são nomes de variáveis legais de JavaScript

Se você quer aplicar o atributo `data-something`, você não pode escrever isso:

    <div data-bind="attr: { data-something: someValue }">...</div>

... porque `data-something` não é um identificador legal nesse ponto. A solução é simples: apenas envolva o identificador em aspas para que se torne uma string literal, que é legal em um objeto JavaScript. Por exemplo:

    <div data-bind="attr: { 'data-something': someValue }">...</div>

### Nota: Usando palavras reservadas com nomes de atributos em navegadores antigos

Em navegadores antigos (IE8 e anteriores) usar palavras reservadas do JavaScript em nomes de atributos causam um erro. Você pode contornar isso usando aspas como isso:

    <input data-bind="attr: { 'for': someValue }" />

Você pode encontrar uma boa lista de palavras reservadas na página da [Mozilla's MDN page here](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Lexical_grammar#Keywords).

### Dependências

Nenhuma, exceto a biblioteca Knockout.