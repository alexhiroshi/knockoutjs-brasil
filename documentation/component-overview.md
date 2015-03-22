---
layout: documentation
title: Componentes e Elementos Customizados – Visão Geral
---

**Componentes** são uma maneira poderosa, limpa de organizar o seu código de UI e são pedaços reutilizáveis. Eles:

 * ...podem representar controles/widgets individuais ou seções inteiras do seu aplicativo
 * ...contém sua própria view e, geralmente (mas não obrigatoriamente), sua própria viewmodel
 * ...podem ser pré-carregados ou carregados de forma assíncrona (on demand) via AMD ou outros sistemas de módulos
 * ...podem receber parâmetros e, opcionalmente, escrever de volta alterações neles ou invocar callbakcs
 * ...podem ser formadas em conjunto (aninhada) ou herdade de outros componentes
 * ...podem ser facilmente empacotados para reutilizar em outros projetos
 * ...permitem definir suas próprias converções/lógicas para configuração e carregamento

Esse padrão é benéfico para grandes aplicações, porque **simplifica o desenvolvimento** através de uma organização clara e encapsulamento e ajuda a **melhorar o desempenho de tempo de execução** através do carregamento incremental do seu código e templates, conforme necessário

**Elementos customizados** are an optional but convenient syntax for consuming components. Instead of needing placeholder `<div>`s into which components are injected with bindings, you can use more self-descriptive markup with custom element names (e.g., `<voting-button>` or `<product-editor>`). Knockout cuida para garantir compatibilidade mesmo com navegadores antigos como IE6.

### Exemplo: Um widget de like/dislike

Para começar, você pode registrar um componente usando `ko.components.register` (tecnicamente, registrar é opcional, mas é a maneira mais fácil de se começar). Uma definição de componente especifica um `viewModel` e `template`. Por exemplo:

{% capture like_dislike_registration %}
    ko.components.register('like-widget', {
        viewModel: function(params) {
            // Data: value is either null, 'like', or 'dislike'
            this.chosenValue = params.value;
            
            // Behaviors
            this.like = function() { this.chosenValue('like'); }.bind(this);
            this.dislike = function() { this.chosenValue('dislike'); }.bind(this);
        },
        template:
            '<div class="like-or-dislike" data-bind="visible: !chosenValue()">\
                <button data-bind="click: like">Like it</button>\
                <button data-bind="click: dislike">Dislike it</button>\
            </div>\
            <div class="result" data-bind="visible: chosenValue">\
                You <strong data-bind="text: chosenValue"></strong> it\
            </div>'
    });
{% endcapture %}
{{ like_dislike_registration }}

**Normalmente, você carregaria a view model e template a partir de arquivos externos** em vez de declara-los inline dessa forma. Chegaremos lá mais tarde.

Agora, para usar esse componente, você pode referencia-lo de qualquer outra view em sua aplicação, ou usando um [binding `component`](component-binding.html) ou usando um [elemento customizado](component-custom-elements.html). Aqui um live example que utiliza-o como um elemento customizado:

<style type="text/css">
    .liveExample ul { margin: 0; }
    .liveExample .product {
        list-style-type: none;
        background-color: rgba(0,0,0,0.1);
        padding: 1em; border-radius: 1em;
        min-height: 3.5em;
    }
    .liveExample .product + .product { margin-top: 1em;  }
    .liveExample > button { margin: 0.5em 0; }
</style>
<script>{{ like_dislike_registration }}</script>

{% capture live_example_id %}component-inline{% endcapture %}
{% capture live_example_view %}
    <ul data-bind="foreach: products">
        <li class="product">
            <strong data-bind="text: name"></strong>
            <like-widget params="value: userRating"></like-widget>
        </li>
    </ul>
{% endcapture %}
{% capture live_example_viewmodel %}
    function Product(name, rating) {
        this.name = name;
        this.userRating = ko.observable(rating || null);
    }

    function MyViewModel() {
        this.products = [
            new Product('Garlic bread'),
            new Product('Pain au chocolat'),
            new Product('Seagull spaghetti', 'like') // This one was already 'liked'
        ];
    }

    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% include live-example-minimal.html %}

Nesse exemplo, o componente exibe e edita uma propriedade observable chamada `userRating` na classe view model `Product`.

### Exemplo: Carregando o widget like/dislike a partir de um arquivo externo, sob demanda

Na maioria das aplicações, você irá querer manter os view models e templates do component em um arquivo externo. Se você configurar o Knockout para procura-los via AMD module loader como [require.js](http://requirejs.org/), então eles podem ser, ou pré-carregados (possivelmente minificado), ou carregados incrementalmente de acordo com a necessidade.

Aqui está um exemplo de configuração:

{% capture like_dislike_amd_registration %}
    ko.components.register('like-or-dislike', {
        viewModel: { require: 'files/component-like-widget' },
        template: { require: 'text!files/component-like-widget.html' }
    });
{% endcapture %}
{{ like_dislike_amd_registration }}

**Requerimentos**

Para que isso funcione, os arquivos [`files/component-like-widget.js`](files/component-like-widget.js) and [`files/component-like-widget.html`](files/component-like-widget.html) precisam existir. Verifique eles (e veja o *código fonte* no arquivo .html) – como você pode ver, é mais limpo e mais conveniente que incluindo o código inline na definição.

Também, você precisa referenciar uma biblioteca module loader adequada (como [require.js](http://requirejs.org/)) ou implementar um [loader de componente customizado](component-loaders.html) que sabe como conseguir seus arquivos.

**Usando o componente**

Agora `like-or-dislike` pode ser usado da mesma maneira como antes, usando um [binding `component`](component-binding.html) ou um [elemento customizado](component-custom-elements.html):

<script>{{ like_dislike_amd_registration }}</script>

{% capture live_example_id %}component-amd{% endcapture %}
{% capture live_example_view %}
    <ul data-bind="foreach: products">
        <li class="product">
            <strong data-bind="text: name"></strong>
            <like-or-dislike params="value: userRating"></like-or-dislike>
        </li>
    </ul>
    <button data-bind="click: addProduct">Add a product</button>
{% endcapture %}
{% capture live_example_viewmodel %}
    function Product(name, rating) {
        this.name = name;
        this.userRating = ko.observable(rating || null);
    }

    function MyViewModel() {
        this.products = ko.observableArray(); // Start empty
    }

    MyViewModel.prototype.addProduct = function() {
        var name = 'Product ' + (this.products().length + 1);
        this.products.push(new Product(name));
    };

    ko.applyBindings(new MyViewModel());
{% endcapture %}
{% include live-example-minimal.html %}

Se você abrir o inspetor de rede das ferramentas de desenvolvedor de seu navegador antes de clicar em Add product verá que os arquivos `.js`/`.html` do componente são buscados sob demanda quando são requeridos pela primeira vez, e consequentemente retidos para reuso.

### Saiba mais

Mais informações detalhes, veja:

 * [Definindo e registrando componentes](component-registration.html)
 * [Usando o binding `component`](component-binding.html)
 * [Usando elementos customizados](component-custom-elements.html)
 * [Avançado: Componentes loaders customizados](component-loaders.html)
