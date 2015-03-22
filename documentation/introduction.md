---
layout: documentation
title: Introdução
---

Knockout é uma biblioteca JavaScript que simplifica a construção de interfaces gráficas dinâmicas. Toda vez que tiver seções de UI que atualizam dinamicamente (ex: mudanças dependendo das ações do usuário ou quando uma fonte de dado externa muda), KO pode ajudá-lo a implementar de forma simples e sustentável.

Características chave:

* **Elegant dependency tracking** - Automaticamente atualiza as partes corretas de sua UI sempre que seu data model é alterado.
* **Declarative bindings** - Uma forma simples e óbvia de conectar partes de sua UI ao seu data model. Você pode construir UIs dinâmicas complexas facilmente usando, arbitrariamente, nested binding contexts.
* **Trivially extensible** - Implemente comportamentos customizados como bindings declarativas para o fácil reuso em apenas algumas linhas.

Benefícios adicionais:

* **Biblioteca JavaScript pura** - funciona com qualquer tecnologia cliente-side e server-side
* **Pode ser adicionado em cima de sua aplicação web já existente** sem necessidade de mudanças arquiteturais
* **Compact** - por volta de 13kb depois de gzipping
* **Works on any mainstream browser** (IE 6+, Firefox 2+, Chrome, Safari, outros)
* **Comprehensive suite of specifications** (desenvolvida estilo BDD) significa que sua funcionalidade correta pode ser verificada em novos browsers e plataforma

Desenvolvedores familiarizados com Ruby on Rails. ASP.NET MVC, ou outras tecnologias MV* pode ver MVVM como um form em tempo real do MVC com sintaxes declarativas. Em um outro sentido, você pode pensar no KO como uma forma geral de criar UIs para editar dados JSON…o que funcionar para você :)

## Ok, como usar?

O maneira mais fácil e divertida de se começar é trabalhando através de [tutoriais interativos](http://learn.knockoutjs.com). Assim que você entender o básico, dê uma olhada nos [live examples](http://knockoutjs.com/examples/index.html) e depois tente usá-lo em seu projeto.

## KO pretende competir com jQuery (ou Prototype, etc.) ou trabalhar com ele?

Todo mundo ama jQuery! É um ótimo substituto para a API DOM inconsistente que tínhamos que lidar no passado. jQuery é uma excelente forma de baixo nível para manipular elementos e eventos em uma página. KO soluciona um problema diferente.

Assim que sua UI deixar de ser importante e houver apenas alguns comportamentos que sobrepõe, as coisas começam a se tornar complicadas e caras de se manter se você usa apenas jQuery. Considere este exemplo: você está exibindo uma lista de itens, indicando o número de itens nesta lista, e quer habilitar um botão “Adicionar” apenas quando tiver menos que 5 itens. jQuery não tem conceito de data model subjacente,  então para conseguir o número de itens você tem que deduzir pelo número de TRs em uma tabela ou número de DIVs com uma certa class CSS. Talvez o número de itens é exibido em algum SPAN, e você tem que lembrar de alterar o texto daquele SPAN quando o usuário adiciona um item. Você também deve lembrar de desabilitar o botão “Adicionar” quando o número de TRs for 5. Depois, você tem que implementar um botão “Deletar”, e descobrir qual elemento DOM alterar quando clicado.

### Como o Knockout é diferente?
É muito fácil com KO. Ele permite você escalar em complexidade sem medo de introduzir inconsistências. Apenas represent seus itens como array, e então use o binding `foreach` para transformar esse array em uma TABLE or conjunto de DIVs. Quando o array é alterado, a UI muda para corresponder(não é necessário entender como injetar novos TRs ou onde injeta-los). O resto da UI permanece em sincronismo. Por exemplo, você pode ligar um SPAN para mostrar o número de itens:

    There are <span data-bind="text: myItems().count"></span> items

É isso! Você não precisa escrever código para atualiza-lo; Ele se atualiza quando o array `myItems` é alterado. Similarmente, para fazer o botão “Adicionar” habilitar e desabilitar dependendo do número de itens, apenas escreva:

    <button data-bind="enable: myItems().count < 5">Add</button>

Depois, quando implementar a funcionalidade 'Deletar', você não precisa entender quais partes da UI ele tem que interagir; you just make it alter the underlying data model.

Para resumir: KO não compete com jQuery ou  APIs DOM de nível baixo similares. KO oferece uma maneira complementar, de alto nível para conectar o data model à UI. KO por si só não depende do jQuery, mas você pode, com certeza, usar jQuery junto, e certamente é útil se quiser coisas como transições animadas.