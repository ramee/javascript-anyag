# Magas szintű programszervezés és alkalmazásarchitektúrák
## 1 Kliensoldali webes alkalmazások fejlődése
- egyoldalas alkalmazásoknak
- programkód megfelelő szervezése

## 2 Kódszervezési problémák
- modulok, publikus interfész
- modulok kusza hálója
- keverednek a *felület író műveletek*, *eseménykezelés*, *megjelenítés az adatokkal* és a *tárolás módja*
- egy alkalmazás feladatai: 
    - modulok kezelése
    - események kezelése
    - DOM kezelése, elemek megjelenítése
    - adatok és üzleti logika definiálása
    - adatok tárolása, mentése (pl. AJAX)
    - URL kezelése

## 3 Modulok kezelése
A modulkezelés tipikusan a következő funkciókból áll össze:
- névterezés (névtér minta)
- minden modul számára egy közös alapfunkcionalitás biztosítása (homokozó minta)
- modulok élettartamának kezelése (regisztrálás, indítás, megállítás)
- eseménykezelő rendszer biztosítása a modulok kommunikációjához
- alkalmazásszintű adatok kezelése
- bővítmények definiálása

A modulkezelő általában előír egy szabványos modulinterfészt, amelyet a moduloknak implementálniuk kell. Ez a legegyszerűbb esetekben a modul elindítását és leállítását végző műveletek.

kód #1:
```javascript
var moduleInterface = {
    start: function () ,
    stop: function () 
};
```

- homokozó objektum: egy-egy példányát minden modul megkapja. A homokozó objektum tartalmazza az összes olyan szolgáltatást, amelyet egy modul az alkalmazásból használhat. Ez általában egy homlokzat mintát valósít meg, absztrakt interfész mögé rejtve a konkrét megvalósításokat. Ezen keresztül lehet eseményeket küldeni vagy feliratkozni rájuk, a DOM-mal kapcsolatos műveleteket elérni, stb. A homokozó tulajdonképpen egy minden modul számára közös API-t jelent, így ennek megtervezése különösen figyelmet igényel, mert később nehezen változtatható

kód #2:
```javascript
var sandboxProto = extendDeep({
    //...
}, observable);

// egy modul definíciója így nézhet ki:
var module1 = function (sandbox) {
    //A modul implementációja
    //sandbox használata
    //Publikus API meghatározása
    return extendDeep(Object.create(moduleInterface), {
        //...
    });
};

// korábbi példával
var weatherModule = function (sandbox) {
    var state = 'sunny',
        changeState = function (newState, hours) {
            state = newState;
            sandbox.notify(newState, hours);
        };
    return extendDeep(Object.create(moduleInterface), {
        changeState: changeState
    });
};
var smartphoneModule = function (sandbox) {
    var warnForRain = function (hours) {
            console.log('Rain is coming in ', hours, 'hours!');
        },
        prepareForSun = function (hours) {
            console.log('The sun will shine in ', hours, 'hours!');
        },
        init = function () {
            sandbox.addObserver('rainy', warnForRain);
            sandbox.addObserver('sunny', prepareForSun);
        };
    return extendDeep(Object.create(moduleInterface), {
        init: init
    });
};

```

A modulok kezelését egy alkalmazásobjektum végzi. Feladata a modulok regisztrálása, indítása és leállítása, a modulok működésében felmerülő hibák kezelése, a modulok közötti kommunikáció biztosítása.

kód #3:
```javascript
var application = (function () {
    var modules = {};
    return {
        register: function (moduleId, module) {
            var moduleInstance = module(Object.create(sandboxProto));
            modules[moduleId] = moduleInstance;
            return moduleInstance;
        },
        start: function (moduleId) {
            modules[moduleId].start();
        },
        stop: function (moduleId) {
            modules[moduleId].stop();
        },
        startAll: function () {
            for (moduleId in modules) {
                if (modules.hasOwnProperty(moduleId)) {
                    this.start(moduleId);
                }
            }
        }
        //...
    };
})();

// PÉLDA
application.register('weather', weatherModule);
application.register('smartphone', smartphoneModule);
application.startAll();
```

Az elképzelést számos függvénykönyvtár implementálta:
- [Terrific](http://terrifically.org/)
- [Aura](http://aurajs.com/)
- [scaleApp](http://scaleapp.org/)
- [Kernel.js](http://alanlindsay.me/kerneljs/)
- [Hydra.js](http://tcorral.github.io/Hydra.js/)

## 4 Események kezelése
### 4.1 Eseményt kiváltó objektumok
- a megfigyelő minta hátránya, hogy a megfigyelőnek referenciát kell tartalmaznia a megfigyelt objektumokra

### 4.2 Eseménygyűjtők
- központi eseményvezérlő minta (mediator + observer)
- homokozó objektum

### 4.3 Eseménybuszok
Az **eseménygyűjtők** egyik nagy **hátránya**, hogy semmi **nem biztosítja**, hogy az **esemény feldolgozásra** is kerül az üzenet kézbesítésekor. Az eseménybuszok egy jól kontrollált és adminisztrált környezetet hoznak létre az üzenetek kezelésére. A beérkező eseményeket feljegyzik, a megfelelő címzetthez továbbítják, és nyomon követik állapotukat, így azok mindig ellenőrizhetők. Az üzenetek általában valamilyen jól meghatározott formában utaznak.

### 4.4 Eseménysorok
Egy speciális eseménybusz, amely elsősorban akkor hasznos, ha az eseményt kiváltó és az eseményt feldolgozó objektum eltérő sebességgel dolgozik, vagy szükséges az eredeti sorrend megtartása.

### 4.5 A megfelelő eseménykezelési stratégia kiválasztása
Mindegyik fenti eseménykezelési megoldásnak megvan a maga helye, egyik sem ad teljes megoldást minden problémára.
- **Aszinkron viselkedést** nyújtó függvénykönyvtárak esetében érdemes **megfelelő eseményeket** kiváltani. Mivel ezeket a könyvtárakat különböző környezetekben használhatják, így nem tudhatunk semmit az eseményekre feliratkozókról, vagy egy központi eseménygyűjtő objektumról.
- Alkalmazásunk egyes részei közötti **kommunikációnál** az **eseménygyűjtő** használata javasolt, ugyanis ezzel biztosítható az egyes részek lazán kapcsoltsága.
- **Nagy méretű alkalmazásoknál** az egyes nagyobb rétegek közötti megbízható kommunikációhoz lehet **eseménybuszokat** használni.

## 5 MN* architektúrák
- modell: feladat elvégzéséhez tartozó adatok és a hozzájuk kapcsolódó feldolgozó függvények. Ebben nem jelenhetnek meg HTML elemek és nem dolgozhatnak fel felületről jövő eseményeket
- nézet: felhasználói felület kezelésével kapcsolatos logikát tartalmazza. HTML elemek manipulálása és események feldolgozása
- vezérlő: maradék logikáért felelős rész. Ennek a logikának nem mindig egyértelmű a feladatköre, ezért *-gal helyettesítik.

### 5.1 Az MNV minta
- Smalltalk-80 tervezésekor 1979-ben
Jellegzetességei a következők voltak:
- A modell teljesen független volt a felhasználói felülettől.
- A megjelenítéssel kapcsolatos logikát a nézet és a vezérlő végezte el közösen, mindig egy nézet-vezérlő párra volt szükség.
- A vezérlő a felhasználótól érkező bemeneti adatok kezeléséért és feldolgozásáért felelt.
- A megfigyelő mintát alkalmazták a nézet frissítéséhez a modell változásakor.

Az MVC minta egyik klasszikusnak mondott képviselője, a **Backbone** keretrendszer, például **nem** definiál külön **vezérlőt**, annak a logikáját a **nézet** logikájába olvasztja bele. Ugyanakkor külön szereplőként jelenik meg a **Router**, amely az URL kezelését végzi, és a változásakor az útvonal alapján választja ki a futtatandó kódot.

### 5.2 Az MNP minta
- 1990-es évek elején
- Modell-Nézet-Prezenter
- a nézetet igyekeznek mentesíteni mindenféle logikától (passzív nézet). Egyetlen feladata, hogy a felületről érkező eseményeket továbbítsa a prezenternek
- a prezenter átveszi a megjelenítési és a bemenet feldolgozását végző logikát. A felületi események alapján módosítja a modellt, akiben bekövetkező változásokra ugyancsak ő figyel
- JavaScriptes világban viszonylag kevés keretrendszer épít erre a mintára.

### 5.3 Az MNNM minta
A Modell-Nézet-Nézetmodell minta még jobban próbálja elválasztani az adatokat és a hozzá tartozó üzleti logikát a nézettől és a megjelenítéshez használt logikát. A modell továbbra is a feladat elvégzéséhez szükséges adatokat tartalmazza, de egyáltalán nem hordoz viselkedésbeli információkat, vagy olyanokat, amelyek a megjelenítéshez kellenek (mint pl. egy dátum formátuma). Ehhez definiálja a Nézetmodellt, ami a modell bizonyos tulajdonságait teszi elérhetővé a nézet számára, tartalmazza az ehhez kapcsolódó logikát, és dolgozza fel a felületről érkező inputot. Egyfajta vezérlőként viselkedik, ami továbbítja a modelladatokat a nézetnek, és a nézetbeli változásokat a modellnek. Nagyon sokban inkább a modellre hasonlít, hiszen adatokkal dolgozik, de a megjelenítésért is felel. A fő különbség ebben a mintában a nézet és a nézetmodell viszonyában van. A nézetmodell ugyanis semmit nem tud a megjelenítésre szolgáló HTML struktúráról. A nézet ebben az esetben passzív olyan szempontból, hogy benne semmilyen logika nincsen. A nézetben deklaratív módon kell jeleznünk, hogy a felületi elemek tulajdonságai a nézetmodell melyik attribútumához kapcsolódnak. Az MNNM minta jellegzetessége ez a deklaratív kétirányú kötés. A nézetmodellben bekövetkező változások automatikusan a felületi elemeken is látszódnak, a felületi elemek tulajdonságaiban bekövetkező változások a nézetmodellben is érvényesülnek. A nézetnek nincsen állapota, a nézetmodellnek van.

### 5.4 Az MN* architektúrák létjogosultsága
Van létjogosultságuk :D

## 6 MN* architektúrák tipikus eszközei
### 6.1 Megfigyelhető modellek
A fent említett MN* architektúrák egyik közös jellemzője, hogy a modell (vagy nézetmodell) változás esetén eseményt bocsát ki, amin keresztül értesíti az őt figyelőket. Tipikusan a nézet iratkozik fel a modell változásaira, és automatikusan frissíti az oldal megjelenését a változásoknak megfelelően. Az egyes architektúrák által igényelt adatkötés is erre a mechanizmusra épít.
A modellek ilyen viselkedése alapvetően két komponensből áll:
- a modellt megfigyelhetővé kell tenni a megfigyelő minta szerint;
- a modellben történt változásokkor eseményt kell kibocsátani.

A második részre nincs még általános megoldás. Az ES szabvány kisérletezik vele, de kicsi a támogatottsága.

#### 6.1.1 get() és set() metódusok
Az egyik megközelítés szerint a modellnek csak egy részét képezik a reprezentatív adatok, így ezek a modell egy speciális adattagjaként jelennek meg (pl. attributes), és ezeket explicit get() és set() függvényekkel lehet lekérdezni vagy beállítani. (Ilyen megoldással él pl. a Backbone.js keretrendszer.)

kód #4:
```javascript
// Példa
var book = {
    author: 'Stoyan Stefanov',
    title: 'JavaScript Patterns'
};
var observableBook = makeObservable(book);
observableBook.addObserver('change', function (book) {
    console.log('Book changed', book);
});
observableBook.addObserver('change:title', function (title) {
    console.log('Title changed', title);
});
observableBook.set('title', 'JavaScript Patterns rev.3.');
var title = observableBook.get('title');


// Megvalósítás
var makeObservable = function(o) {
    return extendDeep(, observable, {
        get: function (attr) {
            return this.attributes[attr];
        },
        set: function (attr, val) {
            this.attributes[attr] = val;
            this.notify('change:' + attr, val);
            this.notify('change', this.attributes);
        },
        attributes: extendDeep(, o)
    })
};
```

#### 6.1.2 Natív getterek és setterek
ES5 már támogatja a getter-setter párost, így a megfigyelhető objektumot nem kell modellbe csomagolni.

kód #5:
```javascript
// Példa
var book = {
    author: 'Stoyan Stefanov',
    title: 'JavaScript Patterns'
};
var observableBook = makeObservable(book);
observableBook.addObserver('change', function (book) {
    console.log('Book changed', book);
});
observableBook.addObserver('change:title', function (title) {
    console.log('Title changed', title);
});
observableBook.title = 'JavaScript Patterns rev.3.';
var title = observableBook.title;


//Megvalósítás
var makeObservable = function(o) {
    var newO = extendDeep(, observable);
    var attributes = ;
    for (var i in o) {
        if (o.hasOwnProperty(i)) {
            attributes[i] = o[i];
            Object.defineProperty(newO, i, {
                enumerable: true,
                get: function() {
                    return attributes[i];
                },
                set: function(val) {
                    attributes[i] = val;
                    this.notify('change:' + i, val);
                    this.notify('change', attributes);
                }
            });
        }
    }
    return newO;
};
```

#### 6.1.3 Adattagok metódusokká alakítása
Egy harmadik megoldás azzal él, hogy az adattagokat egyszerűen függvénnyé alakítja. Ez akkor lehet hasznos, ha olyan böngésző támogatása is cél, amely nem implementálta még a natív getter és setter funkcionalitást. (Ezt a megközelítést használja a Knockout.js keretrendszer.)

kód #6:
```javascript
var book = {
    author: 'Stoyan Stefanov',
    title: 'JavaScript Patterns'
};
var observableBook = makeObservable(book);
observableBook.addObserver('change', function (book) {
    console.log('Book changed', book);
});
observableBook.addObserver('change:title', function (title) {
    console.log('Title changed', title);
});
observableBook.title('JavaScript Patterns rev.3.');
var title = observableBook.title();
//Megvalósítás
var makeObservable = function(o) {
    var newO = extendDeep(, observable);
    var attributes = ;
    for (var i in o) {
        if (o.hasOwnProperty(i)) {
            attributes[i] = o[i];
            newO[i] = function(val) {
                if (arguments.length > 0) {
                    attributes[i] = arguments[0];
                    this.notify('change:' + i, val);
                    this.notify('change', attributes);
                } else {
                    return attributes[i];
                }
            }
        }
    }
    return newO;
};
```

### 6.2 Sablon-kezelés (template)
Manapság már nagyon sokféle sablonleírással találkozhatunk: **Jade**, **Mustache**, **Handlebars**, **Haml**, **EJS**, **Plates**, **Pure**, illetve az **Underscore** függvénykönyvtárnak is van egy sablonkezelő segédfüggvénye. Nézzük meg az előző példát néhány sablonnyelvben.

kód #7:
```
// Jade
li
  a(href='show/'+id)= title

// Handlebars.js
<li>
    <a href="show/{{id}}">{{title}}</a>
</li>

// Underscore.js
<li>
    <a href="show/<%= id %>"><%= title %></a>
</li>
```

Az így megadott sablonokat szövegként is meg lehet adni, de jelezvén, hogy ez már nem a nézet logikai részéhez tartozik, vagy külön fájlba helyezhetjük, vagy egyszerűen a HTML dokumentumba ágyazzuk egy megfelelő típusú script elembe:

kód #8:
```html
<script id="listItem" type="text/template">
    <li>
        <a href="show/<%= id %>"><%= title %></a>
    </li>
</script>
```

A speciális type attribútum miatt a böngésző nem értelmezi a script elem tartalmát, viszont azt az id-n keresztül az innerHTML tulajdonsággal el lehet érni, és átadni a sablonfeldolgozónak.

## 7 Példa MNV keretrendszer megvalósítása (minimvc a forrásban)
### 7.1 Modell
### 7.2 A nézet
### 7.3 Vezérlő
### 7.4 A példa keretrendszer használata
#### 7.4.1 A HTML oldal
#### 7.4.2 A modell definiálása
#### 7.4.3 Az űrlapnézet
#### 7.4.4 A panelnézet
#### 7.4.5 A vezérlő

## 8 Egy professzionális MN* keretrendszer bemutatása – Backbone.js (a forrásban)
### 8.1 Backbone.js
### 8.2 A modell
#### 8.2.1 Megfigyelhető objektumok
#### 8.2.2 get() és set() metódusok
#### 8.2.3 Alapértelmezett értékek
#### 8.2.4 Modell inicializálása
#### 8.2.5 Modell adatainak másolata
#### 8.2.6 Modell mentése
### 8.3 A gyűjtemény
#### 8.3.1 Modellek hozzáadása és elvétele
#### 8.3.2 Modell lekérdezése
#### 8.3.3 A gyűjtemény mint megfigyelhető objektum
#### 8.3.4 Egész gyűjtemény beállítása
#### 8.3.5 Tömbfüggvények
### 8.4 A nézet
#### 8.4.1 Új nézet létrehozása és a megjelenítés
#### 8.4.2 Eseménykezelés
### 8.5 Az útvonalválasztó
### 8.6 Példaalkalmazás bemutatása Backbone.js segítségével
#### 8.6.1 A HTML sablon
#### 8.6.2 A modell
#### 8.6.3 Az űrlap nézet
#### 8.6.4 A statikus nézet
#### 8.6.5 A „főprogram”
### 8.7 További példák
### 8.8 Hivatkozások