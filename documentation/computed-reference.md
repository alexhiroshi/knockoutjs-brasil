---
layout: documentation
title: Referência do Computed Observable
---

A seguinte documentação descreve como construir e trabalhar com computed observable.

## Construindo um computed observable

Um computed observable pode ser construído usando umas das seguintes formas:

1. `ko.computed( evaluator [, targetObject, options] )` --- Esta forma suporta o caso mais comum de criar um computed observable.
  * `evaluator` --- Uma função que é usada para avaliar o valor atual do computed observable.
  * `targetObject` --- Se fornecido, define o valor do `this` sempre que KO invoca as funções callback. Veja a seção em [gerenciando `this`](computedObservables.html#managing-this) para mais informação.
  * `options` --- Um objeto com propriedades adicionais para um computed observable. Veja a lista completa abaixo.

1. `ko.computed( options )` --- Este parâmetro para criar um computed observable aceita um objeto JavaScript com qualquer uma das propriedades a seguir.
  * `read` --- Obrigatório. Uma função usada para avaliar o valor atual do computed observable.
  * `write` --- Opcional. Se fornecido, faz com que o computed observable seja gravável. Esta é uma função que recebe valores que outro código está tentando gravar em seu computed observable. Cabe a você prover uma lógica para lidar com os valores de entrada, tipicamente escrever os valores em algum observable adjacente.
  * `owner` --- Opcional. Se fornecido, define o valor de `this` sempre que KO invoca seus callbacks de `read` ou `write`.
  * `pure` --- Opcional. Se esta opção for `true`, o computed observable será configurado como um [*pure* computed observable](computed-pure.html). Esta opção é uma alternativa para o construtor `ko.pureComputed`.
  * `deferEvaluation` --- Opcional. Se esta opção for `true`, então o valor do computed observable não será avaliado até que alguma coisa tente acessar seu valor ou manualmente se inscreva nele. Por padrão, um computed observable tem seu valor imediatamente determinado durante a criação.
  * `disposeWhen` --- Opcional. Quando fornecido, esta função é executado antes de cada reavaliação para determinar se o computed observable deverá ser descartado. Um resultado `true` irá disparar a eliminação do computed observable.
  * `disposeWhenNodeIsRemoved` --- Opcional. Se fornecido, a eliminação do computed observable será disparado quando um node DOM específico é removido pelo KO. Esta característica é usada para os descartar computed observable usados em bindings quando nodes são removidos pelo `template` e bindings control-flow.
  
1. `ko.pureComputed( evaluator [, targetObject] )` --- Constrói um [*pure* computed observable](computed-pure.html) usando a função dada de avaliação e um objeto opcional para usar com o `this`. Diferente do `ko.computed`, esse não aceita um parâmetro `options`.

1. `ko.pureComputed( options )` ---  Constrói um *pure* computed observable usando um objeto options. Este aceita as opções `read`, `write` e `owner` descritas acima..

## Usando um computed observable

Um computed observable oferece as funções a seguir:

* `dispose()` --- Manualmente descarta o computed observable, removendo todas as inscrições das dependências. Esta função é útil se você quer que um computed observable pare de ser alterado ou quer liberar memória para um computed observable que tem dependências em um observable que não será liberado.
* `extend(extenders)` --- Aplica o [extenders](extenders.html) dado ao computed observable.
* `getDependenciesCount()` --- Retorna o número corrente de dependências do computed observable.
* `getSubscriptionsCount()` --- Retorna o número corrente de inscrições (tanto de outros computed observables ou inscrições manuais) do computed observable.
* `isActive()` --- Retorna se o computed observable será alterado. Um computed observable está inativo se ele não tiver nenhuma dependência.
* `peek()` --- Retorna o valor atual do computed observable sem criar uma dependência (veja a seção sobre [`peek`](computed-dependency-tracking.html#controlling-dependencies-using-peek)).
* `subscribe( callback [,callbackTarget, event] )` --- Registra uma [inscrição manual](observables.html#explicitly-subscribing-to-observables) para ser notificado de alterações ao computed observable.

## Usando o computed context

Durante a execução da função de avaliação do um computed observable, você pode acessar `ko.computedContext` para obter informações sobre a propriedade computed. Ele fornece as seguintes funções:

* `isInitial()` --- Uma função que retorna `true` se chamada durante a primeira avaliação do computed observable, or `false` caso contrário. Para *pure* computed observables, `isInitial()` é sempre `undefined`.

* `getDependenciesCount()` --- Retorna o número de dependências do computed observable detectada até agora durante a atual avaliação.
  * Nota: `ko.computedContext.getDependenciesCount()` é equivalente a chamar `getDependenciesCount()` no computed observable. A razão por existir também no `ko.computedContext` é para oferecer uma maneira de contar as dependências durante a primeira avaliação, mesmo antes do computed observable ser construído.

Exemplo:

    var myComputed = ko.computed(function() {
        // ... Omitted: read some data that might be observable ...

        // Now let's inspect ko.computedContext
        var isFirstEvaluation = ko.computedContext.isInitial(),
            dependencyCount = ko.computedContext.getDependenciesCount(),
        console.log("Evaluating " + (isFirstEvaluation ? "for the first time" : "again"));
        console.log("By now, this computed has " + dependencyCount + " dependencies");

        // ... Omitted: return the result ...
    });

Essas “instalações” são tipicamente úteis apenas em cenários avançados, por exemplo, quando o propósito primário de seu computed observable é causar algum efeito durante sua avaliação, e você quer realizar alguma lógica apenas durante a primeira execução, or apenas se tiver pelo menos uma dependência (e por isso a reavaliação no futuro). A maioria das propriedades computed não precisam se importar se já foram avaliadas, or quantas dependências eles têm.
