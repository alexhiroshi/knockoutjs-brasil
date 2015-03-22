---
layout: documentation
title: Mapping
---

Knockout é projetado para permitir que você objetos JavaScript como view models. Contanto que algumas de suas propriedades do view model são [observables](observables.html), você pode usar o KO para ligar eles à sua UI, e a UI será atualizada automaticamente sempre que a propriedade do observable é alterado.

Muitas aplicações precisam buscar dados a partir de um servidor backend. Visto que o servido não tem nenhum conceito de observable, ele irá apenas fornecer objetos JavaScript simples (normalmente serializados como JSON). O pluging mapping fornece uma maneira direta de mapear esses objetos JavaScript simples em um view model com os observables apropriados. Essa é uma alternativa para manualmente escrever seu próprio código JavaScript que construi um view model baseado em algum dado que você buscou no servidor.


{% capture plugin_download_link %}
 * __[Version 2.0](https://github.com/SteveSanderson/knockout.mapping/tree/master/build/output)__ (8.6kb minified)
{% endcapture %}
{% include plugin-download-link.html %}

### Exemplo: Mapeamento manual sem o plugin ko.mapping

Você quer exibir o server-time e o número de usuários em sua página web. Você poderia representar essa informação usando o seguindo view model:

    var viewModel = {
        serverTime: ko.observable(),
        numUsers: ko.observable()
    }

Você poderia ligar esse view model à alguns elementos HTML como no exemplo seguinte:

    The time on the server is: <span data-bind='text: serverTime'></span>
    and <span data-bind='text: numUsers'></span> user(s) are connected.

Visto que as propriedades do view model são observables, o KO automaticamente atualizará os elementos HTML sempre que essas propriedades são alteradas.

Em seguida, você quer buscar o dado mais recente do servidor. A cada 5 segundos você pode enviar uma requisição Ajax (usando as funções jQuery `$.getJSON` ou `$.ajax`):

    var data = getDataUsingAjax(); // Gets the data from the server

O servidor pode retornar um dado JSON similar ao seguinte dado:
    
    {
        serverTime: '2010-01-07',
        numUsers: 3
    }

Finalmente, para atualizar seu view model usando esse dado (sem usar o plugin mapping), você escreveria:

    // Every time data is received from the server:
    viewModel.serverTime(data.serverTime);
    viewModel.numUsers(data.numUsers);

Você teria de fazer isso para todas as variáveis que você quer exibir em sua página. Se sua estrutura de dado se tornar mais complexo (por exemplo, eles contêm filhos ou arrays) isso se torna um incômodo para lidar manualmente. O que o plugin mapping permite que você faça é criar um mapeamento a partir de um objeto normal JavaScript (ou estrutura JSON) à um view model do observable.

### Exemplo: Usando ko.mapping

Para criar um view model através do plugin mapping, substitua a criação do `viewModel` no código acima pela função `ko.mapping.fromJS`: 

    var viewModel = ko.mapping.fromJS(data);

Isso automaticamente cria propriedades observable para cada uma das propriedades no `data`. Depois, toda vez que receber um novo dado do servidor, você pode atualizar todas as propriedades da `viewModel` em um passo chamando a função `ko.mapping.fromJS` novamente:

    // Every time data is received from the server:
    ko.mapping.fromJS(data, viewModel);

### Como as coisas são mapeadas

 * Todas as propriedades de um objeto são convertidas em um observable. Se um atualização alterar o valor, ele atualizará o observable.
 * Arrays são convertidos em <a title="Trabalhando com Arrays Observable" href="/documentacao/observables/trabalhando-com-arrays-observable/">arrays de observable</a>. Se uma atualização alteraria o número de itens, ele irá realizar as ações apropriadas de adicionar/remover. Ele também tentará manter a mesma ordem do array JavaScript original.

### Desmapeando

Se você quer converter seu objeto mapeado de volta para um objeto JavaScript normal, use:

    var unmapped = ko.mapping.toJS(viewModel);

Isso irá criar um objecto desmapeado contendo apenas as propriedades de um objeto mapeado que era parte do seu objeto JavaScript original, quaisquer propriedades ou funções que você adicionou manualmente ao seu view model são ignorados. Por padrão, a única exceção a essa regra é a propriedade `_destroy` que será também mapeada de volta, porque ela é uma propriedade que o Knockout pode gerar quando você destrói um item de um `ko.observableArray`. Veja a seção "Uso Avançado" para mais detalhes em como configurar isso.

### Trabalhando com JSON

Se a sua chama Ajax retorna uma string JSON (e não deserializa em um objeto JavaScript), você pode, então, usar a função `ko.mapping.fromJSON` para criar e atualizar sua view model. Para desmapear, você pode usar `ko.mapping.toJSON`.

Além do fato de que eles trabalham com strings JSON em vez de objetos JavaScript, essas funções são complemente idênticas às funções `*JS` correspondentes.

### Uso avançado

Algumas vezes pode ser necessário ter mais controle sobre como o mapeando é feito. Isso é realizado usando *mapping options*. Eles podem ser especificados durante a chamada `ko.mapping.fromJS`. Nas próximas chamadas você não precisará especifica-los novamente.

Aqui algumas situações no qual você pode querer usar essas opções de mapeaemento.

###### Identificando objetos usando "keys" de forma única

Digamos que você tem um objeto JavaScript que se parece com isso:

    var data = {
        name: 'Scot',
        children: [
            { id : 1, name : 'Alicw' }
        ]
    }

Você pode mapear isso ao um view model sem qualquer problema:

    var viewModel = ko.mapping.fromJS(data);

Agora, digamos que o dado é atualizado para ficam sem erros de digitação:

    var data = {
        name: 'Scott',
        children: [
            { id : 1, name : 'Alice' }
        ]
    }

Duas coisas aconteceram aqui: nome foi alterado de Scot para Scott e children[0].name foi alterado de Alicw para Alice. Você pode atualizar a viewModel baseando nesse novo dado:

    ko.mapping.fromJS(data, viewModel);

E `name` teria sido alterado como esperado. No entanto, no array `children` array, o filho (Alicw) teria sido completamente removido e um novo (Alice) adicionado. Isso não é completamente o que você esperava. Em vez disso, you você teria esperado que apenas a propriedade `name` do filho fosse alterado de `Alicw` para `Alice`, não que o filho inteiro fosse substituído!

Isso acontece porque, por padrão, o plugin mapping simplesmente comparar dois objetos no array. E visto que o JavaScript o objeto `{ id : 1, name : 'Alicw' }` não é igual a `{ id : 1, name : 'Alice' }` ele acha que o filho `inteiro` precisa ser removido e substituido por um novo.

Para resolver isso, você pode especificar qual `key` do plugin mapping deveria usar para determinar se um objeto é novo ou velho. Você faria dessa forma:

    var mapping = {
        'children': {
            key: function(data) {
                return ko.utils.unwrapObservable(data.id);
            }
        }
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

Dessa forma, toda que o plugin mapping verificar se um item no array `children`, ele apenas irá procurar pela propriedade `id` para determinar se um objeto foi completamente substituído ou apenas necessita de atualização.

###### Customizando a construção de objeto usando "create"

Se você quer controlar a parte do mapeando você mesmo, você também pode providenciar um callback `create`. Se esse call estiver presente, o plugin mapping irá permitir que você faça a parte do mapeamento você mesmo.

Digamos que você tem um objeto JavaScript que se parece com isso:

    var data = {
        name: 'Graham',
        children: [
            { id : 1, name : 'Lisa' }
        ]
    }

Se você quer mapear o array `children` você mesmo, você pode especifica-lo dessa forma:

    var mapping = {
        'children': {
            create: function(options) {
                return new myChildModel(options.data);
            }
        }
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

The `options` argument supplied to your `create` callback is a JavaScript object containing:
 
 * `data`: The JavaScript object containing the data for this child
 * `parent`: The parent object or array to which this child belongs

Of course, inside the `create` callback you can do another call to `ko.mapping.fromJS` if you wish. A typical use-case might be if you want to augment the original JavaScript object with some additional [computed observables](computedObservables.html):

    var myChildModel = function(data) {
        ko.mapping.fromJS(data, {}, this);
         
        this.nameLength = ko.computed(function() {
            return this.name().length;
        }, this);
    }

###### Customizing object updating using “update”

You can also customize how an object is updated by specifying an `update` callback. It will receive the object it is trying to update and an `options` object which is identical to the one used by the `create` callback. You should `return` the updated value.

The `options` argument supplied to your `update` callback is a JavaScript object containing: * `data`: The JavaScript object containing the data for this child * `parent`: The parent object or array to which this child belongs * `observable`: If the property is an observable, this will be set to the actual observable

Here is an example of a configuration that will add some text to the incoming data before updating:

    var data = {
        name: 'Graham',
    }
     
    var mapping = {
        'name': {
            update: function(options) {
                return options.data + 'foo!';
            }
        }
    }
    var viewModel = ko.mapping.fromJS(data, mapping);
    alert(viewModel.name());

This will alert `Grahamfoo!`.

###### Ignoring certain properties using “ignore”

If you want the mapping plugin to ignore some properties of your JS object (i.e. to not map them), you can specify an array of propertynames to ignore:

    var mapping = {
        'ignore': ["propertyToIgnore", "alsoIgnoreThis"]
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

The `ignore` array you specify in the mapping options is combined with the default `ignore` array. You can manipulate this default array like this:

    var oldOptions = ko.mapping.defaultOptions().ignore;
    ko.mapping.defaultOptions().ignore = ["alwaysIgnoreThis"];

###### Including certain properties using “include”

When converting your view model back to a JS object, by default the mapping plugin will only include properties that were part of your original view model, except it will also include the Knockout-generated `_destroy` property even if it was not part of your original object. However, you can choose to customize this array:

    var mapping = {
        'include': ["propertyToInclude", "alsoIncludeThis"]
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

The `include` array you specify in the mapping options is combined with the default `include` array, which by default only contains `_destroy`. You can manipulate this default array like this:

    var oldOptions = ko.mapping.defaultOptions().include;
    ko.mapping.defaultOptions().include = ["alwaysIncludeThis"];

###### Copying certain properties using “copy”

When converting your view model back to a JS object, by default the mapping plugin will create observables based on the rules explained [above](#how-things-are-mapped). If you want to force the mapping plugin to simply copy the property instead of making it observable, add its name to the “copy” array:

    var mapping = {
        'copy': ["propertyToCopy"]
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

The `copy` array you specify in the mapping options is combined with the default `copy` array, which by default is empty. You can manipulate this default array like this:

    var oldOptions = ko.mapping.defaultOptions().copy;
    ko.mapping.defaultOptions().copy = ["alwaysCopyThis"];

###### Observing only certain properties using “observe”

If you want the mapping plugin to only create observables of some properties of your JS object and copy the rest, you can specify an array of propertynames to observe:

    var mapping = {
        'observe': ["propertyToObserve"]
    }
    var viewModel = ko.mapping.fromJS(data, mapping);

The `observe` array you specify in the mapping options is combined with the default `observe` array, which by default is empty. You can manipulate this default array like this:

    var oldOptions = ko.mapping.defaultOptions().observe;
    ko.mapping.defaultOptions().observe = ["onlyObserveThis"];

The arrays `ignore` and `include` still work as normal. The array `copy` can be used for efficiency to copy array or object properties including children. If an array or object property is not specified in `copy` or `observe` then it is recursively mapped:

    var data = {
        a: "a",
        b: [{ b1: "v1" }, { b2: "v2" }] 
    };
     
    var result = ko.mapping.fromJS(data, { observe: "a" });
    var result2 = ko.mapping.fromJS(data, { observe: "a", copy: "b" }); //will be faster to map.

Both `result` and `result2` will be:

    {
        a: observable("a"),
        b: [{ b1: "v1" }, { b2: "v2" }] 
    }

Drilling down into arrays/objects works but copy and observe can conflict:

    var data = {
        a: "a",
        b: [{ b1: "v1" }, { b2: "v2" }] 
    };
    var result = ko.mapping.fromJS(data, { observe: "b[0].b1"});
    var result2 = ko.mapping.fromJS(data, { observe: "b[0].b1", copy: "b" });

O `result` será:

    {
        a: "a",
        b: [{ b1: observable("v1") }, { b2: "v2" }] 
    }

Enquanto o `result2` será:

    {
        a: "a",
        b: [{ b1: "v1" }, { b2: "v2" }] 
    }

###### Specifying the update target

If, like in the example above, you are performing the mapping inside of a class, you would like to have `this` as the target of your mapping operation. The third parameter to `ko.mapping.fromJS` indicates the target. For example:

    ko.mapping.fromJS(data, {}, someObject); // overwrites properties on someObject

So, if you would like to map a JavaScript object to `this`, you can pass `this` as the third argument:

    ko.mapping.fromJS(data, {}, this);

##### Mapping from multiple sources

You can combine multiple JS objects in one viewmodel by applying multiple `ko.mapping.fromJS` calls, e.g.:

    var viewModel = ko.mapping.fromJS(alice, aliceMappingOptions);
    ko.mapping.fromJS(bob, bobMappingOptions, viewModel);

Mapping options that you specify in each call will be merged.

##### Mapped observable array

Observable arrays that are generated by the mapping plugin are augmented with a few functions that can make use of the `keys` mapping:

 * mappedRemove
 * mappedRemoveAll
 * mappedDestroy
 * mappedDestroyAll
 * mappedIndexOf

They are functionally equivalent to the regular `ko.observableArray` functions, but can do things based on the key of the object. For example, this would work:

    var obj = [
        { id : 1 },
        { id : 2 }
    ]
     
    var result = ko.mapping.fromJS(obj, {
        key: function(item) {
            return ko.utils.unwrapObservable(item.id);
        }
    });
     
    result.mappedRemove({ id : 2 });

The mapped observable array also exposes a `mappedCreate` function:

    var newItem = result.mappedCreate({ id : 3 });

It will first check if the key is already present and will throw an exception if it is. Next, it will invoke the create and update callbacks, if any, to create the new object. Finally, it will add this object to the array and return it.

{% include plugin-download-link.html %}