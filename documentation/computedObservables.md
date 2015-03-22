---
layout: documentation
title: Computed Observables
---

E se você tiver um [observable](observables.html) para `firstName` e outro para `lastName` e você quer exibir o nome completo? É aqui onde computed observables entram em ação – eles são funções que dependem de um ou mais observables, e automaticamente alterará sempre que qualquer uma dessas dependências forem alteradas.

Por exemplo, dado a seguinte classe view model

    function AppViewModel() {
        this.firstName = ko.observable('Bob');
        this.lastName = ko.observable('Smith');
    }

... você poderia adicionar um computer observable para retornar o nome completo:

    function AppViewModel() {
        // ... leave firstName and lastName unchanged ...

        this.fullName = ko.computed(function() {
            return this.firstName() + " " + this.lastName();
        }, this);
    }

Agora você poderia ligar os elementos da UI, por exemplo:

    The name is <span data-bind="text: fullName"></span>

... e eles serão alterados sempre que `firstName` ou `lastName` é alterado (sua função de avaliação será chamada toda vez que qualquer de suas dependências forem alteradas, e qualquer valor que você retornar será passado para os observers, como elementos da UI ou outros computed observables).
    
### Encadeamento de dependência funciona

Claro, você pode criar encadeamento de computed observables se desejar. Por exemplo, você pode ter:

* um **observable** chamado `items` representando um conjunto de itens
* outro **observable** chamado `selectedIndexes` guardando qual índices do item foi “selecionado” pelo usuário
* um **computed observable** chamado `selectedItems` que retorna um array de objetos itens correspondendo os índices selecionados
* outro **computed observable** que retorna `true` ou `false` dependendo se qualquer `selectedItems` tem alguma propriedade (como “novo” ou “não salvo”). Algum elemento da UI, como um botão, pode ser ativado ou desativado baseado neste valor.

Alterações ao `items` ou `selectedIndexes` irá afetar o encadeamento dos computed observables, o que irá alterar qualquer elemento da UI ligado a eles.

### Gerenciando 'this'

O segundo parâmetro do `ko.computed` (o bit onde passamos `this` no exemplo acima) define o valor de `this` quando avaliando o computed observable. Sem passar ele, não seria possível referir `this.firstName()` ou `this.lastName()`. Desenvolvedores JavaScript experientes dirão que isto é óbvio, mas se você ainda está começando com JavaScript pode parecer estranho. (Linguagens como C# e Java nunca esperam que o programador coloque um valor para `this`, mas em JavaScript, sim, porque as funções em si não são parte de nenhum objeto.)

#### Uma convenção popular que simplifica as coisas

Existe uma convenção popular que evita a necessidade de rastrear o `this`: se seu construtor do view model copiar a referencia para `this` em uma variável diferente (tradicionalmente chamada `self`), você pode então usar `self` em toda parte de sua view model e não precisa se preocupar se for redefinida para referir a alguma outra coisa. Por exemplo:

    function AppViewModel() {
        var self = this;

        self.firstName = ko.observable('Bob');
        self.lastName = ko.observable('Smith');
        self.fullName = ko.computed(function() {
            return self.firstName() + " " + self.lastName();
        });
    }

Porque `self` é capturado no fim da função, ele se mantém disponível e consistente em qualquer conjunto de funções, como avaliador do computed observable. Esta convenção é ainda mais útil quando se trata de event handlers, que verá em muitos dos [live examples](http://knockoutjs.com/examples/).

### *Pure* computed observables

Se seu computed observable simplesmente calcula e torna um valor baseado em alguma dependência observable, então é declara-lo como um `ko.pureComputed` em vez de `ko.computed`. Por exemplo:

    this.fullName = ko.pureComputed(function() {
        return this.firstName() + " " + this.lastName();
    }, this);

Visto que este computed é declarado para ser `puro` (em outras palavras, o avaliador não modifica diretamente outros objetos ou estado), knockout pode eficientemente gerenciar suas reavaliações e uso de memória. Knockout automaticamente irá suspender ou liberar se nenhum outro código tiver uma dependência ativa.

Pure computeds foram introduzidos na versão 3.2.0 do Knockout. Veja também: [mais sobre pure computed observables](computed-pure.html).

### Forçando computed observables para sempre notificar seus subscribers

Quando um computed observable retorna um valor primitivo (um número, string, boolean ou null), as dependências do observable são normalmente notificados se o valor for realmente alterado. No entanto, é possível usar o [extender](extenders.html) `notify` para garantir que os subscribers do computed observables sejam sempre notificados na alteração, mesmo se o valor for o mesmo. Você aplicaria o extender desta forma:

    myViewModel.fullName = ko.pureComputed(function() {
        return myViewModel.firstName() + " " + myViewModel.lastName();
    }).extend({ notify: 'always' });

### Adiando ou suprindo notificações de alteração

Normalmente, um observable notifica seus subscribers imediatamente, logo que suas dependências são alteradas. Mas se um observable tem muitas dependências ou tem muitas alterações, você pode ter uma performance melhor limitando ou adiando as notificações. Isto é feito usando a [`rateLimit` extender](rateLimit-observable.html) desta forma:

    // Ensure updates no more than once per 50-millisecond period
    myViewModel.fullName.extend({ rateLimit: 50 });
    
### Determinando se uma propriedade é um computed observable

Em alguns casos, é útil determinar programaticamente se você está lidando com computed observable. Knockout oferece uma função de utilitário, `ko.isComputed` para ajudar nesta situação. Por exemplo, você quer excluir computed observables de um dato que você está enviando para o servidor.

    for (var prop in myObject) {
      if (myObject.hasOwnProperty(prop) && !ko.isComputed(myObject[prop])) {
          result[prop] = myObject[prop];
      }
    }

Adicionalmente, Knockout oferece funções similares que podem operar nos observables e computed observables:

* `ko.isObservable` - retorna true para observable, arrays observable, e todos os computed observables.
* `ko.isWritableObservable` - retorna true para observables, observable arrays, e computed observables graváveis (também chamado de `ko.isWriteableObservable`).

