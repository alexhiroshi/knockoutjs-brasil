---
layout: documentation
title: O binding "template"
---

### Propósito
O binding `template` preenche o elemento DOM associado com os resultado da renderização de um template. Templates são uma maneira simples e conveniente para contruir estruturas de UI sofisticadas – repetindo blocos ou com blocos aninhados – em funções de dados do seu view model.

Existem duas maneiras principais de usar templates:

  * *Template Nativo* é um mecanismo que suporta `foreach`, `if`, `with` e outros fluxos de controle. Internamente, esses fluxos de controle capturam a marcação HTML no seu elemento e usa como um template para renderizar com itens de dados. Este recurso está no Knockout e não necessita de qualquer biblioteca externa.
  * *Template String-based* é uma maneira de conectar Knockout com um template engine de terceiros. Knockout irá passar valores do seu model para o template engine externo e injetar o resultado no seu documento. Veja abaixo exemplos que usam os templates *jquery.tmpl* e *Underscore*.

### Parâmetros

  * Parâmetro principal

      * Shorthand syntax: If you just supply a string value, KO will interpret this as the ID of a template to render. The data it supplies to the template will be your current model object.

      * For more control, pass a JavaScript object with some combination of the following properties:

          * `name` --- the ID of an element that contains the template you wish to render - see [Note 5](#note-5-dynamically-choosing-which-template-is-used) for how to vary this programmatically.
          * `nodes` --- directly pass an array of DOM nodes to use as a template. This should be a non-observable array and note that the elements will be removed from their current parent if they have one. This option is ignored if you have also passed a nonempty value for `name`.
          * `data` --- an object to supply as the data for the template to render. If you omit this parameter, KO will look for a `foreach` parameter, or will fall back on using your current model object.
          * `if` --- if this parameter is provided, the template will only be rendered if the specified expression evaluates to `true` (or a `true`-ish value). This can be useful for preventing a null observable from being bound against a template before it is populated.
          * `foreach` --- instructs KO to render the template in "foreach" mode - see [Note 2](#note-2-using-the-foreach-option-with-a-named-template) for details.
          * `as` --- when used in conjunction with `foreach`, defines an alias for each item being rendered - see [Note 3](#note-3-using-as-to-give-an-alias-to-foreach-items) for details.
          * `afterRender`, `afterAdd`, or `beforeRemove` --- callback functions to be invoked against the rendered DOM elements - see [Note 4](#note-4-using-afterrender-afteradd-and-beforeremove)

### Nota 1: Renderizando um template

Normalmente, quando usamos bindings de fluxo de controle (`foreach`, `with`, `if` etc.), não há necessicade de dar nomes aos seus templates: eles são definidos implicitamente e anonimamente pela marcação dentro do seu elemento DOM. Mas se você quiser, você pode escrever os templates em um elemento separado e, então, referenciá-los pelo nome:

    <h2>Participants</h2>
    Here are the participants:
    <div data-bind="template: { name: 'person-template', data: buyer }"></div>
    <div data-bind="template: { name: 'person-template', data: seller }"></div>

    <script type="text/html" id="person-template">
        <h3 data-bind="text: name"></h3>
        <p>Credits: <span data-bind="text: credits"></span></p>
    </script>

    <script type="text/javascript">
         function MyViewModel() {
             this.buyer = { name: 'Franklin', credits: 250 };
             this.seller = { name: 'Mario', credits: 5800 };
         }
         ko.applyBindings(new MyViewModel());
    </script>

Nesse exemplo, a marcação `person-template` é usada duas vezes: uma para `buyer` e outra para `seller`. Note que a marcação do template é envolvida por `<script type="text/html">` –-- o valor do atributo type é necessário para garantir que a marcação não seja executada como JavaScript e que o Knockout não tente aplicar bindings para a marcação, exceto quando for usado como um template.

It's not very often that you'll need to use named templates, but on occasion it can help to minimise duplication of markup.

### Nota 2: Usando a opção “foreach” com um template

Se você quiser o equivalente com um `foreach`, mas usando templates nomeados, você pode fazer de forma natural:

    <h2>Participants</h2>
    Here are the participants:
    <div data-bind="template: { name: 'person-template', foreach: people }"></div>

    <script type="text/html" id="person-template">
        <h3 data-bind="text: name"></h3>
        <p>Credits: <span data-bind="text: credits"></span></p>
    </script>

     function MyViewModel() {
         this.people = [
             { name: 'Franklin', credits: 250 },
             { name: 'Mario', credits: 5800 }
         ]
     }
     ko.applyBindings(new MyViewModel());

This gives the same result as embedding an anonymous template directly inside the element to which you use `foreach`, i.e.:

    <div data-bind="foreach: people">
        <h3 data-bind="text: name"></h3>
        <p>Credits: <span data-bind="text: credits"></span></p>
    </div>

### Nota 3: Usando “as” para dar um apelido para itens do “foreach”

When nesting `foreach` templates, it's often useful to refer to items at higher levels in the hierarchy. One way to do this is to refer to `$parent` or other [binding context](binding-context.html) variables in your bindings.

A simpler and more elegant option, however, is to use `as` to declare a name for your iteration variables. For example:

    <ul data-bind="template: { name: 'employeeTemplate',
                                      foreach: employees,
                                      as: 'employee' }"></ul>

Notice the string value `'employee'` associated with `as`. Now anywhere inside this `foreach` loop, bindings in your child templates will be able to refer to `employee` to access the employee object being rendered.

This is mainly useful if you have multiple levels of nested `foreach` blocks, because it gives you an unambiguous way to refer to any named item declared at a higher level in the hierarchy. Here's a complete example, showing how `season` can be referenced while rendering a `month`:

    <ul data-bind="template: { name: 'seasonTemplate', foreach: seasons, as: 'season' }"></ul>

    <script type="text/html" id="seasonTemplate">
        <li>
            <strong data-bind="text: name"></strong>
            <ul data-bind="template: { name: 'monthTemplate', foreach: months, as: 'month' }"></ul>
        </li>
    </script>

    <script type="text/html" id="monthTemplate">
        <li>
            <span data-bind="text: month"></span>
            is in
            <span data-bind="text: season.name"></span>
        </li>
    </script>

    <script>
        var viewModel = {
            seasons: ko.observableArray([
                { name: 'Spring', months: [ 'March', 'April', 'May' ] },
                { name: 'Summer', months: [ 'June', 'July', 'August' ] },
                { name: 'Autumn', months: [ 'September', 'October', 'November' ] },
                { name: 'Winter', months: [ 'December', 'January', 'February' ] }
            ])
        };
        ko.applyBindings(viewModel);
    </script>

Tip: Remember to pass a *string literal value* to as (e.g., `as: 'season'`, *not* `as: season`), because you are giving a name for a new variable, not reading the value of a variable that already exists.

### Nota 4: Usando “afterRender”, “afterAdd” e “beforeRemove”

Sometimes you might want to run custom post-processing logic on the DOM elements generated by your templates. For example, if you're using a JavaScript widgets library such as jQuery UI, you might want to intercept your templates' output so that you can run jQuery UI commands on it to transform some of the rendered elements into date pickers, sliders, or anything else.

Generally, the best way to perform such post-processing on DOM elements is to write a [custom binding](custom-bindings.html), but if you really just want to access the raw DOM elements emitted by a template, you can use `afterRender`.

Pass a function reference (either a function literal, or give the name of a function on your view model), and Knockout will invoke it immediately after rendering or re-rendering your template. If you're using `foreach`, Knockout will invoke your `afterRender` callback for each item added to your observable array. For example,

    <div data-bind='template: { name: "personTemplate",
                                data: myData,
                                afterRender: myPostProcessingLogic }'> </div>

... and define a corresponding function on your view model (i.e., the object that contains `myData`):

    viewModel.myPostProcessingLogic = function(elements) {
        // "elements" is an array of DOM nodes just rendered by the template
        // You can add custom post-processing logic here
    }

If you are using `foreach` and only want to be notified about elements that are specifically being added or are being removed, you can use `afterAdd` and `beforeRemove` instead. For details, see documentation for the [`foreach` binding](foreach-binding.html).

### Nota 5: Escolhendo dinamicamente qual template é usado

If you have multiple named templates, you can pass an observable for the `name` option. As the observable's value is updated, the element's contents will be re-rendered using the appropriate template. Alternatively, you can pass a callback function to determine which template to use. If you are using the `foreach` template mode, Knockout will evaluate the function for each item in your array, passing that item's value as the only argument. Otherwise, the function will be given the `data` option's value or fall back to providing your whole current model object.

Por exemplo,

    <ul data-bind='template: { name: displayMode,
                               foreach: employees }'> </ul>

    <script>
        var viewModel = {
            employees: ko.observableArray([
                { name: "Kari", active: ko.observable(true) },
                { name: "Brynn", active: ko.observable(false) },
                { name: "Nora", active: ko.observable(false) }
            ]),
            displayMode: function(employee) {
                // Initially "Kari" uses the "active" template, while the others use "inactive"
                return employee.active() ? "active" : "inactive";
            }
        };

        // ... then later ...
        viewModel.employees()[1].active(true); // Now "Brynn" is also rendered using the "active" template.
    </script>

If your function references observable values, then the binding will update whenever any of those values change.  This will cause the data to be re-rendered using the appropriate template.

If your function accepts a second parameter, then it will receive the entire [binding context](binding-context.html). You can then access `$parent` or any other [binding context](binding-context.html) variable when dynamically choosing a template. For example, you could amend the preceding code snippet as follows:

    displayMode: function(employee, bindingContext) {
        // Now return a template name string based on properties of employee or bindingContext
    }

### Nota 6: Usando jQuery.tmpl, um template engine externo string-based

Na grande maioria dos casos, o template nativo do Knockout, com `foreach`, `if`, `with` e outros fluxos de controle, será o que você precisa para construir uma sofisticada UI. Mas se você precisar integrar com bibliotecas de templates externos, tal como [Underscore template engine](http://documentcloud.github.com/underscore/#template) ou [jquery.tmpl](http://api.jquery.com/jquery.tmpl/), Knockout oferece maneiras de fazer isso.

Por padrão, Knockout vem com suporte para [jquery.tmpl](https://github.com/BorisMoore/jquery-tmpl). Para usar, você precisa referenciar as seguintes bibliotecas, nessa ordem:

    <!-- First jQuery -->     <script src="http://code.jquery.com/jquery-1.7.1.min.js"></script>
    <!-- Then jQuery.tmpl --> <script src="jquery.tmpl.js"></script>
    <!-- Then Knockout -->    <script src="knockout-x.y.z.js"></script>

Então, você pode usar a sintaxe jQuery.tmpl em seus templates. Por exemplo:

    <h1>People</h1>
    <div data-bind="template: 'peopleList'"></div>

    <script type="text/html" id="peopleList">
        {{'{{'}}each people}}
            <p>
                <b>${name}</b> is ${age} years old
            </p>
        {{'{{'}}/each}}
    </script>

    <script type="text/javascript">
        var viewModel = {
            people: ko.observableArray([
                { name: 'Rod', age: 123 },
                { name: 'Jane', age: 125 },
            ])
        }
        ko.applyBindings(viewModel);
    </script>

This works because `{{'{{'}}each ...}}` and `${ ... }` are jQuery.tmpl syntaxes. What's more, it's trivial to nest templates: because you can use data-bind attributes from inside a template, you can simply put a `data-bind="template: ..."` inside a template to render a nested one.

Please note that, as of December 2011, jQuery.tmpl is no longer under active development. We recommend the use of Knockout's native DOM-based templating (i.e., the `foreach`, `if`, `with`, etc. bindings) instead of jQuery.tmpl or any other string-based template engine.

### Nota 7: Usando o template engine Underscore.js

O [Underscore.js template engine](http://documentcloud.github.com/underscore/#template) por padrão usa delimitadores estilo ERB (`<%= ... %>`). Aqui está o template, como o exemplo anterior, com Undercore:

    <script type="text/html" id="peopleList">
        <% _.each(people(), function(person) { %>
            <li>
                <b><%= person.name %></b> is <%= person.age %> years old
            </li>
        <% }) %>
    </script>

Here's [a simple implementation of integrating Underscore templates with Knockout](http://jsfiddle.net/rniemeyer/NW5Vn/). The integration code is just 16 lines long, but it's enough to support Knockout `data-bind` attributes (and hence nested templates) and Knockout [binding context](binding-context.html) variables (`$parent`, `$root`, etc.).

If you're not a fan of the `<%= ... %>` delimiters, you can configure the Underscore template engine to use any other delimiter characters of your choice.

### Dependências

 * **Template Nativo** não necessita de nenhuma outra biblioteca, exceto a biblioteca Knockout
 * **Template string-based** works only once you've referenced a suitable template engine, such as jQuery.tmpl or the Underscore template engine.
