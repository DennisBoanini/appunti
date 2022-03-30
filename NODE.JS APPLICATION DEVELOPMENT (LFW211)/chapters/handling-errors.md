# HANDLING ERRORS

Possiamo affermare che gli errori possono essere divisi in due grandi gruppi:

- Gli errori sulle operazioni: sono errori che si hanno durante l'esecuzione di task
- Gli errori commessi dagli sviluppatori: sono errori commessi dallo sviluppatore che ha sviluppato la feature

Tipicamente un errore di input viene gestito con la parola chiave throw, come nell'esempio seguente

```js
function doTask (amount) {
  if (typeof amount !== 'number') throw new Error('amount must be a number')
  return amount / 2
}
```

Al crash del programma solitamente viene stampato uno stack trace che ha origine dall'oggetto Error che abbiamo creato facendo il throw. Solitamente è sempre raccomandabile sollevare errori utilizzando Error, ma sarebbe anche possibile farlo senza Error, in questo modo 

```js
function doTask (amount) {
  if (typeof amount !== 'number') throw new Error('amount must be a number')
  // THE FOLLOWING IS NOT RECOMMENDED:
  if (amount <= 0) throw 'amount must be greater than zero'
  return amount / 2
}

doTask(-1)
```

e in questo caso non verrà stampato nessuno stack trace dato che non abbiamo utilizzato nessun oggetto Error, ma verrà stampato il seguente output

![](images/no-stack.png)

Ci sono altre sei classe per la gestione degli errori che ereditano direttamente da Error e sono:

- EvalError
- SyntaxError
- RangeError
- ReferenceError
- TypeError
- URIError

Vediamo un piccolo esempio di utilizzo

```js
function doTask (amount) {
  if (typeof amount !== 'number') throw new TypeError('amount must be a number')
  if (amount <= 0) throw new RangeError('amount must be greater than zero')
  return amount / 2
}

// L'esecuzione produce il seguente output
> doTask('ciao')
Uncaught TypeError: amount must be a number
at doTask (REPL5:2:41)

> doTask(-10)
Uncaught RangeError: amount must be greater than zero
at doTask (REPL5:3:26)

```

Tutti gli errori visti fino ad adesso non sono assolutamente in grado di gestire tutti i possibili errori che si possono avere in un'applicazione, ci deve essere quindi un modo per crearne di nuovi (di errori). Due di questi modi sono estendere dalla classe Error e/o utilizzare la proprietà code di Error.

Aggiungiamo una validazione al metodo doTask tale che l'argomento deve necessariamente essere un numero pari.

```js
function doTask (amount) {
  if (typeof amount !== 'number') throw new TypeError('amount must be a number')
  if (amount <= 0) throw new RangeError('amount must be greater than zero')
  if (amount % 2) {
    const err = Error('amount must be even')
    err.code = 'ERR_MUST_BE_EVEN'
    throw err
  }
  return amount / 2
}

doTask(3)

// Risultato
> doTask(3)
Uncaught Error: amount must be even
at doTask (REPL10:5:17) {
    code: 'ERR_MUST_BE_EVEN'
}
```

Creiamo invece una classe errore custom per gestire questo errore

```js
class OddError extends Error {
  constructor (varName = '') {
    super(varName + ' must be even')
  }
  get name () { return 'OddError' }
}

function doTask (amount) {
    if (typeof amount !== 'number') throw new TypeError('amount must be a number')
    if (amount <= 0) throw new RangeError('amount must be greater than zero')
    if (amount % 2) throw new OddError('amount')
    return amount / 2
}

doTask(3)
```

Quindi eseguendo il codice sopra otteniamo 

```bash
$ node example.js
C:\Users\dennis\Desktop\corsi\NodeJS\example.js:11
  if (amount % 2) throw new OddError('amount')
                  ^
OddError: amount must be even
    at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:11:25)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:15:1)
←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)←[39m
←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m
←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)←[39m
←[90m    at node:internal/main/run_main_module:17:47←[39m

```

Da notare che la console ha stampato la nostra classe di errore custom OddError.

Volendo possiamo utilizzare anche la classe di errore custom e la proprietà code insieme, come nel seguente esempio

```js
class OddError extends Error {
    constructor (varName = '') {
        super(varName + ' must be even')
        this.code = 'ERR_MUST_BE_EVEN'
    }

    get name () {
        return 'OddError [' + this.code + ']'
    }
}
```

ed ecco il risultato di esecuzione

```bash
$ node example.js
C:\Users\dennis\Desktop\corsi\NodeJS\example.js:15
  if (amount % 2) throw new OddError('amount')
                  ^
OddError [ERR_MUST_BE_EVEN]: amount must be even
    at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:15:25)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:19:1)
←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)←[39m
←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m
←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)←[39m
←[90m    at node:internal/main/run_main_module:17:47←[39m {
  code: ←[32m'ERR_MUST_BE_EVEN'←[39m
```

Un errore sollevato in una funzione sincrona può essere catturato con un blocco try/catch

```js
try {
  const result = doTask(3)
  console.log('result', result)
} catch (err) {
  console.error('Error caught: ', err)
}

// Risultato
$ node example.js
Error caught:  OddError [ERR_MUST_BE_EVEN]: amount must be even
at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:15:25)
at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:20:18)
at Module._compile (node:internal/modules/cjs/loader:1101:14)
at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
at Module.load (node:internal/modules/cjs/loader:981:32)
at Function.Module._load (node:internal/modules/cjs/loader:822:12)
at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
at node:internal/main/run_main_module:17:47 {
    code: 'ERR_MUST_BE_EVEN'
}
```

Se l'input alla funzione fosse corretto 

```js
try {
    const result = doTask(4)
    console.log('result', result)
} catch (err) {
    console.error('Error caught: ', err)
}
```

avremmo questo output

```js
$ node example.js
result 2
```

Oltre che loggare semplicemente l'errore possiamo anche determinare il tipo di errore sollevato. Guardiamo ad esempio il seguente codice

```js
try {
  const result = doTask(4)
  console.log('result', result)
} catch (err) {
  if (err instanceof TypeError) {
    console.error('wrong type')
  } else if (err instanceof RangeError) {
    console.error('out of range')
  } else if (err instanceof OddError) {
    console.error('cannot be odd')
  } else {
    console.error('Unknown error', err)
  }
}
```

Nel codice sopra si può vedere come nel blocco catch andiamo a controllare il tipo di errore che è stato sollevato andando a vedere l'instanceof. Se proviamo ad eseguire il codice sopra con i seguenti input

- doTask(3)
- doTask('invalid input')
- doTask(-1)

otteniamo i seguenti risultati

```bash
$ node example.js
cannot be odd

$ node example.js
wrong type

$ node example.js
out of range
```

Fare il check dell'instaceof del tipo di errore può essere debole. Consideriamo il seguente codice

```js
try {
  const result = doTask(4)
  result()
  console.log('result', result)
} catch (err) {
  if (err instanceof TypeError) {
    console.error('wrong type')
  } else if (err instanceof RangeError) {
    console.error('out of range')
  } else if (err.code === 'ERR_MUST_BE_EVEN') {
    console.error('cannot be odd')
  } else {
    console.error('Unknown error', err)
  }
}

// Output
$ node example.js
wrong type
```

Nel codice sopra sembra che l'errore sia sollevato da doTask ma in realtà non è così visto che stiamo cercando di eseguire una funzione che non è una funzione (result è un numero non una funzione).

Per migliorare la situazione possiamo utilizzare quello che è il duck typing di JavaScript (se sembra un'anatra e starnazza come un'anatra allora è un'anatra). Scriviamo questo codice

```js
function codify (err, code) {
  err.code = code
  return err
}

function doTask (amount) {
    if (typeof amount !== 'number') throw codify(
        new TypeError('amount must be a number'),
        'ERR_AMOUNT_MUST_BE_NUMBER'
    )
    if (amount <= 0) throw codify(
        new RangeError('amount must be greater than zero'),
        'ERR_AMOUNT_MUST_EXCEED_ZERO'
    )
    if (amount % 2) throw new OddError('amount')
    return amount/2
}

try {
    const result = doTask(4)
    result()
    console.log('result', result)
} catch (err) {
    if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
        console.error('wrong type')
    } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
        console.error('out of range')
    } else if (err.code === 'ERR_MUST_BE_EVEN') {
        console.error('cannot be odd')
    } else {
        console.error('Unknown error', err)
    }
}

// Risultato
$ node example.js
Unknown error TypeError: result is not a function
at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:32:5)
at Module._compile (node:internal/modules/cjs/loader:1101:14)
at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
at Module.load (node:internal/modules/cjs/loader:981:32)
at Function.Module._load (node:internal/modules/cjs/loader:822:12)
at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
```

Molto più comprensibile adesso qual è il tipo di errore, no?

I try/catch non catchano gli errori sollevati in una callback che verrà chiamamta in qualche momento del futuro

```js
// WARNING: NEVER DO THIS:
try {
  setTimeout(() => {
    const result = doTask(3)
    console.log('result', result)
  }, 100)
} catch (err) {
  if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
    console.error('wrong type')
  } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
    console.error('out of range')
  } else if (err.code === 'ERR_MUST_BE_EVEN') {
    console.error('cannot be odd')
  } else {
    console.error('Unknown error', err)
  }
}

// Risultato
$ node example.js
C:\Users\dennis\Desktop\corsi\NodeJS\example.js:26
if (amount % 2) throw new OddError('amount')
^

OddError [ERR_MUST_BE_EVEN]: amount must be even
at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:26:27)
at Timeout._onTimeout (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:33:20)
←[90m    at listOnTimeout (node:internal/timers:557:17)←[39m
←[90m    at processTimers (node:internal/timers:500:7)←[39m {
    code: ←[32m'ERR_MUST_BE_EVEN'←[39m
}
```

Come si vede dal risultato l'errore stampato è quello lanciato dalla funzione doTask e non quello presente nel blocco catch, e questo proprio perché la funzione doTask viene chiamata in qualche momento del futuro (100ms in questo caso) e quindi risulta essere in un ciclo di eventi successivo.

Un fix veloce consiste nel muovere tutto il blocco try/catch all'interno della callback.

```js
setTimeout(() => {
  try {
    const result = doTask(3)
    console.log('result', result)
  } catch (err) {
    if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
      console.error('wrong type')
    } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
      console.error('out of range')
    } else if (err.code === 'ERR_MUST_BE_EVEN') {
      console.error('cannot be odd')
    } else {
      console.error('Unknown error', err)
    }
  }
}, 100)
```

Un errore sollevato in codice sincrono si dice essere un'eccezione, mentre una promise che termina in uno stato di reject rappresenta un errore in contesto asincrono. Trasformiamo il doTask in modo che sia asincrono

```js
function doTask (amount) {
  return new Promise((resolve, reject) => {
    if (typeof amount !== 'number') {
      reject(new TypeError('amount must be a number'))
      return
    }
    if (amount <= 0) {
      reject(new RangeError('amount must be greater than zero'))
      return
    }
    if (amount % 2) {
      reject(new OddError('amount'))
      return
    }
    resolve(amount/2)
  })
}

doTask(3)

// Risultato
$ node example.js
C:\Users\dennis\Desktop\corsi\NodeJS\example.js:28
reject(new OddError('amount'))
^

OddError [ERR_MUST_BE_EVEN]: amount must be even
at C:\Users\dennis\Desktop\corsi\NodeJS\example.js:28:14
at new Promise (<anonymous>)
    at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:18:10)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:35:1)
        ←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
        ←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)←[39m
        ←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
        ←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m
        ←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)←[39m
        ←[90m    at node:internal/main/run_main_module:17:47←[39m {
            code: ←[32m'ERR_MUST_BE_EVEN'←[39m
        }
```

Come si vede dal codice sopra la sua esecuzione termina in uno stato di errore di reject. La reject non viene catturata in quanto, con le promise, bisogna utilizzare il catch (come spiegato più sopra nel capitolo delle promise). Aggiungiamo quindi il catch al nostro codice.

```js
doTask(3)
  .then((result) => {
    console.log('result', result)
  })
  .catch((err) => {
    if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
      console.error('wrong type')
    } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
      console.error('out of range')
    } else if (err.code === 'ERR_MUST_BE_EVEN') {
      console.error('cannot be odd')
    } else {
      console.error('Unknown error', err)
    }

  })

// Risultato
$ node example.js
cannot be odd
```

Col codice sopra l'esecuzione è del tutto simile alla sua versione sincrona. Se non ci sono errori il catch non viene eseguito ma viene eseguito il then stampando, in questo caso, il risultato.

async/await supporta anche il try/catch quindi lo possiamo benissimo utilizzare al posto del .then.catch. Vediamo un esempio di codice

```js
async function run () {
  try {
    const result = await doTask(3)
    console.log('result', result)
  } catch (err) {
    if (err instanceof TypeError) {
      console.error('wrong type')
    } else if (err instanceof RangeError) {
      console.error('out of range')
    } else if (err.code === 'ERR_MUST_BE_EVEN') {
      console.error('cannot be odd')
    } else {
      console.error('Unknown error', err)
    }
  }
}

run()

// Risultato 
$ node example.js
cannot be odd
```

Utilizzare una funzione async col try/catch è zucchero sintattico in quanto il blocco catch nella funzione async run è l'equivalente del catch dello snippet di codice precedente (quello dopo il then). Una funziona async ritorna sempre una promise che si risolve col risultato a meno che non incontra un throw all'interno della funzione stessa e in questo caso la promise passa allo stato di reject. Quindi la funzione doTask la possiamo semplificare parecchio scrivendola in questo modo

```js
async function doTask (amount) {
  if (typeof amount !== 'number') throw new TypeError('amount must be a number')
  if (amount <= 0) throw new RangeError('amount must be greater than zero')
  if (amount % 2) throw new OddError('amount')
  return amount/2
}
```

Con propagazione di errori si intende che, invece che gestirli facendo il throw, si demanda al chiamante la responsabilità di gestire l'errore.

```js
class OddError extends Error {
  constructor (varName = '') {
    super(varName + ' must be even')
    this.code = 'ERR_MUST_BE_EVEN'
  }
  get name () {
    return 'OddError [' + this.code + ']'
  }
}

function codify (err, code) {
  err.code = code
  return err
}

async function doTask (amount) {
  if (typeof amount !== 'number') throw codify(
    new TypeError('amount must be a number'),
    'ERR_AMOUNT_MUST_BE_NUMBER'
  )
  if (amount <= 0) throw codify(
    new RangeError('amount must be greater than zero'),
    'ERR_AMOUNT_MUST_EXCEED_ZERO'
  )
  if (amount % 2) throw new OddError('amount')
  throw Error('some other error')
  return amount/2
}

async function run () {
  try {
    const result = await doTask(4)
    console.log('result', result)
  } catch (err) {
    if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
      throw Error('wrong type')
    } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
      throw Error('out of range')
    } else if (err.code === 'ERR_MUST_BE_EVEN') {
      throw Error('cannot be odd')
    } else {
      throw err
    }
  }
}
run().catch((err) => { console.error('Error caught', err) })
```

(da notare che il codice di esempio solleva l'errore anche con input corretto, è solo un esempio!). Dato che l'errore non corrisponde a nessuno di quelli conosciuti, questo viene propagato verso l'alto anziché essere gestito come gli errori conosciuti.

Il risultato del codice precedente è

```js
$ node example.js
Error caught Error: some other error
    at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:26:9)
    at run (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:32:26)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:46:1)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47

```

La propagazione degli errori nel codice sincrono funziona esattamente allo stesso modo

```js
function doTask (amount) {
    if (typeof amount !== 'number') throw codify(
        new TypeError('amount must be a number'),
        'ERR_AMOUNT_MUST_BE_NUMBER'
    )
    if (amount <= 0) throw codify(
        new RangeError('amount must be greater than zero'),
        'ERR_AMOUNT_MUST_EXCEED_ZERO'
    )
    if (amount % 2) throw new OddError('amount')
    throw Error('some other error')
    return amount/2
}

function run () {
    try {
        const result = doTask('not a valid input')
        console.log('result', result)
    } catch (err) {
        if (err.code === 'ERR_AMOUNT_MUST_BE_NUMBER') {
            throw Error('wrong type')
        } else if (err.code === 'ERRO_AMOUNT_MUST_EXCEED_ZERO') {
            throw Error('out of range')
        } else if (err.code === 'ERR_MUST_BE_EVEN') {
            throw Error('cannot be odd')
        } else {
            throw err
        }
    }
}

try { run() } catch (err) { console.error('Error caught', err) }
```

Con risultato

```bash
$ node example.js
Error caught ReferenceError: codify is not defined
    at doTask (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:2:35)
    at run (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:17:20)
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example.js:32:7)
    at Module._compile (node:internal/modules/cjs/loader:1101:14)
    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153:10)
    at Module.load (node:internal/modules/cjs/loader:981:32)
    at Function.Module._load (node:internal/modules/cjs/loader:822:12)
    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/run_main:81:12)
    at node:internal/main/run_main_module:17:47

```