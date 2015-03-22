---
layout: documentation
title: O binding "enable"
---

### Propósito
O binding `enable` faz com que o elemento DOM associado seja habilitado somente quando o valor do parâmetro for `true`. Isso é útil com elementos de formulário como `input`, `select`, e `textarea`.

### Exemplo
    <p>
        <input type='checkbox' data-bind="checked: hasCellphone" />
        I have a cellphone
    </p>
    <p>
        Your cellphone number:
        <input type='text' data-bind="value: cellphoneNumber, enable: hasCellphone" />
    </p>
    
    <script type="text/javascript">
        var viewModel = {
            hasCellphone : ko.observable(false),
            cellphoneNumber: ""
        };
    </script>

Nesse exemplo, o campo texto “Your cellphone number” inicialmente será desabilitado. Ele será habilitado somente quando o usuário marcar  o checkbox “I have a cellphone”.

### Parâmetros

  * Parâmetro principal
   
    Um valor que controla se o elemento DOM associado deve ser ativado.
   
    Valores não boolean são interpretados livremente como boolean. Por exemplo, `0` e `null` são tratados como `false`, enquanto `21` e objetos não `null` são tratados como `true`.
   
    Se o seu parâmetro faz referência a um valor observable, o binding irá atualizar o estado ativado/desativado sempre que o valor de observable for alterado. Se o parâmetro não faz referência  a um valor observable, ele só irá definir o estado uma vez.
      
  * Parâmetros adicionais 

     * Nenhum

### Nota: Usando expressões JavaScript

Você não está limitado a referenciar variáveis - você pode referenciar qualquer expressão para controlar o "ativamento" do elemento. Por exemplo:

    <button data-bind="enable: parseAreaCode(viewModel.cellphoneNumber()) != '555'">
        Do something
    </button>

### Dependências

Nenhuma, exceto a biblioteca Knockout.