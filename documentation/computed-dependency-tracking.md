---
layout: documentation
title: Como dependency tracking funciona
---

*Iniciantes não precisam saber sobre isto, mas desenvolvedores experientes irão querer saber porque nós ficamos impondo que o KO automaticamente localiza as dependências e atualiza as partes certas da UI…*

É, na verdade, bem simples e agradável. O algoritmo de localização é assim:

1. Sempre que você declara um computed observable, KO imediatamente invoca sua função de avaliação para conseguir seu valor inicial.
1. While the evaluator function is running, KO sets up a subscription to any observables (including other computed observables) that the evaluator reads. The subscription callback is set to cause the evaluator to run again, looping the whole process back to step 1 (disposing of any old subscriptions that no longer apply).
1. Ko notifica qualquer subscribers sobre o novo valor de seu computed observable.

Knockout não detecta dependências na primeira vez que o avaliador é executado – ele re-detecta eles toda vez. Isto significa que, por exemplo, aquela dependência pode variar dinamicamente: dependência A poderia terminar se o computed observable também dependerá do B ou C. Em seguida, ele apenas será reavaliado quando ou A ou sua escolha entre B ou C ser alterado. Você não precisa declarar dependências: elas são determinadas na execução do código.

Um outro truque é que declarative bindings são simplesmente implementados como computed observables. Então, se um binding lê o valor de um observable, este binding se torna dependente daquele observable, o que causa aquele binding ser reavaliado se o observable ser alterado.

*Pure* computed observables funcionam um pouco diferente. Para mais detalhes, veja a documentação para [*pure* computed observables](computed-pure.html).

### Controlando dependência usando peek

Knockout's automatic dependency tracking normally does exactly what you want. But you might sometimes need to control which observables will update your computed observable, especially if the computed observable performs some sort of action, such as making an Ajax request. The `peek` function lets you access an observable or computed observable without creating a dependency.

No exemplo abaixo, um computed observable é usado para recarregar um observable chamado `currentPageData` usando Ajax com dados de outras duas propriedades observable. O computed observable irá atualizar sempre que `pageIndex` é alterado, mas ele ignora alterações de `selectedItem` porque ele é acessado usando `peek`. Neste caso, o usuário pode querer usar o valor atual de `selectedItem` apenas com finalidade de rastreamento quando um novo conjunto de dados é carregado.

    ko.computed(function() {
        var params = {
            page: this.pageIndex(),
            selected: this.selectedItem.peek()
        };
        $.getJSON('/Some/Json/Service', params, this.currentPageData);
    }, this);

Nota: Se você só quer prevenir que um computed observable seja atualizado frequentemente, veja [`rateLimit` extender](rateLimit-observable.html).

### Nota: Por que dependências circulares não são significantes

Computed observables são supostos a mapear um conjunto de entradas do observable em uma única saída. Como tal, não faz sentido incluir ciclos em seus encadeamentos de dependência. Ciclos *não* seriam análogos para recursão; eles seriam análogos a ter duas células da planilha que são computados como funções uma da outra. Isto levaria a uma loop infinito de avaliação.

Então o que o Knockout faz se você tiver um ciclo em sua dependência? Evita loops infinitos aplicando a seguinte regra: **Knockout não reiniciará a avaliação de um computed enquanto ele já estiver avaliando**. Isto é pouco provável que afetará seu código. É relevante em duas situações: quando dois computed observable são dependentes um do outro (possível apenas se um ou ambos usam a opção `deferEvaluation`), ou quando um computed observable escreve em outro observable no tem um dependência (ou diretamente, ou via um encadeamento de dependência). Se você pre usar um desses exemplos e quer evitar a dependência circular, você pode usar a função `peek` descrita acima.
