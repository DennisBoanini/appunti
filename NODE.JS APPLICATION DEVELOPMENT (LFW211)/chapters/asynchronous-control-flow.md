# ASYNCHRONOUS CONTROL FLOW

Una callback è una funzione che viene chiamata in un qualche punto del futuro una volta che il task ha finito la sua esecuzione. Prima dell'introduzione di async/await le callback erano l'unico modo per gestire il flusso asincrono.

Vediamo come funzionano le callback partendo da questo semplice codice

```js
const { readFile } = require('fs')
readFile(__filename, (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(contents.toString())
})
```

Lo script precedente, se eseguito, non fa altro che leggere se stesso e stamparsi in console. Il secondo argomento della funzione readFile è quella che viene chiamata callback e non è altro che una funzione che prende due argomenti che sono err e contents (err è sempre il primo argomento per convenzioni di Node). Quindi se readFile viene eseguita con errore allora la callback avrà il primo argomento err valorizzato se invece non ci sono errori allora err sarà null e sarà valorizzato contents. Il tempo di completamento, in questo caso, dipende dalla grandezza dei file letti. Supponiamo di avere più o meno il solito codice che però legge tre file diversi di diverse dimensioni

```js
const { readFile } = require('fs')
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

const print = (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(contents.toString())
}
readFile(bigFile, print)
readFile(mediumFile, print)
readFile(smallFile, print)
```

Anche se readFile(bigFile, print) è eseguita per prima, il suo contenuto verrà stampato per ultimo in quando il file sarà di dimensioni maggiori di smallFile e quindi il tempo di esecuzione sarà più lungo. Quindi l'ordine di stampa del risultato sarà sicuramente smallFile, mediumFile e bigFile. E questo è uno dei modi con cui si ottiene l'esecuzione parallela di espressioni in NodeJS.

Ma se volessimo, per un qualche motivo, stampare prima bigFile poi mediumFile e infine smallFile? Beh allora dobbiamo annidare le callback una dentro l'altra.

```js
const { readFile } = require('fs')
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)
const print = (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(contents.toString())
}
readFile(bigFile, (err, contents) => {
  print(err, contents)
  readFile(mediumFile, (err, contents) => {
    print(err, contents)
    readFile(smallFile, print)
  })
})
```

Infine, con un'altra piccola modifica, possiamo fare in modo che la stampa sia fatta solamente quando tutti i file sono stati letti

```js
const { readFile } = require('fs')
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)
const data = []
const print = (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(contents.toString())
}
readFile(bigFile, (err, contents) => {
  if (err) print(err)
  else data.push(contents)
  readFile(mediumFile, (err, contents) => {
    if (err) print(err)
    else data.push(contents)
    readFile(smallFile, (err, contents) => {
      if (err) print(err)
      else data.push(contents)
      print(null, Buffer.concat(data))
    })
  })
})
```

Supponiamo adesso di avere, anziché tre file, un array di file che vogliamo leggere in ordine

```js
const { readFile } = require('fs')
const files = Array.from(Array(3)).fill(__filename)
const data = []
const print = (err, contents) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(contents.toString())
}
let count = files.length
let index = 0
const read = (file) => {
  readFile(file, (err, contents) => {
    index += 1
    if (err) print(err)
    else data.push(contents)
    if (index < count) read(files[index])
    else print(null, Buffer.concat(data))
  })
}

read(files[index])
```

L'esecuzione di istruzioni in serie basate su callback può diventare facilmente complicata e illeggibile, un consiglio è quello di utilizzare una qualche libreria esterna come ad esempio fastseries

```js
const { readFile } = require('fs')
const series = require('fastseries')()
const files = Array.from(Array(3)).fill(__filename)

const print = (err, data) => {
  if (err) {
    console.error(err)
    return
  }
  console.log(Buffer.concat(data).toString())
}

const readers = files.map((file) => {
    return (_, cb) => {
        readFile(file, (err, contents) => {
            if (err) {
                print(err)
                cb(null, Buffer.alloc(0))
            } else cb(null, contents)
        })
    }
})

series(null, readers, null, print)
```

Una Promise è un oggetto che rappresenta un'operazione asincrona. Tale operazione può essere sospesa o risolta e se risolta può essere risolta correttamente o rifiutata (in inglese il concetto rende meglio "It's either pending or settled, and if it is settled it's either resolved or rejected", in italiano fa pena).

```js
// Approccio con callback
function myAsyncOperation (cb) {
    doSomethingAsynchronous((err, value) => { cb(err, value) })
}

myAsyncOperation(functionThatHandlesTheResult)

// Approccio con Promise
function myAsyncOperation () {
    return new Promise((resolve, reject) => {
        doSomethingAsynchronous((err, value) => {
            if (err) reject(err)
            else resolve(value)
        })
    })
}

const promise = myAsyncOperation()
// next up: do something with promise
```

I metodi per gestire il successo o il fallimento della promise sono then e catch, come si vede nell'esempio qua sotto

```js
const promise = myAsyncOperation()
promise
  .then((value) => { console.log(value) })
  .catch((err) => { console.error(err) })
```

I metodi then e catch ritornano a loro volta una promise, quindi possono essere concatenati gli uni con gli altri

```js
const { readFile } = require('fs').promises
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

const print = (contents) => {
  console.log(contents.toString())
}
readFile(bigFile)
  .then((contents) => {
    print(contents)
return readFile(mediumFile)
})
.then((contents) => {
print(contents)
return readFile(smallFile)
})
.then(print)
.catch(console.error)
```

Le keyword async/await vengono utilizzate per far sembrare il codice come se fosse sincrono. async è utilizzata prima della dichiarazione di una funzione `async function myFunction () { }` tutte le funzioni async ritornano una promise. await può essere utilizzato solamente all'interno di una funzione async. Praticamente mette in pausa l'esecuzione di una funzione asincrono fino a quando la promise non viene risolta.

```js
const { readFile } = require('fs').promises

async function run () {
    const contents = await readFile(__filename)
    console.log(contents.toString())
}

run().catch(console.error)
```

Nel codice sopra abbiamo creato una funzione asincrona di nome run utilizzando la parola chiave async. All'interno della funzione utilizziamo la parolina chiave await per "aspettare" che `readFile(__filename)` si risolva dandoci il risultato. L'esecuzione di tutta la funzione run è in uno stato di pausa fino a quando la promise `readFile(__filename)` non è risolta, ovvero fino a quando l'espressione non ha prodotto un risultato.

Una funzione async viene invocata come una qualsiasi altra funzione e dato che essa ritornerà sempre una promise abbiamo messo anche un catch per catturare un eventuale stato di errore della promise (`run().catch(console.error)`).

Async/Await rendono molto più pulito l'approccio dell'esecuzione in serie di espressioni. Vediamo l'esempio dell'esecuzione in serie di file di diversi formati utilizzando async/Await

```js
const { readFile } = require('fs').promises
const print = (contents) => {
  console.log(contents.toString())
}
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

async function run () {
  print(await readFile(bigFile))
  print(await readFile(mediumFile))
  print(await readFile(smallFile))
}

run().catch(console.error)
```

Se abbiamo un ordine di preferenza per il completamento delle operazioni non bisogna fare altro che "attendere" (await) in ordine le operazioni (come si può vedere nel corpo del metodo run (se volessimo un altro ordine è sufficiente cambiare l'ordine delle operazioni)).

Anche concatenare i file in un array dopo che sono stati caricati è semplicissimo con async/await

```js
const { readFile } = require('fs').promises
const print = (contents) => {
  console.log(contents.toString())
}
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

async function run () {
  const data = [
    await readFile(bigFile),
    await readFile(mediumFile),
    await readFile(smallFile)
  ]
  print(Buffer.concat(data))
}

run().catch(console.error)
```

E se avessimo un array di files con una lunghezza sconosciuta?

```js
const { readFile } = require('fs').promises

const print = (contents) => {
  console.log(contents.toString())
}

const files = Array.from(Array(3)).fill(__filename)

async function run () {
  const data = []
  for (const file of files) {
    data.push(await readFile(file))
  }
  print(Buffer.concat(data))
}

run().catch(console.error)
```

Il codice sopra è OK per scenari in cui le operazioni devono essere chiamate con un certo ordine, ma per scenari in cui l'output deve essere ordinato ma l'ordine in cui le operazioni asincrone si risolvono è sconosciuto allora possiamo utilizzare `await Promise.all`

```js
const { readFile } = require('fs').promises
const files = Array.from(Array(3)).fill(__filename)
const print = (contents) => {
  console.log(contents.toString())
}

async function run () {
  const readers = files.map((file) => readFile(file))
  const data = await Promise.all(readers)
  print(Buffer.concat(data))
}

run().catch(console.error)
```

Nel codice sopra leggiamo tutti i file e li mettiamo in un array di readers che altro non è che un array di promise. Dopodiché con l'istruzione `await Promise.all(readers)` aspettiamo che tutte le promise siano risolte e le stampiamo a video. 

Promise.all va in uno stato di rifiuto se una qualsiasi promise va in errore. Per avere una certa tolleranza degli errori possiamo utilizzare Promise.allSettled

```js
const { readFile } = require('fs').promises
const files = [__filename, 'foo', __filename]
const print = (contents) => {
console.log(contents.toString())
}

async function run () {
const readers = files.map((file) => readFile(file))
const results = await Promise.allSettled(readers)

results
.filter(({status}) => status === 'rejected')
.forEach(({reason}) => console.error(reason))

const data = results
.filter(({status}) => status === 'fulfilled')
.map(({value}) => value)

print(Buffer.concat(data))
}

run().catch(console.error)
```

async/await è molto utile per il controllo seriale delle operazioni, il problema è che l'esecuzione parallela di async con l'aiuto di  Promise.all, Promise.allSettled, Promise.any o Promise.race può diventare molto confusionario e difficile da leggere.

Riprendiamo l'esempio di esecuzione parallela

```js
const { readFile } = require('fs')
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

const print = (err, contents) => {
if (err) {
console.error(err)
return
}
console.log(contents.toString())
}
readFile(bigFile, print)
readFile(mediumFile, print)
readFile(smallFile, print)
```

e trasformiamolo in esecuzione parallela facendo uso di async/await

```js
const { readFile } = require('fs').promises
const [ bigFile, mediumFile, smallFile ] = Array.from(Array(3)).fill(__filename)

const print = (contents) => {
console.log(contents.toString())
}

async function run () {
const big = readFile(bigFile)
const medium = readFile(mediumFile)
const small = readFile(smallFile)

big.then(print)
medium.then(print)
small.then(print)

await small
await medium
await big
}

run().catch(console.error)
```

Alcune volte si rende necessario interrompere alcune operazioni asincrone anche se sono già in esecuzione. La soluzione semplice è di non iniziare l'operazione fino a quando non è strettamente necessario, ma questa è una soluzione molto lenta. Un altro approccio è quella d'iniziare l'operazione e poi cancellarla non appena le condizioni di esecuzione cambiano. Per fare tutto ciò possiamo utilizzare AbortController e AbortSignal.

```js
const timeout = setTimeout(() => {
  console.log('will not be logged')
}, 1000)

setImmediate(() => { clearTimeout(timeout) })
```

Come si ottiene lo stesso risultato quando abbiamo a che fare con le Promise?

```js
import { setTimeout } from 'timers/promises'

const timeout = setTimeout(1000, 'will be logged')

setImmediate(() => {
  clearTimeout(timeout) // do not do this, it won't work
})

console.log(await timeout)
```

Nell'esempio precedente anziché utilizzare il setTimeout global abbiamo utilizzato il setTimeout esportato da timers/promises che non ha bisogno di una callback ma restituisce una promise che risolve dopo il tempo indicato come primo argomento. Se viene passato un secondo argomento (come nel nostro esempio) allora la promise si risolve con il valore del secondo argomento. Quindi la costante che abbiamo dichiarato timeout è una promise che viene passato a clearTimeout e dal momento che è una promise e non un timeout questa viene ignorata da clearTimeout.

Ecco che quindi entra in gioco AbortController e AbortSignal per cancellare le promise fatte.

```js
import { setTimeout } from 'timers/promises'

const ac = new AbortController()
const { signal } = ac
const timeout = setTimeout(1000, 'will NOT be logged', { signal })

setImmediate(() => {
    ac.abort()
})

try {
    console.log(await timeout)
} catch (err) {
    // ignore abort errors:
    if (err.code !== 'ABORT_ERR') throw err
}
```