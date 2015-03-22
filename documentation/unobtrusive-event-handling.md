---
layout: documentation
title: Usando manipuladores de eventos unobtrusive
---

In most cases, data-bind attributes provide a clean and succinct way to bind to a view model. However, event handling is one area that can often result in verbose data-bind attributes, as anonymous functions were typically the recommended techinique to pass arguments.  For example:

    <a href="#" data-bind="click: function() { viewModel.items.remove($data); }">
        remove
    </a>

As an alternative, Knockout provides two helper functions that allow you to identify the data associated with a DOM element:

 * `ko.dataFor(element)` - returns the data that was available for binding against the element
 * `ko.contextFor(element)` - returns the entire [binding context](binding-context.html) that was available to the DOM element.

Essas funções auxiliares podem ser utilizada em manipuladores de eventos que são adicionadas "unobtrusivamente" usando algo parecido com `bind` ou `click`, do jQuery. A função acima poderia ser adicionada para cada link com a classe `remove`:

    $(".remove").click(function () {
        viewModel.items.remove(ko.dataFor(this));
    });

Melhor ainda, essa técnica poderia ser usada para apoiar delegação de evento. As funções jQuery `live/delegate/on` são uma maneira fácil de fazer isso acontecer:

    $(".container").on("click", ".remove", function() {
        viewModel.items.remove(ko.dataFor(this));
    });

Now, a single event handler is attached at a higher level and handles clicks against any links with the `remove` class. This method has the added benefit of automatically handling additional links that are dynamically added to the document (perhaps as the result of an item being added to an observableArray).

### Exemplo: filhos aninhados

Esse exemplo mostra links "add" e "remove" em vários níveis de pais e filhos com um único manipulador "unobtrusivo" anexado para cada tipo de link.
<style type="text/css">
   .liveExample a.add { font-size: .7em; color: #aaa; }
   .liveExample a.remove { font-size: .9em; }
</style>

{% capture live_example_view %}
<ul id="people" data-bind='template: { name: "personTmpl", foreach: people }'>
</ul>

<script id="personTmpl" type="text/html">
    <li>
        <a class="remove" href="#"> x </a>
        <span data-bind='text: name'></span>
        <a class="add" href="#"> add child </a>
        <ul data-bind='template: { name: "personTmpl", foreach: children }'></ul>
    </li>
</script>
{% endcapture %}

{% capture live_example_viewmodel %}
var Person = function(name, children) {
    this.name = ko.observable(name);
    this.children = ko.observableArray(children || []);
};

var PeopleModel = function() {
    this.people = ko.observableArray([
        new Person("Bob", [
            new Person("Jan"),
            new Person("Don", [
                new Person("Ted"),
                new Person("Ben", [
                    new Person("Joe", [
                        new Person("Ali"),
                        new Person("Ken")
                    ])
                ]),
                new Person("Doug")
            ])
        ]),
        new Person("Ann", [
            new Person("Eve"),
            new Person("Hal")
        ])
    ]);

    this.addChild = function(name, parentArray) {
        parentArray.push(new Person(name));
    };
};

ko.applyBindings(new PeopleModel());

//attach event handlers
$("#people").on("click", ".remove", function() {
    //retrieve the context
    var context = ko.contextFor(this),
        parentArray = context.$parent.people || context.$parent.children;

    //remove the data (context.$data) from the appropriate array on its parent (context.$parent)
    parentArray.remove(context.$data);

    return false;
});

$("#people").on("click", ".add", function() {
    //retrieve the context
    var context = ko.contextFor(this),
        childName = context.$data.name() + " child",
        parentArray = context.$data.people || context.$data.children;

    //add a child to the appropriate parent, calling a method off of the main view model (context.$root)
    context.$root.addChild(childName, parentArray);

    return false;
});

{% endcapture %}
{% include live-example-minimal.html %}

Não importa o quão aninhados os links venham a ser, o manipulador é sempre capaz de identificar e operar sobre os dados apropriados. Usando essa técnica, podemos evitar sobrecarga de anexar manipuladores para cada link e podemos manter a marcação limpa e concisa.