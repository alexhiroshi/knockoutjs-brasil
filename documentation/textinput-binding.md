---
layout: documentation
title: O binding "textInput"
---

### Propósito
O binding `textInput` vincula um text box (`<input>`) ou text area (`<textarea>`) com uma propriedade viewmodel, fornecendo autalizações two-way entre a propriedade viewmodel e o valor do elemento. Diferente do binding value, `textInput` fornece atualizações instantâneas do DOM para todos os tipos de entrada do usuário, inluindo autocomplete, drag-and-drop e eventos do clipboard.

### Example

    <p>Login name: <input data-bind="textInput: userName" /></p>
    <p>Password: <input type="password" data-bind="textInput: userPassword" /></p>

    ViewModel:
    <pre data-bind="text: ko.toJSON($root, null, 2)"></pre>

    <script>
        ko.applyBindings({
            userName: ko.observable(""),        // Initially blank
            userPassword: ko.observable("abc")  // Prepopulate
        });
    </script>

### Parâmetros

  * Parâmetro principal

    KO define o conteúdo texto do elemento para o seu valor de parâmetro. Qualquer outro valor anterior será substituído.

    Se esse parâmetro é um valor observable, o binding atualizará o valor do elemento sempre que o valor for trocado. Se o parâmetro não for observable, ele irá definir o valor do elemento apenas uma vez e não atualizará depois.

    Se você fornecer algo diferente de um número ou uma string (ex. você passar um objeto ou um array), o texto exibido será equivalente a `yourParameter.toString()` (que normalmente não é muito útil, por isso é melhor fornecer valores numéricos ou strings).

    Sempre que o usuário editar o valor no controle do formulário associado, KO atualizará a propriedade na sua view model. KO sempre tentará atualizar a sua view model quando o valor for modificado pelo usuário ou quaisquer eventos do DOM.
 
  * Parâmetros adicionais

     * Nenhum


### Nota 1: binding `textInput` vs `value`

Embora o [ binding `value`](value-binding.html) também possa executar two-way binding entre text boxes e propriedades viewmodel, você deve preferir textInput sempre que quiser atualizações imediatas. As principais diferenças são:

  * **Atualizações imediatas**

    `value`, por padrão, só atualiza o seu model quando o usuário tira o foco do text box. `textInput` atualiza seu model imediatamente em cada caractere digitado ou outro mecanismo de entradas de texto (como cortar ou arrastar um texto que não necessariamente mude o foco).

  * **Browser event quirks handling**

    Navegadores são muito inconsistentes nos eventos disparados em resposta a entradas de textos incomuns, como cutting, dragging ou aceitar sugestões de autocomplete. O binding `value`, mesmo com opções extras como `valueUpdate: afterkeydown` para obter atualizações sobre eventos particulares, não abrange todos os cenários de entrada de texto em todos os navegadores.

    O binding `textInput` é especificamente desenvolvido para lidar com uma ampla gama de peculiaridades dos navegadores para fornecer uma consistente e imediata atualização do model, até mesmo em resposta a métodos de entrada de textos incomuns.

Não tente usar o binding `value` e `textInput` juntos no mesmo elemento, você não vai conseguir nada de útil.

### Dependências

Nenhuma, exceto a biblioteca Knockout.

