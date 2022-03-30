# WRITING UNIT TESTS

Un'asserzione controlla il valore di una certa condizione e solleva un errore nel caso in cui questa non sia verificata. Le asserzioni sono fondamentali nei test unitari e di integrazione.

Il modulo `assert` esporta una funzione che solleva un errore AssertionError quando il valore passato è falsy.

```bash
$ node -p "assert(false)"
node:assert:400
    throw err;
    ^

AssertionError [ERR_ASSERTION]: false == true
    at [eval]:1:1
    at Script.runInThisContext (node:vm:129:12)
    at Object.runInThisContext (node:vm:305:38)
    at node:internal/process/execution:81:19
    at [eval]-wrapper:6:22
    at evalScript (node:internal/process/execution:80:60)
    at node:internal/main/eval_string:27:3 {
  generatedMessage: true,
  code: 'ERR_ASSERTION',
  actual: false,
  expected: true,
  operator: '=='
}
```

Se invece il valore passato ad `assert` è un truthy, la funzione non solleverà nessun errore

```bash
dennis@Phate MINGW64 ~
$ node -p "assert(true)"
undefined

dennis@Phate MINGW64 ~
$
```

Il modulo `assert` ha i seguenti metodi per le asserzioni

- assert.ok(val) che fa lo stesso mestiere di assert(val)
- assert.equal(val1, val2) che fa un'uguaglianza "leggera" (val1 == val2)
- assert.notEqual(val1, val2) fa val1 != val2
- assert.strictEqual(val1, val2) fa un'uguaglianza "pesante" (val1 === val2)
- assert.notStrictEqual(val1, val2)
- assert.deepEqual(obj1, obj2) fa come assert.equal(val1, val2) ma su tutti i valori dell'oggetto
- assert.notDeepEqual(obj1, obj2)
- assert.deepStrictEqual(obj1, obj2)
- assert.notDeepStrictEqual(obj1, obj2)
- assert.throws(sunction) controlla se effettivamente la funzione solleva un'eccezione
- assert.doesNotThrow(function) controlla che la funzione non sollevi alcuna eccezione
- assert.rejects(promise|async function) controlla che la promise venga "rifiutata"
- assert.doesNotRejects(promise|async function)
- assert.ifError(err) controlla che l'errore sia falsy
- assert.match(string, regex)
- assert.doesNotMatch(string, regex)
- assert.fail() forza l'errore AssertionError

Generalmente quando facciamo il check di un valore vogliamo anche che il suo tipo sia quello che ci aspettiamo. Immaginiamo di voler testare una funzione di addizione

```js
const assert = require('assert')
const add = require('./get-add-from-somewhere.js')
assert.equal(add(2, 2), 4)
```

Il test precedente passerà in quanto, effettivamente, `add(2, 2)` restituisce il valore 4, ma il test passerebbe anche se l'output fosse '4' (4 a stringa) o anche se fosse un oggetto con questa forma `{ valueOf: () => 4 }` e questo succede perché assert.equal fa un confronto di tipo coercitivo (confronto debole, non strict), meglio scrivere un test come il seguente che controlla anche che il valore restituito sia di tipo number.

```js
const assert = require('assert')
const add = require('./get-add-from-somewhere.js')
const result = add(2, 2)
assert.equal(typeof result, 'number')
assert.equal(result, 4)
```

Un altro modo per raggiungere lo stesso scopo è utilizzare la versione strict

```js
const assert = require('assert')
const add = require('./get-add-from-somewhere.js')
assert.strictEqual(add(2, 2), 4)
```

in questo modo non solo controlliamo che il risultato sia 4, ma che sia effettivamente 4 numero e non stringa o altro.

Il codice sopra potrebbe anche essere riscritto, meglio, in questo modo

```js
const assert = require('assert')
const add = require('./get-add-from-somewhere.js')
assert.strict.equal(add(2, 2), 4)
```

Un errore di asserzione viene sollevato tutte le volte che una condizione non viene rispettata. Un altro modo ancora, utilizzando la libreria expect è il seguente

```js
const assert = require('assert')
const add = require('./get-add-from-somewhere.js')
assert.strict.equal(add(2, 2), 4)
```

se l'asserzione fallisce viene sollevato l'errore JestAssertionError 

Proviamo qualche asserzione sugli oggetti. Supponiamo di avere il seguente oggetto `const obj = { id: 1, name: { first: 'David', second: 'Clements' } }` e di volerlo comparare con un altro oggetto identico. Una semplice uguaglianza in questo caso non basta

```js
const assert = require('assert')
const obj = {
  id: 1,
  name: { first: 'David', second: 'Clements' }
}
// this assert will fail because they are different objects:
assert.equal(obj, {
  id: 1,
  name: { first: 'David', second: 'Clements' }
})

// Risultato
$ node example53.js
node:assert:123
  throw new AssertionError(obj);
  ^

AssertionError [ERR_ASSERTION]: {
  id: 1,
  name: {
    first: 'David',
    second: 'Clements'
  }
} == {
  id: 1,
  name: {
    first: 'David',
    second: 'Clements'
  }
}
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example53.js:7:8)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47 {
  generatedMessage: true,
  code: 'ERR_ASSERTION',
  actual: { id: 1, name: { first: 'David', second: 'Clements' } },
  expected: { id: 1, name: { first: 'David', second: 'Clements' } },
  operator: '=='
}
```

Come si vede il test fallisce, anche se gli oggetti sembrano identici in realtà si trovano in allocazioni di memoria diversa e quindi sono due oggetti diversi. Per dire se gli oggetti sono uguali, nel senso che hanno la stessa struttura, doppiamo utilizzare il deep equality check

```js
const assert = require('assert')
const obj = {
  id: 1,
  name: { first: 'David', second: 'Clements' }
}
assert.deepEqual(obj, {
  id: 1,
  name: { first: 'David', second: 'Clements' }
})

// Risultato
dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node example54.js

dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$
```

Come si vede non c'è output, segno che significa che il test è passato con successo.

La differenza fra deepEqual e deepStrictEqual è che il primo effettua una valutazione corcitiva, del tipo che 1 e '1' sono uguali, quindi 

```js
const assert = require('assert')
const obj = {
  id: 1,
  name: { first: 'David', second: 'Clements' }
}
// id is a string but this will pass because it's not strict
assert.deepEqual(obj, {
  id: '1',
  name: { first: 'David', second: 'Clements' }
})
```

passa in quando per il deepEqual 1 e '1' sono uguali e di conseguenza gli oggetti hanno le stesse proprietà e quindi sono uguali. Utilizziamo deepStrictEqual

```js
const assert = require('assert')
const obj = {
  id: 1,
  name: { first: 'David', second: 'Clements' }
}
// this will fail because id is a string instead of a number
assert.strict.deepEqual(obj, {
  id: '1',
  name: { first: 'David', second: 'Clements' }
})

// Risultato
$ node example56.js
node:assert:123
  throw new AssertionError(obj);
  ^

AssertionError [ERR_ASSERTION]: Expected values to be strictly deep-equal:
+ actual - expected ... Lines skipped

  {
+   id: 1,
-   id: '1',
    name: {
...
      second: 'Clements'
    }
  }
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example56.js:7:15)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47 {
  generatedMessage: true,
  code: 'ERR_ASSERTION',
  actual: { id: 1, name: { first: 'David', second: 'Clements' } },
  expected: { id: '1', name: { first: 'David', second: 'Clements' } },
  operator: 'deepStrictEqual'
}
```

deepStrictEqual si accorge, giustamente, che 1 e '1' sono due cose diverse e quindi il test fallisce.

Le assertions throws, ifError e rejects sono molto comode per testare la presenza di errore sia in codice sincrono che in codice basato su callback o promise.

```js
const assert = require('assert')
const add = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw Error('inputs must be numbers')
  }
  return a + b
}
assert.throws(() => add('5', '5'), Error('inputs must be numbers'))
assert.doesNotThrow(() => add(5, 5))
```

assert.ifError passa solamente se il valore dell'argomento è null o undefined

```js
const assert = require('assert')
const pseudoReq = (url, cb) => {
  setTimeout(() => {
    if (url === 'ht‌tp://error.com') cb(Error('network error'))
    else cb(null, Buffer.from('some data'))
  }, 300)
}

pseudoReq('ht‌tp://example.com', (err, data) => {
  assert.ifError(err)
})

pseudoReq('ht‌tp://error.com', (err, data) => {
  assert.deepStrictEqual(err, Error('network error'))
})
```

Tipicamente è spesso utilizzato con le callback e a cui viene passato err per controllare che sia null o undefined, quindi che effettivamente non ci sia un errore.

Vediamo anche un esempio di asserzione per codice basato su promise

```js
const assert = require('assert')
const { promisify } = require('util')
const timeout = promisify(setTimeout)
const pseudoReq = async (url) => {
  await timeout(300)
  if (url === 'ht‌tp://error.com') throw Error('network error')
  return Buffer.from('some data')
}
assert.doesNotReject(pseudoReq('ht‌tp://example.com'))
assert.rejects(pseudoReq('ht‌tp://error.com'), Error('network error'))`
```

Anche se sono uno strumento molto potente se una di questi assert dovesse fallire, verrà sollevato un errore di tipo AssertionError che causerà il crash del processo sul quale gira lo script o il programma Node. Sarebbe fantastico se riuscissimo a raggruppare tutte le asserzioni in un unico gruppo e l'output verrà stampato sul terminale e questo è ciò che fanno i test harnesses (non so tradurlo). Questo tipo di test possono essere raggruppati in due gruppi: librerie e framework.

Le librerie sono moduli caricati all'interno del codice e poi usati per raggruppare i test. Un esempio di libreria di test è tap.

Un framework è un qualcosa che fornisce uno o più moduli e delle variabili globali e richiedono un'altra CLI per essere eseguiti. Un esempio di framework di test è jest.

Supponiamo di avere tre file add.js, req.js e req-prom.js la cui implementazione è la seguente, in ordine

```js
// add.js
'use strict'
module.exports = (a, b) => {
  if (typeof a !== 'number' || typeof b !== 'number') {
    throw Error('inputs must be numbers')
  }
  return a + b
}

// req.js
'use strict'
module.exports = (url, cb) => {
  setTimeout(() => {
    if (url === 'ht‌tp://error.com') cb(Error('network error'))
    else cb(null, Buffer.from('some data'))
  }, 300)
}

// req-prom.js
'use strict'
const { promisify } = require('util')
const timeout = promisify(setTimeout)
module.exports = async (url) => {
  await timeout(300)
  if (url === 'ht‌tp://error.com') throw Error('network error')
  return Buffer.from('some data')
}
```

Come prima cosa diamo il comando npm init -y per creare il file package.json utile per poter installare tap e jest.

```bash
npm init -y // crea il package.json

npm install --save-dev tap // installa nelle dev dependencies la libreria tap per i test
```

Nella cartella test dove abbiamo i tre file sopra elencati creiamo un nuovo file di test per il file add, add.test.js

```js
const { test } = require('tap')
const add = require('../add')

// con la funzione test possiamo creare un gruppo di test che testano un comportamento comune
test('throw when inputs are not numbers', async ({ throws }) => {
  throws(() => add('5', '5'), Error('inputs must be numbers'))
  throws(() => add(5, '5'), Error('inputs must be numbers'))
  throws(() => add('5', 5), Error('inputs must be numbers'))
  throws(() => add({}, null), Error('inputs must be numbers'))
})

test('adds two numbers', async ({ equal }) => {
  equal(add(5, 5), 10)
  equal(add(-5, 5), 0)
})

// Risultato
$ node add.test.js
TAP version 13
# Subtest: throw when inputs are not numbers
    ok 1 - expected to throw: Error inputs must be numbers
    ok 2 - expected to throw: Error inputs must be numbers
    ok 3 - expected to throw: Error inputs must be numbers
    ok 4 - expected to throw: Error inputs must be numbers
    1..4
ok 1 - throw when inputs are not numbers # time=10.941ms

# Subtest: adds two numbers
    ok 1 - should be equal
    ok 2 - should be equal
    1..2
ok 2 - adds two numbers # time=1.501ms

1..2
# time=17.121ms

// Oppure
$ ./node_modules/.bin/tap add.test.js
​ PASS ​ add.test.js 6 OK 12.582ms
​


  🌈 SUMMARY RESULTS 🌈

​
Suites:   ​1 passed​, ​1 of 1 completed​
Asserts:  ​​​6 passed​, ​of 6​
​Time:​   ​1s​
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |     100 |      100 |     100 |     100 |
 add.js   |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|-------------------
```

Adesso facciamo i test per le API basate su callback, file req.js

```js
'use strict'
const { test } = require('tap')
const req = require('../req')

test('handles network errors', ({ strictDeepEqual, end }) => {
  req('ht‌tp://error.com', (err) => {
    strictDeepEqual(err, Error('network error'))
    end() // Visto che stavolta non stiamo utilizzando una funzione async, ma una callback, questa riga sta a significare che abbiamo finito l'esecuzione dei test nel gruppo
  })
})

test('responds with data', ({ ok, strictDeepEqual, ifError, end }) => {
  req('ht‌tp://example.com', (err, data) => {
    ifError(err)
    ok(Buffer.isBuffer(data))
    strictDeepEqual(data, Buffer.from('some data'))
    end()
  })
})

// Risultato
$ node req.test.js
TAP version 13
# Subtest: handles network errors
    ok 1 - should be equivalent strictly
    1..1
ok 1 - handles network errors # time=323.725ms

(node:12524) DeprecationWarning: strictDeepEqual() is deprecated, use strictSame() instead
(Use `node --trace-deprecation ...` to show where the warning was created)
# Subtest: responds with data
    ok 1 - should not error
    ok 2 - expect truthy value
    ok 3 - should be equivalent strictly
    1..3
ok 2 - responds with data # time=311.758ms

(node:12524) DeprecationWarning: ifError() is deprecated, use error() instead
1..2
# time=644.491ms

// Oppure
$ ./node_modules/.bin/tap req.test.js
req.test.js 2> (node:12188) DeprecationWarning: strictDeepEqual() is deprecated, use strictSame() instead
req.test.js 2> (Use `node --trace-deprecation ...` to show where the warning was created)
req.test.js 2> (node:12188) DeprecationWarning: ifError() is deprecated, use error() instead
​ PASS ​ req.test.js 4 OK 618.035ms
​


  🌈 SUMMARY RESULTS 🌈

​
Suites:   ​1 passed​, ​1 of 1 completed​
Asserts:  ​​​4 passed​, ​of 4​
​Time:​   ​1s​
----------|---------|----------|---------|---------|-------------------
File      | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
----------|---------|----------|---------|---------|-------------------
All files |     100 |      100 |     100 |     100 |
 req.js   |     100 |      100 |     100 |     100 |
----------|---------|----------|---------|---------|-------------------
```

Adesso testiamo il file req-prom.js. Creiamo req-prom.test.js

```js
'use strict'
const { test } = require('tap')
const req = require('../req-prom')

test('handles network errors', async ({ rejects }) => {
  await rejects(req('ht‌tp://error.com'), Error('network error'))
})

test('responds with data', async ({ ok, strictDeepEqual }) => {
  const data = await req('ht‌tp://example.com')
  ok(Buffer.isBuffer(data))
  strictDeepEqual(data, Buffer.from('some data'))
})

// Risultato
$ node req-prom.test.js
TAP version 13
# Subtest: handles network errors
    ok 1 - expect rejected Promise: Error network error
    1..1
ok 1 - handles network errors # time=322.239ms

# Subtest: responds with data
    ok 1 - expect truthy value
    ok 2 - should be equivalent strictly
    1..2
ok 2 - responds with data # time=308.217ms

(node:5432) DeprecationWarning: strictDeepEqual() is deprecated, use strictSame() instead
(Use `node --trace-deprecation ...` to show where the warning was created)
1..2
# time=638.727ms

// Oppure
$ ./node_modules/.bin/tap req-prom.test.js
req-prom.test.js 2> (node:24372) DeprecationWarning: strictDeepEqual() is deprecated, use strictSame() instead
req-prom.test.js 2> (Use `node --trace-deprecation ...` to show where the warning was created)
​ PASS ​ req-prom.test.js 3 OK 631.478ms
​


  🌈 SUMMARY RESULTS 🌈

​
Suites:   ​1 passed​, ​1 of 1 completed​
Asserts:  ​​​3 passed​, ​of 3​
​Time:​   ​1s​
-------------|---------|----------|---------|---------|-------------------
File         | % Stmts | % Branch | % Funcs | % Lines | Uncovered Line #s
-------------|---------|----------|---------|---------|-------------------
All files    |     100 |      100 |     100 |     100 |
 req-prom.js |     100 |      100 |     100 |     100 |
-------------|---------|----------|---------|---------|-------------------
```