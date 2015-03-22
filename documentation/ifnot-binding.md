---
layout: documentation
title: O binding "ifnot"
---

### Propósito
O binding `ifnot` é exatamente como o [binding if](if-binding.html), exceto que ele inverte o resultado de qualquer expressão que você passar. Para mais detalhes, veja a documentação do [binding if](if-binding.html).

### Nota: “ifnot” é o mesmo que um “if” negado.

O seguinte código:

    <div data-bind="ifnot: someProperty">...</div>

... é o mesmo que:

    <div data-bind="if: !someProperty()">...</div>

... assumindo que `someProperty` é observable, você precisa chamar como uma função para obter o valor atual.

A única razão para usar um `ifnot` em vez de negar um `if`, é apenas por questão de gosto: muitos desenvolvedores acham mais bonito.