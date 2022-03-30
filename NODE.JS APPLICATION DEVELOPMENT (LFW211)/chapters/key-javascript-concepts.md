# KEY JAVASCRIPT CONCEPTS

JavaScript è un linguaggio debolmente tipizzato. Ci sono 7 primitive, mentre tutto il resto può essere considerato come un oggetto. Le primitive sono

- null --> tipicamente descrive l'assenza di un oggetto
- undefined --> tipicamente indica l'assenza di un valore. Ogni variabile inizializzata senza un valore è undefined. Un qualcosa che tenta di accedere ad una proprietà non esistente di un oggetto darà come risposta un undefined. Ogni funzione senza un'istruzione di return restituisce undefined.
- number --> double-precision floating-point. Ammette sia decimali che interi, gli interi assegnabili stanno nel range 2^53-1 to 2^53-1
- BigInt --> come sopra ma senza limiti superiori e inferiori
- String
- Boolean
- Symbol --> possono essere utilizzati come chiavi uniche all'interno di oggetti

Oltre ai tipi primitivi sopra descritti, tutto il resto, in JS, è un oggetto, che è un insieme di coppie chiave-valore dove il valore può essere un qualsiasi tipo primitivo o un altro oggetto. LE chiavi di un oggetto vengono chiamate proprietà. 

Tutti gli oggetti JS hanno un prototype. Un prototype è una referenza implicita a un altro oggetto che viene interrogato durante la ricerca di proprietà. Se un oggetto non ha una particolare proprietà allora questa viene cercata all'interno del prototype dell'oggetto. Se nemmeno il prototype dell'oggetto ha quella particolare proprietà allora viene interrogato il prototype del prototype, ecc... e così è come funziona l'ereditarietà in JS.

Le funzioni si dice che sono cittadine di prima classe in JS. Una funzione è un oggetto e come tale può essere utilizzato come tutti gli altri oggetti.

Una funzione può essere ritornata da una funzione

```js
function factory () {
  return function doSomething () {}
}
```

una funzione può essere passata a un'altra funzione

```js
setTimeout(function () { console.log('hello from the future') }, 100)
```

può essere assegnata a un oggetto

```js
const obj = { id: 999, fn: function () { console.log(this.id) } }
obj.fn() // prints 999
```

> NB: Quando una funzione è assegnata a un oggetto, quando utilizziamo la parola chiave this all'interno della funzione dichiarata all'interno dell'oggetto allora questo si riferisce all'oggetto sul quale la funzione viene chiamata. Ecco perché, nell'esempio precedente, il risultato è 999. Quindi, SI RIFERISCE ALL'OGGETTO SUL QUALE LA FUNZIONE VIENE CHIAMATA, NON ALL'OGGETTO CHE CONTIENE LA FUNZIONE

```js
const obj = { id: 999, fn: function () { console.log(this.id) } }
const obj2 = { id: 2, fn: obj.fn }
obj2.fn() // prints 2
obj.fn() // prints 999
```

Tutte le funzioni hanno un metodo chiamato call che viene utilizzato per settare il contesto del this.

```js
function fn() { console.log(this.id) }
const obj = { id: 999 }
const obj2 = { id: 2 }
fn.call(obj2) // prints 2
fn.call(obj) // prints 999
fn.call({id: ':)'}) // prints :)
```

Un altro modo per scrivere funzioni è attraverso le lambda function (dette anche fat arrow function). Le lambda non hanno il loro contesto per il this e questo si riferisce al più vicino this genitore

```js
function fn() {
  return (offset) => {
   console.log(this.id + offset)
  }
}
const obj = { id: 999 }
const offsetter = fn.call(obj)
offsetter(1) // prints 1000 (999 + 1)
```

Le funzioni normali hanno il prototype, mentre le lambda no.

```js
function normalFunction () { }
const fatArrowFunction = () => {}
console.log(typeof normalFunction.prototype) // prints 'object'
console.log(typeof fatArrowFunction.prototype) // prints 'undefined'
```

### Prototypal Inheritance (Functional)

In JS l'ereditarietà viene fatta attraverso una catena di prototype. Esistono diversi modi per creare una catena di prototype, i tre più comuni sono:

- funzionale
- funzioni costruttore
- class-syntax constructor (non so tradurlo)

Il modo funzionale per creare una catena di prototype è attraverso l'utilizzo di Object.create

```js
const wolf = {
  howl: function () { console.log(this.name + ': awoooooooo') }
}

const dog = Object.create(wolf, {
  woof: { value: function() { console.log(this.name + ': woof') } }
})

const rufus = Object.create(dog, {
  name: {value: 'Rufus the dog'}
})

rufus.woof() // prints "Rufus the dog: woof"
rufus.howl() // prints "Rufus the dog: awoooooooo"
```

wolf è un semplice oggetto javascript e il prototype di un oggetto javascript è Object.prototype.

Object.create può accettare due argomenti: il primo di questi è il prototipo dell'oggetto che verrà creato. Ritornando all'esempio di sopra quando l'oggetto dog viene istanziato gli argomenti di Object.create sono wolf, che quindi è il prototype di dog, e l'oggetto da creare. Allo stesso modo quando rufus viene istanziato, dog è il primo argomento e quindi il prototype di rufus. Quindi, alla fine del palo, avremmo che

- il prototype di rufus è dog
- il prototype di dog è wolf
- il prototype di wolf è Object.prototype

Il secondo argomento di Object.create è un oggetto, opzionale, detto Properties Descriptor. Properties Descriptor è un oggetto che che contiene le chiavi che diverranno la chiave dell'oggetto che sarà creato. Per recuperare queste Properties Descriptor si usa Object.getOwnPropertyDescriptor

```bash
C:\Windows\SysWOW64>node -p "Object.getOwnPropertyDescriptor(process, 'title')"
{
  value: `Windows Command Processor - node  -p "Object.getOwnPropertyDescriptor(process, 'title')"`,
  writable: true,
  enumerable: true,
  configurable: true
}
```

Per descrivere il valore di una proprietà il descriptor può utilizzare sia value che get/set per creare proprietà getter/setter. La proprietà writable ci dice se la proprietà può essere riassegnata. La proprietà enumerable ci dice se la proprietà è enumerabile o no (utilizzando Object.keys) ad esempio. Tutti questi metadati di default hanno valore false.

Nel caso di dog e rufus il property descriptor imposta il valore di value il quale mette a false writable, enumerable, configurable. Quando il prototipo dog viene creato, il property descriptor è un oggetto con una chiave woof la quale fa riferimento ad un oggetto che ha come valore una funzione. Il risultato è un oggetto che ha un metodo chiamato woof. Quando rufus.woof() viene eseguito l'oggetto rufus non ha una proprietà chiamata woof, quindi il runtime di js va a vedere se il prototipo ha questa proprietà. Il prototype di rufus è dog che ha la proprietà woof.

### Prototypal Inheritance (Constructor Functions)

La creazione di un oggetto con uno specifico prototype può essere fatto anche chiamando una funzione con la keyword new. Tutte le funzioni hanno una proprietà prototype e l'approccio che utilizza il costruttore per creare una catena di prototype viene realizzato definendo le proprietà nell'oggetto prototipo della funzione e dopo chiamarla utilizzando la keyword new.

```js
function Wolf (name) {
  this.name = name
}

Wolf.prototype.howl = function () {
  console.log(this.name + ': awoooooooo')
}

function Dog (name) {
  Wolf.call(this, name + ' the dog')
}

function inherit (proto) {
  function ChainLink(){}
  ChainLink.prototype = proto
  return new ChainLink()
}

Dog.prototype = inherit(Wolf.prototype)

Dog.prototype.woof = function () {
  console.log(this.name + ': woof')
}

const rufus = new Dog('Rufus')

rufus.woof() // prints "Rufus the dog: woof"
rufus.howl() // prints "Rufus the dog: awoooooooo"
```

### Prototypal Inheritance (Class-Syntax Constructors)

Javascript moderno (EcmaScript 2015+) ha la keyword class che non è da confondere con la keyword class di linguaggi OOP. In JS class è zucchero sintattico per creare funzioni, nello specifico crea funzioni che poi possono essere chiamate col new. Crea funzioni costruttore. 

```js
class Wolf {
  constructor (name) {
    this.name = name
  }
  howl () { console.log(this.name + ': awoooooooo') }
}

class Dog extends Wolf {
  constructor(name) {
    super(name + ' the dog')
  }
  woof () { console.log(this.name + ': woof') }
}

const rufus = new Dog('Rufus')

rufus.woof() // prints "Rufus the dog: woof"
rufus.howl() // prints "Rufus the dog: awoooooooo"
```

La parola extends fa in modo che l'ereditarietà fra prototype sia più semplice. Quindi class Dog extends Wolf si traduce in "il prototype di Dog.prototype è Wolf.prototype".

Il costruttore in ogni classe è l'equivalente delle funzioni costruttore. Quindi `function Wolf (name) { this.name = name }` è la stessa cosa di `Wolf { constructor (name) { this.name = name } }`. 

La parola super viene utilizzata per richiamare il costruttore padre. Quindi il super dentro al costruttore della classe Dog dell'esempio precedente viene utilizzato per richiamare il costruttore della classe padre Wolf.

### Closure Scope

Quando una funzione viene creata, viene creato anche un oggetto "invisibile" che è noto col nome di "closure scope". Parametri e variabili creati nel corpo della funzione vengono salvati all'interno di questo oggetto. Una funzione dentro ad una funzione può accedere sia al proprio scope che allo scope della funzione che la contiene

```js
function outerFn () {
  var foo = true
  function print() { console.log(foo) }
  print() // prints true
  foo = false
  print() // prints false
}
outerFn()
```

in caso di nomi di variabili uguali (collisione di nomi) viene utilizzata la variabile più vicina allo scope

```js
function outerFn () {
  var foo = true
  function print(foo) { console.log(foo) }
  print(1) // prints 1
  foo = false
  print(2) // prints 2
}
outerFn()
```

Non è possibile accedere allo scope di una funzione dall'esterno.

```js
function outerFn () {
  var foo = true
}
outerFn()
console.log(foo) // will throw a ReferenceError
```

Dal momento che non è possibile accedere allo scope dall'esterno, se una funzione ritorna un'altra funzione, la funzione ritornata può fornire un accesso controllato allo scope della funzione padre.

```js
function init (type) {
  var id = 0
  return (name) => {
    id += 1
    return { id: id, type: type, name: name }
  }
}
const createUser = init('user')
const createBook = init('book')
const dave = createUser('Dave')
const annie = createUser('Annie')
const ncb = createBook('Node Cookbook')
console.log(dave) //prints {id: 1, type: 'user', name: 'Dave'}
console.log(annie) //prints {id: 2, type: 'user', name: 'Annie'}
console.log(ncb) //prints {id: 1, type: 'book', name: 'Node Cookbook'}
```

Le closure possono essere utilizzate anche come alternativa all'ereditarietà tramite prototype.

```js
function wolf (name) {
  const howl = () => {
    console.log(name + ': awoooooooo')
  }
  return { howl: howl }
}

function dog (name) {
  name = name + ' the dog'
  const woof = () => { console.log(name + ': woof') }
  return {
    ...wolf(name),
    woof: woof
  }
}
const rufus = dog('Rufus')

rufus.woof() // prints "Rufus the dog: woof"
rufus.howl() // prints "Rufus the dog: awoooooooo"
```