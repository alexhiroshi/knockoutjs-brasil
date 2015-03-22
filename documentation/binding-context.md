---
layout: documentation
title: Binding context
---

Um *binding context* é um objeto que guarda dados que voc6e pode referenciar a partir de seus bindings. Enquanto aplicando bindings, Knockout automaticamente cria e gerencia uma hierarquia de binding contexts. A raiz da hierarquia refere-se ao parâmetro `viewModel` que você forneceu para `ko.applyBindings(viewModel)`. Então, cada vez que usar um binding de fluxo de controle como [`with`](with-binding.html) ou [`foreach`](foreach-binding.html), ele cria um  binding context filho que refere-se ao view model data aninhado.

Bindings contexts oferecem as seguintes propriedades especiais que você pode referenciar em qualquer binding:

  * `$parent`

     Esse é o objeto view model do contexto pai, imediatamente fora do contexto atual. No contexto raiz, este é undefined. Exemplo:

         <h1 data-bind="text: name"></h1>

         <div data-bind="with: manager">
             <!-- Now we're inside a nested binding context -->
             <span data-bind="text: name"></span> is the
             manager of <span data-bind="text: $parent.name"></span>
         </div>

  * `$parents`

     Esse é um array que representa todos os pais da view model:

     `$parents[0]` é a view model do contexto pai (ou seja, é o mesmo que `$parent`)

     `$parents[1]` é a view model do contexto do avô

     `$parents[2]` é a view model do contexto do bisavô

     ... e assim por diante.

  * `$root`

     Esse é o objeto principal da view model no contexto raiz, ou seja, o contexto pai mais alto. Geralmente é o objeto que é passado para `ko.applyBindings`. É equivalente a `$parents[$parents.length - 1]`.

  * `$component`

     Se você está dentro de um contexto de um [componente](component-overview.html) template específico, então `$component` refere-se ao viewmodel para aquele componente. É o equivalente `$root` do component-specific. Em caso de componentes aninhados, `$component` refere-se ao viewmodel para o componente mais próximo.

     Isso é útil, por exemplo, se um template do componente inclui um ou mais blocos `foreach` no qual você deseja referir à alguma propriedade ou função no viewmodel do componente em vez do item atual.

  * `$data`

     Esse é o objeto view model do contexto atual. No contexto raiz, `$data` e `$root` são equivalentes. Dentro de um binding context aninhado, esse parâmetro será definido para o dado do item atual (por exemplo, dentro de um binding `with: person`, `$data` será definido como `person`). `$data` é útil para quando você quer referenciar a própria viewmodel, em vez da uma propriedade da viewmodel. Exemplo:
    
         <ul data-bind="foreach: ['cats', 'dogs', 'fish']">
             <li>The value is <span data-bind="text: $data"></span></li>
         </ul>

  * `$index` (disponível apenas dentro do binding `foreach`)

     É um índice baseado em zero do array atual que está sendo processado por um binding `foreach`. Ao contrário das outras propriedades do binding context, `$index` é um observable e é atualizado sempre que o índice do item é alterado (por exemplo, se os itens são adicionados ou removidos do array).

  * `$parentContext`

     Isso refere-se ao objeto binding context no nível pai. É diferente de `$parent`, que refere-se ao *dado* (não binding context) no nível pai. Isso é útil, por exemplo, se você precisa acessar o valor do índice de um item `foreach` externo a partir de um contexto interno (uso: `$parentContext.$index`). Isso é indefinido no contexto root.
     
  * `$rawData`

     Isso é um valor cru do view model no contexto atual. Normalmente será o mesmo que `$data`, mas se o view model fornecido ao Knockout está envolvido em um observable, `$data` será o view model não envolvido, e `$rawData` será o próprio observable.

  * `$componentTemplateNodes`

    Se você está dentro de um contexto de um [componente](component-overview.html) template específico,, então `$componentTemplateNodes` é um array contendo qualquer nó DOM que foi passado ao componente. Isso faz com que seja fácil construir componentes que recebem templates, por exemplo um componente grid que aceita um template para definir suas linhas de saída. Para um exemplo completo, veja [passando marcação para componentes](component-custom-elements.html#passing-markup-into-components).

As seguintes variáveis especiais também estão disponíveis nos bindings, mas não fazem parte do objeto do binding context:

  * `$context`

     Refere-se ao objeto binding context atual. Pode ser útil se você quer acessar propriedades de um contexto quando eles podem também existir no view model, ou se você quer passa o objeto context para uma função helper em seu view model.
     
  * `$element`

     Esse é o objeto do elemento DOM (para elementos virtuais, será o objeto DOM de comentário) do binding atual. Isso é útil se um binding precisa acessar um atributo do elemento atual. Exemplo:

         <div id="item1" data-bind="text: $element.id"></div>

### Controlando ou modificando o binding context em um binding customizado

Assim como os bindings internos [`with`](with-binding.html) e [`foreach`](foreach-binding.html), bindings customizados podem mudar o contexto para os seus elementos descendentes ou fornecer propriedades especiais, estendendo o objeto do contexto. Isso está descrito em detalhes em [criando bindings customizados que controlam bindings descendentes](custom-bindings-controlling-descendant-bindings.html).
