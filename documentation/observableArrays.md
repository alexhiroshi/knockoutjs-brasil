---
layout: documentation
title: Arrays Observable
---

Se você quiser detectar e responder a alterações de um objeto, você usaria [observables](observables.html). Se você quiser detectar e responder a alterações de uma *coleção de coisas*, use um `observableArray`. Isto é útil em muitos cenários onde você está exibindo ou editando múltiplos valores e precisa que seções repetidas de sua UI apareça e desapareça quando itens são adicionados ou removidos.

### Exemplo

    var myObservableArray = ko.observableArray();    // Initially an empty array
    myObservableArray.push('Some value');            // Adds the value and notifies observers

Para saber como você pode ligar o `observableArray` a uma UI e deixar que o usuário modifique, veja [a lista simples de exemplos](http://knockoutjs.com/examples/simpleList.html).

### Ponto chave: Um observableArray localiza quais objetos estão no array, não o estado desses objetos

Simplesmente colocando um objeto em um `observableArray` não faz com que todas as propriedades deste objeto sejam observable. Claro, você pode fazê-los ser se desejar, mas não é um escolha independente. Um `observableArray` apenas localiza quais objetos ele guarda, e notifica os listeners quando os objetos são adicionados ou removidos.

## Pré populando um observableArray

Se você **não** quiser que seu array de observable se inicialize vazio, mas que contenha itens inicialmente, passe estes itens como um array para o construtor. Por exemplo:

    // This observable array initially contains three objects
    var anotherObservableArray = ko.observableArray([
        { name: "Bungle", type: "Bear" },
        { name: "George", type: "Hippo" },
        { name: "Zippy", type: "Unknown" }
    ]);

## Lendo informação de um observableArray

Por debaixo dos lençóis, um `observableArray` é na verdade um [observable](observables.html) no qual o valor é um array (e `observableArray` inclui algumas características adicionais descritas abaixo). Então, você consegue o array básico do JavaScript invocando o observableArray como uma função sem parâmetros, como qualquer outro observable. Então você consegue ler informações do array. Por exemplo:

    alert('The length of the array is ' + myObservableArray().length);
    alert('The first element is ' + myObservableArray()[0]);

Tecnicamente você pode usar quaisquer funções array nativa do JavaScript para operar naquele array, mas normalmente há um alternativa melhor. `observableArray` do KO tem funções equivalente próprias, e elas são mais úteis porque:

 1. Elas funcionam em todos os navegadores. (Por exemplo, a função `indexOf` nativa do JavaScript não funciona no IE 8 ou anterior, mas o `indexOf` do KO funciona em qualquer lugar.)
 1. Para as funções que modificam os conteúdos do array, como `push` e `splice`, os métodos do KO automaticamente disparam os mecanismos dependency tracking para que todos os listeners registrados sejam notificados quando houver alteração, e sua UI é automaticamente atualizada.
 1. A sintaxe é mais conveniente. Para chamar o método push do KO, apenas escreva `myObservableArray.push(...)`. Assim é um pouco mais bonito que chamar o método push escrevendo `myObservableArray().push(...)`.

O resto desta página descreve as funções do `observableArray` para ler e escrever informação.

### indexOf

A função `indexOf` retorna o índice do primeiro item do array que é igual ao seu parâmetro. Por exemplo, `myObservableArray.indexOf('Blah')` retornará o índice da primeira entrada do array que for igual a Blah, ou o valor `-1` se não for encontrado.

### slice

A função `slice` do `observableArray` é equivalente à função `slice` nativa do JavaScript (em outras palavras, ele retorna as entradas do seu array a partir de um índice inicial até um índice final). Chamando `myObservable.slide(...)` é equivalente a chamar o mesmo método no array subjacente (`myObservableArray().slide(...)`).

## Manipulando um observableArray

`observableArray` expõe um conjunto de funções familiares para modificar os conteúdos do array e notificando os listeners.

### pop, push, shift, unshift, reverse, sort, splice

All of these functions are equivalent to running the native JavaScript array functions on the underlying array, and then notifying listeners about the change:

 * `myObservableArray.push('Some new value')` adiciona um novo item no final do array
 * `myObservableArray.pop()` remove o último valor do array e retorna-o
 * `myObservableArray.unshift('Some new value')` insere um novo item no começo do array
 * `myObservableArray.shift()` remove o primeiro valor do array e retorna-o
 * `myObservableArray.reverse()` reverte a ordem do array
 * `myObservableArray.sort()` ordena o conteúdo do array.
   * A ordenação patrão é alfabética, mas você opcionalmente passar um função para controlar como o array deve ser ordenado. Sua função deve aceitar quaisquer dois objetos do array e retornar um valor negativo se o primeiro argumento for menor, um valor positivo se o segundo argumento for menor, or retornar zero para trata-los igualmente. Por exemplo, para ordenar um array de objetos ‘pessoas’ pelo sobrenome, você poderia escrever `myObservableArray.sort(function(left, right) { return left.lastName == right.lastName ? 0 : (left.lastName < right.lastName ? -1 : 1) })`
 * `myObservableArray.splice()` remove e retorna um dado número de elementos começando a partir de um dado índice. Por exemplo, `myObservableArray.splice(1, 3)` remove 3 elementos começando a partir do índice 1 (ou seja, segundo, terceiro, e quarto elemento) e retorna eles como um array.

Para mais detalhes sobre essas funções observableArray, veja a documentação equivalente das [funções de array padrão do JavaScript](https://developer.mozilla.org/en/JavaScript/Reference/Global_Objects/Array#Methods_2).

### remove e removeAll

`observableArray` adiciona mais alguns métodos úteis que não são encontrados nos arrays JavaScript por padrão:

 * `myObservableArray.remove(someItem)` remove todos os valores iguais a `someItem` e retorna eles em um array
 * `myObservableArray.remove(function(item) { return item.age < 18 })` remove todos os valores onde a propriedade `age` é menor que 18, e retorna eles em um array
 * `myObservableArray.removeAll(['Chad', 132, undefined])` remove todos os valores iguais a `"Chad"`, `123`, ou `undefined`, e retorna eles em um array
 * `myObservableArray.removeAll()` remove todos os valores e retorna eles em um array

### destroy e destroyAll (Nota: relevante apenas para desenvolvedores Ruby on Rails) {#destroy-and-destroyall}

As funções `destroy` e `destroyAll` são principalmente destinadas como conveniência para desenvolvedores que usam Ruby on Rails:

 * `myObservableArray.destroy(someItem)` procura qualquer objeto no array que seja igual a someItem e dá à eles uma propriedade especial chamada `_destroy` com valor `true`
 * `myObservableArray.destroy(function(someItem) { return someItem.age < 18 })` procura qualquer objeto no array onde a propriedade `age` é menor que 18, e dá à eles uma propriedade especial chamada _destroy com valor `true`
 * `myObservableArray.destroyAll(['Chad', 132, undefined])` procura qualquer objeto no array igual a `'Chad'`, `123` ou `undefined` e dá à eles uma propriedade especial chamada `_destroy` com valor `true`
 * `myObservableArray.destroyAll()` dá uma propriedade especial chamada `_destroy` com valor `true` para todos os objetos no array

Então, o que é esse negócio todo sobre `_destroy`? É interessante apenas para o desenvolvedores Rails. A convenção no Rails é que, quando você passa para uma ação um objeto JSON, o framework automaticamente pode converter em um objeto Active Record e depois salvar em seu banco de dados. Ele sabe qual dos objetos já estão em seu banco de dados, e envia corretamente o INSERT ou UPDATE. Para dizer ao framework para DELETAR um registro, você deve marca-lo com `_destroy` com valor `true`.

Note que quando KO executar um binding `foreach`, automaticamente ele esconderá qualquer objeto marcado com `_destroy` com valor `true`. Para que você possa ter tipo um botão “deletar” que chama o método `destroy(someItem)`, e isso imediatamente fará com o que o item desapareça da UI. Quando enviar um objeto JSON graph para Rails, este item também será deletado do banco de dados (enquanto os outros itens serão inseridos ou alterados como de costume).

## Adiando e/ou suprindo notificações de alteração

Normalmente, um `observableArray` notifica seus subscribers imediatamente, logo que é alterado. Mas se um `observableArray` é alterado repetidamente, você pode ter uma performance melhor limitando ou adiando as notificações. Isto é feito usando o [`rateLimit` extender](rateLimit-observable.html) desta forma:

    // Ensure it notifies about changes no more than once per 50-millisecond period
    myViewModel.myObservableArray.extend({ rateLimit: 50 });
