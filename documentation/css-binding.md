---
layout: documentation
title: O binding "css"
---

### Propósito
O binding `css` adiciona ou remove um ou mais classes CSS associado no elemento DOM. Isso é útil, por exemplo, para destacar algum valor em vermelho se for negativo.

(Nota: Se você não quer aplicar uma classe CSS, mas quer atribuir um valor para o atributo `style`  diretamente, veja [o binding style](style-binding.html).)

### Exemplo com classes estáticas
    <div data-bind="css: { profitWarning: currentProfit() < 0 }">
       Profit Information
    </div>
    
    <script type="text/javascript">
        var viewModel = {
            currentProfit: ko.observable(150000) // Positive value, so initially we don't apply the "profitWarning" class
        };
        viewModel.currentProfit(-50); // Causes the "profitWarning" class to be applied
    </script>

Isso irá aplicar a classe CSS `profitWarning` sempre que o valor de `currentProfit` for menor que zero e vai remover a classe sempre que o valor for maior que zero.

### Exemplo com classes dinâmicas
    <div data-bind="css: profitStatus">
       Profit Information
    </div>

    <script type="text/javascript">
        var viewModel = {
            currentProfit: ko.observable(150000)
        };

        // Evalutes to a positive value, so initially we apply the "profitPositive" class
        viewModel.profitStatus = ko.pureComputed(function() {
            return this.currentProfit() < 0 ? "profitWarning" : "profitPositive";
        }, viewModel);

        // Causes the "profitPositive" class to be removed and "profitWarning" class to be added
        viewModel.currentProfit(-50);
    </script>

Isso irá aplicar a classe CSS `profitPositive` quando o valor de `currentProfit` for positivo, caso contrário ele irá apliar a classe CSS `profitWarning`.

### Parâmetros

  * Parâmetro principal
   
    Se você estiver usando nomes de classe CSS estáticos, então você pode passa um objeto JavaScript em que os nomes da propriedade serão as suas classes e seus valores serão `true` ou `false`, de acordo com a classe que deve ser aplicada.
 
    Você pode definir multiplas classes de uma só vez. Por exemplo, se a sua view model tem uma propriedade chamada `isSevere`,
   
        <div data-bind="css: { profitWarning: currentProfit() < 0, majorHighlight: isSevere }">

    Você pode até definir multiplas classes CSS baseada na mesma condição envolvendo os nomes como:

        <div data-bind="css: { profitWarning: currentProfit() < 0, 'major highlight': isSevere }">
   
    Valores não-boolean são interpretados livremente como boolean. Por exemplo, `0` e `null` são tratados como `false`, enquanto `21` e objetos não-`null` são tratados como `true`.
   
    Se o parâmetro faz referência a um valor observable, o binding irá adicionar ou remover a classe CSS sempre que o valor for alterado. Se o parâmetro não faz referência a um valor observable, ele vai apenas adicionar ou remover a classe uma vez.

    Se você quer usar um nome de classe CSS dinâmica, então você pode passar uma string correspondente para a classe ou classes que você quer adicionar ao elemento. Se o parâmetro referencia um valor observable, então o binding irá remover qualquer classe adicionada anteriormente e vai adicionar a classe ou classes correspondentes ao novo valor.
   
    Como de costume, você pode usar qualquer expressão JavaScript ou função como valor de parâmetro. KO irá avaliar e usar os valores resultantes para determinar as classes CSS apropriadas para adicionar ou remover.
   
  * Parâmetros adicionais

      * Nenhum

### Nota: Aplicando classes CSS cujos nomes não são nomes de variáveis legais de JavaScript

Se você quer aplicar a classe CSS `my-class`, você não pode escrever isso:

    <div data-bind="css: { my-class: someValue }">...</div>

... porque `my-class` não é um identificador legal nesse ponto. A solução é simples: apenas envolva o identificador em aspas para que se torne uma string literal, que é legal em um objeto JavaScript. Por exemplo:

    <div data-bind="css: { 'my-class': someValue }">...</div>

### Dependências

Nenhuma, exceto a biblioteca Knockout.