---
layout: documentation
title: O binding "selectedOptions"
---

### Propósito
O binding `selectedOptions` controla quais elementos em um select múltiplo estão atualmente selecionados. Isso destina-se a ser utilizado em conjunto com um elemento `<select>` e o binding `options`.

Quando o usuário seleciona ou retira a seleção de um item em um select múltiplo, esse adiciona ou remove o valor correspondente de um array em sua view model. Da mesma forma, supondo que o array é um *observable* em sua view model, então sempre que você adicionar ou remover (por exemplo, via `push` ou `splice`) itens nesse array, o item correspondente em sua UI ficará selecionado ou não. É um binding de 2 vias.

Nota: Para controlar quais elementos de um drop-down de seleção única é selecionado, você pode usar [o binding `value`](value-binding.html).

### Exemplo
    <p>
        Choose some countries you'd like to visit: 
        <select data-bind="options: availableCountries, selectedOptions: chosenCountries" size="5" multiple="true"></select>
    </p>
    
    <script type="text/javascript">
        var viewModel = {
            availableCountries : ko.observableArray(['France', 'Germany', 'Spain']),
            chosenCountries : ko.observableArray(['Germany']) // Initially, only Germany is selected
        };
        
        // ... then later ...
        viewModel.chosenCountries.push('France'); // Now France is selected too
    </script>
    
### Parâmetros

  * Parâmentro principal
   
    Este deve ser um array (ou um array observable). KO define o elemento option selected conforme o conteúdo do array. Qualquer estato selecionado anteriormente será substituído.
   
    Se o seu parâmetro é um array observable, o binding irá atualizar o elemento selecionado sempre qeu o array for alterado (ex., via `push`, `pop` ou [outros métodos de array observable](observableArrays.html)). Se o parâmetro não é observable, ele irá definir a visibilidade do elemento apenas uma vez sem alterá-lo depois.
   
    Se o parâmetro for ou não um array observable, KO irá detectar quando o usuário selecionar ou não um item em um select múltiplo e irá atualizar o array para corresponder. Isso é como você pode ler que um option é selecionado.
      
  * Parâmetros adicionais

      * Nenhum
   
### Nota: Deixando o usuário selecionar a partir de objetos JavaScript

No código de exemplo acima, o usuário pode escolher entre um array de strings. Você *não* está limitado a fornecer apenas strings - seu array de `options` pode conter objetos JavaScript, se desejar. Veja [o binding `options`](options-binding.html) para detalhes de como controlar como objetos devem ser exibidos na lista.

Nesse cenário, os valores que você pode ler e escrever usando `selectedOptions` são os próprios objetos, *não* suas representações textuais. Isso deixa o código muito mais limpo e elegante, na maioria dos casos. Sua view model pode imaginar que o usuário escolhe entre um array de objetos, sem ter que se preocupar como esses objetos são mapeados para uma representação na tela.
     
### Dependências

Nenhuma, exceto a biblioteca Knockout.