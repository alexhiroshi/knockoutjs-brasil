---
layout: documentation
title: A sintaxe data-bind
---

O sistema declarativo de binding do Knockout fornece uma maneira concisa e poderosa para conectar dados com a UI. Em geral é fácil e óbvio vincular propriedades de dados simples ou usar um único binding. Para bindings mais complexos, ele ajuda a entender melhor o comportamento e a sintaxe do sistema de bindings do Knockout.

### Sintaxe binding

Um binding é composto por dois itens, o nome e o valor, separados por dois pontos. Aqui está um exemplo simples de binding:

    Today's message is: <span data-bind="text: myMessage"></span>

Um elemento pode incluir vários bindings (relacionada ou não), com cada binding separado por uma vírgula. Aqui alguns exemplos:

    <!-- related bindings: valueUpdate is a parameter for value -->
    Your value: <input data-bind="value: someValue, valueUpdate: 'afterkeydown'" />

    <!-- unrelated bindings -->
    Cellphone: <input data-bind="value: cellphoneNumber, enable: hasCellphone" />

O binding *name* deve geralmente corresponder ao binding handler registrado (tanto internamente or [customizado](custom-bindings.html)) ou ser um parâmetro para outro binding. Se o nome condizer com nenhum desses, Knockout irá ignora-lo (sem nenhum erro ou aviso). Então se um binding parecer não funcionar, verifique se o nome está correto.

#### Binding values

O binding *value* pode ser um único [valor, variável, literal](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Values,_variables,_and_literals) ou quase qualquer [expressão JavaScript](https://developer.mozilla.org/en-US/docs/JavaScript/Guide/Expressions_and_Operators) válida. Aqui são exemplos com vários valores:

    <!-- variable (usually a property of the current view model -->
    <div data-bind="visible: shouldShowMessage">...</div>

    <!-- comparison and conditional -->
    The item is <span data-bind="text: price() > 50 ? 'expensive' : 'cheap'"></span>.

    <!-- function call and comparison -->
    <button data-bind="enable: parseAreaCode(cellphoneNumber()) != '555'">...</button>

    <!-- function expression -->
    <div data-bind="click: function (data) { myFunction('param1', data) }">...</div>

    <!-- object literal (with unquoted and quoted property names) -->
    <div data-bind="with: {emotion: 'happy', 'facial-expression': 'smile'}">...</div>

Esses exemplos mostram que o valor pode ser qualquer expressão JavaScript. Até mesmo a vírgula não tem problema quando está fechado por colchetes, chaves, ou parênteses. Quando o valor é um objeto literal, o nome da propriedade do objeto deve ser um identificador JavaScript válido ou ser fechado em aspas. Se o binding value for uma expressão inválida ou referencia uma variável desconhecida, Knockout irá produzir um erro e parar de processar os bindings.

#### Espaço em branco

Bindings podem incluir qualquer quantidade de espaços em branco (espaços, tab e novas linhas), então você pode usar isso para organizar os seus bindings do jeito que preferir. Os seguintes exemplos são todos equivalentes:

    <!-- no spaces -->
    <select data-bind="options:availableCountries,optionsText:'countryName',value:selectedCountry,optionsCaption:'Choose...'"></select>

    <!-- some spaces -->
    <select data-bind="options : availableCountries, optionsText : 'countryName', value : selectedCountry, optionsCaption : 'Choose...'"></select>

    <!-- spaces and newlines -->
    <select data-bind="
        options: availableCountries,
        optionsText: 'countryName',
        value: selectedCountry,
        optionsCaption: 'Choose...'"></select>

#### Pulando o binding value

Começando com o Knockout 3.0, você pode especificar bindings sem um valor, que irá lhe dar ao binding um valor `undefined`. Por exemplo:

    <span data-bind="text">Text that will be cleared when bindings are applied.</span>

Essa habilidade é especialmente útil quando pareada com [preprocessando binding](binding-preprocessing.html), que pode atribuir um valor padrão para o binding. 