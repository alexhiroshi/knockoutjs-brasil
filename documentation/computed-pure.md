---
layout: documentation
title: Pure computed observables
---

*Pure* computed observables, introduzido no Knockout 3.2.0, fornece benefícios de performance e memória sobre os computed observables comuns em muitas aplicações. Isto é porque um *pure* computed observables não mantém inscrições em suas dependências quando eles não possuem subscribers. Esta característica:

 * **Previne vazamento de memória** dos computed observables que não são mais referenciados em uma aplicação, mas as dependências ainda existem.
 * **Reduz a sobrecarga de computação** não recalculando os computed observable do quais os valores não estão mais sendo observados.

Um *pure* computed observable automaticamente troca entre dois estados baseado na presença de subscribers.

1. Sempre que não é há subscribers, ele está ***sleeping***. Quando entra no estado *sleeping*, ele descarta todas as inscrições de suas dependências. Durante este estado, ele não se inscreverá em nenhum observable acessado na função de avaliação (porém ele os conta para que `getDependenciesCount()` seja sempre preciso). Se o valor do computed observable é lido enquanto está em sleeping, ele sempre será reavaliado para garantir que o valor é atual..

2. Sempre que tiver qualquer subscribers, ele está ***listening***. Quando entra em estado listening, ele imediatamente invoca a função de avaliação e se inscreve em qualquer observables que é acessado. Neste estado, ele opera como um computed observable comum, como descrito em [Como dependency tracking funciona](computed-dependency-tracking.md).

#### Por que "pure"? {#pure-computed-function-defined}

Pegamos o termo emprestado das [funções pure](http://en.wikipedia.org/wiki/Pure_function) porque esta característica geralmente é apenas aplicável para computed observables dos quais o avaliador é uma *pure function* como segue:

1. Avaliando o computed observable não deve causa nenhum efeito colateral.
2. O valor do computed observable não deve variar baseado no número de avaliações ou outras informações “escondidas”. Seu valor deve ser baseado somente nos valores de outros observables na aplicação, o que para definição de pure function, são considerados seus parâmetros.

#### Sintaxe

O método padrão para definir um pure computed observable é usar `ko.pureComputed`:

    this.fullName = ko.pureComputed(function() {
        return this.firstName() + " " + this.lastName();
    }, this);
    
Alternativamente, você pode usar a opção `pure` junto com `ko.computed`:

    this.fullName = ko.computed(function() {
        return this.firstName() + " " + this.lastName();
    }, this, { pure: true });
    
Para a sintaxe completa, veja a [referência computed observable](computed-reference.html).

### Quando usar um *pure* computed observable

Você pode usar o *pure* para qualquer computed observable que segue as orientações do [*pure function* guidelines](#pure-computed-function-defined). No entanto, você verá um benefício maior quando é aplicado ao design da aplicação que envolve persistent view models que são usados e compartilhados por views e view models temporárias. Usando pure computed observables em uma persistent view model oferece benefício de desempenho de computação ([mas nem sempre](#using-pure-computed-can-hurt-performance)). Usando-os em view models temporários oferece benefícios no gerenciamento de memória.

No seguinte exemplo de uma simples interface wizard, o *pure* computed `fullName` é ligado apenas à view durante o etapa final e é, portanto, atualizado somente quando aquela etapa está ativa.

<style>
#wizard-example {
    position: relative;
    height: 6.5em;
}
#wizard-example .log {
    float: right;
    height: 6em;
    background: white;
    border: 1px solid black;
    width: 20em;
    overflow-y: scroll;
}
#wizard-example button {
    position: absolute;
    bottom: 1em;
}
</style>

{% capture live_example_id %}wizard-example{% endcapture %}
{% capture live_example_view %}
<div class="log" data-bind="text: computedLog"></div>
<!--ko if: step() == 0-->
    <p>First name: <input data-bind="textInput: firstName" /></p>
<!--/ko-->
<!--ko if: step() == 1-->
    <p>Last name: <input data-bind="textInput: lastName" /></p>
<!--/ko-->
<!--ko if: step() == 2-->
    <div>Prefix: <select data-bind="value: prefix, options: ['Mr.', 'Ms.','Mrs.','Dr.']"></select></div>
    <h2>Hello, <span data-bind="text: fullName"> </span>!</h2>
<!--/ko-->
<p><button type="button" data-bind="click: next">Next</button></p>
{% endcapture %}

{% capture live_example_viewmodel %}
function AppData() {
    this.firstName = ko.observable('John');
    this.lastName = ko.observable('Burns');
    this.prefix = ko.observable('Dr.');
    this.computedLog = ko.observable('Log: ');
    this.fullName = ko.pureComputed(function () {
        var value = this.prefix() + " " + this.firstName() + " " + this.lastName();
        // Normally, you should avoid writing to observables within a pure computed 
        // observable (avoiding side effects). But this example is meant to demonstrate 
        // its internal workings, and writing a log is a good way to do so.
        this.computedLog(this.computedLog.peek() + value + '; ');
        return value;
    }, this);

    this.step = ko.observable(0);
    this.next = function () {
        this.step(this.step() === 2 ? 0 : this.step()+1);
    };
};
ko.applyBindings(new AppData());
{% endcapture %}

{% include live-example-minimal.html %}

### Quando *não* usar um *pure* computed observable

#### Efeitos colaterais

Você não deve usar a característica *pure* para um computed observable que é destinado a realizar uma ação quando suas dependências serem alteradas. Exemplos incluem:

* Usando um computed observable para executar um callback baseado em múltiplos observables.

        ko.computed(function () {
            var cleanData = ko.toJS(this);
            myDataClient.update(cleanData);
        }, this);
    
* Em uma função de binding `init`, usando um computed observable para atualizar o elemento ligado.

        ko.computed({
            read: function () {
                element.title = ko.unwrap(valueAccessor());
            },
            disposeWhenNodeIsRemoved: element
        });

The reason you shouldn't use a *pure* computed if the evaluator has important side effects is simply that the evaluator will not run whenever the computed has no active subscribers (and so is sleeping). If it's important for the evaluator to always run when dependencies change, use a [regular computed](computedObservables.html) instead.

#### Performance {#using-pure-computed-can-hurt-performance}

Pode haver casos onde usar o *pure* para um computed observable resulta em sobrecarga de computação maior. Um computed observable normal é reavaliado apenas quando uma de suas dependências é alterada. Mas um *pure* computed observable é também reavaliado toda vez que é acessa quando está em estado de *sleeping* e sempre que ele entra em estado de *listening*. Portanto é possível que um computed observable seja reavaliado mais vezes do que deveria quando usamos o *pure*.

