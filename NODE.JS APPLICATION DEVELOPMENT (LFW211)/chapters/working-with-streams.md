# WORKING WITH STREAMS

Il modulo stream, che fa parte del core di Node, espone sei diversi tipi di costruttore, che sono: 

- Stream
- Readable
- Writable
- Duplex
- Transform
- PassThrough

Altre API del core di Node come process, net, http, fs, child_process espongono streams creati con i costruttori elencati sopra.

Il costruttore Stream è l'export di default del modulo stream e eredita da EventEmitter.

```bash
$ node -p "stream + ''"
function Stream(opts) {
  EE.call(this, opts);
}

dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node -p "stream.prototype"
EventEmitter { pipe: [Function (anonymous)] }

dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node -p "Object.getPrototypeOf(stream.prototype)"
{
  _events: undefined,
  _eventsCount: 0,
  _maxListeners: undefined,
  setMaxListeners: [Function: setMaxListeners],
  getMaxListeners: [Function: getMaxListeners],
  emit: [Function: emit],
  addListener: [Function: addListener],
  on: [Function: addListener],
  prependListener: [Function: prependListener],
  once: [Function: once],
  prependOnceListener: [Function: prependOnceListener],
  removeListener: [Function: removeListener],
  off: [Function: removeListener],
  removeAllListeners: [Function: removeAllListeners],
  listeners: [Function: listeners],
  rawListeners: [Function: rawListeners],
  listenerCount: [Function: listenerCount],
  eventNames: [Function: eventNames]
}
```

I vari eventi emessi dalle varie implementazioni dello Stream sono

- data
- end
- finish
- close
- error

Ci sono due tipi di stream

- stream binario
- stream di oggetti

e questo modo viene determinato dall'opzione objectMode passata quando lo stream viene inizializzato. Il suo valore di default è false, il che significa che lo stream sarà binario. Il modo binario legge e scrive solamente su Buffer. Nel modo a oggetti gli stream possono leggere e scrivere oggetti e primitive (strings, number, ...) tranne il null

Lo stream Readable serve per creare stream che sono leggibili. Questo tipo di stream possono essere utilizzati per leggere file, leggere i dati di una risposta HTTP o l'input utente fatto da terminale. Readable eredita da Stream che eredita da EventEmitter quindi possiamo affermare che uno stream Readable è un EventEmitter. Quando i dati sono disponibili lo stream readable emette un evento 'data'.

Vediamo di seguito un esempio di Readable stream

```js
'use strict'
const fs = require('fs')
const readable = fs.createReadStream(__filename)
readable.on('data', (data) => { console.log(' got data', data) })
readable.on('end', () => { console.log(' finished reading') })

// Risultato
$ node example3.js
got data <Buffer 27 75 73 65 20 73 74 72 69 63 74 27 0a 63 6f 6e 73 74 20 66 73 20 3d 20 72 65
71 75 69 72 65 28 27 66 73 27 29 0a 63 6f 6e 73 74 20 72 65 61 64 61 62 ... 166 more bytes>
finished reading
```

Il metodo createReadStream di fs è utilizzato per creare un'istanza di uno stream Readable. 

Gli stream readable solitamente sono connessi al layer I/O tramite il C-binding, ma è possibile creare uno stream anche utilizzando il costruttore di Readable.

```js
'use strict'
const { Readable } = require('stream')
const createReadStream = () => {
  const data = ['some', 'data', 'to', 'read']
  return new Readable({
    read () {
      if (data.length === 0) this.push(null)
      else this.push(data.shift())
    }
  })
}
const readable = createReadStream()
readable.on('data', (data) => { console.log('got data', data) })
readable.on('end', () => { console.log('finished reading') })

// Risultato
$ node example3.js
got data <Buffer 73 6f 6d 65>
got data <Buffer 64 61 74 61>
got data <Buffer 74 6f>
got data <Buffer 72 65 61 64>
finished reading
```

Per creare lo stream si utilizza la parola chiave new davanti a Readable. A Readable viene passato un oggetto che ha un metodo read. Questo metodo viene chiamato tutte le volte che gli internal di Node hanno bisogno di leggere dati dallo stream readable.

Quando creiamo un'istanza di Readable con il new, nell'oggetto di configurazione è possibile pure passare la codifica dello stream, come nel seguente esempio

```js
'use strict'
const { Readable } = require('stream')
const createReadStream = () => {
  const data = ['some', 'data', 'to', 'read']
  return new Readable({
    encoding: 'utf8',
    read () {
      if (data.length === 0) this.push(null)
      else this.push(data.shift())
    }
  })
}
const readable = createReadStream()
readable.on('data', (data) => { console.log('got data', data) })
readable.on('end', () => { console.log('finished reading') })

// Risultato
$ node example3.js
got data some
got data data
got data to
got data read
finished reading
```

Come si vede dall'esempio sopra, questa volta anziché ricevere un buffer, quando l'evento data viene emesso quello che si riceve è una stringa, proprio perché abbiamo detto al nostro stream di effettuare una codifica utf8. E' anche possibile creare uno stream readable impostando l'opzione objectMode a true, come nel seguente esempio

```js
'use strict'
const { Readable } = require('stream')
const createReadStream = () => {
  const data = ['some', 'data', 'to', 'read']
  return new Readable({
    objectMode: true,
    read () {
      if (data.length === 0) this.push(null)
      else this.push(data.pop())
    }
  })
}
const readable = createReadStream()
readable.on('data', (data) => { console.log('got data', data) })
readable.on('end', () => { console.log('finished reading') })

// Risultato
$ node example3.js
got data some
got data data
got data to
got data read
finished reading
```

A differenza di prima però, in questo caso, la stringa viene mandata direttamente as-is nello stream, mentre prima era convertita a buffer.

Se volessimo condensare un po' il codice di prima potremmo scrivere


```js
'use strict'
const { Readable } = require('stream')
const readable = Readable.from(['some', 'data', 'to', 'read'])
readable.on('data', (data) => { console.log('got data', data) })
readable.on('end', () => { console.log('finished reading') })
```

che produrrebbe lo stesso output. 

> NB: Readable.from imposta objectMode a true di default, al contrario del costruttore che andava esplicitato.

Il costruttore Writable crea uno stream scrivibile. Uno stream di questo tipo può essere usato per scrivere su un file, scrivere una risposta HTTP, scrivere sul terminale ecc... Anche Writable eredita da Stream che eredita da EventEmitter quindi anche Writable è un EventEmitter.

Vediamo un esempio di come possiamo mandare dati su uno stream Writable

```js
'use strict'
const fs = require('fs')
const writable = fs.createWriteStream('./out')
writable.on('finish', () => { console.log('finished writing') })
writable.write('A\n')
writable.write('B\n')
writable.write('C\n')
writable.end('nothing more to write')

// Risultato
$ node example4.js
finished writing

dennis@Phate MINGW64 ~/Desktop/corsi/NodeJS
$ node -p "fs.readFileSync('./out').toString()"
A
B
C
nothing more to write
```

Come si vede dall'esempio precedente il metodo write può essere chiamato più volte, mentre il metodo end viene chiamato solo una volta e anch'esso può scrivere un payload nello stream prima di terminare. Quando lo stream termina viene emesso un evento chiamato 'finish' .

Come per gli stream di tipo Readable, anche gli stream Writable sono maggiormente utilizzati per operazioni di tipo I/O, ma è anche possibile creare uno stream di scrittura programmaticamente, come si vede nel codice seguente.

```js
'use strict'
const { Writable } = require('stream')
const createWriteStream = (data) => {
  return new Writable({
    write (chunk, enc, next) {
      data.push(chunk)
      next()
    }
  })
}
const data = []
const writable = createWriteStream(data)
writable.on('finish', () => { console.log('finished writing', data) })
writable.write('A\n')
writable.write('B\n')
writable.write('C\n')
writable.end('nothing more to write')

// Risultato
$ node example5.js
finished writing [
    <Buffer 41 0a>,
    <Buffer 42 0a>,
    <Buffer 43 0a>,
    <Buffer 6e 6f 74 68 69 6e 67 20 6d 6f 72 65 20 74 6f 20 77 72 69 74 65>
]
```

Come per lo stream readable, anche lo stream writable viene creato col costruttore Writable e la parola chiave new. Questo costruttore prende come argomento un oggetto di opzioni che può avere una funzione write (readable aveva la funzione read) che prende tre argomenti e che sono chunk, enc e next. Il chunk rappresenta ogni pezzo di dato scritto nello stream, enc rappresenta la codifica e next è una callback che dice che siamo pronti per il successivo pezzo di dato (chunk). Anche qua, come per i readable stream, il valore di default della proprietà objectMode è false, quindi ogni stringa nello stream writable viene convertita a buffer prima di diventare un chunk passato alla funzione write. Per evitare che siano buffer, possiamo spengere la proprietà decodeStrings (mettendola a false), come nel seguente esempio

```js
const createWriteStream = (data) => {
  return new Writable({
    decodeStrings: false,
    write (chunk, enc, next) {
      data.push(chunk)
      next()
    }
  })
}
const data = []
const writable = createWriteStream(data)
writable.on('finish', () => { console.log('finished writing', data) })
writable.write('A\n')
writable.write('B\n')
writable.write('C\n')
writable.end('nothing more to write')

// Risultato
$ node example5.js
finished writing [ 'A\n', 'B\n', 'C\n', 'nothing more to write' ]
```

Il codice sopra permette solamente la scrittura di buffer o stringhe, ogni altro valore produce un errore

```js
'use strict'
const { Writable } = require('stream')
const createWriteStream = (data) => {
  return new Writable({
    decodeStrings: false,
    write (chunk, enc, next) {
      data.push(chunk)
      next()
    }
  })
}
const data = []
const writable = createWriteStream(data)
writable.on('finish', () => { console.log('finished writing', data) })
writable.write('A\n')
writable.write(1) // --> questa riga produce l'errore
writable.end('nothing more to write')

// Risultato
$ node example6.js
node:internal/streams/writable:312
throw new ERR_INVALID_ARG_TYPE(
^

TypeError [ERR_INVALID_ARG_TYPE]: The "chunk" argument must be of type string or
an instance of Buffer or Uint8Array. Received type number (1)
←[90m    at new NodeError (node:internal/errors:371:5)←[39m
←[90m    at _write (node:internal/streams/writable:312:13)←[39m
←[90m    at Writable.write (node:internal/streams/writable:334:10)←[39m
    at Object.<anonymous> (C:\Users\dennis\Desktop\corsi\NodeJS\example6.js:16:1
0)
←[90m    at Module._compile (node:internal/modules/cjs/loader:1101:14)←[39m
←[90m    at Object.Module._extensions..js (node:internal/modules/cjs/loader:1153
:10)←[39m
←[90m    at Module.load (node:internal/modules/cjs/loader:981:32)←[39m
←[90m    at Function.Module._load (node:internal/modules/cjs/loader:822:12)←[39m

←[90m    at Function.executeUserEntryPoint [as runMain] (node:internal/modules/r
un_main:81:12)←[39m
←[90m    at node:internal/main/run_main_module:17:47←[39m {
    code: ←[32m'ERR_INVALID_ARG_TYPE'←[39m
}
```

Per avere il supporto a tutti i valori JavaScript possiamo mettere a true il valore di objectMode

```js
'use strict'
const { Writable } = require('stream')
const createWriteStream = (data) => {
  return new Writable({
    objectMode: true,
    write (chunk, enc, next) {
      data.push(chunk)
      next()
    }
  })
}
const data = []
const writable = createWriteStream(data)
writable.on('finish', () => { console.log('finished writing', data) })
writable.write('A\n')
writable.write(1)
writable.end('nothing more to write')

// Risultato
$ node example7.js
finished writing [ 'A\n', 1, 'nothing more to write' ]
```

Duplex eredita da Readable ma implementa anche funzionalità di Writable. Con Duplex entrambi i metodi di read e write sono implementati ma non ci deve essere una relazione casuale fra di essi. Solo perché qualcosa è scritto in uno stream Duplex non significa necessariamente che si tradurrà in qualcosa che sarà letto dallo stream. Vediamo un esempio con una socket TCP.

```js
'use strict'
const net = require('net')
net.createServer((socket) => {
  const interval = setInterval(() => {
    socket.write('beat')
  }, 1000)
  socket.on('data', (data) => {
    socket.write(data.toString().toUpperCase())
  })
  socket.on('end', () => { clearInterval(interval) })
}).listen(3000)
```

La funzione net.createServer accetta un listener che viene eseguita ogni volta che un client contatta il server. Al listener viene passata un'istanza di Duplex che abbiamo chiamato socket. Ogni secondo viene chiamata la funzione socket.write che rappresenta la prima parte della parte writable dello stream. Lo stream è anche in ascolto di eventi data ed end e in questo caso abbiamo a che fare con la parte readable dello stream. Creiamo un piccolo client per interagire col piccolo server creato 

```js
'use strict'
const net = require('net')
const socket = net.connect(3000)

socket.on('data', (data) => {
  console.log('got data:', data.toString())
})
socket.write('hello')
setTimeout(() => {
  socket.write('all done')
  setTimeout(() => {
    socket.end()
  }, 250)
}, 3250)

// Risultato
$ node example9.js
got data: HELLO
got data: beat
got data: beat
got data: beat
got data: ALL DONE
```

Il metodo net.connect ritorna uno stream Duplex che rappresenta una socket TCP (lato client).

Lo stream Trasnform eredita da Duplex. Gli stream Trasnform sono stream duplex con una constraint addizionale per aumentare la casualità della relazione fra le interfacce read e write.

```js
'use strict'
const { createGzip } = require('zlib')
const transform = createGzip()
transform.on('data', (data) => {
  console.log('got gzip data', data.toString('base64'))
})
transform.write('first')
setTimeout(() => {
  transform.end('second')
}, 500)

// Risultato
$ node example10.js
got gzip data H4sIAAAAAAAACg==
got gzip data S8ssKi4pTk3Oz0sBAP/7ZB0LAAAA
```

Quando i dati vengono scritti sull'istanza readable dello stream, vengono emessi degli eventi di tipo data sulla parte readable dello stream in formato compresso.

Il modo in cui Transform crea questa relazione casuale è legato a come lo stream viene creato. Anziché passare un metodo read o write, nell'oggetto di configurazione passato al costruttore di Transform viene definito il metodo transform.

```js
'use strict'
const { Transform } = require('stream')
const { scrypt } = require('crypto')
const createTransformStream = () => {
  return new Transform({
    decodeStrings: false,
    encoding: 'hex',
    transform (chunk, enc, next) {
      scrypt(chunk, 'a-salt', 32, (err, key) => {
        if (err) {
          next(err)
          return
        }
        next(null, key)
      })
    }
  })
}
const transform = createTransformStream()
transform.on('data', (data) => {
  console.log('got data:', data)
})
transform.write('A\n')
transform.write('B\n')
transform.write('C\n')
transform.end('nothing more to write')

// Risultato
$ node example11.js
got data: 0b2fc9e2da62d8be9c0e20aa2a024ccb682ca4b980c960719faf7961cd4614ab
got data: bf5d30321e62edb7799305f55b9ae8dd0903f7f80153285e3b7288d72dc6a61d
got data: 5948d4437ac61783075269db7efa33650f537f8f775661f3d782b96580bcb22a
got data: d552a104ef061267719651e4d1a87c9590690d15cc7989f7ba8224f60e224e2a
```

transform ha la stessa firma di write passata a Writable con la differenza che alla funzione next può essere passato un secondo argomento che rappresenta il risultato dell'applicazione di una qualche trasformazione al chunk.

Ci sono 4 modi in cui uno stream può diventare inoperativo.

- evento close
- evento error
- evento finish
- evento end

Anziché mettersi in ascolto su ognuno di questi eventi ci viene incontro una utility, la funzione stream.finished che fornisce un modo semplificato per capire se uno stream ha finito o meno, vediamo un esempio

```js
'use strict'
const net = require('net')
const { finished } = require('stream')
net.createServer((socket) => {
  const interval = setInterval(() => {
    socket.write('beat')
  }, 1000)
  socket.on('data', (data) => {
    socket.write(data.toString().toUpperCase())
  })
  finished(socket, (err) => {
    if (err) {
      console.error('there was a socket error', err)
    }
    clearInterval(interval)
  })
}).listen(3000)
```

In modo molto basico il pipe in Node legge l'output da destra e lo scrive a sinistra. Riprendiamo il piccolo server TCP e adattiamolo all'utilizzo del pipe

```js
'use strict'
const net = require('net')
const socket = net.connect(3000)

socket.pipe(process.stdout)

socket.write('hello')
setTimeout(() => {
  socket.write('all done')
  setTimeout(() => {
    socket.end()
  }, 250)
}, 3250)

// Risultato
$ node example12.js
HELLObeatbeatbeatALL DONE
```

process.stdout è uno stream di tipo Writable. Qualsiasi cosa scritta in process.stdout verrà stampato come output di processo. Il metodo pipe esiste sugli stream Readable e viene passato allo stream Writable. Internalmente pipe imposta un listener su data e scrive automaticamente sullo stream writable tutto quello che diventa disponibile. Dal momento che pipe ritorna uno stream, è possibile concatenare le pipe `streamA.pipe(streamB).pipe(streamC)`. Anche se la concatenazione precedente è molto usata è una bad practice fare quel tipo di concatenamento in quanto se per un qualche motivo lo stream nel mezzo si chiude tutti gli altri stream nella pipeline non si chiudono automaticamente e questo può causare dei memory leak. Per risolvere questo problema ci viene in aiuto la funzione di utility stream.pipeline.

Vediamo un esempio di pipeline

```js
'use strict'
const net = require('net')
const { Transform, pipeline } = require('stream')
const { scrypt } = require('crypto')
const createTransformStream = () => {
  return new Transform({
    decodeStrings: false,
    encoding: 'hex',
    transform (chunk, enc, next) {
      scrypt(chunk, 'a-salt', 32, (err, key) => {
        if (err) {
          next(err)
          return
          }
        next(null, key)
      })
    }
  })
}

net.createServer((socket) => {
  const transform = createTransformStream()
  const interval = setInterval(() => {
    socket.write('beat')
  }, 1000)
  pipeline(socket, transform, socket, (err) => {
    if (err) {
      console.error('there was a socket error', err)
    }
    clearInterval(interval)
  })
}).listen(3000)
```