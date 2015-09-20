# Programozási minták
## 1 Alapértelmezett értékek

kód #1:
```javascript
var defaultValues = {
    a: 1,
    b: {
        c: 2
    }
};
// bővítéssel
var o1 = extendDeep(, defaultValues);
var o2 = extendDeep(, defaultValues);

// prototípus objektummal - teljes másolat szükséges a referenciatípusok miatt
var o1 = Object.create(defaultValues);
var o2 = Object.create(defaultValues);
```

## 2 Flyweight minta

kód #2:
```javascript
//A prototípus
var moleculeProto = {
    velocity: 10,
    position: {
        x: 100,
        y: 100
    },
    setPosition: function setPosition(x, y) {
        this.position = {
            x: x,
            y: y
        };
    }
};
//A konkrét molekulák
var m1 = Object.create(moleculeProto);
var m2 = Object.create(moleculeProto);
//Változások m1-ben
m1.velocity = 5;
m1.setPosition(90, 102);
```

Mivel a prototípusban a position objektum referenciatípus, így annak tagját közvetlenül módosítva minden gyerekobjektumban a változás érvényesülne. Annak érdekében, hogy a változás csak az adott gyerekobjektumon következzen be, vezettük be a setPosition() metódust, mely a gyerekobjektumban hoz létre egy új position objektumot.

## 3 Prototype minta
A klasszikus OOP világában Prototype mintának nevezik azt, amikor egy objektumot egy meglévő sablon alapján hozunk létre.

## 4 Az Egyke (Singleton) minta
Az Egyke minta a klasszikus OOP-ben arra szolgál, hogy egy osztály csak egy példányban létezzen. JavaScriptben nincsenek osztályok, csak objektumok. Amikor létrehozunk egy objektumot, akkor az egyedi, nincs más hozzá hasonló, tehát megfelel az Egyke mintának. JavaScriptben tehát az objektumliterállal létrehozott objektumok egykék, nincs szükség speciális szintaxisra.

Az egykék annyiban különböznek a statikus osztályoktól, hogy létrehozásukat késleltethetjük. Az objektumliterállal létrehozott objektumok azonnal létrejönnek függetlenül attól, hogy szükségünk van-e rá vagy sem. Ha csak akkor szeretnénk létrehozni őket, ha szükségünk van rá, akkor létrehozásukat egy függvénybe csomagolhatjuk, és closure-ben tárolhatjuk a létrejött objektumot.

kód #3:
```javascript
var childSingleton = (function () {
    var instance,
        createInstance = function createInstance() {
            //Privát adattagok és metódusok
            //...
            return {
                name: 'Sári',
                dateOfBirth: {
                    year: 2004,
                    month: 11,
                    day: 14
                },
                getName: function getName() {
                    return this.name;
                },
                setName: function setName(name) {
                    this.name = name;
                }
            };
        };
    return {
        get: function () {
            if (!instance) {
                instance = createInstance();
            }
            return instance;
        }
    };
})();
var c1 = childSingleton.get();
var c2 = childSingleton.get();
ok( c1 === c2, 'A két objektum ugyanaz');
```

## 5 A közvetítő minta (Mediator)
Böngészők esetében a közvetítő minta valósul meg események delegálása esetén. Ekkor egy központi objektumhoz, például a dokumentum legfelső szintjén lévő document elemhez rendeljük az eseménykezelő függvényeket, majd azokban döntjük el, hogy az esemény körülményeitől függően milyen logikát futtatunk. Ekkor a document tölti be a mediátor szerepét.

kód #4:
```javascript
$(document)
    .on('click', 'div', function (e) {
        //A kattintás div elemen történt
    })
    .on('click', 'a', function (e) {
        //A kattintás a elemen történt
    });
```

## 6 A megfigyelő minta (Observer)
Ebben az esetben az adott DOM objektum a megfigyelt, és a függvény a megfigyelő:
kód #5:
```javascript
observable.addEventListener('esemény', observer, false);
```

A megfigyelt egy listában tárolja a hozzá feljelentkezetteket, és beszédes metódusokon keresztül teszi lehetővé ezek kezelését:
- registerObserver(observer)
- unregisterObserver(observer)
- notifyObservers()


A minta szereplőire többféleképpen hivatkozik a szakirodalom:
- megfigyelő/előfizető/feliratkozó (observer/subscriber/listener)
- megfigyelt/tárgyobjektum/kiadó/értesítő/eseményküldő (observable/subject/publisher/notifier/event emitter)

A naiv megoldás az alábbi (többek között nem tartalmaz típusellenőrzést, nem rugalmas a paraméterek elhagyására, stb.):

kód #6:
```javascript
var observable = {
    observers: [],
    addObserver: function(type, fn) {
        if (typeof this.observers[type] === "undefined") {
            this.observers[type] = [];
        }
        this.observers[type].push(fn);
    },
    removeObserver: function(type, fn) {
        var i,
            observers = this.observers[type];
        for (i = 0; i < observers.length; i += 1) {
            if (observers[i] === fn) {
                observers.splice(i, 1);
            }
        }
    },
    notify: function(type) {
        var i,
            observers = this.observers[type] || [],
            args = [].slice.call(arguments, 1);
        for (i = 0; i < observers.length; i += 1) {
            observers[i].apply(this, args);
        }
    }
};
```

kód #7:
```javascript
// observerable,  extendDeep-pel kiterjesztjük a weatherService objektumot
var weatherService = extendDeep(
    {
        state: 'sunny',
        changeState: function (newState, hours) {
            this.state = newState;
            this.notify(newState, hours);
        }
    }, 
    observable);
//observer
var smartPhone = {
    warnForRain: function (hours) {
        console.log('Rain is coming in ', hours, 'hours!');
    },
    prepareForSun: function (hours) {
        console.log('The sun will shine in ', hours, 'hours!');
    }
};
//registering at observable
weatherService.addObserver('rainy', smartPhone.warnForRain);
weatherService.addObserver('sunny', smartPhone.prepareForSun);
//notify observers about changes in the weather
weatherService.changeState('rainy', 3);
weatherService.changeState('rainy', 2);
weatherService.changeState('rainy', 1);
weatherService.changeState('sunny', 1);
```


## 7 A központi eseményvezérlő minta (pubsub)

A megfigyelő minta egyik nagy hátránya, hogy ugyan értesítéskor közvetlen metódushívás nincsen a megfigyelt és megfigyelő között, mégis a regisztráláshoz a megfigyelőnek tartalmaznia kell referenciát a megfigyeltre. Ilyen szempontból a két objektum között még mindig szoros a kapcsolat. Az objektumok közötti kapcsolatot a közvetítő minta segítségével lazíthatjuk úgy, hogy minden objektum egy központi eseményvezérlő objektumon keresztül hívja meg eseményeit, és ennek az objektumnak az eseményeire is regisztrál. Ez a minta tehát tulajdonképpen két mintának az együttes használatából született. A központi eseményvezérlő objektum a közvetítő mintát valósítja meg, ami egyben megfigyelhető is, ami a megfigyelő mintának felel meg.

kód #8:
```javascript
var pubsub = extendDeep(, observable);
//observable
var weatherService = {
    state: 'sunny',
    changeState: function (newState, hours) {
        this.state = newState;
        pubsub.notify(newState, hours);
    }
};
//observer
var smartPhone = {
    warnForRain: /*...*/,
    prepareForSun: /*...*/
};
//registering at observable
pubsub.addObserver('rainy', smartPhone.warnForRain);
pubsub.addObserver('sunny', smartPhone.prepareForSun);
//notify observers about changes in the weather
weatherService.changeState('rainy', 3);
weatherService.changeState('rainy', 2);
weatherService.changeState('rainy', 1);
weatherService.changeState('sunny', 1);
```

## 6 Összefoglalás
- Chaining: a getter metódusokban adjunk inkább objektumokat (legtöbbször this-t) vissza, így láncolható
- Curry: olyan függvények létrehozását végzi el, amelyek korábbi függvények meghívása, néhány paraméterükkel automatikusan kitöltve
    - a closure-ben tárolja el a hívandó függvényt és paramétereit. híváskor az eltárolt paraméterek mellé teszi az aktuálisat

kód #9:
```javascript
var curry function(func){
    var args = Array.prototype.slice.apply(arguments, [1]);

    return function () {
        return func.apply(
            null,
            args.concat(Array.prototype.slice.applay(arguments));
        );
    }
}

// Példa 1:
var add = function (a, b) {
    return a + b;
}

var inc = curry(add, 1), 
    add2= curry(add, 2);

inc(6);     // 7
add2(6);    // 8

// Példa 2:
var elemeketSzinez = function(hol, milyenElemeket, milyenSzinre){
    return $(hol).find(milyenElemeket).css('backgroundColor', milyenSzinre);
};
elemeketSzinez('#adatok', '.adat', 'yellow');

// ha sokszor használjuk ezt a két szelektort
var adatokatSzinez = function(milyenSzinre){
    return elemeketSzinez('#adatok', '.adat', milyenSzinre);
}
// !!!!! ehelyettt !!!!!
var adatokatSzinez = curry(elemeketSzinez, '#adatok', '.adat');
adatokatSzinez('yellow');

```

- alaptípusok bővítése

kód #10:

```javascript
if (typeof String.prototype.trim !== 'function') {
    String.prototype.trim = function(){
        return this.replace(/^\s*(\S+)\s*$/, "$1");
    }
}

' I wanna whitespace! :( '.trim();
// "I wanna whitespace! :("
```

- factory
- mediator (közvetítő)
- observer (megfigyelő)
- pubsub (központi eseményvezérlés): mediator + observer