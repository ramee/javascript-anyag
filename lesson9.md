# Eseményvezérelt programozás
## 1 Aszinkron JavaScript
- fogalmak: multithreaded, deadlock, race condition
- javascript csak egy szálat használ

## 2 Szinkron-aszinkron-multithreaded
### 2.1 Technológiai háttér
A futtatókörnyezet 3 fontos része:
- `heap`: **tartalmazza a programunk azon részét, ami már lefutott** — így ide kerülnek a **függvénydefiníciók**, a lefuttatott függvények által létrehozott, még létező **névterek**, az alkalmazás jelen állapotában elérhető **összes változó**, és így tovább. Ez a referencia és adathalmaz meglehetősen kaotikus, semmi sem biztosítja a rendezettségét
- `stack`: **éppen futás alatt álló függvények, programrészek** vannak — egy olyan rendezett tárként képzelhetjük el, amely egyre mélyül, ahogy a függvények egymást hívogatják és várnak a másik visszatérési értékére. A futtatókörnyezet ezen része folyamatos kapcsolatban áll a **Heap**-el, hiszen bármikor új elemet emelhet át onnan a Stack-be, attól függően, hogy az épp futó folyamatok hogyan is döntenek
- `queue`: Minden JavaScriptben írt aszinkron kód egy **callback függvényt** hív meg egy, az őt befejező esemény bekövetkezésekor. Egy egyszerű setTimeout esetén ez a beállított késleltetéskor következik be; egy AJAX hívásnál akkor mikor visszatér valamilyen eredménnyel az aszinkron request; egy click eseménynél pedig a felhasználó által kiváltott klikkelésre várjuk egy függvény lefutását. Ezeket a függvényeket a megfelelő időben a **Message Queue**-ba pakolja a futtatókörnyezet, egy sorba a jövőben futtatandó feladatok számára. Ebből a tárból már egyesével kerülnek át a callback függvények a **Stack**-be, egymás után, ahogyan az képes feldolgozni őket. Mivel a feladatok lefutási ideje különböző lehet és több ugyanakkor indukált függvény feltorlódhat a Queue-ban, ezért sohasem várható el azok pontos futtatási ideje. A setTimeout második paramétereként átadott 500 csak annyit jelent, hogy függvényünk 500ms múlva kerül a Queue végére, de semmi sem biztosítja, hogy ekkor le is fog futni. Amennyiben több konkurens aszinkron callback is várakozik, akkor a késleltetett metódusunk is megvárja a sorát.

kód #1:
```javascript
// Aszinkron hívás késleltetett metódussal:
// a setTimeout 1000ms-al a deklaráció után a Queue-ba
// teszi az első paraméterben található függvényt
setTimeout(function() {
    console.log('Aszinkron hívás történt!');
}, 1000);
console.log('Az Aszinkron hívás előtt lefut, hiszen' +
            'ez a parancs már a Stack-ben van!');
// Aszinkron hívás user inputra:
// a callback függvény akkor kerül be a Queue-ba, ha
// a felhasználó rákattint a gombra
document.getElementById('supaButton').addEventListener(
    'click', function() {
        console.log('Aszinkron hívás történt!');
    }
);
```

### 2.2 Event loop

A JavaScript a **Heap-Stack-Queue** felépítés fölött egy úgynevezett **Event loop** folyamatot futtat, ami az aszinkron hívások külső eseményeit figyeli és az általuk visszaküldött jelzéseket, **callback**-eket ütemezi a **Queue**-ba. Az **Event loop** legfontosabb tulajdonsága, hogy mindeközben **nem blokkolja** a JavaScript folyamatokat!

### 2.3 Aszinkron kód a szerver oldalon
kód #2:
```javascript
fs.readFile('usernames.csv', function (err, data) {
  if (err) throw err;
  console.log(data);
});
```

Az `fs.readFile` meghívása után a vezérlés továbbadódik — mivel a fájl beolvasása a háttérben történik, nem blokkol, így a szerver a többi kérés kiszolgálásával foglalkozhat. Amint a futtatókörnyezet beolvasta a fájlt, az Event loop segítségével a **Queue**-ba teszi a **callback** függvényt, paraméterként átadva a fájl tartalmát.

### 2.4 Aszinkron kód a kliens oldalon
A szerver oldalhoz hasonlóan a böngészőkben való futtatás esetén is törekedni kell arra, hogy a számításigényes feladatokat aszinkron műveletekre osszuk fel, mivel a Stack feldolgozása során a felhasználó interfész blokkolva van!

### 2.5 Aszinkron JavaScript
Összesítve:
- A JavaScript parancsok a futási idejükre blokkolják a felhasználói felületet, szerver oldal esetén pedig egyéb, más felhasználók által futtatott folyamatokat. A böngésző-közeli világra vetítve ez azt jelenti, hogy a UI csak akkor használható, ha üres a Queue. Ennek tükrében, ha minél több utasítást hívunk meg aszinkron módon, a felhasználói felület annál többször jut levegőhöz — két aszinkron esemény között a futtatókörnyezet be tudja ütemezni a UI-t érintő műveleteket is, ezzel sokat javítva a felhasználói élményen.
- Sosem fordulhat elő, hogy egy program inkonzisztens állapotba kerül az aszinkron kódok miatt, hiszen minden parancs megvárja az őt megelőzőek lefutását.
- Az eseményvezérelt kód nem lesz gyorsabb, mint a szinkron — sőt rengeteg egymás után kiváltott aszinkron esemény telítheti a Queue-t és ideiglenesen blokkolhatja a futtatókörnyezetet vagy a UI-t.

### 2.6 Problémák láncolt eseményekkel
#### 2.6.1 Aszinkron kód callback-ekkel
- pyramid of doom, callback hell

kód #3:
```javascript
var dialog = new Dialog(),
    mapRenderer = new MapRenderer(dialog);
$('#homeButton').on('click', function() {
    dialog.open(function() {
        mapRenderer.render();
        $.getJSON('/getHomeInfo', function(home) {
            GMaps.geocode({
                address: home.address,
                callback: function(position) {
                    mapRenderer.addAddress(home, position);
                }
            });
        });
    });
});
```

#### 2.6.2 Aszinkron kód kiegyenesítése a függvények kiemelésével

kód #4:
```javascript
var HomeDialog = function() {
    this.dialog = new Dialog();
    this.mapRenderer = new MapRenderer(this.dialog);
};
HomeDialog.prototype.open = function() {
    var render = this.render.bind(this);
    this.dialog.open.call(this.dialog, render);
};
HomeDialog.prototype.render = function() {
    this.mapRenderer.render();
    this.getHomeData(this.renderHome);
};
HomeDialog.prototype.getHomeData = function(callback) {
    $.getJSON('/getHome', (function(homeInfo) {
        this.home = homeInfo;
        callback();
    }).bind(this);
};
HomeDialog.prototype.renderHome = function() {
    GMaps.geocode({
        address: this.home.address,
        callback: this.renderPosition.bind(this)
    });
};
HomeDialog.prototype.renderPosition = function(position) {
    mapRenderer.addAddress(this.home, position);
};
var homeDialog = new HomeDialog();
$('#homeButton').on('click', homeDialog.open.bind(homeDialog));
```

- jól elkülöníthetó, átlátható
- további refaktorálásra ad lehetőséget
- a HomeDialog osztálynak túl sok különböző felelőssége van, olyan dolgokat tesz, amit nem biztos, hogy neki kellene
- a metódusok meghívási sorrendje nem triviálist, könnyű elrontani a működést

A getHomeData metódust és a `renderHome`-ban található `Google API kérést` ki lehetne emelni egy `HomeInfo` illetve egy `PositionProvider` osztályba.

kód #5:
```javascript
var HomeDialog = function() {
    this.dialog = new Dialog();
    this.mapRenderer = new MapRenderer(this.dialog);
    this.homeInfo = new HomeInfo();
    this.positionProvider = new PositionProvider();
};
HomeDialog.prototype.render = function() {
    this.mapRenderer.render();
    this.homeInfo.get(this.renderHome.bind(this));
};
HomeDialog.prototype.renderHome = function(homeInfo) {
    this.positionProvider.get(homeInfo, this.renderPosition.bind(this));
};
HomeDialog.prototype.renderPosition = function(homeInfo, position) {
    mapRenderer.addAddress(homeInfo, position);
};
```

A függőségek csökkentése érdekében egyszerűsíthetjük a kódot.

kód #6:
```javascript
var HomeDialog = function() {
    this.dialog = new Dialog();
    this.mapRenderer = new MapRenderer(this.dialog);
    this.homePositionProvider = new HomePositionProvider();
};
HomeDialog.prototype.render = function() {
    this.mapRenderer.render();
    this.homePositionProvider.get(this.renderHome.bind(this));
};
HomeDialog.prototype.renderHome = function(homeInfo, position) {
    mapRenderer.addAddress(homeInfo, position);
};
```

Még egy kicsit egyszerűsítve:

kód #7:
```javascript
var HomeDialog = function() {
    this.dialog = new Dialog();
    this.mapRenderer = new MapRenderer(this.dialog);
    this.homePositionProvider = new HomePositionProvider();
};
HomeDialog.prototype.render = function() {
    this.mapRenderer.render();
    this.homePositionProvider.get(mapRenderer.addAddress);
};
```

Hibakezelés:

kód #8:
```javascript
HomeDialog.prototype.render = function() {
    this.mapRenderer.render();
    this.homePositionProvider.get(
        mapRenderer.addAddress,
        this.showError);
};
HomeDialog.prototype.showError = function() {
    alert('Sorry to say, but something bad happened! :( ');
    this.dialog.destroy();
}
```

Természetesen a megfelelő wrapper osztályokat (HomePositionProvider, HomeInfo, PositionProvider) is ki kell bővíteni a hibakezeléssel.

**Probléma**: Továbbra is nekünk kell menedzselnünk a párhuzamos feladatokat, fejben kell tartani, hogy melyik metódus aszinkron és melyik nem. - Adott esetben nehéz lehet átlátni a kódot: például a HomeDialog osztályra rátekintve fogalmunk sem lehet arról, hogy hány aszinkron hívás fog történni a háttérben. - A debuggoláson nem javít, hiszen egy hiba esetén a StackTrace-ben csak a legközelebbi aszinkron hívásig fogjuk látni a meghívott metódusokat — hiszen ilyenkor az utasítások már csak eddig vannak a Stack-ben! - Az egyre sokasodó osztályok jelentősen növelhetik a teljes kódbázis komplexitását. Sok olyan osztályunk is születhet, amit máshol nem fogunk felhasználni, így kiemelésük nem biztos, hogy megérte. Érdemes szem előtt tartani, hogy a nagyobb kódbázist mindig nehezebb is karbantartani!

#### 2.6.3 Aszinkron kód Promise-al

Ajánlott irodalom: [Jake Archibald - JavaScript Promises](http://www.html5rocks.com/en/tutorials/es6/promises/)

A promise egy aszinkron hívást körbeölelő objektum, amely a művelet aktuális állapotát hordozza magában. Minden promise egyetlen egyszer futhat le, és a futás eredménye csak sikeres vagy sikertelen lehet. Az eredménytől függően képes a megfelelő callback metódusokat meghívni.

Állapotok:
- fulfilled: az aszinkron művelet sikeresen lefutott
- rejected: sikertelen aszinkron művelet
- pending: a művelet még fut
- settled: a hívás megtörtént, függetlenül a sikerességtől

kód #9:
```javascript
promise.then(onFulFilled, onRejected);

// példa
fs.readFile('usernames.csv')
    .then(function(data) {
        console.log('Got data: ', data);
    }, function() {
        console.log('Error happened!');
    });

```

A Promise osztály az ES6-ban már natívan is elérhető, az azt nem támogató futtatókörnyezetekben számos letölthető [promise library](http://www.promisejs.org/implementations/) közül választhatunk vagy egyszerűen húzzunk be egy kis méretű [polyfill](https://github.com/jakearchibald/es6-promise)-t.

A then egy nagyszerű tulajdonsága, hogy mindenképpen egy új promise-t fog visszaadni, ami az előző aszinkron hívás után hajtódik végre, így könnyedén egymás után **láncolhatjuk** az aszinkron műveleteinket. Ha azonban egy promise lánc bármelyik eleme visszautasításra kerül, akkor a **rejected** státuszú hívás utáni then-ek már **nem futnak le**!

Ha az onFulfilled callback-el visszaadunk egy értéket, akkor az a **következő then utasításban paraméterként** fog megjelenni. Amennyiben nincs szükségünk egyedi hibakezelésekre, a lánc végére tehetünk egy **catch** hívást, amely akkor hívódik meg, ha bármelyik promise rejected állapotba kerül vagy ha egy then callback-ben kivételt dobunk:

kód #10:
```javascript
fs.readFile('usernames.csv')
    .then(filterValidUsernames)
    .then(writeUsernames)
    .then(showSuccessMessage)
    .catch(function() {
        alert('Something bad happened! :( ');
    });
```

A Node.js fs.readFile és fs.writeFile metódusai nem promise-t adnak vissza, de használhatjuk a Promise konstruktorfüggvényt:

kód #11:
```javascript
var readFilePromise = function(filename) {
    return new Promise(function(resolve, reject) {
        fs.readFile(filename, function (err, data) {
            if (err) reject();
            else resolve(data);
        });
    });
};
readFilePromise('usernames.csv')
    .then(filterValidUsernames)
    .then(writeUsernames)
    .then(showSuccessMessage)
    .catch(function() {
        alert('Something bad happened! :( ');
    });
```

A Promise-nak átadott függvény két paramétert kap: a **resolve**-ot meghívva a promise **fullfilled** állapotba kerül és a rákötött then **sikeres** ágát indukálja; míg a **reject** hívásakor értelemszerűen a **rejected** állapot lép érvénybe, a **hiba-callbackek** futtatásával

A jQuery hibás promise implementációt valósít meg. // ellenőrizendő az újabb verziókban

Natív promise használata:
kód #12:
```javascript
var nativePromise = Promise.cast($.getJSON('/something.json'));
nativePromise.then(function(things) {
    console.log(things);
});
```

Több egymástól független ág futtatása:

kód #13:
```javascript
var promiseOriginal = readFilePromise('something.txt');
// Első ág
promiseOriginal
    .then(function(data) {
        console.log(data);
        return 'Ez egy üzenet';
    })
    .then(function(message) {
        console.log(message);
    });
// Második ág - bárhol máshol
promiseOriginal
    .then(function(data) {
        console.log('data');
    });
```

Az ágak futásának sorrendje nem garantált.
A Promise-nak még rengeteg hasznos segédfüggvénye van az aszinkron hívások kezelésére, érdemes az [MDN oldalán](https://developer.mozilla.org/en-US/docs/Web/JavaScript/Reference/Global_Objects/Promise) található példákat is végiglapozni.

kód #14:
```javascript
// A Google API bewrappelése, hogy promise-t adjon vissza
var geocode = function( address, datas ) {
    return new Promise(function(resolve) {
        GMaps.geocode({
            address: address,
            callback: function(response) {
                datas = datas || ;
                datas.position = response;
                resolve(datas);
            }
        });
    });
};
dialog.open()
    .then(mapRenderer.render)
    .then(function() {
        return Promise.cast($.getJSON('/getHomeInfo'));
    })
    .then(function(homeInfo) {
        return geocode(home.address, { home: homeInfo });
    })
    .then(function(datas) {
        mapRenderer.addAddress(datas.home, datas.position);
    });
```

#### 2.6.4 Aszinkron kód generátorokkal

kód #15:
```javascript
var renderHome = function *() {
    yield dialog.open();
    mapRenderer.render();
    var home = yield Promise.cast($.getJSON('/getHome'));
    var position = yield geocode(home.address);
    mapRenderer.addAddress(data.home, data.position);
};
```

A fenti generátor függvénynek mindig hívogatni kéne a next metódusát a promise then metódusára. Erre a feladatra sokan egy általános wrapper metódust használnak, melyet legtöbbször spawn-nak hívnak.

kód #16:
```javascript
$('#homeButton').on('click', function() {
    spawn(renderHome);
});
```

A **spawn** metódus figyeli a neki átadott generátorfüggvény promise-ait és azok állapotváltásaikor meghívja a renderHome.next() metódust, továbbléptetve a függvényt a következő yield-ig, vagy annak hiányában a függvénymag végéig. **Implementációja** Jake Archibald [cikk](http://www.html5rocks.com/en/tutorials/es6/promises/)ében található.

A két technika ötvözése:

kód #17:
```javascript
var fixUsernames = function*() {
    try {
        var usernames = yield readFilePromise('usernames.csv');
        usernames = filterValidUsernames(usernames);
        yield writeUsernames(usernames);
        showSuccessMessage();
    } catch() {
        alert('Something bad happened! :( ');
    }
};
spawn(fixUsernames);
```

#### 2.6.5 Aszinkron kód deferred függvényekkel

Az EcmaScript 6 bevezeti az úgynevezett [Deferred function](http://wiki.ecmascript.org/doku.php?id=strawman:async_functions&s=await) fogalmát, olyan függvényekre értve, melyek lefutása aszinkron módon is történhet, állapotát pedig nyomon lehet követni. Ehhez kapcsolódóan egy új vezérlési szerkezet is megjelenik: az **await** kulcsszó után álló Deferred futásának idejére a futtatókörnyezet elmenti, majd kiüríti a Stack-et, átadva helyét a következő esemény számára a Queue-ból. Az elmentett műveletsor akkor folytatódik, ha a Deferred lefutott.

A promise a Deferred function-ök részét képezi, így hasonló tulajdonságokkal rendelkezik. Ennek köszönhetően használható az await kulcsszó után arra, hogy a függvény futását felfüggesszük addig, amíg az aszinkron műveletünk visszatérésére várunk.

kód #18:
```javascript
var renderHome = function() {
    var home, position;
    await dialog.open();
    mapRenderer.render();
    await home = Promise.cast($.getJSON('/getHome'));
    await position = geocode(home.address);
    mapRenderer.addAddress(data.home, data.position);
};
renderHome();
```

Az első **await**-hez érve a kód megvárja, hogy megnyíljon az ablak, vagyis felfüggeszti a **renderHome** függvény futását, elmenti a **Stack** aktuális állapotát és felszabadítja azt. Ha megnyílt az ablak (azaz a **dialog.open()** által visszaadott **promise fulfilled** állapotba kerül), akkor az elmentett **Stack**-et visszatölti és folytatja a függvény futtatását. Így megy ez egészen addig, amíg a függvény véget nem ér. Az await-et tartalmazó függvény visszatérési értéke mindenképpen egy Promise lesz.

### 2.7 Funkcionális reaktív programozás
#### 2.7.1 Reaktív programozás
#### 2.7.2 Funkcionális reaktív programozás
Használt: [Bacon.js](https://github.com/baconjs/bacon.js/tree/master)

kód #19:
```javascript
var $input = $('input[name=textfield]'),
    $label = $('.label');

// hagyományos 
$input.on('keyup', function(e) {
    if (e.keyCode === 13) {
        $label.text($input.val());
    }
});

// FRP
$input
    .asEventStream('keyup')
    .onValue(function(e) {
        if (e.keyCode === 13) {
            $label.text($input.val());
        }
    });

// egérkattintásra is
var $button = $('button');

var enterStream = $input
        .asEventStream('keyup')
        .filter(function(e) { return e.keyCode === 13; });

var clickStream = $button
        .asEventStream('click');

enterStream.merge(clickStream).onValue(function() {
    $label.text($input.val());
});
```

- a billentyűleütést egy úgynevezett Event Stream-é alakítjuk
- az asEventStream metódus egy jQuery eseményből képes létrehozni egy Event Stream-et, az onValue pedig egy esemény kiváltódásakor a folyamba került értékkel hívja meg a callback-jét, jelen esetben ez a billentyű-esemény event objektuma.
- Property: ez egy egyszerű, állandó állapottal rendelkező értéknek tekinthető, melynek van valamilyen kezdőértéke, majd folyamatosan követi a folyamot

kód #20:
```javascript
var keyStream = $input.asEventStream('keyup'),
    enterStream = keyStream
        .filter(function(e) { return e.keyCode === 13; }),
    clickStream = $button.asEventStream('click'),
    showStream = enterStream.merge(clickStream),
    isEmpty = keyStream
        .map(function() { return $input.val().trim() === ''; })
        .toProperty(true);

showStream.onValue(function() {
    $label.text($input.val());
});

isEmpty.onValue(function(state) {
    $button.attr('disabled', state);
});

// VAGY egyenesen a gombra kötjük
keyStream
    .map(isEmpty)
    .toProperty(true)
    .assign($button, 'attr', 'disabled');
```

- Az újonnan megjelent isEmpty egy olyan Property, mely igaz kezdőértékkel rendelkezik, azonban minden keyStream-beli eseményre (vagyis billentyűlenyomásra) ellenőrzi, hogy üres-e még a szöveges mező — ha nem, akkor false-ra állítja a saját állapotát

#### 2.7.3 Funkcionális reaktív programozás a szerver oldalon

Node.js és Bacon.js

kód #21:
```javascript
var getUsernames = function(content) {
        return content.split('\r\n');
    },
    filterValidUsernames = function(usernames) {
        return usernames.filter(function(username) {
            return Boolean(username.trim());
        });
    },
    fixUsernamesIn = function(file) {
        var fileContent = Bacon.fromNodeCallback(fs.readFile, file),
            usernames = fileContent.map(getUsernames),
            validUsernames = usernames.map(filterValidUsernames);
        validUsernames.onValue(function(usernames) {
            fs.writeFile(file, usernames.join('\r\n'));
        });
    };
fixUsernamesIn('usernames.csv');
```