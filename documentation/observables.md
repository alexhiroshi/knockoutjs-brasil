---
layout: documentation
title: Observables
---

Knockout é construído em volta de três características:

1. Observables e dependency tracking
1. Declarative bindings
1. Templating

Nesta página, você irá aprender sobre o primeiro desses três. Mas antes, vamos examinar o modelo MVVM e o conceito de *view model*.

# MVVM e View Models

*Model-View-View Model (MVVM)* é um modelo de design para criar interfaces de usuário. Descreve como você pode manter uma possível UI sofisticada, simples, divindindo em três partes:

  * Um *model*: dados armazenados de sua aplicação. Esses dados representam objetos e operações em seu domínio de negócios (ex: contas de banco que realizam transferências de dinheiro) e é independente de qualquer UI. Usando o KO, você irá geralmente fazer chamadas Ajax para algum código server-side para ler e escrever estes dados.

  * Uma *view model*: uma representação de dado e operações em uma UI em puro código. Por exemplo, se você está implementando uma lista, sua view model seria um objeto guardando essa lista de itens, e expondo métodos para adicionar ou remover itens.

    Note que isso não é a UI em si: não há nenhum conceito de botões ou telas. Também não é data model persistente – o que guarda dados não salvos que o usuário está utilizando. Usando o KO, suas view models são objetos puro JavaScript que não guardam nenhum HTML. Mantendo a view model abstrata dessa forma faz com que ela se mantenha simples, para que você possa gerenciar mais comportamentos sofisticados sem se perder.

  * Uma *view*: uma UI visível e interativa representando o estado da view model. Exibe informações de sua view model, envia comandos para a view model (quando o usuário clica nos botões), e atualizar sempre que o estado da view model é alterado.

    Usando o KO, sua view é simplesmente seu documento HTML com declaratives bindings que conectam à view model. Alternativamente, você pode usar templates para gerar HTML usando os dados de sua view model.

Para criar uma view mode utilizando KO, declare qualquer objeto JavaScript. Por exemplo:

    var myViewModel = {
        personName: 'Bob',
        personAge: 123
    };

Você pode criar então, uma view bem simples dessa view model utilizando um declarative binding. Por exemplo, o seguinte HTML exibe o valor `personName`:

    The name is <span data-bind="text: personName"></span>

## Ativando o Knockout

O atributo `data-bind` não é nativo do HTML, porém é perfeitamente OK (é estritamente flexível no HTML5, e não causa problema algum no HTML4 mesmo que o validador irá dizer que há um atributo desconhecido). Porém, visto que o navegador não saberá o que significa, você precisa ativar o Knockout para que reconheça.

Para ativar o Knockout, adicione a seguinte linha em um bloco `<script>`:

    ko.applyBindings(myViewModel);

Você pode tanto colocar o block script no fim do seu documento HTML, ou você pode colocá-lo no topo e envolve-lo em um handler DOM como o [`$` function do jQuery](http://api.jquery.com/jQuery/#jQuery3).

Pronto! Agora, sua view irá exibir como se tivesse escrito o seguinte HTML:

    The name is <span>Bob</span>

Caso esteja se perguntando o que os parâmetros do `ko.applyBindings` fazem:

* O primeiro parâmetro fala qual objeto view model você quer usar com os declaratives bindings quando ele ativar

* Opcionalmente, você pode passar um segundo parâmetro que define qual parte do documento você quer procurar os atributos `data-bind`. Por exemplo, `ko.applyBindings(myViewModel, document.getElementById('someElementId'))`. Isso limita a ativação do elemento com ID `someElementId` e seus descendentes, o que é útil se você quer ter múltiplas view models e associar cada uma com diferentes regiões da página.

É simples.

# Observables

Ok, você viu como criar uma view model básica e como exibir uma de suas propriedades usando binding. Mas um dos benefícios do KO é que ele atualiza sua UI automaticamente quando a view model é alterada. Como KO pode saber quando as partes de sua view model é alterada? Resposta: você precisa declarar as propriedades de seu model como observables, porque esses são objetos JavaScript especiais que pode notificar os subscribers sobre alterações, e pode automaticamente detectar dependências.

Por exemplo, reescreva o objeto view model anterior:

    var myViewModel = {
        personName: ko.observable('Bob'),
        personAge: ko.observable(123)
    };

Você não precisa alterar a view – a mesma sintaxe `data-bind` continuará funcionando. A diferença é que agora é capaz de detectar alterações, e quando é detectado, a view será atualizada automaticamente.

## Lendo e escrevendo observables

Nem todos os navegadores suportam os getters e setters do JavaScript (ex: IE), então para compatibilidade, objetos `ko.observable` são, na verdade, *funções*.

* Para **ler** o valor atual de um observable, apenas chame o observable sem parâmetros. Neste exemplo, `myViewModel.personName()` retornará `'Bob'`, e `myViewModel.personAge()` retornará `123`.

* Para **escrever** um novo valor a um observable, chame o observable e passe um novo valor como parâmetro. Por exemplo, chamando `myViewModel.personName('Mary')` mudará o valor do nome para `'Mary'`.

* Para escrever valores para **várias propriedades de um observable** em um objeto model, você pode usar chaining syntax (encadeamento de sintaxe). Por exemplo, `myViewModel.personName('Mary').personAge(50)` mudará o valor do nome para Mary e o valor da idade para `50`.

A ideia por trás dos observables é que eles podem ser observados, por exemplo, outro código pode dizer que ele quer ser notificado quando houver alterações. É isto que muitos bindings do KO fazem internamente. Então, quando você escreveu `data-bind="text: personName"`, o binding `text` se registrou para ser notificado quando `personName` ser alterado (presumindo que ele seja um valor observable, o que é, agora).

Quando você muda o valor do nome para `'Mary'` ao chamar `myViewModel.personName('Mary')`. o `text` binding automaticamente atualizará os conteúdos do texto do element DOM associado. É assim que alterações na view model automaticamente se propaga na view.

## Explicitamente inscrevendo aos observables

*Você, normalmente, não precisa configurar inscrições manualmente, então iniciantes podem pular esta seção.*

Para usuário avançados, se você quiser registrar suas próprias inscrições para serem notificadas quando houver alterações no observables, você chamar a função `subscribe`. Por exemplo:

    myViewModel.personName.subscribe(function(newValue) {
        alert("The person's new name is " + newValue);
    });

A função `subscribe` é como muitas partes do KO funcionam internamente. Muitas vezes você não precisa usar isto, porque os bindings internos e o sistema de template gerenciam as inscrições.

A função `subscribe` aceita três parâmetros: `callback` é a função que é chamada sempre que uma notificação acontece, `target` (opcional) define o valor do `this` na função callback, e event (opcional; valor padrão é “`change`“) é o nome do evento que recebe a notificação.

Você também pode terminar uma inscrição se desejar: primeiramente capture o valor retornado como uma variável, então chame a função `dispose`, por exemplo:

    var subscription = myViewModel.personName.subscribe(function(newValue) { /* do stuff */ });
    // ...then later...
    subscription.dispose(); // I no longer want notifications

Se você quiser ser notificado quando o valor de um observable está prestes de ser alterado, você pode se inscrever no evento `beforeChange`. Por exemplo:

    myViewModel.personName.subscribe(function(oldValue) {
        alert("The person's previous name is " + oldValue);
    }, null, "beforeChange");
    
Nota: Knockout não garante que os eventos `beforeChange` e `change` irão ocorrer em pares, visto que outras partes de seu código podem levantar qualquer um dos eventos individualmente. Se precisar localizar o valor anterior de um observable, cabe a você usa uma inscrição para capturar e localizar.

## Forçando observable para sempre notificar os subscribers

Quando escrevemos um observable que contém um valor primitivo (um número, string, boolean, ou null), as dependências do observable são normalmente apenas notificadas se o valor realmente foi alterado. No entanto, é possível usar o [extender](extenders.html) `notify` para assegurar que um subscriber do observable será sempre notificado quando alterado, mesmo que o valor seja o mesmo. Você aplicaria um extender a um observable desta forma:

    myViewModel.personName.extend({ notify: 'always' });

## Adiando e/ou suprindo notificações de alteração

Normalmente, um observable notifica seus subscribers imediatamente, logo que é alterado. Mas se um observable é alterado repetidamente, você pode ter uma performance melhor limitando ou adiando as notificações. Isto é feito usando o [`rateLimit` extender](rateLimit-observable.html), desta forma:

    // Ensure it notifies about changes no more than once per 50-millisecond period
    myViewModel.personName.extend({ rateLimit: 50 });
