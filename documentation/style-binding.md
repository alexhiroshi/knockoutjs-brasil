---
layout: documentation
title: O binding "style"
---

### Propósito
O binding `style` adiciona ou remove um ou mais valores de estilo associado no elemento DOM. Isso é útil, por exemplo, para destacar alguns valores em vermelho se for negativo ou para setar a largura de uma barra que altera o seu valor numérico.

(Nota: Se você não quer aplicar um estilo explícito, mas quer atribuir uma classe CSS, veja [o binding css](css-binding.html).)

### Exemplo
    <div data-bind="style: { color: currentProfit() < 0 ? 'red' : 'black' }">
       Profit Information
    </div>
    
    <script type="text/javascript">
        var viewModel = {
            currentProfit: ko.observable(150000) // Positive value, so initially black
        };
        viewModel.currentProfit(-50); // Causes the DIV's contents to go red
    </script>

Isso vai definir a propriedade `style.color` do elemento para `red` sempre que o valor de `currentProfit` estiver abaixo de zero e `black` sempre que estiver acima de zero.

### Parâmetros

  * Parâmetro principal
   
    Você deve passar um objeto JavaScript em que os nomes da propriedade corresponde aos nomes de estilo e os valores correspondem aos valores de estilo que você quer aplicar.
 
    Você pode setar multiplos valores de estilo de uma vez. Por exemplo, se o seu view model tiver uma propriedade chamada `isSevere`,
   
    `<div data-bind="style: { color: currentProfit() < 0 ? 'red' : 'black', fontWeight: isSevere() ? 'bold' : '' }">...</div>`
   
    Se o seu parâmetro referencia um valor observable, o binding irá atualizar os estilos sempre que o valor for alterado. Se o parâmetro não faz referência a um valor observable, ele irá apenas setar o estilo uma vez.
   
    Como de costume, você pode usar qualquer expressão JavaScript ou função como valores de parâmetro. KO irá avalidar e usar os valores resultantes para determinar os valores de estilo a aplicar.
   
  * Parâmetros adicionais 

      * Nenhum

### Nota: Aplicando estilos cujos nomes não são nomes de variáveis legais de JavaScript

Se você quiser aplicar um estilo `font-weight`, `text-decoration` ou qualquer outro estilo que o nome não é um identificador JavaScript (ex., porque contém um hífen), você deve usar o nome JavaScript para  esse estilo. Por exemplo:

* Não escreva `{ font-weight: someValue }`; escreva `{ fontWeight: someValue }`
* Não escreva `{ text-decoration: someValue }`; escreva `{ textDecoration: someValue }`

Veja também: [uma longa lista de nomes de estilos e seus equivalentes em JavaScript](http://www.comptechdoc.org/independent/web/cgi/javamanual/javastyle.html)

### Dependências

Nenhuma, exceto a biblioteca Knockout.