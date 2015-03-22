---
layout: documentation
title: Criando bindings customizado que suportam elementos virtuais
---

*Nota: Isso é uma técnica avançada, normalmente usada apenas ao criar bibliotecas com bindings reutilizáveis. Não é algo que você normalmente precisa fazer ao criar aplicativos com Knockout*

Os bindings de fluxo de controle do Knockout (ex., [`if`](if-binding.html) e [`foreach`](foreach-binding.html)) não são aplicados apenas em elementos normais do DOM, mas também para elementos “virtuais” definidos por um sintaxe especial baseado em comentários. Por exemplo:

    <ul>
        <li class="heading">My heading</li>
        <!-- ko foreach: items -->
            <li data-bind="text: $data"></li>
        <!-- /ko -->
    </ul>

Bindings customizados também podem trabalhar com elementos virtuais, mas para isso, você deve dizer explicitamente para o Knockout entender elementos virtuais, usando a API `ko.virtualElements.allowedBindings`.

### Exemplo

Para começar, aqui está um binding cutomizado que randomiza a ordem dos nós DOM:

    ko.bindingHandlers.randomOrder = {
        init: function(elem, valueAccessor) {
            // Pull out each of the child elements into an array
            var childElems = [];
            while(elem.firstChild)
                childElems.push(elem.removeChild(elem.firstChild));

            // Put them back in a random order
            while(childElems.length) {
                var randomIndex = Math.floor(Math.random() * childElems.length),
                    chosenChild = childElems.splice(randomIndex, 1);
                elem.appendChild(chosenChild[0]);
            }
        }
    };

Isso funciona muito bem com elementos normais do DOM. Os seguintes elementos serão embaralhados em uma ordem aleatória:

    <div data-bind="randomOrder: true">
        <div>First</div>
        <div>Second</div>
        <div>Third</div>
    </div>

    Contudo, isso *não* funciona com elementos virtuais. Se você tentar isso:

    <!-- ko randomOrder: true -->
        <div>First</div>
        <div>Second</div>
        <div>Third</div>
    <!-- /ko -->

... então você terá o erro `The binding 'randomOrder' cannot be used with virtual elements`. Vamos corrigir isso. Para usar `randomOrder` com elementos virtuais, comece dizendo para o Knockout permitir isso. Adicione o seguinte:

    ko.virtualElements.allowedBindings.randomOrder = true;

Agora não haverá erro. No entanto, ainda não funcionará corretamente, porque nosso binding `randomOrder` está codificado usando chamadas de API DOM normais (`firstChild`, `appendChild`, etc.), o que não entende elementos virtuais. Esta é a razão pela qual KO exige que você opte explicitamente no suporte de elementos virtuais: a não ser que seu binding customizado está codificado usando APIs de elementos virtuais, não funcionará corretamente!

Vamos atualizar o código para `randomOrder`, desta vez usando a API do KO de elementos virtuais:

    ko.bindingHandlers.randomOrder = {
        init: function(elem, valueAccessor) {
            // Build an array of child elements
            var child = ko.virtualElements.firstChild(elem),
                childElems = [];
            while (child) {
                childElems.push(child);
                child = ko.virtualElements.nextSibling(child);
            }

            // Remove them all, then put them back in a random order
            ko.virtualElements.emptyNode(elem);
            while(childElems.length) {
                var randomIndex = Math.floor(Math.random() * childElems.length),
                    chosenChild = childElems.splice(randomIndex, 1);
                ko.virtualElements.prepend(elem, chosenChild[0]);
            }
        }
    };

Note como, em vez de usando APIs como `domElement.firstChild`, nós estamos usando `ko.virtualElements.firstChild(domOrVirtualElement)`. O binding `randomOrder` irá agora funcionar corretamente com os elementos virtuais, por exemplo, `<!-- ko randomOrder: true -->...<!-- /ko -->`.

Além disso, `randomOrder` ainda irá trabalhar com elementos regulares do DOM, porque toda API de `ko.virtualElements` são retrocompatível com elementos reulares do DOM.

### API de Elementos Virtuais

Knockout fornece as seguintes funções para trabalhar com elementos virtuais.

  * `ko.virtualElements.allowedBindings`

    Um objeto cujas chaves determinar quais bindings são utilizáveis com elementos virtuais. Defina `ko.virtualElements.allowedBindings.mySuperBinding = true` para permitir que `mySuperBinding` seja usado como elemento virtual.

  * `ko.virtualElements.emptyNode(containerElem)`

    Remove todos os filhos dos elementos reais ou elementos virtuais `containerElem` (limpando todos os dados associados para evitar vazamento de memória).

  * `ko.virtualElements.firstChild(containerElem)`

    Retorna o primeiro filho do elemento real ou elemento virtual `containerElem`, ou `null` se não houver filhos.

  * `ko.virtualElements.insertAfter(containerElem, nodeToInsert, insertAfter)`

    Insere `nodeToInsert` como um elemento filho real ou elemento virtual `containerElem`, posicionado imediatamente depois `insertAfter` (quando `insertAfter` deve ser um filho de `containerElem`).

  * `ko.virtualElements.nextSibling(node)`

    Retorna o nó irmão que segue `node` em seu elemento pai real ou virtual, ou `null` se não houver seguinte irmão.

  * `ko.virtualElements.prepend(containerElem, nodeToPrepend)`

    Insere `nodeToPrepend` como o primeiro elemento filho real ou virtual de `containerElem`.

  * `ko.virtualElements.setDomNodeChildren(containerElem, arrayOfNodes)`

    Remove todos os filhos do elemento real ou virtual de `containerElem` (nesse processo, limpando todos os dados associados para evitar vazamento de memória), e então insere todos os nós de `arrayOfNodes` com esse novo filho.

Note que isto *não* é intencionado para ser um substituto ao conjunto de APIs DOM. Knockout apenas fornece um conjunto mínimo de APIs de elementos virtuais para ser possível realizar esses tipos de transformações necessárias quando implementando controle de fluxo dos bindings.