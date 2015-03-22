---
layout: documentation
title: Asynchronous Module Definition (AMD) com RequireJs
---

### Visão geral do AMD

Trecho de [Writing Modular JavaScript With AMD, CommonJs & ES Harmony](http://addyosmani.com/writing-modular-js/):

> Quando dizemos que um aplicativo é módular, queremos geralmente dizer que ele é composto de um conjunto altamente dissociado, peças distintas de funcionalidade armazenados em módulos. Como provavelmente você sabe, o baixo acoplamento facilita a fácil manutenção de aplicativos, removendo dependências quando possível. Quando isso for implementado de forma eficiente, é bem fácil de ver como as mudanças para uma parte do sistema pode afetar outra.
>
> Ao contrário de algumas linguagens de programação mais tradicionais, a interação atual de JavaScript (ECMA-262) não fornece aos desenvolvedores os meios para importar esses módulos de código de forma limpa, organizada. É uma das preocupações com especificações que não tenham exigido grande atenção até os anos mais recentes, onde a necessidade de aplicações JavaScript mais organizado tornou-se aparente.
>
> Instead, developers at present are left to fall back on variations of the module or object literal patterns. With many of these, module scripts are strung together in the DOM with namespaces being described by a single global object where it's still possible to incur naming collisions in your architecture. There's also no clean way to handle dependency management without some manual effort or third party tools.
>
> Enquanto soluções nativas para esses problemas estão chegando no ES Harmony, a boa notícia é que escrever JavaScript modular nunca foi tão fácil e você pode começar hoje.

### Carregando Knockout.js e uma classe ViewModel via RequireJs

HTML

    <html>
        <head>
            <script type="text/javascript" data-main="scripts/init.js" src="scripts/require.js"></script>
        </head>
        <body>
            <p>First name: <input data-bind="value: firstName" /></p>
            <p>First name capitalized: <strong data-bind="text: firstNameCaps"></strong></p>
        </body>
    </html>

scripts/init.js

    require(['knockout-x.y.z', 'appViewModel', 'domReady!'], function(ko, appViewModel) {
        ko.applyBindings(new appViewModel());
    });

scripts/appViewModel.js

    // Main viewmodel class
    define(['knockout-x.y.z'], function(ko) {
        return function appViewModel() {
            this.firstName = ko.observable('Bert');
            this.firstNameCaps = ko.pureComputed(function() {
                return this.firstName().toUpperCase();
            }, this);
        };
    });

É claro, `x.y.z` deve ser substituido com o número da versão do script Knockout que você estier carregando (ex.: `knockout-3.1.0`).

### Carregando Knockout.js, um manipulador de Binding, e uma class ViewModel via RequireJs

Documentação sobre manipulador de binding, em geral, pode ser encontrada [aqui](http://knockoutjs.com/documentation/custom-bindings.html). Essa seção destina-se a demonstrar o poder que os módulos AMD fornecem em manter seus manipuladores personalizados. Vamos tomar como exemplo o `ko.bindingHandlers.hasFocus`, exemplo da documentação do manipulador de binding. Envolvendo o monipulador em seu próprio módulo, você pode restringir o uso apenas para as páginas que precisam dele. O módulo envolvido parece com:

    define(['knockout-x.y.z'], function(ko){
        ko.bindingHandlers.hasFocus = {
            init: function(element, valueAccessor) { ... },
            update: function(element, valueAccessor) { ... }
        }
    });

Depois que você definiu o módulo de atualização do elemento input do exemplo HTML acima para ser:

    <p>First name: <input data-bind="value: firstName, hasFocus: editingName" /><span data-bind="visible: editingName"> You're editing the name!</span></p>

Incluir o módulo na lista de dependências para a sua view model:

    define(['knockout-x.y.z', 'customBindingHandlers/hasFocus'], function(ko) {
        return function appViewModel(){
            ...
            // Add an editingName observable
            this.editingName = ko.observable();
        };
    });

Note que o módulo de manipulador de binding customizado não injeta qualquer coisa em nosso módulo ViewModel, que é porque ele não retorna nada. Ele só acrescenta um comportamento adicional para o módulo do Knockout.

### Download do RequireJs

RequireJs pode ser baixando em [http://requirejs.org/docs/download.html](http://requirejs.org/docs/download.html).
