# Objektum-orientált minták JavaScriptben
## 1 Prototípusosság mint a kód-újrahasznosítás eszköze
- a javascript protoípusos nyelv: lehetőség van egy objektumot egy másik objektum mintájára elkészíteni

## 2 Kód-újrahasznosítás (prototípusosság) dinamikus objektumokkal
### 2.1 Tulajdonságok másolása – a Mixin minta
- az újrahasznosítandó kódrészletet egyszerűen bemásoljuk a célobjektumba

kód #1:
```javascript
var o1 = {
    a: 1,
    b: 2
};
var o2 = {
    b: 42,
    c: 3
};
//Mixin
o2.a = o1.a;
o2.b = o1.b;
```

### 2.2 Az extendShallow() segédfüggvény
- hogy ne tulajdonságról tulajdonságra kelljen a másolást megtenni, érdemes egy segédfüggvényt bevezetni

kód #2:
```javascript
var extendShallow = function (objTo, objFrom) {
    if (arguments[2] && typeof arguments[2] === 'string') {
        for (var i = 2; i < arguments.length; i++) {
            var propName = arguments[i];
            objTo[propName] = objFrom[propName];
        }
    } else {
        [].slice.call(arguments, 1).forEach(function(source) {
            for (var prop in source) {
                if (source.hasOwnProperty(prop)) {
                    objTo[prop] = source[prop];
                }
            }
        });
    }
    return objTo;
};

// Példa
var oFrom = {
    arr: [],
    obj: {
        a: 1
    },
    put: function (elem) {
        this.arr.push(elem);
    }
};
//Célobjektumok
var o1 = extendShallow({}, oFrom);
var o2 = extendShallow({}, oFrom);

```

### 2.3 Referenciatípusok másolása
- objektumok, tömbök és függvények másolása csak referencia szinten, vagyis az új/másolt objektumok közös referenica tulajdonságokat használnak
- elemi objektumoknál egy új érték létrejöttét jelenti, hiszen ezek érték szerint másolódnak, összetett objektumoknál azonban a másolás csak referenciaszinten valósul meg, a belső objektum tulajdonságai közösek. Ha viszont az egész objektumot lecseréljük, akkor megszűnik az egyezés.

### 2.3 Tömbök és objektumok másolása – az extendDeep() segédfüggvény
- a referenciatípusok érték szerinti másolása, KIVÉVE a függvényeket, melyek referencia szerint hívhatók
- rekurzívan hívjuk meg a függvényt. Addig megy, amíg egyszerű értéket nem talál.

kód #3:
```javascript
var extendDeep = function (objTo, objFrom) {
    var copy = function (objTo, objFrom, prop) {
        if (typeof objFrom[prop] === "object") {
            objTo[prop] = (Object.prototype.toString.call(objFrom[prop]) === '[object Array]') ? [] : {};
            extendDeep(objTo[prop], objFrom[prop]);
        } else {
            objTo[prop] = objFrom[prop];
        }
    }

    if (arguments[2] && typeof arguments[2] === 'string') {
        for (var i = 2; i < arguments.length; i++) {
            var prop = arguments[i];
            copy(objTo, objFrom, prop);
        }
    } else {
        [].slice.call(arguments, 1).forEach(function(source) {
            for (var prop in source) {
                if (source.hasOwnProperty(prop)) {
                    copy(objTo, source, prop);
                }
            }
        });
    }
    return objTo;
};
```

### 2.4 Metódusok kötése – az extendBind() segédfüggvény
- oly módon másolja át egy modellobjektum összes metódusát, hogy a this objektum kontextusa meghatározható

kód #4:
```javascript
var extendBind = function (objTo, objFrom, context) {
    context = context || objFrom;
    for (var prop in objFrom) {
        if (objFrom.hasOwnProperty(prop) && typeof objFrom[prop] === 'function') {
            objTo[prop] = objFrom[prop].bind(context);
        }
    }
    return objTo;
}
```

### 2.5 Tulajdonságok klónozása – funkcionális bővítés és az extendFunc() segédfüggvény
- funkcionális bővítésnek nevezzük

kód #5:
```javascript
// Egyszerű használat
var funcFrom = function funcFrom() {
    this.arr = [];
    this.obj = {
        a: 1
    };
    this.put = function (elem) {
        this.arr.push(elem);
    };
    return this;
};
var o1 = funcFrom.call();
var o2 = funcFrom.call();

// Segédfüggvény használat
var extendFunc = function extendFunc(obj) {
    [].slice.call(arguments, 1).forEach(function(source) {
        if (typeof source === 'function') {
            source.call(obj);
        }
    });
    return obj;
}

var o1 = extendFunc(, func1, func2);
```


### 2.6 Összefoglalás
- **extendShallow():** egyik objektumot másolja a másikba. Az értéktípusok érték szerint másolódnak, azaz célobjektumonként eltérő értékek lehetnek. A referenciatípusok (objektumok, tömbök, függvények) referencia szerint másolódnak, azaz a célobjektumok egy közös értéken osztoznak.
- **extendDeep():** egyik objektum rekurzív mélymásolása a másikba. Az értéktípusok érték szerint másolódnak, azaz célobjektumonként eltérő értékek lehetnek. A referenciatípusok közül egyedül a függvények másolódnak referencia szerint, azaz a célobjektumok a modellobjektum metódusain közösen osztozkodnak.
- **extendBind():** metódusok másolására szolgál, beállítható a metódusok this objektuma.
- **extendFunc():** funkcionalitást injektál az objektumba. Alapvetően függvényekkel dolgozik, amelyekbe kell a funkcionalitást csomagolni. Minden becsomagolt tulajdonság klónozódik. az célobjektumonként eltérőek

## 3 Kód-újrahasznosítás (prototípusosság) prototípus-objektummal
- az újrahasznosítandó kódot tartalmazó modellobjektumot most nem bemásoljuk a célobjektumba, hanem azt prototípus-objektumként vesszük fel

kód #6:
```javascript
var oFrom = {
    a: 1,
    obj: {
        l: true
    },
    hello: function () {
        return 'hello';
    }
};
//Célobjektumok
var o1 = Object.create(oFrom);
var o2 = Object.create(oFrom);
```

## 4 Objektum-létrehozási minták
### 4.1 Egy objektum létrehozása
- egyszerűen az objektumliterállal {}
- Object.create metódussal

### 4.2 Több objektum létrehozása – Gyár (Factory) minta
- Ha több ugyanolyan funkcionalitású objektumot szeretnénk létrehozni, akkor egy olyan függvényt kell készítenünk, amely a megfelelő objektumot adja vissza. Ez nem más, mint a Gyár (Factory) minta.

kód #7:
```javascript
var child = {
    create: function create() {
        return {
            name:        /*...*/,
            dateOfBirth: /*...*/,
            getName:     /*...*/,
            setName:     /*...*/
        };
    }
};
var c1 = child.create();
var c2 = child.create();
```

### 4.3 Paraméteres gyárfüggvények
- egy inicializáló objektumot adunk át a gyárfüggvénynek, amivel felülírjuk a modellobjektumból származó tulajdonságok értékét

kód #8:
```javascript
var child = {
    create: function create(props) {
        return extendDeep({
            name:        'Attila',
            dateOfBirth: { year: 1940, month: 4, day: 1 },
            getName:     function(){ return this.name; },
            setName:     function(name){ this.name = name; }
        }, props || {});
    }
};
var c1 = child.create();
var c2 = child.create({
    name: 'Zsófi',
    dateOfBirth: {
        year: 2006,
        month: 9,
        day: 1
    }
});
```

### 4.4 Privát adattagok és metódusok
- JavaScriptben az objektumok adattagjai és metódusai mind publikusak, a nyelvben nincsen speciális szintaxis privát, védett vagy publikus tulajdonságok létrehozására. **Closure** segítségével azonban létrehozhatók **privát adattagok és metódusok**. Ekkor a gyárfüggvényen belül kell ezeket a privát tulajdonságokat definiálni, amik kívülről közvetlenül nem érhetők el, viszont megmaradnak a gyárfüggvény closure-jében, és a publikus metódusokon keresztül meghívhatók. Azokat a publikus függvényeket, amelyek privát adattagokat vagy metódusokat használnak, privilegizált metódusoknak hívjuk.

kód #9:
```javascript
var child = {
    create: function create(props) {
        //Privát adattag
        var secretNickName = '';
        return extendDeep({
            //Publikus adattagok és metódusok
            name: 'Anonymous',
            dateOfBirth: {
                year: 1970,
                month: 1,
                day: 1
            },
            getName: function getName() {
                return this.name ;
            },
            setName: function setName(name) {
                this.name = name;
            },
            //Privilegizált metódusok
            setSecretNickName: function (name) {
                secretNickName = name;
            },
            getSecretNickName: function () {
                return secretNickName;
            }
        }, props || {});
    }
};
var c1 = child.create();
c1.setSecretNickName('Fairy');
```

### 4.5 Metódusok hatékony tárolása
- állapotteret objektumonként,
- metódusokat közös helyen.

### 4.6 Metódusok hatékony tárolása bővítéssel

kód #10:
```javascript
var child = (function child() {    
    var childProto = {
        name:        /*...*/,
        dateOfBirth: /*...*/,
        getName:     /*...*/,
        setName:     /*...*/
    };
    return {
        create: function create(props) {
            return extendDeep({}, childProto, props || {});
        }
    }
})();
var c1 = child.create();
var c2 = child.create();
```

### 4.7 Metódusok hatékony tárolása bővítéssel 2.

kód #11:
```javascript
var child = (function child() {    
    var methodsProto = {
        getName:     /*...*/,
        setName:     /*...*/
    };
    var dataProto = {
        name:        /*...*/,
        dateOfBirth: /*...*/
    };
    return {
        create: function create(props) {
            var obj = extendShallow(, methodsProto);
            return extendDeep(obj, dataProto, props || {});
        },
        methodsProto: methodsProto,
        dataProto: dataProto
    }
})();
var c1 = child.create();
var c2 = child.create();
```

### 4.8 Metódusok hatékony tárolása prototípuslánccal
- A másik lehetőség, hogy a metódusokat egy objektumba helyezzük, majd ezt adjuk meg a létrejövő objektumok prototípusának. A létrejövő objektumokba pedig csupán az adattagokat másoljuk. A metódusok objektumát is elérhetővé tesszük a methods tulajdonságon későbbi felhasználás céljából (ld. öröklés lejjebb).

kód #12:
```javascript
var child = (function() {
    var methods = {
        getName: /*...*/,
        setName: /*...*/
    };
    var data = {
        name:        /*...*/,
        dateOfBirth: /*...*/
    };
    return {
        create: function(props) {
            return extendDeep(
                Object.create(methods),
                data,
                props || {}
            );
        },
        methods: methods
    };
})();
```


### 4.9 Metódusok hatékony tárolása és a privát adattagok
- JavaScriptben nem lehet megtenni azt, hogy a privilegizált metódusokat közös helyre tegyük. Ennek az egyszerű oka az, hogy privát adattagok csak closure-ben létezhetnek. A closure viszont függvényhez tartozik. Egy függvény, egy closure. Ha tehát objektumonként szeretnénk privát adatokat tárolni, akkor annyi closure-t és ennek megfelelően annyi függvényt is kell létrehozni. És csak ezen függvényen belüli függvények érik el a privát adattagokat. A closure-t kívülről elérni vagy paraméterként átadni nem lehet.
- A privát adattagokat és privilegizált metódusokat objektumonként kell felvenni továbbra is a gyárfüggvényen belül.

kód #13:
```javascript
var child = (function() {
    var publicMethods = {
        getName: /*...*/,
        setName: /*...*/
    };
    var publicData = {
        name:        /*...*/,
        dateOfBirth: /*...*/
    };
    return {
        create: function(props) {
            //Privát adattag
            var secretNickName = /*...*/;
            //Privilegizált metódusok
            var privilegedMethods = {
                setSecretNickName: /*...*/,
                getSecretNickName: /*...*/
            };
            return extendDeep(
                Object.create(publicMethods),
                privilegedMethods
                publicData,
                props || {}
            );
        },
        methods: publicMethods
    };
})();
```

## 5 Öröklési minták
### 5.1 Öröklés prototípuslánccal
### 5.1 Öröklés bővítéssel – kompozíció
## 6 Magas szintű segédfüggvények objektumok létrehozására
### 6.1 Modellobjektumok
### 6.2 Prototípusláncot alkalmazó gyárfüggvények készítése
### 6.3 Kompozíciót alkalmazó gyárfüggvények készítése
## 7 Objektumok létrehozása klasszikus objektum-orientált szintaxissal
### 7.1 A konstruktorfüggvények
### 7.2 A konstruktorhívás folyamatának háttere
### 7.3 A konstruktorhívási minta alkalmazásának hátrányai
### 7.4 Gyárfüggvények
### 7.5 Paraméteres gyárfüggvények
### 7.6 Privát adattagok és privilegizált metódusok
### 7.7 Metódusok hatékony tárolása
### 7.8 Becsomagolás „osztály-modulba”
### 7.9 Öröklés
## 8 EcmaScript 6 újdonságok az objektumkezelésben