---
layout: documentation
title: O binding "hasFocus"
---

### Propósito
O binding `hasFocus` liga o estado de um elemento DOM com uma propriedade viewmodel. Isso é um two-way binding, assim:

 * Se você definir a propriedade viewmodel para `true` ou `false`, o elemento associado ficará com ou sem foco.
 * Se o usuário der ou retirar o foco manualmente no elemento associado, a propriedade viewmodel será definida para `true` ou `false` de acordo com a ação.

This is useful if you're building sophisticated forms in which editable elements appear dynamically, and you would like to control where the user should start typing, or respond to the location of the caret.

### Exemplo 1: O básico
Esse é um exemplo simplesmente exibe uma mensagem se o textbox receber o foco e usa um botão para mostrar que você pode adicionar o foco programaticamente.

{% capture live_example_view %}
<input data-bind="hasFocus: isSelected" />
<button data-bind="click: setIsSelected">Focus programmatically</button>
<span data-bind="visible: isSelected">The textbox has focus</span>
{% endcapture %}

{% capture live_example_viewmodel %}
var viewModel = {
    isSelected: ko.observable(false),
    setIsSelected: function() { this.isSelected(true) }
};
ko.applyBindings(viewModel);
{% endcapture %}

{% include live-example-minimal.html %}


### Exemplo 2: Clique para editar

Because the `hasFocus` binding works in both directions (setting the associated value focuses or unfocuses the element; focusing or unfocusing the element sets the associated value), it's a convenient way to toggle an "edit" mode. In this example, the UI displays either a `<span>` or an `<input>` element depending on the model's `editing` property. Unfocusing the `<input>` element sets `editing` to `false`, so the UI switches out of "edit" mode.

{% capture live_example_id %}click_to_edit{% endcapture %}
{% capture live_example_view %}
<p>
	Name: 
	<b data-bind="visible: !editing(), text: name, click: edit">&nbsp;</b>
	<input data-bind="visible: editing, value: name, hasFocus: editing" />
</p>
<p><em>Click the name to edit it; click elsewhere to apply changes.</em></p>
{% endcapture %}

{% capture live_example_viewmodel %}
function PersonViewModel(name) {
    // Data
    this.name = ko.observable(name);
    this.editing = ko.observable(false);
        
    // Behaviors
    this.edit = function() { this.editing(true) }
}

ko.applyBindings(new PersonViewModel("Bert Bertington"));
{% endcapture %}

{% include live-example-minimal.html %}


### Parameters

  * Parâmetro principal
 
    Passar `true` (ou valores que representem true) para focar o elemento associado. Caso contrário, o elemento associado será desfocado.

    Quando o usuário foca o desfoca o elemento manualmente, o seu valor será definido para `true` ou `false`.

    Se o valor que você fornecer é observable, o binding `hasFocus` irea atualizar o estado de foco do elemento sempre que esse valor for alterado.
     
  * Parâmetros adicionais

      * Nenhum

### Dependências

Nenhuma, exceto a biblioteca Knockout.