# JavaScript Design Patterns

## The Constructor Pattern 

Object constructors are used to create specific types of objects - both preparing the object for use and accepting arguments which a constructor can use to set the values of member properties and methods when the object is first created.

## The Module Pattern

The Module pattern was originally defined as a way to provide both private and public encapsulation for classes in conventional software engineering.

**Advantages:**
* *cleaner code*
* *private data*

**Disadvantages:**
* *no access to private members in methods that are added to the object at a later point*
* *unit testing privates*

## The Revealing Module Pattern

This is an upgraded module pattern which returns an anonymous object with pointers to the private functionality we wished to reveal as public. All of our functions and variables are in the private scope.

**Advantages:**
* *more consistent syntax and readability*

**Disadvantages:**
* *no patching or overriding of public members*

## The Singleton Pattern

Singleton pattern can be implemented by creating a class with a method that creates a new instance of the class if one doesn't exist. In the event of an instance already existing, it simply returns a reference to that object.

```javascript
const singleton = (function () {
    let foods = ['banana', 'apple', 'strawberry', 'mango'];
    let instance;

    let createInstance = () => {
        return { food: foods[Math.floor(Math.random() * foods.length)] };
    }

    return {
        getInstance() {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    }
})();

var a = singleton.getInstance();
var b = singleton.getInstance();

console.log(a, b);
```

## The Observer Pattern

Define a one-to-many dependency between objects so that when one object changes state, all its dependents are notified and updated automatically.

```javascript
class Subject {
    constructor() {
        this.observers = [];
    }
    sub(observer) {
        this.observers.push(observer);
    }
    unsub(observer) {
        this.observers = this.observers.filter(o => o !== observer);
    }
    notify() {
        for (const o of this.observers) {
            o.update();
        }
    }
    increment(observer) {
        observer.obj.num++;
        this.notify();
    }
}

class Observer {
    constructor(obj) {
        this.obj = obj;
    }
    update() {
        console.log('Observer', this.obj, 'is notifed');
    }
}

var subject = new Subject();

var o1 = new Observer({ num: 1 });
var o2 = new Observer({ num: 2 });

subject.sub(o1);
subject.sub(o2);

console.log(subject.observers); // [ Observer { obj: { num: 1 } }, Observer { obj: { num: 2 } } ]
subject.notify(o2);
// Observer { num: 1 } is notifed
// Observer { num: 2 } is notifed

subject.increment(o2);
// Observer { num: 1 } is notifed
// Observer { num: 3 } is notifed

console.log(subject.observers); // [ Observer { obj: { num: 1 } }, Observer { obj: { num: 3 } } ]
```

## The Mediator Pattern

The Mediator pattern provides central authority over a group of objects by encapsulating how these objects interact

**Advantages:**
* *reduces the communication channels between objects from many to many to many to one*
* *adding new pubs/subs*

**Disadvantages:**
* *single point of failure*
* *performance hit*

```javascript
// Colleague
class User {
    constructor(username) {
        this.username = username;
        this.msgsystem = null;
    }
    send(message, to) {
        this.msgsystem.store(message, this, to);
    }
    showAll() {
        const msgArr = this.msgsystem.retrieveAll(this.username);

        if (msgArr.length > 0) {
            for (const m of msgArr) {
                console.log(`${m.from} -> ${m.to}: ${m.message}`);
            }
        } else {
            console.log('no messages!');
        }
    }
}
// Mediator
class MsgSystem {
    constructor() {
        this.users = {};
        this.messages = [];
    }
    join(user) {
        this.users[user.username] = user;
        user.msgsystem = this;
    }
    store(message, from, to) {
        this.messages.push({
            from: from.username,
            to: to.username,
            message
        });
    }
    retrieveAll(to) {
        return this.messages.filter(m => m.to === to);
    }
}

const john = new User('john');
const george = new User('george');
const oscar = new User('oscar');

const msgSystem = new MsgSystem();
msgSystem.join(john);
msgSystem.join(george);
msgSystem.join(oscar);

john.send('text message', george);
john.send('another text message', george);
oscar.send('another text message', john);

george.showAll();
// john -> george: text message
// john -> george: another text message
oscar.showAll();
// no messages!

console.log(JSON.stringify(msgSystem.messages, 0, 3));
/*
[
   {
      "from": "john",
      "to": "george",
      "message": "text message"
   },
   {
      "from": "john",
      "to": "george",
      "message": "another text message"
   },
   {
      "from": "oscar",
      "to": "john",
      "message": "another text message"
   }
]
*/
```

## The Prototype Pattern

The Prototype Pattern creates new objects, but rather than creating non-initialized objects it returns objects that are initialized with values it copied from a prototype object. The most common approach is the use of `Object.create(proto, [propertiesObject])`.

## The Command Pattern

The Command pattern aims to encapsulate method invocation, requests or operations into a single object and gives us the ability to both parameterize and pass method calls around that can be executed at our discretion.

```javascript
class MyArray {
    constructor() {
        this.arr = [];
    }
    add(item) {
        if (!this.arr.includes(item)) {
            this.arr.push(item);
            console.log('added item:', item);
        } else {
            console.log('duplicate:', item, '(no change)');
        }
    }
    remove(item) {
        if (this.arr.includes(item)) {
            this.arr = this.arr.filter(v => v !== item);
            console.log('removed item:', item);
        } else {
            console.log('item:', item, 'not found (no change)');
        }
    }
    do(name) {
        const args = Array.from(arguments);
        args.shift();

        if (typeof this[name] === 'function') {
            for (const arg of args) {
                this[name](arg);
            }
        } else {
            console.log('function does not exist');
        }
    }
}

const arr = new MyArray();

arr.do('add', 1);		// added item: 1
arr.do('add', 2);		// added item: 2
arr.do('add', 2);		// duplicate: 2 (no change)
arr.do('add', 3);		// added item: 3

arr.do('remove', 2);	// removed item: 2
arr.do('remove', 4);	// item: 4 not found (no change)

arr.do('add', 1, 5, 6, 7, 8);
/*
duplicate: 1 (no change)
added item: 5
added item: 6
added item: 7
added item: 8
*/
arr.do('remove', 6, 8, 9);
/*
removed item: 6
removed item: 8
item: 9 not found (no change)
*/
arr.do('something', 1, 2, 3); // function does not exist

console.log(arr);   // MyArray { arr: [ 1, 3, 5, 7 ] }
```

## The Facade Pattern

Provides an interface which shields clients from complex functionality.

**Pros:**
* *hiding complex code from the user*

**Cons:**
* *performance costs*

```javascript
var addEvent = function (el, ev, fn) {
    if (el.addEventListener) {
        el.addEventListener(ev, fn, false);
        // for older browsers
    } else if (el.attachEvent) {
        el.attachEvent("on" + ev, fn);
    } else {
        el["on" + ev] = fn;
    }
};

addEvent(window, 'load', alert('window loaded'));
```

## The Factory Pattern

Provides an interface for creating objects that let subclasses decide which class to instantiate.

**When to use:**
* *object setup involves high level of complexity*
* *easily generate different instances of objects*
* *small objects that share same properties*

**When NOT to use:**
* *applying to the wrong type of problem*
* *can produce problems with unit testing*

```javascript
class Book {
    constructor(name, category, author) {
        this.name = name;
        this.category = category;
        this.author = author;
    }
}

class Author {
    constructor(name) {
        this.name = name;
    }
}

const bookFactory = {
    create(bookName, category, authorName) {
        const author = new Author(authorName);
        const book = new Book(bookName, category, author);

        return {
            book,
            info() {
                return `${book.name} is a book by ${book.author.name} categorized as ${book.category}`;
            }
        }
    },
}

const book = bookFactory.create('The Hobbit', 'Fantasy', 'J.R.R. Tolkien');

console.log(book.info());   // The Hobbit is a book by J.R.R. Tolkien categorized as Fantasy
console.log(book.book instanceof Book); // true
console.log(book.book.author instanceof Author);    // true
```

## The Mixin Pattern

Mixins are classes which offer functionality that can be easily inherited by a sub-class or group of sub-classes for the purpose of function re-use.

**Advantages:**
* *decrease function repetition and increase function re-use*

**Disadvantages:**
* *prototype pollution*

```javascript
// THE MIXIN PATTERN
class User {
    constructor(name) {
        this.name = name;
    }
}

const Mixin = {
    greet() {
        console.log(`Hello ${this.name}!`);
    }
};

Object.assign(User.prototype, Mixin);

const user = new User('John');
user.greet();   // Hello John!
```

## The Decorator Pattern

The Decorator pattern extends (decorates) an object’s behavior dynamically.

```javascript
function User(name) {
    this.name = name;
    this.info = () => `Name: ${this.name} #`;
}

function addAddress(user, address) {
    this.address = address;

    const info = user.info();
    user.info = () => `${info} Address: ${address} #`;
}

function addCountry(user, country) {
    this.country = country;

    const info = user.info();
    user.info = () => `${info} Country: ${country} #`;
}

var user = new User('John');
console.log(user.info());   // Name: John #

addAddress(user, '27 Colmore Row, Birmingham');
addCountry(user, 'England');
console.log(user.info());   // Name: John # Address: 27 Colmore Row, Birmingham # Country: England #
```

## The Flyweight Pattern

Essentially Flyweight is an 'object normalization technique' in which common properties are factored out into shared flyweight objects.

```javascript
// THE FLYWEIGHT PATTERN
class Flyweight {
    constructor(make, model, processor) {
        this.make = make;
        this.model = model;
        this.processor = processor;
    }
};

var FlyWeightFactory = (function () {
    var flyweights = {};

    return {
        get: function (make, model, processor) {
            if (!flyweights[make + model]) {
                flyweights[make + model] = new Flyweight(make, model, processor);
            }

            return flyweights[make + model];
        },
        getCount: function () {
            return Object.entries(flyweights).length;
        }
    }
})();

class Computer {
    constructor(make, model, processor, memory, tag) {
        this.flyweight = FlyWeightFactory.get(make, model, processor);
        this.memory = memory;
        this.tag = tag;
        this.getMake = function () {
            return this.flyweight.make;
        };
    }
}

function ComputerCollection() {
    var computers = {};

    return {
        add: function (make, model, processor, memory, tag) {
            computers[tag] = new Computer(make, model, processor, memory, tag);
        },
        get: function (tag) {
            return computers[tag];
        },
        getCount: function () {
            return Object.entries(computers).length;
        }
    };
}

const computers = new ComputerCollection();

computers.add("Dell", "Studio XPS", "Intel", "5G", "Y755P");
computers.add("Dell", "Studio XPS", "Intel", "6G", "X997T");
computers.add("Dell", "Studio XPS", "Intel", "2G", "U8U80");
computers.add("Dell", "Studio XPS", "Intel", "2G", "NT777");
computers.add("Dell", "Studio XPS", "Intel", "2G", "0J88A");
computers.add("HP", "Envy", "Intel", "4G", "CNU883701");
computers.add("HP", "Envy", "Intel", "2G", "TXU003283");

console.log("Computers:", computers.getCount());            // Computers: 7
console.log("Flyweights:", FlyWeightFactory.getCount());    // Flyweights: 2
```
