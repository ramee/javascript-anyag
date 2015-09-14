# 1 Objektumok
## 1.1 dinamikus objektumok (ES5)
### 1.1.1 tulajdonságleíró objektumok

- tulajdonságok
    - writable (módosítható)
    - enumerable (felsorolható)
    - configurable (törölhető)
- getter, setter
- descriptor (tulajdonságleíró objektum)
    - értékei:
        + value: a tulajdonság értéke. Alapértelmezetten undefined.
        + writable: megváltoztatható-e egy tulajdonság értéke? Alapértelmezetten hamis.
        + enumerable: megjelenik-e az adott tulajdonság a for..in ciklusban vagy az Object.keys() felsorolásban. Alapértelmezetten hamis.
        + configurable: törölhető-e a tulajdonság, vagy változtatható-e bármelyik itt tárgyalt attribútuma? Alapértelmezetten hamis.
        + set: setter. Alapértelmezetten undefined.
        + get: getter. Alapértelmezetten undefined.

kód #1:
```
var descriptor = {
    value: "érték",
    writable: true,
    enumerable: true,
    configurable: true,
    set: function(value) { valami = value},
    get: function() { return test }
};
```
    
- objektum létrehozása (üresen)

kód #2
```
var obj = {};
var obj = Object.create(Object.prototype);
var obj = Object.create(null); // itt az objektumnak nem lesz prototype-objektuma
```

- objektum létrehozása (előre megadott tulajdonságokkal)
    - egyszerűbb verzió (kód #3)
    - objektumliterállal kompatibilis objektumként, megadva a descriptor tulajdonságokat (kód #4)

kód #3:
```
var obj = {
    alma: 'piros',
    'körte': 'sárga'
};
```

kód #4:
```
var obj = Object.create(Object.prototype, {
    alma: {
        value: 'piros',
        configurable: true,
        enumerable: true,
        writable: true
    },
    'körte': {
        value: 'sárga',
        configurable: true,
        enumerable: true,
        writable: true
    }
});
```

- objektum tulajdonságainak dinamikus megadása

kód #5:
```
// objektum megadása
var obj = {
    a: 1,
    b: 2
}
// új tulajdonság
obj.c = 3; // VAGY obj['c'] = 3;
// tulajdonság módosítása
obj.b = 42;
// tulajdonság törlése
delete obj.c // VAGY delete obj['c'];
```

VAGY

kód #6:
```
// Üres objektum létrehozása
var obj = Object.create(Object.prototype);
// Tulajdonság létrehozása
Object.defineProperty(obj, 'a', {
    value: 1,
    writable: true,
    configurable: true,
    enumerable: true
});
// Leíróobjektum lekérdezése
var desc = Object.getOwnPropertyDescriptor(obj, 'a');
// desc.writable, desc.configurable ...

// Tulajdonság módosítása és új létrehozása egyszerre
// a objektum módosul, b objektum létrejön
Object.defineProperties(obj, {
    a: {
        value: 42,
        enumerable: false
    },
    b: {
        value: 2
    }
});
```

- objektumszintű metódusok
    - Object.keys(obj): az objektum **felsorolható** tulajdonságainak nevét tartalmazó tömb.
    - Object.getOwnPropertyNames(obj): az objektum **összes** tulajdonságának nevét tartalmazó tömb.
    - Object.preventExtensions(obj): megakadályozza új tulajdonság hozzáadását az objektumhoz.
    - Object.isExtensible(obj): a fenti tulajdonság lekérdezése.
    - Object.seal(obj): a tulajdonságok és leírók attribútumának változtatását tiltja le, kivéve az értékek megváltoztatását.
    - Objcect.isSealed(obj): a fenti tulajdonság lekérdezése.
    - Object.freeze(obj): az objektum befagyasztása, azaz semmiféle változtatás sem engedélyezett az objektumon.
    - Object.isFrozen(obj): a fenti tulajdonság lekérdezése.

## 1.2 prototípus-objektum (későbbiekben prototípus)
### 1.2.1 prototípuslánc
### 1.2.2 prototípus beállítása és lekérdezése
- Object.create(prototípus)
- prototípus lekérdezése
    - a.isPrototypeOf(b): visszaadja, hogy a szerepel-e b prototípusláncában.
    - Object.getPrototypeOf(obj): megadja obj prototípus-objektumát.
    - obj.\_\_proto\_\_: a \_\_proto\_\_ tulajdonság az obj prototípus-objektumára mutat. **Csak pár böngészőben érhető el, így használata nem javasolt.**

kód #7:
```
//A prototípuslánc kialakítása
var obj1 = Object.create(Object.prototype);
var obj2 = Object.create(obj1);

//Tesztek
ok( obj2.__proto__ === obj1, 'obj2 prototípusa obj1' );
ok( obj1.__proto__ === Object.prototype, 'obj1 prototípusa Object.prototype' );
ok( obj1.isPrototypeOf(obj2), 'obj1 szerepel obj2 prototípusláncában' );
ok( Object.prototype.isPrototypeOf(obj2), 'Object.prototype szerepel obj2 prototípusláncában' );
ok( Object.getPrototypeOf(obj2) === obj1, 'obj2 prototípusa obj1' );
```

### 1.2.3 tulajdonság lekérdezése (olvasás)

A keresés végigmegy a prototípusláncon az érintett objektummal kezdve.
Ha nem találja az objektumban, akkor megy tovább a prototípusra, aztán a prototípus prototípusára, amíg meg nem találja a tulajdonságot. Ha nem találja, *undefined* -del tér vissza.
Az alábbi kódban létrehozzuk o1, o2 objektumokat. o2 prototípusa o1. o2.b tulajdonsága eltakarja o1.b tulajdonságát.

kód #8:
```
//A prototípuslánc létrehozása előre feltöltött objektumokkal
var o1 = {
    a: 1,
    b: 2
};
var o2 = Object.create(o1);
o2.b = 22;
o2.c = 3;

o2.b // 22
Object.getPrototypeOf(o2).b // 2

```

### 1.2.4 saját tulajdonságok lekérdezése
- obj.hasOwnProperty('a'): megnézi, hogy a **saját** tulajdonsága-e az obj-nak 

### 1.2.5 tulajdsonságok felsorolása
- for...in bejárás: a prototípuslánc összes felsorolható tulajdonságát figyelembe veszi
- leszűrhetjük a hasOwnProperty metódussal
- Object.keys(obj), Object.getOwnPropertyNames(obj) csak az objektum saját tulajdonságait adják vissza

### 1.2.6 tulajdonság beállítása
- a tulajdonság írása mindig csak az adott objektumot érinti, a prototípusát nem

# 2 Tömbök
- új elem hozzáadása push metódussal
- hosszát a length tulajdonság tartalmazza
- keresés indexOf metódussal
- az objektum tulajdonságként megadott értéket nem rakja a lista közé, így iterálásnál nem veszi azt figyelembe
- tetszőleges indexelés, de a köztes elemek *undefined*-ok lesznek

## 2.1 collection műveletek
- forEach: végigiterál az összes elemen
- map: új tömböt épít a függvény visszatérési értékeiből
- filter: függvény visszatérési értékei alapján épít fel egy új tömböt
- every: eldönti, hogy a tömb összes eleme megfelel-e az átadott függvény viszgálatának
- some: az every-hez hasonló, de itt elég egy visszatérési érték, a végeredmény igaz lesz
- reduce: egy értéket állít elő az iteráció végén. Az előállítandó értéket a második paraméterben inicializálhatjuk, minden iterációs lépésnek átadja az eddig kiszámolt értékeket és az aktuális elemet. A visszatérési érték lesz átadva a következő hívásnak

kód #9:
```
var numbers = [1,2,3];
// forEach
var minimumValue = Number.MAX_VALUE;
numbers.forEach(function(item) {
    if (item < minimumValue) {
        minimumValue = item;
    }
});
// map
var multiplied = numbers.map(function(item) {
    return item * 2;
});
// filter
var smallNumbers = numbers.filter(function(item) {
    if (item < 3) {
        return true;
    }
    return false;
});
// every
var isNumbers = numbers.every(function(item) {
    return typeof item === 'number';
});
// some
var hasThree = numbers.some(function(item) {
    return item === 3;
});
// reduce
var sum = numbers.reduce(function(previous, item) {
    return previous + item;
}, 0);
```

## 2.2 Tömb műveletek
- slice: adott indextől visszaadja egy tömb értékeit
- concat: konkatenál két tömböt
- join: tömbök sztringé alakítása egy szeparátor megadásával
- split: sztringek tömbbé alakítása egy szeparátor megadásával