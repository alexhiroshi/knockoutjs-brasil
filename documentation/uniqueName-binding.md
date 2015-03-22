---
layout: documentation
title: O binding "uniqueName"
---

### Propósito
O binding `uniqueName` garante que o elemento DOM associado tenha um atributo `name` que não seja vazio. Se o elemento não tiver um atributo `name`, ele cria e adiciona um valor único.

Você não vai precisar usar isso muitas vezes. É útil apenas em alguns casos raros, exemplo:

  * Outras tecnologias podem depender do pressuposto de que certos elementos têm nomes, apesar dos nomes serem irrelevante quando você está usando KO. Por exemplo, [jQuery Validation](http://jqueryvalidation.org/) atualmente só irá validar os elementos que têm nomes. Para utilizar com a UI do Knockout, às vezes é necessário aplicar o binding `uniqueName` para não confundir o jQuery Validation. Veja um [um exemplo de uso do jQuery Validation com KO](http://knockoutjs.com/examples/gridEditor.html).

  * O IE 6 não permite que radio buttons sejam marcados se ele não tiver um atributo `name`. Em muitos casos isso é irrelevante, porque seus elementos radio buttons terão atributos name para adicioná-los em grupos exclusivos. No entando, no caso de você não adicionar um atributo name porque é desnecessário no seu caso, o KO usará internamente `uniqueName` nos elementos para assegurar que eles possam ser marcados.

### Exemplo
    <input data-bind="value: someModelProperty, uniqueName: true" />

### Parâmetros

  * Parâmetro principal
 
    Passe `true` (ou qualquer valor equivalente a true) para habilitar o binding `uniqueName`, como no exemplo anterior.
     
  * Parâmentros adicionais 

      * Nenhum

### Dependências

Nenhuma, exceto a biblioteca Knockout.