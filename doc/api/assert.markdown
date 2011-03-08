## Assertions

Ce module est utilisé pour écrire des tests unitaires pour vos applications.
Vous pouvez y accéder avec `require('assert')`.

### assert.fail(actual, expected, message, operator)

Teste si `actual` est égal  à `expected` en utilisant l'`operator` donné

### assert.ok(value, [message])

Teste si la valeur `value` est `true`. Ceci est équivalent à
`assert.equal(true, value, message)`.

### assert.equal(actual, expected, [message])

Teste l'égalité superficielle avec l'opérateur `==`.

### assert.notEqual(actual, expected, [message])

Teste l'inégalité superficielle avec l'opérateur `!=`.

### assert.deepEqual(actual, expected, [message])

Teste l'égalité profonde.

### assert.notDeepEqual(actual, expected, [message])

Teste l'inégalité profonde.

### assert.strictEqual(actual, expected, [message])

Teste l'égalité stricte avec l'opérateur `===`.

### assert.notStrictEqual(actual, expected, [message])

Teste l'inégalité stricte avec l'opérateur `!==`.

### assert.throws(block, [error], [message])

S'attend à ce que `block` provoque une erreur. `error` peut être un 
constructeur, une epression régulière ou une fonction de validation

Validation de `instanceof` en utilisant un constructeur :

    assert.throws(
      function() {
        throw new Error("Lauvaise valeur");
      },
      Error
    );

Validation du message d'erreur en utilisant une regex :

    assert.throws(
      function() {
        throw new Error("Mauvaise valeur");
      },
      /valeur/
    );

Validation d'erreur personnalisée :

    assert.throws(
      function() {
        throw new Error("Mauvaise valeur");
      },
      function(err) {
        if ( (err instanceof Error) && /valeur/.test(err) ) {
          return true;
        }
      },
      "erreur inatendue"
    );

### assert.doesNotThrow(block, [error], [message])

S'attend à ce que `block` ne provoque pas d'erreur. voir `assert.throws`
pour plus de détails.

### assert.ifError(value)

Teste si `value` n'est pas `false`, lance un exception si égale à `true`.
Utile lorsque l'on teste le premier argument `erreur` dans les fonctions
de rappel.
