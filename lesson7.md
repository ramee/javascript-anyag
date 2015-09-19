# A felhasználói felület kezelése
Bárhol is használjuk azonban a JavaScriptet, a futtatókörnyezetnek biztosítania kell egy olyan kapcsolódási felületet, amelyen keresztül a nyelvi elemekkel a gazdakörnyezet szolgáltatásai elérhetőek. Ez a kapcsolódási felület nem más, mint a futtatókörnyezet szolgáltatásai elé készített programozói felület (API).

A böngészők tehát a JavaScript nyelvi maggal való kapcsolattartást alapvetően két modell segítségével biztosítják:
- a böngésző objektummodell (angolul Browser Object Model, BOM) a böngésző szolgáltatásait reprezentáló API-objektumokat összefogó óriási hierarchikus objektum. Ezen keresztül érjük el a böngésző szolgáltatásait, de ez adja programunk globális névterét is.
- a dokumentum objektummodell (angolul Document Object Model, DOM) a megjelenített HTML oldal elé tett API-objektum. Majd látni fogjuk, hogy nem más, mint a HTML struktúránk JavaScript objektumokra való leképezése. Segítségével az egész oldalunkat képesek vagyunk elérni egyetlen objektumon keresztül. Természetesen ahogy a dokumentum része a böngészőablaknak, úgy a DOM is része a BOM-nak.

## 1 A böngésző objektummodell – BOM
A **window** objektumnak két főbb szerepe van:
- egyrészt ez alá vannak becsatolva a böngésző szolgáltatásait képviselő további objektumok,
- másrészt ez lesz az oldalhoz tartozó JavaScript programnak a globális névtere, ennek részeként jönnek létre a változóink, metódusaink, további objektumaink. A JavaScript nyelvi magnak tehát ez biztosítja a futási környezetét.

kód #1:
```javascript
//Globális változó létehozása
var globalis = 42;
ok( globalis === window.globalis, 'A window objektumon keresztül elérhető a globális változó');
ok( globalis === window['globalis'], 'A globális változók a window objektum adattagjaként jönnek létre');
```

Néhány fontosabb szolgáltatás:
- document
- location
- history
- screen
- localStorage

### 1.1 A böngészőablak (window)
A window objektum néhány fontosabb tulajdonsága és metódusa:

- **innerHeight, innerWidth**: a tartalmi rész (dokumentum) magassága és szélessége.
- **outerHeight, outerWidth**: a böngészőablak magassága és szélessége.
- **screenX, screenY**: az ablaknak a főképernyő bal felső sarkától való távolsága. Több monitoros elrendezés esetén a főképernyő az elrendezés bal felső sarkának megfelelő képernyő. Egyes böngészőkben **screenLeft** és **screenTop** néven is elérhető ez a tulajdonság.
- **scrollX, scrollY**: a tetejétől hány pixelnyire van lejjebb gördítve a dokumentum.
- **resizeTo(szél, mag)**: ablak átméretezése a megadott szélességre és magasságra. A modern böngészők már különböző megkötésekkel élnek az átméretezést illetően. Általában csak a főablakot nem engedik átméretezni, csak azt, amit már eleve JavaScripttel nyitottunk, valamint azoknak az ablakoknak az átméretezése is tiltott, amelyben egynél több oldal van megnyitva füleken. Ennek okai érthetőek: az egyik oldalbeli szkriptnek ne engedjük meg, hogy elrontsa a másik oldal méreteit, illetve hogy elállítsák a felhasználó által beállított böngészőméretet.
- **resizeBy(dszél, dmag)**: ablak méretének megváltoztatása a paraméterben megadott értékekkel. Az átméretezésre ugyanolyan megkötések igazak, mint a resizeTo esetén.
- **moveTo(x, y)**: az ablak mozgatása a paraméterként megadott pozícióra. A fenti megkötések itt is érvényesek.
- **moveBy(dx, dy)**: az ablak pozíciójának megváltoztatása a paraméterként megadott értékekkel (ld. megkötések).
- **scrollTo(x, y)**: a dokumentum görgetése a megadott pozícióra.
- **scrollBy(dx, dy)**: a dokumentum görgetése a paraméterben megadott értékekkel.
- **scrollByLines(l)**: a dokumentum görgetése paraméternyi sorral.
- **scrollByPages(p)**: a dokumentum görgetése paraméternyi oldallal.
- **window.open(url, név, tulajdonságok)**: új böngészőablak vagy -lap nyitását teszi lehetővé. A megnyitni kívánt oldal **elérhetőségét az első**, a **hivatkozási alapnak szolgáló ablaknevet a második**, az **új ablak megjelenését és viselkedését befolyásoló paramétereket a harmadik** paraméterben kell megadni.

Az ablak méretéhez kapcsolódóan két eseményt kell kiemelnünk:
- **resize**: az ablak átméretezésekor hívódik meg.
- **scroll**: a dokumentum görgetésekor aktiválódik.

### 1.2 A képernyő (window.screen)
- **width, height**: a képernyő szélessége és magassága.
- **availWidth, availHeight**: a rendelkezésre álló szélesség és magasság a képernyőn. A tálca vagy egyéb elemek miatt ez az érték kisebb lehet, mint a width és height paraméterek.
- **availTop, availLeft**: egy monitor esetében ez a két érték általában 0. Több monitor esetén a legbaloldalibb monitortól való távolságát adja meg a képernyőnek. Például két monitoros elrendezés esetén a jobb oldali képernyőn elhelyezett böngészőben lekérdezve az értéket a bal oldali képernyő felbontását kapjuk az availLeft paraméterre.
- **colorDepth, pixelDepth**: a képernyő szín- és bitmélységét adja meg.

### 1.3 URL-információk (window.location)
- **hash**: az URL # utáni része.
- **host**: a hostname és protocol együttesen.
- **hostname**: a hosztnév.
- **href**: az egész URL.
- **origin**: a protocol és a host együtt.
- **pathname**: a host után megadott útvonal.
- **port**: portszám.
- **protocol**: az URL protokollja.
- **search**: a ? után megadott szövegrész.

A metódusai:
- **assign(url)**: a paraméterül megadott URL betöltését végzi el.
- **replace(url)**: ugyancsak a megadott URL betöltése a feladata, de úgy, hogy lecseréli az oldalt, így az előzmények között nem lesz új bejegyzés.
- **reload()**: újratölti az oldalt.

### 1.4 Böngészési előzmények (window.history)
- **back()**: előző oldal
- **forward()**: következő oldal
- **go(n)**: ugrás n állapotot

HTML5:
- **state**: állapotleíró objektum
- **pushState(state, title [, url])**: új előzményt rögzít újratöltődés nélkül
- **popstate**: amikor a felhasználó megnyomja a vissza gombot
- **replaceState(state, title [, url])**: az aktuális url-t cseréli le


### 1.5 Böngészőinformációk lekérdezése
- böngésző detektálás elavult
- helyette inkább tulajdonság detektálás

kód #2:
```javascript
//Böngésződetektálás (elavult)
if (navigator.userAgent.indexOf('MSIE') !== -1) {
    document.attachEvent('onclick', console.log);
}
//Tulajdonságdetektálás
if (document.attachEvent) {
    document.attachEvent('onclick', console.log);
}
//Vagy még specifikusabban
if (typeof document.attachEvent !== "undefined") {
    document.attachEvent('onclick', console.log);
}
```

### 1.6 Időzítők
- **setTimeout(fn, ms)**: késleltetés
- **setInterval(fn, ms)**: ütemezés

- **clearTimeout(fn, ms)**: késleltetés megállítása
- **clearInterval(fn, ms)**: ütemezés megállítása

kód #3:
```javascript
// setInterval alternatíva
var d;
(function tick() {
    console.log('de lehet nem ette meg!');
    d = setTimeout(tick, 1000);
}());
clearTimeout(d);
```

## 2 A dokumentum objektummodell – DOM
A DOM objektumban megváltoztatva egy értéket, az azonnal a megjelenítésben is változással jár, és visszafele, a felületi változások rögtön kiolvasható az adott elemnek megfelelő DOM objektumon keresztül. Ennek az élő kapcsolatnak a fenntartása rengeteg erőforrást vesz el a böngészőtől, ezért lehetőleg ritkán nyúljunk hozzá. Általában szkriptünk leglassabb része a felületi elemek kezelésével kapcsolatos

A HTML DOM sokféle csomópontot különböztet meg, ezek közül a fontosabbak a következők:
- dokumentum: a fa gyökéreleme, rajta keresztül érhető el a többi csomópont.
- elem: a HTML elemnek megfelelő csomópontok.
- attribútum: az elemekhez kapcsolt további információ név-érték formában.
- szöveges csomópont: a HTML elemek között megjelenő szöveges elemek reprezentánsai.

Az alábbi csoportokba soroljuk a DOM műveleteket:
- elemek kiválasztása,
- dokumentum bejárása,
- elemek módosítása,
- attribútumok módosítása.

### 2.1 Elemek kiválasztása
Dokumentumszinten:
- egyedi azonosító: **document.getElementById(id)**
- neve: **document.getElementsByName(name)**
Dokumentum vagy bármilyen elem szinten
- elem neve: **elem.getElementsByTagName(tagname)**
- stílusosztállyal: **elem.getElementsByClassName(className)**
- CSS szelectorral: 
    - egy elemet **elem.querySelector(selector)**
    - több elemet **elem.querySelectorAll(selector)**

Ezek HTMLCollection vagy NodeList típussal térnek vissza. Változik a hosszuk és a tartalmuk, ha közben a dokumentum szerkezete is megváltozik. Ez alól kivételt a querySelector() és querySelectorAll() metódusok jelentenek. Úgy viselkednek, mint a tömbök, de nem azok. Számokkal indexelhetők, és van length tulajdonságuk, ezért egy for ciklusban feldolgozhatók, de tömbmetódusok nem érhetők el rajtuk. Ha ezeket használni szeretnénk, akkor át kell alakítanunk őket, pl. a slice() metódus kölcsönvételével, de ekkor élő tulajdonságukat elvesztjük.

kód #4:
```
//Paragrafusok lekérdezése
var pars = document.getElementsByTagName('p');
//Tömbbé alakítás
var arrayOfPars = [].slice.call(pars);
//Sorrend megfordítása
arrayOfPars.reverse();
```

### 2.2 A struktúra bejárása

Az alábbi felsorolásban mindegyik tulajdonságként (és nem metódusként) érhető el, és null értékkel térnek vissza, ha nincs megfelelő csomópont: 
- Gyerekelemek lekérdezése:
    - **childNodes** tulajdonság: az összes gyerekcsomópont listája (NodeList)
    - **firstChild**: az első gyerekcsomópont
    - **lastChild**: az utolsó gyerekcsomópont
- Szülőelem lekérdezése:
    - **parentNode** tulajdonság
- Testvércsomópontok lekérdezése:
    - **nextSibling** tulajdonság: a következő testvércsomópont.
    - **previousSibling**: az előző testvércsomópont.

A DOM bejárásában gyakran nehézséget okoz, hogy a szöveges csomópontok az elemek csomópontjaival együtt jelennek meg a tulajdonságok által visszaadott gyűjteményekben. Ezeket a nodeType attribútumuk alapján ki lehet szűrni, ugyanis ez szöveges csomópontoknál 3.

### 2.3 Attribútumok kezelése

Az elemhez tartozó attribútumok kezelésére az alábbi metódusok és tulajdonságok állnak rendelkezésre:
- **getAttribute(name)**: adott nevű attribútum értékének lekérdezése.
- **setAttribute(name, value)**: adott nevű attribútum értékének beállítása.
- **hasAttribute(name)**: adott nevű attribútum létezésének vizsgálata.
- **removeAttribute(name)**: adott nevű attribútum eltávolítása.
- **attributes**: elem összes attribútumának gyűjteménye.

### 2.4 Elemek kezelése

Az elemek eggyel magasabb szintű módosítása, amikor nem az elem tulajdonságait módosítjuk, hanem új elemeket hozunk létre, meglévőeket helyezünk át vagy törlünk a dokumentumból.

- Új elem létrehozása:
    - **document.createElement(elem) **
    - az elem **innerHTML** tulajdonságának módosításával
- Elem áthelyezése:
    - **appendChild(újelem)**: az adott elem gyerekei közül utolsóként szúrja be az újelem-et.
    - **insertBefore(újelem, refelem)**: beszúrja az újelem csomópontot az adott elem gyerekei közül a létező refelem gyerekcsomópont elé. Ha ez utóbbi null, akkor a gyerekelemek végére szúrja be az újelem-et.
    - **replaceChild(régielem, újelem)**: az adott elem gyerekelemei közül a régielem csomópontot az újelem csomópontra cseréli le, és a régi csomóponttal tér vissza.
- Elem törlése:
    - **removeChild(childNode)**: egy szülőcsomópontról meghívva törli a paraméterül megkapott gyerekcsomópontot.
    - az **innerHTML** tulajdonság üressé tételével egy adott csomópont belseje teljes egészében törölhető.

## 3 jQuery
### 3.1 A jQuery története
### 3.2 Filozófiája
### 3.3 Elemek kiválasztása
- alap szelektorok: elem, id, osztály
- attribútum szelektorok (pl. [name=”value”])
- űrlap szelektrok (pl. :checkbox)
- tartalomszűrő szelektorok (pl. :contains())
- pszeudo-szelektor (pl. :first, :animated, :hidden)
- összetett szelektorok: többszörös, hierarchikus

Az elemek kiválasztása költséges lehet. Ennek megoldása:
- jQuery objektum cache-elése
- láncolt metódusok írása

Sokszor a változó a magyar jelölés szerint $ jellel kezdődik, jelezvén, hogy a $() függvény eredményeképpen jött létre

kód #5:
```javascript
// cache-elés
var $p = $('p');
// láncolás
$('p').addClass('important').hide();
```

### 3.4 Elemek bejárása
A kiválasztott elem(ek)hez képesti elmozdulásért, illetve az ottani elemek kiválasztásáért a következő metódusok felelősek:
- gyerekek (children()), leszármazottak (find())
- szülő(k) (parent(), parents(), parentsUntil()), legközelebbi ős (closest())
- testvérek (siblings(), next(), nextAll(), nextUntil(), prev(), prevAll(), prevUntil())

A kiválasztott elemeket tovább módosíthatjuk:
- szűrés (is(), has(), filter(), first(), last())
- bővítés (add())

kód #6:
```javascript
//Bejárás
// szülő kiválasztása
$("#adatok").parent();    
// a következő elem kiválasztása
$("li").next();              
// az előző elem kiválasztása
$("li").prev();              
// az előző elem kiválasztása, de csak ha adat osztályú
$("li").prev(".adat");       
// az elem felmenői közül az első, amelyikre igaz, hogy div
$("b:first").closest('div'); 
// az elem leszármazottai, melyekre igaz, hogy adat osztályúak
$("#adatok").find(".adat");  
// a kiválasztott elemek közül az első
$("li").first();           
// a kiválasztott elemek közül az utolsó
$("li").last();              
// a kiválasztott elemek közül a harmadik
$("li").eq(2);               
// az elem azon testvérei, melyek h1 típusúak
$("#adatok").siblings("h1"); 
//Szűrés
// az elem jQuery objektumához hozzáadjuk az összes li-t is
$("#adatok").add("li");      
// a választásból kivesszük az összes li típusút
$("#adatok, li").not("li");  
// a kiválasztott elemekből csak a képek maradnak
$(".adat").filter("img");    
//Visszalépéses szelekció
$("#adatok")                                         // hatókör: div#adatok
    .find("li")                                      // hatókör: div#adatok
        .css({ padding: "5px" })                     // hatókör: div#adatok li
        .find("b")                                   // hatókör: div#adatok li
            .html('Csak a címeket hackoltam meg!!!') // hatókör: div#adatok li b
            .end()                                   // hatókör: div#adatok li b
        .end()                                       // hatókör: div#adatok li
    .animate({ paddingTop: '+=100' }, 200); 
```

### 3.5 Iterálás a szelekcióban

kód #7:
```javascript
var cimkek = [];
$("b").each(function() {
    cimkek.push( $(this).text().replace(':','') );
});
cimkek.join(', ');
```

### 3.6 HTML struktúra megváltoztatása

Néhány műveletcsoport:
- elem(ek) létrehozása: ahogy fentebb már említettük, erre a jQuery() függvény szolgál, paraméterül a létrehozandó HTML szöveget megadva.
- elem lemásolása (clone())
- elem hozzáadása, mozgatása (append(), prepend(), after(), before(), appendTo(), prependTo(), insertAfter(), insertBefore(), replace())
- elem törlése (remove(), detach())
- tartalom kezelése (html(), text())
- attribútumok kezelése (attr())

### 3.7 Hatékony DOM manipuláció
- Ha egy HTML struktúrát építünk fel JavaScriptben és ezt akarjuk hozzácsatolni a weboldalunkhoz, akkor az elemeket ne egyenként adjuk hozzá az oldalhoz, hanem egy lépésben.
- Ha több CSS tulajdonságot egyszerre módosítunk valamely elemen, akkor megfontolandó, hogy inkább egy CSS osztályba tegyük ezeket az értékeket és egyszerűen ezt az osztályt adjuk át az elemnek – a sok kis attribútum-változtatás helyett így csak egyetlenegyszer nyúlunk a DOM objektumhoz.
- Ha nem lehetünk biztosak benne, hogy a kiválasztott elem létezik, akkor mindenképpen ellenőrizzük mielőtt több metódust is futtatnánk rajta – bár hiba nem fog történni, de az összes metódust meghívja, felesleges függvényhívásokat generálunk ezzel.
- A jQuery-ben van még egy speciális lehetőség a DOM-manipulálások lassúságának elkerülésére. A detach metódus segítségével úgy választhatjuk le a módosítandó elemet vagy elemeket az oldalstruktúráról, hogy annak minden tulajdonságát, eseménykezelőjét megtartjuk. Ezt követően nyugodtan állítgathatunk bármit, módosíthatjuk a tartalmát, csak a jQuery objektumon fog megtörténni a változtatás, a DOM-ban nem. Ha végeztünk minden módosítással, akkor visszailleszthetjük az oldalunkba.

kód #8:
```javascript
//Sok egyedi manipulálás
$("#adatok").css({ width: '100px', color: 'red', opacity: 0.5 }).html('Izébizé');
//Helyette: leválasztás, manipulálás, visszaillesztés
$("#adatok")
    .detach()
    .css({ width: '100px', color: 'red', opacity: 0.5 }).html('Izébizé')
    .appendTo($("body"));
```

### 3.8 Stílusok kezelése
- css()
- addClass(), removeClass(), toggleClass(), hasClass()
- animate(tulajdonság, mennyi ideig tartson (ms), csillapítási függvény (linear, swing stb), callback függvény)
- show(), hide(), toggle()
- fadeIn(), fadeOut(), fadeToggle()
- slideDown(), slideUp(), slideToggle()
- Több elemen meghívott animáció viszont párhuzamosan fut le. Az animációs sor kezelésére is kapunk függvényeket a jQuery-től a queue(), dequeue() és stop() személyében.

- height(): számított magasság
- innerHeight(): magasság + padding
- outerHeight(): magasság + padding + border (+margó true paraméter esetén)
- width(), innerWidth(), outerWidth(): ld. a height függvényeit
- position(): a szülőobjektumhoz viszonyított elhelyezkedés (top, left)
- offset(): a dokumentumhoz viszonyított elrendezés (top, left)
- scrollTop(), scrollLeft()

A mai böngészőkben hatékonyabb a CSS3 animáció.
CSS3:
- transform (nem animáció)
- transition: átmenet két stílusállapot között
- animation: keyfraem alapú, ismétlődő

## 4 Eseménykezelés
Az eseménykezeléshez kapcsolódó ismeretek az alábbiak:
- eseménykezelő regisztrálása
- eseményobjektum megszerzése
- az eseményobjektum tulajdonságai
- az alapértelmezett művelet megakadályozása
- buborékolás és megakadályozása
- esemény kiváltása

### 4.1 Eseménykezelő regisztrálása
- addEventListener('type', fn, useCapture): az fn eseménykezelő függvény regisztrálása a type típusú eseményre
- removeEventListener('type', fn, useCapture): az fn eseménykezelő függvény regisztrálásának megszüntetése a type eseménynél

kód #9:
```javascript
var hello = function () {
    console.log('Hello!');
};
var adatok = document.getElementById('adatok');
adatok.addEventListener('click', hello, false);
```

### 4.2 Eseményobjektum megszerzése
kód #10:
```javascript
var listener = function (e) {};
```

### 4.3 Eseményobjektum tulajdonságai
kód #11:
```javascript
var listener = function (e) {
    console.log(e);
};
```

A fontosabb közös tulajdonságok a következők:
- type: az esemény típusa ('click', 'mousemove', stb.)
- target: az eseményt eredetileg kiváltó DOM objektum
- currentTarget: az a DOM objektum, akihez az eseménykezelő csatolva van

Egy egéresemény ezek mellett a következő információkat tartalmazhatja:
- screenX, screenY: az egér pozíciója a képernyő bal felső sarkához képest
- clientX, clientY: az egér pozíciója a böngésző bal felső sarkához képest
- ctrlKey, sshiftKey, altKey: le volt-e nyomva a megfelelő módosító gomb?
- button: a lenyomott egérgomb kódja
- keyCode: a billentyűzet esemény billentyűzet kódját tartalmazza

### 4.4 Az alapértelmezett művelet megakadályozása
- preventDefault()

### 4.5 Buborékolás és megakadályozása

A kiváltott események számát és a programozási lehetőségeket jelentősen megnöveli az, hogy egy eseményt nemcsak az az objektum jelez, aki közvetlenül érintett volt, hanem az események láncszerűen az összes szülőelemen is jelzésre kerülnek. Az esemény tehát az adott elemtől felmegy a legfelső, dokumentumszintig. Ez alapján ezt a jelenséget buborékolásnak nevezzük.

A teljes képhez hozzájárul az is, hogy ezt követően dokumentumszinttől a közvetlenül érintett elemig is kiváltódnak az események. Ez utóbbit elkapási fázisnak nevezik. Az addEventListener() és removeEventListener() metódusok utolsó, logikai paramétere éppen azt határozta meg, hogy szeretnénk-e az eseménykezelőt az elkapási fázisban aktivizálni. A legtöbb feladat elvégezhető buborékolási fázisban, ezért általában oda hamis értéket adunk meg.

Bármelyik szinten kapjuk is el az eseményt, az eseményt elsőként jelző elem mindig lekérdezhető az eseményobjektum target tulajdonságán keresztül.

A buborékolás, vagy az esemény továbbhaladása megállítható, az eseményobjektum stopPropagation() metódusával.

kód #11:
```javascript
var listener = function (e) {
    console.log(e.target);
    e.stopPropagation();
};
```

### 4.6 Delegálás
A buborékoláshoz egy gyakran alkalmazott, hatékony programozási minta is kapcsolódik. Képzeljük el, hogy egy táblajátékot fejlesztünk, ahol minden cellában történő kattintásra valamilyen módon szeretnénk reagálni. Ha minden cellához külön-külön rendelnénk eseménykezelő függvényt, akkor egyrészt igencsak megnövekedne az eseménykezelők száma, ami mind a programozó, mind a böngésző részéről plusz erőforrásokat kíván a kezelésükhöz, másrészt további sorok és oszlopok beszúrására nem lenne elég rugalmas. Ilyen esetekben kihasználhatjuk a buborékolás jelenségét, és az eseménykezelőt az összes érintett elem valamelyik közös ősén, ebben az esetben például a <table> elemen kezelhetjük, viszont az eseményobjektumon keresztül az eseményben érintett <td> cella lekérdezhető. Az egész feladathoz így egyetlen eseménykezelő regisztrálása szükséges, ráadásul a megoldás a táblázaton belüli cellák számától és dinamikus változásától is független. Ezt a fajta eseménykezelést hívjuk delegálásnak.

### 4.7 Esemény kiváltása
- dispatchEvent(): események kiváltására szolgál. Paraméterül egy Event interfészt vagy annak valamilyen leszármazottját megvalósító objektumot kell megadni. Ilyen lehet pl. többek között a MouseEvent vagy KeyboardEvent, egyedi eseményeknél pedig a CustomEvent.

kód #12:
```javascript
//Beépített esemény kiváltása: a függvény a paraméterül megkapott jelölőmezőre kattintást szimulálja
var simulateClick = function(checkbox) {
    var event = new MouseEvent('click', {
        'view': window,
        bubbles: true,
        cancelable: true
    });
    return checkbox.dispatchEvent(event);
};
//Egyedi esemény kiváltása
var publishCustomEvent = function(obj, type, data) {
    var event = new CustomEvent(type, {
        detail: data,
        bubbles: true,
        cancelable: true
    });
    return obj.dispatchEvent(event);
};
var body = document.body;
body.addEventListener('something', function(e) {
    console.log(e.detail);
}, false);
publishCustomEvent(body, 'something', { foo: 42 });
```

### 4.8 jQuery
- on() és off()
- paraméterezése:
    - $obj.on('type', fn)
    - $obj.on('type', 'selector', fn) // delegálás
- névterek használata
    - $('table').on('click.myGame', 'td', function () );
    - $('table').off('click.myGame', 'td', function () );
- az ismétlődő eseménykezelők megoldása:
    - $('table').off('click.myGame').on('click.myGame', 'td', function () );
- trigger(event): beépített vagy saját események programozott kiváltása

kód #13:
```javascript
$("#adatok").trigger("click");
```