# USING BUFFERS

Il costruttore della classe Buffer è globale, quindi non è necessario includere nessun modulo core di Node per poterlo utilizzare

```bash
$ node -p "Buffer"
[Function: Buffer] {
  poolSize: 8192,
  from: [Function: from],
  of: [Function: of],
  alloc: [Function: alloc],
  allocUnsafe: [Function: allocUnsafe],
  allocUnsafeSlow: [Function: allocUnsafeSlow],
  isBuffer: [Function: isBuffer],
  compare: [Function: compare],
  isEncoding: [Function: isEncoding],
  concat: [Function: concat],
  byteLength: [Function: byteLength],
  [Symbol(kIsEncodingSymbol)]: [Function: isEncoding]
}
```

Quando Buffer è stato per la prima volta introdotto in NodeJS, il linguaggio JavaScript non aveva nessun tipo binario. Con l'evoluzione del linguaggio è stato introdotto ArrayBuffer e una serie di typed arrays per fornire diverse "viste" del buffer. Ad esempio un'istanza di ArrayBuffer può essere acceduta da Float64Array dove ogni set di 8bytes vengono interpretati come un numero floating point a 64bit oppure può essere acceduta da Int32Array dove ogni 4bytes rappresentano un numero a 32bit a complemento a 2 oppure ancora può essere acceduta da Uint8Array dove ogni byte rappresenta un numero intero senza segno da 0 a 255. 

Quando tutte queste strutture dati sono state aggiunte al linguaggio l'internal di Buffer è stato refactorizzato sopra a Uint8Array per questo motivo un oggetto buffer è sia un'istanza di Buffer che di Uint8Array

```node
$ node
Welcome to Node.js v16.13.0.
Type ".help" for more information.
> const buffer = Buffer.alloc(10)
undefined
> buffer instanceof Buffer
true
> buffer instanceof Uint8Array
true
```

Un'altra cosa da notare è che il metodo Buffer.prototype.slice fa l'override di Uint8Array.prototype.slice e fornisce un comportamento diverso. Il metodo slice di Uint8Array esegue una copia del buffer compreso fra due indici, il metodo slice di Buffer ritorna una nuova istanza di Buffer che fa riferimento ai dati originali del buffer sul quale è chiamato il metodo slice.

```node
$ node
Welcome to Node.js v16.13.0.
Type ".help" for more information.
> var buf1, buf2, buf3, buf4
undefined
> buf1 = Buffer.alloc(10)
<Buffer 00 00 00 00 00 00 00 00 00 00>
> buf2 = buf1.slice(2, 3)
<Buffer 00>
> buf2[0] = 100
100
> buf2
<Buffer 64>
> buf1
<Buffer 00 00 64 00 00 00 00 00 00 00>
> buf3 = new Uint8Array(10)
Uint8Array(10) [
  0, 0, 0, 0, 0,
  0, 0, 0, 0, 0
]
> buf4 = buf3.slice(2, 3)
Uint8Array(1) [ 0 ]
> buf4[0] = 100
100
> buf4
Uint8Array(1) [ 100 ]
> buf3
Uint8Array(10) [
  0, 0, 0, 0, 0,
  0, 0, 0, 0, 0
]
```

new Buffer è deprecato, quindi per creare una nuova istanza di Buffer non c'è da usare il new, al suo posto si utilizza Buffer.alloc.

```js
const buffer = Buffer.alloc(10)
```

l'istruzione sopra alloca un buffer di 10bytes che inzialmente sarà riempito di zeri.

```node
$ node
Welcome to Node.js v16.13.0.
Type ".help" for more information.
> const buffer = Buffer.alloc(10)
undefined
> buffer
<Buffer 00 00 00 00 00 00 00 00 00 00>

// Ogni coppia è un numero esadecimale.
```

Per creare un buffer a partire da una stringa si utilizza Buffer.from. Il codice `const buffer = Buffer.from('hello world')` crea un buffer a partire dalla stringa "hello world".

```bash
$ node -p "Buffer.from('hello world')"
<Buffer 68 65 6c 6c 6f 20 77 6f 72 6c 64>
```

Per convertire una stringa in binario è necessario specificare la codifica. Di default Buffer.from utilizza come codifica UTF8 che utilizza fino a 4bytes per carattere quindi non possiamo assumere che la lunghezza della stringa sia sempre uguale alla lunghezza della stessa stringa codificata.

```js
console.log('👀'.length) // will print 2
console.log(Buffer.from('👀').length) // will print 4
```

Per specificare un diverso tipo di codifica possiamo passare un secondo argomento a Buffer.from. Esistono due tipi diversi di codifica: la codifica binaria e la codifica binary-to-text

```bash
$ node -p "Buffer.from('hello')"
<Buffer 68 65 6c 6c 6f>
$ node -p "Buffer.from('hello', 'utf16le')"
<Buffer 68 00 65 00 6c 00 6c 00 6f 00>
```

Così com'è possibile convertire una stringa in un buffer è possibile fare anche l'operazione inversa, ovvero convertire un buffer in stringa. Lo si fa chiamando semplicemente il metodo toString.

```js
const buffer = Buffer.from('👀')
console.log(buffer) // prints <Buffer f0 9f 91 80>
console.log(buffer.toString()) // prints 👀
console.log(buffer + '') // prints 👀
```

Al toString è anche possibile passare il tipo di codifica

```js
const buffer = Buffer.from('👀')
console.log(buffer) // prints <Buffer f0 9f 91 80>
console.log(buffer.toString('hex')) // prints f09f9180
console.log(buffer.toString('base64')) // prints 8J+RgA==
```