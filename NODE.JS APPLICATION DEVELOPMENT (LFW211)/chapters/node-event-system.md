#  NODE'S EVENT SYSTEM

Per creare un EventEmitter abbiamo bisogno del modulo events

```js
const { EventEmitter } = require('events')

// oppure 
const EventEmitter = require('events')

// Per creare un nuovo evento si chiama il costruttore con la parola chiave new
const myEmitter = new EventEmitter()

// oppure creiamo una nostra classe e ereditiamo da EventEmitter
class MyEmitter extends EventEmitter {
    constructor (opts = {}) {
        super(opts)
        this.name = opts.name
    }
}

// Per emettere un evento si utilizza il metodo emit
const { EventEmitter } = require('events')
const myEmitter = new EventEmitter()
myEmitter.emit('an-event', some, args)

// oppure
const { EventEmitter } = require('events')
class MyEmitter extends EventEmitter {
    constructor (opts = {}) {
        super(opts)
        this.name = opts.name
    },
    destroy (err) {
        if (err) { this.emit('error', err) }
        this.emit('close')
    }
}

// Per rimanere in ascolto su un evento emesso si utilizza addListener oppure on
const { EventEmitter } = require('events')

const ee = new EventEmitter()
ee.on('close', () => { console.log('close event fired!') }) 
/*
* La riga sopra poteva anche essere scritta così
* ee.addListener('close', () => { console.log(close event fired!') })
*/
ee.emit('close')

// Gli argomenti passati dall'emitter vengono ricevuti dal listener
ee.on('add', (a, b) => { console.log(a + b) }) // logs 13
ee.emit('add', 7, 6)

// L'ordine delle istruzioni è importante, nelle seguenti righe prima viene fatto l'emit, poi
// poi ci mettiamo in ascolto, ma ormai ho già emesso, quindi l'on non cattura nulla
ee.emit('close')
ee.on('close', () => { console.log('close event fired!') })

// I listener vengono anche chiamati nello stesso ordine in cui sono scritti
const { EventEmitter } = require('events')
const ee = new EventEmitter()
ee.on('my-event', () => { console.log('1st') })
ee.on('my-event', () => { console.log('2nd') })
ee.emit('my-event')

// Per aggiungere listener in cima si utilizza il metodo prependListener
const { EventEmitter } = require('events')
const ee = new EventEmitter()
ee.on('my-event', () => { console.log('2nd') })
ee.prependListener('my-event', () => { console.log('1st') })
ee.emit('my-event')

// Un evento può essere emesso anche più di una volta
const { EventEmitter } = require('events')
const ee = new EventEmitter()
ee.on('my-event', () => { console.log('my-event fired') })
ee.emit('my-event')
ee.emit('my-event')
ee.emit('my-event')
// Il risultato di sopra sarà la scritta my-event fired ripetuta tre volte.

// Per far sì che il listener sia utilizzato solo una volta si utilizza once
const { EventEmitter } = require('events')
const ee = new EventEmitter()
ee.once('my-event', () => { console.log('my-event fired') })
ee.emit('my-event')
ee.emit('my-event')
ee.emit('my-event')
// In questo caso il risultato del codice sopra sarà la scritta my-event fired stampata solo una volta

// Il metodo removeListener può essere utilizzaato per rimuovere un listener precedentemente registrato. Questo metodo prende due argomenti: il nome dell'evento e la funzione di listener
const { EventEmitter } = require('events')
const ee = new EventEmitter()

const listener1 = () => { console.log('listener 1') }
const listener2 = () => { console.log('listener 2') }

ee.on('my-event', listener1)
ee.on('my-event', listener2)

setInterval(() => {
    ee.emit('my-event')
}, 200)

setTimeout(() => {
    ee.removeListener('my-event', listener1)
}, 500)

setTimeout(() => {
    ee.removeListener('my-event', listener2)
}, 1100)
// il risultato del codice sopra è che il listener1 viene chiamato 2 volte, mentre il listener2 viene chiamato 5 volte

// il metodo removeAllListener può essere utilizzato per rimuovere in un colpo solo tutti i listener esistenti: se viene utilizzato senza argomenti allora elimina tutti i listener esistenti, se viene passato un argomento (il nome del listener) elimina tutti i listener con quel nome
const { EventEmitter } = require('events')
const ee = new EventEmitter()

const listener1 = () => { console.log('listener 1') }
const listener2 = () => { console.log('listener 2') }

ee.on('my-event', listener1)
ee.on('my-event', listener2)
ee.on('another-event', () => { console.log('another event') })

setInterval(() => {
    ee.emit('my-event')
    ee.emit('another-event')
}, 200)

setTimeout(() => {
    ee.removeAllListeners('my-event')
}, 500)

setTimeout(() => {
    ee.removeAllListeners()
}, 1100)
// il risultato del codice sopra è che listener1 e listener2 vengono stampati 2 volte mentre another event viene stampato 5 volte.

// Emettere un evento 'error' causa il sollevamento di un'eccezione se il listener per 'error' non è stato registrato.
const { EventEmitter } = require('events')
const ee = new EventEmitter()

process.stdin.resume() // keep process alive

ee.emit('error', new Error('oh oh')) 
// il codice sopra produce l'errore nella seguente immagine
```

![](images/event-error.jpeg)

```js
// se invece registriamo un listener per l'errore, allora il processo non va in crash
const { EventEmitter } = require('events')
const ee = new EventEmitter()

process.stdin.resume() // keep process alive

ee.on('error', (err) => {
    console.log('got error:', err.message )
})

ee.emit('error', new Error('oh oh'))
```

AbortController può essere utilizzato pure per cancellare eventi che sono delle promise. events.once ritorna una promise che si risolve una volta che l'evento è stato emesso.

```js
import someEventEmitter from './somewhere.js'
import { once } from 'events'

await once(someEventEmitter, 'my-event')
// arrivati alla riga sopra l'esecuzione viene messa in pausa fino a quando un evento non viene registrato. Se l'evento non viene mai emesso, l'esecuzione rimarrà sempre ferma a questa riga.

import { once, EventEmitter } from 'events'
const uneventful = new EventEmitter()

await once(uneventful, 'ping')
console.log('pinged!')
// pinged non verrà mai stampato in quanto l'evento non verrà mai emesso. Per evitare di rimanere bloccati per sempre su questa riga si utilizza AbortController come segue
import { once, EventEmitter } from 'events'
import { setTimeout } from 'timers/promises'

const uneventful = new EventEmitter()

const ac = new AbortController()
const { signal } = ac

setTimeout(500).then(() => ac.abort())

try {
    await once(uneventful, 'ping', { signal })
    console.log('pinged!')
} catch (err) {
    // ignore abort errors:
    if (err.code !== 'ABORT_ERR') throw err
    console.log('canceled')
}
```