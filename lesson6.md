# Kódszervezés és modularitás

## 1 Kódszervezési koncepciók általában
### 1.1 Függvény
A kódot nagyon sokféleképpen lehet függvényekre osztani:
- ne legyen ismétlődő kódrészlet (DRY - don't repeat yourself)
- a függvény egy dologért legyen felelős (DOT - do one thing)
- legyen minél egyszerűbb (KISS - keep it simple stupid)
- a kevesebb néha több
- kerüljük a mellékhatásokat 

### 1.2 Osztály
- interface
- egységbe zárás

### 1.3 Modul
A modularitás alapelvei szerint egy modulnak a következő feltételeket kell teljesítenie:

- **Specializált**: a modulnak egy adott feladatot kell jól elvégeznie, publikus interfészének egyszerűnek és érthetőnek kell lennie.
- **Független**: egy modulnak a lehető legkevesebbet kell tudnia a többi modul létezéséről. A direkt kapcsolatok helyett lehetőleg valamilyen közvetítőn keresztül kommunikáljanak (pl. központi eseménykezelő objektum).
- **Leválasztható**: a modult a többi modultól függetlenül is tudni kell használni és tesztelni.
- **Újrafelhasználható**: a modult tudni kell felhasználni különböző helyzetekben, alkalmazásokban más modulokkal együtt.
- **Helyettesíthető**: egy modult mindaddig tudni kell kicserélni, amíg a helyettesítő ugyanazt az interfészt biztosítja.

Az interfészek tervezésére tehát különösen oda kell figyelni. Ezzel kapcsolatban is jól meghatározható néhány alapelv.

- **Nyílt-zárt szabály**: egy modul interfészének nyitottnak kell lennie a bővítésre, de zártnak a módosításra. A módosítás a program többi komponensére is kihat, és elég nagy feladat a változások végigvitele a rendszeren.
- **Megjósolhatóság**: a modul interfészének olyannak kell lennie, hogy egy részének ismeretében következtetni lehessen a többi részére, elkerülve így az állandó keresgélést a referenciában. Következetesnek kell lennie az elnevezésekben, a kis- és nagybetű használatában, vagy egy másik, gyakrabban használt interfész konvencióit veheti át.
- **Többrétegűség**: ha egy feladat elvégzéséhez sok paraméter kell, akkor érdemes egy egyszerűbb interfészt kialakítani a leggyakoribb használathoz igazítva minimális paraméterrel, és egy bonyolultabbat, ahol az összes paraméter tetszés szerint testre szabható.

## 2 Kódszervezés JavaScriptben
### 2.1 Függvények
A kód alapvető strukturálására természetesen JavaScriptben is a függvények szolgálnak. 

kód #1:
```javascript
var elems = [],
    push = function (e) {
        elems.push(e);
    },
    pop = function () {
        return elems.pop();
    },
    top = function () {
        return elems[size()-1];
    },
    size = function () {
        return elems.length;
    };
```

### 2.2 Objektumok

kód #2:
```javascript
var stack = {
    elems: [],
    push: function (e) {
        this.elems.push(e);
        return this;
    },
    pop: function () {
        return this.elems.pop();
    },
    top: function () {
        return this.elems[this.size()-1];
    },
    size: function () {
        return this.elems.length;
    }
};
stack.push(10).push(20);
```

### 2.3 Névterek
kód #3:
```javascript
var myApp = myApp || {};
myApp.dataStructures = myApp.dataStructures || {};
myApp.dataStructures.stack = {
    elems: [],
    push: function (e) { /* ... */ },
    pop:  function () { /* ... */ },
    top:  function () { /* ... */ },
    size: function () { /* ... */ }
};
```

### 2.4 Modul minta
Lényeges különbség az objektumliterálos formával szemben, hogy a privát tagokra a this nélkül, míg a publikus tagokra a this kulcsszón keresztül kell hivatkozni.

kód #4:
```javascript
var module = (function () {
    //Inicializáló kód
    //Rejtett változók és függvények
    //Visszatérés egy objektummal
    return {
        //Publikus interfész
    };
})();

// Példa
var stack = (function () {
    var elems = [];
    return {
        push: function (e) {
            elems.push(e);
            return this;
        },
        pop: function () {
            return elems.pop();
        },
        top: function () {
            return elems[this.size()-1];
        },
        size: function () {
            return elems.length;
        }
    };
})();

```

### 2.5 A modul minta variánsai
#### 2.5.1 Névterek
kód #5:
```javascript
MyApp.namespace('dataStructures.stack') = (function () {
    /* ... */
})();
```

#### 2.5.2 Globális változók importálása
Globális változókat az önkioldó függvény paramétereként lehet megadni. Így akár ugyanannak a függvénykönyvtárnak több különböző verziója is a modul hatókörébe injektálható.

kód #6:
```javascript
var module = (function (win, doc, $, undefined) {
    /* ... */
})(window, document, jQuery);
```

#### 2.5.3 Modul módosítása
kód #7:
```javascript
var module = (function (module) {
    var old_method = module.method;
    /* ... */
    return extendDeep(module, {
        //Kiegészítés, illetve felülírás
        method: function () {
            var old = old_method();
            /* ... */
        }
    })
})(module || {});
```

#### 2.5.4 Gyárfüggvény visszaadása
Sokszor célszerű nem egy objektumot, hanem egy függvényt, sőt egy gyárfüggvényt visszaadni. Ez történhet úgy, hogy az önkioldó függvényt egyszerű függvényre cseréljük, de úgy is, hogy az önkioldó függvényen belül definiált függvényt adja vissza az önkioldó függvény (ld. a gyárfüggvények becsomagolását az objektumokról szóló fejezetben).

kód #8:
```javascript
var module = function () {};
//vagy
var module = (function () {
    var Constr = function ()  {
        /* ... */  
    };
    return Constr;
})();
```

#### 2.5.5 Felfedő modul minta
A felfedő modul minta az eredeti modul mintának azt a hiányosságát orvosolja, hogy másképpen kell hivatkozni a privát és a publikus adattagokra és metódusokra. Ebben a mintában minden adatot és metódust a függvény rejtett részében deklarálunk, majd a visszatérési értékként megadott objektum publikus interfészét képviselő tulajdonságaihoz rendeljük hozzá a rejtett adattagokat és metódusokat. A hivatkozás így mindig a this nélkül történik

kód #9:
```javascript
var stack = (function () {
    var elems = [],
        push = function (e) {
            elems.push(e);
            return this;
        },
        pop = function () {
            return elems.pop();
        },
        top = function () {
            return elems[size()-1];
        },
        size = function () {
            return elems.length;
        };
    return {
        push: push,
        pop: pop,
        top: top,
        size: size
    };
})();
```

### 2.6 Alkalmazásfüggetlen modul minta
A modul minta egy másik nagy hátránya, hogy alkalmazásával mindig bővül a globális névtér. Erre megoldásként a névterezés szolgál, de ebben az esetben pedig a modul definíciójába bekerül az alkalmazás névtere, így egy másik alkalmazásba nem lehet a modult átírás nélkül újra felhasználni.

Az általános megoldást az jelenti, hogy egy előre megadott exports nevű objektumot bővítünk a modulon belül, és a modulon kívül adjuk át az exports konkrét értékét. Ezzel a modult mindenféle körülménytől (globális objektumtól, névterektől) függetlenítettük, és a felhasználási helye dönti el, hogy mihez adja hozzá a modult. Ezt a mintát alkalmazzák a modern modulkezelő könyvtárak.

kód #10:
```javascript
(function (exports) {
    /* ... */
    extendDeep(exports, {
        //Publikus interfész
    })
})(exports);
```

### 2.7 Homokozó minta (Sandbox)
A homokozó minta az alkalmazásfüggetlen modul mintához hasonlóan a globális névtérszennyezést próbálja orvosolni (gyakorlatilag ugyanazzal a módszerrel, miszerint egy függvényben várja el a megvalósítást), de ezen túl biztosít számára közös eszközöket úgy, hogy közben nem interferál a modul más modul megoldásaival.


## 3 Modulkezelő könyvtárak
JavaScriptben a modulok egységes kezelésére tett kísérletek mára két kikristályosodott módszert eredményeztek: **CommonJS** és **AMD**.

Mindegyik modulkezelőnek két alapvető eleme van:
- modulok definiálása (exportálás)
- függőségek kezelése (importálás)

### 3.1 CommonJS
 A CommonJS szabvány szerinti modul mintát a szerveroldali JavaScriptes környezetekben használják, és kiválóan alkalmas modulok szinkron betöltésére. A CommonJS szabványnak igen egyszerű a szintaxisa. Minden modul külön fájlban helyezkedik, így nincs is szükség a modult körülvevő, hatókört biztosító függvényre. Minden fájl külön hatókörrel bír. A fájlon belül az exportálandó API-t az exports nevű objektumhoz kell fűzni. Egy modult használni a require kulcsszóval lehet

kód #11:
```javascript
// stack.js
var elems = [],
    push = function (e) { /* ... */ },
    pop = function () { /* ... */ },
    top = function () { /* ... */ },
    size = function () { /* ... */ };

extendDeep(exports, {
    push: push,
    pop: pop,
    top: top,
    size: size
});

// app.js
var stack = require('./stack.js');
```

### 3.2 Aszinkron Modul Definíció – AMD
Az AMD kulcseleme a modulok definiálásáért felelős define() függvény. 
- Ennek első paramétere az opcionálisan megadható modulnév, amit általában nem adunk meg a modul nagyobb hordozhatósága érdekében (névtelen modulok). 
- Második paramétere a modul függőségeit leíró tömb, amelyben az importálandó modulok nevei vannak felsorolva. 
- Utolsó paramétere pedig egy függvény, mely a modul definícióját tartalmazza (ez a hatókört biztosító függvény).

Ez a függvény explicit paraméterekként kapja meg a függőségi listában szereplő modulokat. A függvény interfészét visszatérési értékként kell megadni, akárcsak a klasszikus modul mintában.

kód #12:
```javascript
define(modulnév, [modul1, modul2], function (modul1, modul2) {
    //Modul definíciója
    return {
        //Publikus API
    }
});

// stack.js
define('stack', [], function () {
    var elems = [],
        push = function (e) { /* ... */ },
        pop = function () { /* ... */ },
        top = function () { /* ... */ },
        size = function () { /* ... */ };
    return {
        push: push,
        pop: pop,
        top: top,
        size: size
    };
});

// meghívás
require(['stack'], function (stack) {
    stack.push(10).push(20);
});

```

Az AMD szabványt számos modulbetöltő függvénykönyvtár implementálta. Legnépszerűbbek:
- RequireJS
- curl.js

### 3.3 CommonJS vs.AMD
- http://chimera.labs.oreilly.com/books/1234000000262/ch04.html#node-modules
- http://addyosmani.com/resources/essentialjsdesignpatterns/book/#detailamd
- http://addyosmani.com/resources/essentialjsdesignpatterns/book/#detailcommonjs
- http://requirejs.org/docs/whyamd.html

### 3.4 UMD
- http://addyosmani.com/resources/essentialjsdesignpatterns/book/#detailcommonjs 

### 3.5 EcmaScript 6 modulok
Az EcmaScript szabvány 6-os verziója kész megoldást próbál adni a modulok egységes kezelésére, mely egyaránt használható a szerveren és a böngészőben. Modul definiálására a module kulcsszó használatos. Modulon belül az import kulcsszóval tudjuk a függőségeinket megadni, a publikus interfészt az export kulcsszóval kell jelezni.

kód #13:
```javascript
module staff{
    // specify (public) exports that can be consumed by
    // other modules
    export var baker = {
        bake: function( item ){
            console.log( "Woo! I just baked " + item );
        }
    }
}
module skills{
    export var specialty = "baking";
    export var experience = "5 years";
}
module cakeFactory{
    // specify dependencies
    import baker from staff;
    // import everything with wildcards
    import * from skills;
    export var oven = {
        makeCupcake: function( toppings ){
            baker.bake( "cupcake", toppings );
        },
        makeMuffin: function( mSize ){
            baker.bake( "muffin", size );
        }
    }
}
```