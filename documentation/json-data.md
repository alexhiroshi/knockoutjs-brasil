---
layout: documentation
title: Carregando e Salvando dados JSON
---

Knockout allows you to implement sophisticated client-side interactivity, but almost all web applications also need to exchange data with the server, or at least to serialize the data for local storage. The most convenient way to exchange or store data is in [JSON format](http://json.org/) - the format that the majority of Ajax applications use today.

### Carregando ou salvando dados

Knockout não força você a usar qualquer técnica particular para carregar ou salvar dados. Você pode usar qualquer mecanismo conveniente que se encaixe em sua tecnologia server-side. O mais comum são os métodos ajax do jQuery como [`getJSON`](http://api.jquery.com/jQuery.getJSON/), [`post`](http://api.jquery.com/jQuery.post/), e [`ajax`](http://api.jquery.com/jQuery.ajax/). Você pode buscar dados do servidor:

    $.getJSON("/some/url", function(data) { 
    	// Now use this data to update your view models, 
    	// and Knockout will update your UI automatically 
    })

... ou enviar dados para o servidor:

	var data = /* Your data in JSON format - see below */;
	$.post("/some/url", data, function(returnedData) {
		// This callback is executed if the post was successful		
	})

Ou, se você não quer usar jQuery, você pode usar qualquer outro mecanismo para carregar ou salvar JSON. Então, tudo o que o Knockout precisa para te ajudar é:

 * For saving, get your view model data into a simple JSON format so you can send it using one of the above techniques
 * For loading, update your view model using data that you've received using one of the above techniques

### Convertendo dados do View Model para JSON simples

Suas view models *são* objetos JavaScripts, nesse caso, você pode apenas serializar JSON usando qualquer padrão JSON, como [JSON.stringify](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify) (uma função nativa em navegadores modernos), ou a biblioteca [`json2.js`](https://github.com/douglascrockford/JSON-js/blob/master/json2.js). Contudo, sua view model provavelmente contém observables, coputed observables e arrays observables, que são implementados com funções JavaScript e, portanto, nem sempre terá um serialize limpo sem trabalho adicional para você.

Para ficar mais fácil serializar dados de uma view model, incluindo observable e afins, Knockout inclui duas funções auxiliares:

 * `ko.toJS` --- this clones your view model's object graph, substituting for each observable the current value of that observable, so you get a plain copy that contains only your data and no Knockout-related artifacts
 * `ko.toJSON` --- this produces a JSON string representing your view model's data. Internally, it simply calls `ko.toJS` on your view model, and then uses the browser's native JSON serializer on the result. Note: for this to work on older browsers that have no native JSON serializer (e.g., IE 7 or earlier), you must also reference the [`json2.js`](https://github.com/douglascrockford/JSON-js/blob/master/json2.js) library.
 
Por exemplo, definir uma view model da seguinte forma:

    var viewModel = {
        firstName : ko.observable("Bert"),
        lastName : ko.observable("Smith"),
        pets : ko.observableArray(["Cat", "Dog", "Fish"]),
        type : "Customer"
    };
    viewModel.hasALotOfPets = ko.computed(function() {
        return this.pets().length > 2
    }, viewModel)
    
Isso contém uma mistura de observables: computed observables, arrays observables e um valor normal. Você pode converter isso para uma string JSON adequado para enviar para o servidor usando `ko.toJSON`, como por exemplo:

    var jsonData = ko.toJSON(viewModel);
    
    // Result: jsonData is now a string equal to the following value
    // '{"firstName":"Bert","lastName":"Smith","pets":["Cat","Dog","Fish"],"type":"Customer","hasALotOfPets":true}'

Ou, se você quer somente um objeto JavaScript antes de serializar, use `ko.toJS` como o seguinte:

    var plainJs = ko.toJS(viewModel);
    
    // Result: plainJS is now a plain JavaScript object in which nothing is observable. It's just data.
    // The object is equivalent to the following:
    //   {
    //      firstName: "Bert",
    //      lastName: "Smith",
    //      pets: ["Cat","Dog","Fish"],
    //      type: "Customer",
    //      hasALotOfPets: true
    //   }

Note that `ko.toJSON` accepts the same arguments as [JSON.stringify](https://developer.mozilla.org/en-US/docs/JavaScript/Reference/Global_Objects/JSON/stringify). For example, it can be useful to have a "live" representation of your view model data when debugging a Knockout application. To generate a nicely formatted display for this purpose, you can pass the *spaces* argument into `ko.toJSON` and bind against your view model like:

    <pre data-bind="text: ko.toJSON($root, null, 2)"></pre>


### Atualizando dados do View Model usando JSON

If you've loaded some data from the server and want to use it to update your view model, the most straightforward way is to do it yourself. For example,

    // Load and parse the JSON
    var someJSON = /* Omitted: fetch it from the server however you want */;
    var parsed = JSON.parse(someJSON);

    // Update view model properties
    viewModel.firstName(parsed.firstName);
    viewModel.pets(parsed.pets);
    
Em muitos cenários, essa abordagem direta é a solução mais simples e mais flexível. Claro, ao atualizar as propriedades no seu view model, Knockout irá cuidar da atualização da UI visível para corresponder à sua atualização.

No entanto, muito desenvolvedores preferem usar uma abordagem mais convencional para atualizar seus view models usando dados de entrada sem escrever manualmente uma linha de código para cada propriedade a ser atualizada. Isso pode ser benéfico se seus view models possuem muitas propriedades, ou estruturas de dados profundamente aninhados, porque pode reduzir uma grande quantidade de código de mapeamento que você precisa escrever manualmente. Para mais detalhes sobre essa técnica, veja [o plugin knockout.mapping](plugins-mapping.html).