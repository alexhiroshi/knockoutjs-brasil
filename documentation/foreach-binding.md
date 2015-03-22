---
layout: documentation
title: O binding "foreach"
---

### Propósito
O binding `foreach` repete uma determinada marcação para cada item presente em um array, e associa cada elemento da marcação ao item correspondente. Sendo muito usado para renderizar listas ou tabelas.

Quando o array utilizado for um [observable array](observableArrays.html), sempre que for adicionado ou removido um item ou quando o array for reordenado, o binding vai de forma muito eficiente atualizar a interface do usuário para adicionar/ remover mais cópias da marcação ou ordenar os elementos associados, sem afetar os outros elementos DOM presentes na tela. Esta forma é muito mais rápida do que gerar todos os elementos novamente para cada mudança no array.

E com certeza, você pode, se necessário, juntar vários `foreach` e também pode usá-lo com outros bindings como o `if` e o `with`.

### Exemplo 1: Navegando por um array

O exemplo a seguir usa o `foreach` para criar uma tabela onde cada linha representa um item do array.

    <table>
        <thead>
            <tr><th>First name</th><th>Last name</th></tr>
        </thead>
        <tbody data-bind="foreach: people">
            <tr>
                <td data-bind="text: firstName"></td>
                <td data-bind="text: lastName"></td>
            </tr>
        </tbody>
    </table>

    <script type="text/javascript">
        ko.applyBindings({
            people: [
                { firstName: 'Bert', lastName: 'Bertington' },
                { firstName: 'Charles', lastName: 'Charlesforth' },
                { firstName: 'Denise', lastName: 'Dentiste' }
            ]
        });
    </script>

### Exemplo 2: Adicionando/Removendo itens de um array

O próximo exemplo mostra que, se o array for do tipo observable, então a interface vai estar sincronizada com as mudanças que ocorrerem com o array.

{% capture live_example_view %}
<h4>People</h4>
<ul data-bind="foreach: people">
    <li>
        Name at position <span data-bind="text: $index"> </span>:
        <span data-bind="text: name"> </span>
        <a href="#" data-bind="click: $parent.removePerson">Remove</a>
    </li>
</ul>
<button data-bind="click: addPerson">Add</button>
{% endcapture %}

{% capture live_example_viewmodel %}
function AppViewModel() {
    var self = this;

    self.people = ko.observableArray([
        { name: 'Bert' },
        { name: 'Charles' },
        { name: 'Denise' }
    ]);

    self.addPerson = function() {
        self.people.push({ name: "New at " + new Date() });
    };

    self.removePerson = function() {
        self.people.remove(this);
    }
}

ko.applyBindings(new AppViewModel());
{% endcapture %}

{% include live-example-minimal.html %}

### Parâmetros

  * Parâemtro principal

    Pass the array that you wish to iterate over. The binding will output a section of markup for each entry.

    Alternatively, pass a JavaScript object literal with a property called `data` which is the array you wish to iterate over. The object
    literal may also have other properties, such as `afterAdd` or `includeDestroyed` --- see below for details of these extra options and
    examples of their use.

    If the array you supply is observable, the `foreach` binding will respond to any future changes in the array's contents by adding or
    removing corresponding sections of markup in the DOM.

  * Parâmetros adicionais

      * Nenhum

### Nota 1: Acessando itens do array usando $data

Como mostrado anteriormente, os bindings dentro de um `foreach` podem referenciar as propriedades dos itens de um array. Por exemplo, [Exemplo 1](#example-1-iterating-over-an-array) referencia as propriedades `firstName` e `lastName` de cada item do array.

Mas, e quando é necessário referenciar o item e não uma de suas propriedades? Nesse caso, pode ser utilizado o [propriedade de contexto](binding-context.html) `$data`. Dentro de um `foreach`, o $data  significa “o item atual”. Por exemplo:

    <ul data-bind="foreach: months">
        <li>
            The current item is: <b data-bind="text: $data"></b>
        </li>
    </ul>

    <script type="text/javascript">
        ko.applyBindings({
            months: [ 'Jan', 'Feb', 'Mar', 'etc' ]
        });
    </script>

Se preferir, pode-se usar o `$data` para acessar as propriedades do item atual, mesmo se não for um `foreach`. Por exemplo o [Exemplo 1](#example-1-iterating-over-an-array) poderia ser escrito da seguinte forma:

    <td data-bind="text: $data.firstName"></td>

... mas isso não é obrigatório, porque a propriedade `firstName` vai estar associado a um `$data` por padrão.

### Nota 2: Usando o $index, $parent, e outras proriedades de contexto

Como pode ser visto no Exemplo 2, é possível utilizar `$index` para saber o índice do item atual. Lembrando que o `$index` é do tipo observable, ou seja, seu valor é alterado sempre que o índice de um item muda. Por exemplo, se um item é adicionado ou removido do array.

Além disso, existe o `$parent` que é utilizado para acessar dados que estão fora do escopo do `foreach`, por exemplo:

    <h1 data-bind="text: blogPostTitle"></h1>
    <ul data-bind="foreach: likes">
        <li>
            <b data-bind="text: name"></b> likes the blog post <b data-bind="text: $parent.blogPostTitle"></b>
        </li>
    </ul>

Para mais informações sobre o `$index` e outras propriedades de contexto como o `$parent`, acesse a [propriedade binding context](binding-context.html).

### Nota 3: Apelidando itens em um “foreach” utilizando o “as”

Como mencionado na Nota 1, pode-se acessar o item atual de um array utilizando o `$data`. Em alguns casos pode ser útil dar um nome mais descritivo para o item atual do `foreach`, para isso pode se utilizar a opção `as`:

    <ul data-bind="foreach: { data: people, as: 'person' }"></ul>

Agora dentro de um  loop `foreach` , vai ser possível acessar o item atual do array `people` utilizando o alias `person`, que está sendo renderizado. Isso pode ser extremamente útil dentro de blocos `foreach` que estejam um dentro do outro e seja necessário acessar uma variável de um nível acima. Por exemplo:

    <ul data-bind="foreach: { data: categories, as: 'category' }">
        <li>
            <ul data-bind="foreach: { data: items, as: 'item' }">
                <li>
                    <span data-bind="text: category.name"></span>:
                    <span data-bind="text: item"></span>
                </li>
            </ul>
        </li>
    </ul>

    <script>
        var viewModel = {
            categories: ko.observableArray([
                { name: 'Fruit', items: [ 'Apple', 'Orange', 'Banana' ] },
                { name: 'Vegetables', items: [ 'Celery', 'Corn', 'Spinach' ] }
            ])
        };
        ko.applyBindings(viewModel);
    </script>

Dica: Lembre-se de passar o nome da variável como uma *string* para o `as` (`as: 'category'`, e *não* `as: category`), pois está sendo definido um nome para uma nova variável e não uma leitura de uma variável que já existe.

### Nota 4: Usando um foreach sem um elemento container

Em alguns casos, pode ser necessario duplicar uma parte da marcação, mas sem ter um elemento container para declarar o binding `foreach`. Por exemplo:

    <ul>
        <li class="header">Header item</li>
        <!-- The following are generated dynamically from an array -->
        <li>Item A</li>
        <li>Item B</li>
        <li>Item C</li>
    </ul>

Nesse exemplo, não existe um elemento para declarar o binding `foreach`. Não pode ser declarado no elemento `<ul>` (porque o item de “cabeçalho” seria duplicado), e também não seria possível colocar outro elemento dentro do `<ul>` (porque apenas elementos `<li>`  são permitidos dentro de `<ul>`s).

Para este caso, pode ser usado a técnica *containerless control flow syntax (Controle de fluxo sem container)*, que é escrito na forma de comentários. Por exemplo:

    <ul>
        <li class="header">Header item</li>
        <!-- ko foreach: myItems -->
            <li>Item <span data-bind="text: $data"></span></li>
        <!-- /ko -->
    </ul>

    <script type="text/javascript">
        ko.applyBindings({
            myItems: [ 'A', 'B', 'C' ]
        });
    </script>

Os comentários `<!-- ko -->` e `<!-- /ko -->` funcionarão como marcadores de início/fim, definindo um “elemento virtual” que contém a marcação. Knockout entende esta sintaxe de elemento virtual e liga como se você tivesse um conjunto de elementos reais.

### Nota 5: Como mudanças de array são detectadas e tratadas

Quando você modifica o conteúdo de seu model array (adicionando, movendo, ou deletando entradas), o binding `foreach` use um algoritmo eficiente de diferenciação para descobrir o que foi alterado, para que ele possa, então, atualizar o BOM para corresponder. Isso significa que ele pode controlar combinações de alterações simultâneas aleatórias.

* Quando você **adiciona** entradas no array , `foreach` irá apresentar novas cópias de seu template e inseri-los no DOM existente
* Quando você **deleta** entradas no array, `foreach` irá simplesmente remover os elementos DOM correspondentes
* Quando você **reordena** entradas no array (mantendo as mesmas instâncias do objeto), `foreach` irá tipicamente apenas mover os elementos DOM correspondentes para nova posição.

Note que a detectação de reordenação não é garantida: para ter certeza de que o algoritmo termine rápido, ele é otimizado para detectar movimentos "simples" de um pequeno número de entradas do array. Se o algoritmo detectar muitas reordenções simultâneas combinadas com inserções e deleção não relacionadas, então para velocidade ele pode escolher conseiderar reordenação como um "deletar" mais um "adicionar" em vez de um único "mover", e nesse caso os elementos DOM correspondentes irão ser derrubados e recriados. Muitos desenvolvedores terão esse caso, e mesmo que tiver, a experiência para o usuário final será idêntica.

### Nota 6: Entradas destruídas estão escondidas por padrão

Algumas vezes voc6e pode querer marcar uma entrada do array como deletada, mas sem realmente perder a existência do registro. Isso é conhecido como uma *deleção não-destrutiva*. Para detalhes de como fazer isso, veja [a função destruir no `observableArray`](observableArrays.html#destroy-and-destroyall).

Por padrão, o binding `foreach` irá pular (esconder) qualquer entradas do array que estão marcadas como destruídas. Se você quiser mostrar as entradas destruídas, use a opção `includeDestroyed`. Por exemplo,

    <div data-bind='foreach: { data: myArray, includeDestroyed: true }'>
        ...
    </div>


### Nota 7: Pós-processamento ou animando os elementos DOM gerados

Se você precisa executar alguma lógica customizado nos elementos DOM gerados, você pode usar qualquer um dos callbacks `afterRender`/`afterAdd`/`beforeRemove`/`beforeMove`/`afterMove` descritos abaixo.

> **Nota:** Esses callbacks são *apenas* para ativar animações relacionadas a mudanças em uma lista. Se seu objetivo é na verdade atribuir outros comportamentos para novos elementos DOM quando eles são adicionados (exemplo: event handlers, or para ativar controles de UI de terceiros), então seu trabalho será facilitado se você implementar esse novo comportamento como um [custom binding](custom-bindings.html) ao invés, porque você poderá usar esse comportamento em qualquer lugar, independentemente do binding `foreach`.

Aqui um exemplo trivial que utiliza `afterAdd` para aplicar o efeito clássico "fade amarelo" para os itens adicionados recentemente. Requere o [jQuery Color plugin](https://github.com/jquery/jquery-color) para habilitar a animação de cores do fundo.

    <ul data-bind="foreach: { data: myItems, afterAdd: yellowFadeIn }">
        <li data-bind="text: $data"></li>
    </ul>

    <button data-bind="click: addItem">Add</button>

    <script type="text/javascript">
        ko.applyBindings({
            myItems: ko.observableArray([ 'A', 'B', 'C' ]),
            yellowFadeIn: function(element, index, data) {
                $(element).filter("li")
                          .animate({ backgroundColor: 'yellow' }, 200)
                          .animate({ backgroundColor: 'white' }, 800);
            },
            addItem: function() { this.myItems.push('New item'); }
        });
    </script>

Detalhes completos:

  * `afterRender` --- é invocado sempre que o bloco `foreach` é duplicado e inserido no documento, ambos quando `foreach` é inicializado pela primeira vez, e quando novas entradas são adicionadas ao array associado depois. Knockout fornecerá os seguintes parâmetros ao seu callback:

      1. Um array de elementos DOM inseridos
      2. O dado do item que estão sendo ligados

  * `afterAdd` --- é como `afterRender`, exceto que é invocado apenas quando novas entradas são adicionadas ao seu array (and *not* when `foreach` first iterates over your array's initial contents). Um uso comum para `afterAdd` é chamar um método como `$(domNode).fadeIn()` do jQuery para que você consigo transições animadas sempre que itens são adicionados. Knockout fornecerá os seguintes parâmetros ao seu callback:

      1. Um nó DOM sendo adicionado ao documento
      2. O índice do elemento adicionado ao array
      3. O elemento adicionado ao array

  * `beforeRemove` --- é invocado quando um item do array é removido, mas antes dos nós DOM correspondentes serem removidos. Se você especificar um callback `beforeRemove`, então *se torna sua responsabilidade remover os nós DOM*. O caso de uso óbvio aqui é chamar alguma coisa como `$(domNode).fadeOut()` do jQuery para animar a remoção dos nós DOM correspondentes -- nesse caso, Knockout não sabe o quão cedo ele está permitido remover os nós DOM fisicamente (quem sabe quando tempo sua animação irá durar?), então cabe a você removê-los. Knockout fornecerá os seguintes parâmetros ao seu callback:

      1. Um nó DOM que você deveria remover
      2. O índice do elemento removido do array
      3. O elemento removido do array

  * `beforeMove` --- é invocado quando um item do array muda de posição, mas antes dos nós DOM correspondentes serem movidos. Note que  `beforeMove` aplica para todos os elementos do array dos quais os índices foram alterados, então se você inserir um novo item no começo do array, o callback (se especificado) disparará para todos os outros elementos, já que seu índice de posição foi incrementado por um. Você pode usar `beforeMove` para guardar as coordenadas originais da tela dos elementos afetados para que você possa animar seus movimentos no callback `afterMove`. Knockout fornecerá os seguintes parâmetros ao seu callback:
  
      1. Um nó DOM que pode estar preste a ser movido
      2. O índice do elemento movido do array
      3. O elemento movido do array

  * `afterMove` --- é invocado depois que um item do array muda de posição, e depois que o `foreach` atualiza o DOM para corresponder / NOte que para `afterMove` aplica para todos os elementos do array dos quais os índices foram alterados, então se você inserir um novo item no começo do array, o callback (se especificado) disparará para todos os outros elementos, já que seu índice de posição foi incrementado por um. Knockout fornecerá os seguintes parâmetros ao seu callback:
  
      1. Um nó DOM que pode ser movido
      2. O índice do elemento movido do array
      3. O elemento movido do array

Para exemplos de `afterAdd` e `beforeRemove`, veja [transições animadas](http://knockoutjs.com/examples/animatedTransitions.html). 

### Dependências

Nenhuma, exceto a biblioteca Knockout.