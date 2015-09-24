# Tesztvezérelt fejlesztés
## 1 Egységtesztelés JavaScriptben
- Igazán gyors visszajelzés érdekében a fejlesztéshez válasszunk úgynevezett headless, vagyis felülettel nem rendelkező böngészőt (pl. PhantomJS)
- A JavaScript legnépszerűbb teszt-keretrendszerei a [Mocha](https://mochajs.org/), a [Jasmine](http://jasmine.github.io/), a jQuery tesztelésére is használt [QUnit](https://qunitjs.com/) és a [JsTestDriver](https://code.google.com/p/js-test-driver/) által biztosított [Assertion library](https://code.google.com/p/js-test-driver/wiki/Assertions).

A teszt-keretrendszerekben általában tesztkészleteket definiálhatunk, melyek mindegyike egy-egy osztály egységtesztelésére szolgál. A JsTestDriver Assertion libraryben ezt a TestCase függvénnyel tehetjük meg:

kód #1:
```javascript
// Tegyük fel, hogy a következő osztályra szeretnénk tesztet írni:
var Calculator = function() ;
Calculator.prototype.add = function(a, b) {
    return a + b;
};
// Definiáljuk a tesztkészletet:
TestCase('CalculatorTests', );
```

A TestCase második paraméterében egy objektumot kell átadnunk, melyben felsorolhatjuk a teszteseteinket

Minden tesztesetben legalább egy assertion-nek, vagyis ellenőrzésnek kell szerepelnie. Az xUnit keretrendszerek a legkülönbözőbb függvényeket biztosítják az érték- és állapotellenőrzésre:
- általában léteznek logikai helyességet vizsgáló metódusok, esetünkben ilyen az assertTrue illetve az assertFalse. Paraméterként egy kifejezést várnak és ha ez az előbbi esetében hamis, utóbbiban igaz lesz, akkor az ellenőrzés elbuktatja a tesztet és értesít minket.
- vannak összehasonlító vizsgálatok, ilyen pl. az assertEquals, mely első paramétereként az elvárt értéket, második paraméterként az általunk kiszámoltat várja. Az előbbi tagadó változata az assertNotEquals, de létezik reguláris kifejezéssel összehasonlító, egy sztring tartalmában kereső, stb. metódus is.
- ellenőrizhetünk típusra (pl. assertTypeOf, assertNumber, assertArray, stb.), kivételre (pl. assertException), de akár DOM tulajdonságokra is (pl. assertElementId).

A JsTestDriver Assertion library használatánál minden teszt-függvényt a test kulcsszóval kell prefixelni, azonban ezt követően bármilyen nevet írhatunk. A nem test-el kezdődő metódusokat közvetlenül nem hívja meg a test runner, így nyugodtan írhatunk vagy egy összetett tesztből kiemelhetünk saját metódusokat.

kód #2:
```javascript
TestCase('CalculatorTests', {
    testAdd_GivenTwoNumbers_ReturnsSumOfThem: function() {
        var calculator = new Calculator();
        assertEquals(3, calculator.add(1,2));
    },
    testAdd_GivenAZero_ReturnsTheOtherNumber: function() {
        var calculator = new Calculator();
        assertEquals(2, calculator.add(0,2));
        assertEquals(3, calculator.add(3,0));
    }
});
```

A fenti példában a kalkulátor példányosítása egy csúnya ismétlés, kiált azért, hogy emeljük ki valahova. A teszt-keretrendszerek általában biztosítanak egy minden teszteset előtt lefutó metótust — ez esetünkben a setUp lesz. Kitűnő lehetőség arra, hogy kiemeljük az objektum konstruálást:

kód #3:
```javascript
TestCase('CalculatorTests', {
    setUp: function() {
        // A this.calculator minden tesztesetben egy teljesen
        // új objektumpéldány lesz!
        this.calculator = new Calculator();
    },
    testAdd_GivenTwoNumbers_ReturnsSumOfThem: function() {
        assertEquals(3, this.calculator.add(1,2));
    },
    testAdd_GivenAZero_ReturnsTheOtherNumber: function() {
        assertEquals(2, this.calculator.add(0,2));
        assertEquals(3, this.calculator.add(3,0));
    }
});
```

## 2 Hagyományos tesztelés
- nem igazán közkedvelt

### 2.1 Jellemzői és problémái
- Egy már működő kód alátámasztására és csak annak biztosítására készül, azért hogy a kód a későbbiekben is jól működjön.
- Mivel közvetlenül nem kapcsolódik a fejlesztéshez, csak annak ellenőrzéséhez, így a teszteket sokszor nem is a fejlesztők írják, erre külön munkakört biztosítanak. Ez rengeteg erőforrást köthet le mind anyagi, mind humán téren, ráadásul szükségszerűen nagy mennyiségű plusz dokumentációval jár.
- Egy meglévő kódhoz tesztet írni meglehetősen lassú és körülményes folyamat, utólag is nagy odafigyelést igényel. A tesztet író könnyedén érezhet késztetést arra, hogy az aktuálisan vizsgált programrész működését jónak tekintse, és a jelenlegi futási eredményeit írja a tesztbe. Ha valamilyen véletlen folytán az adott programrész eleve hibásan működik és ez nem derül ki, akkor a teszt a hibás működést fogja beégetni a szoftverbe.
- Semmi sem mondja meg, hogy hol a határ, hogy egész pontosan mit is ellenőrizzünk, így gyakran felesleges esetek tömkelegét fedjük le tesztekkel — ilyenkor nem biztos, hogy annyi értéket adunk a teszthalmazhoz, mint amennyivel növeljük a méretét és komplexitását.
- Hajlamosak vagyunk elbagatellizálni, ámde mégis fontos probléma, hogy egy már elkészült kódhoz meglehetősen monoton és a legtöbb ember számára unalmas munka tesztet írni.

## 3 Miben más a tesztvezérelt fejlesztés?
### 3.1 Hogyan működik a TDD?
- A tesztben megfogalmazzuk, hogy mit szeretnénk elérni, milyen osztályra, és annak milyen metódusára van szükségünk és azt hogyan szeretnénk használni. Ekkor valójában egy „interfészt” definiálunk, vagyis megadjuk, hogy jelen pillanatban, hogyan lenne a legkényelmesebb az adott szoftverrészt használnunk
- A TDD során az osztályok publikus interfészeit vizsgáljuk és mindig az elvárt működésre, sohasem az implementációra írunk tesztet!

### 3.2 A tesztvezérelt fejlesztés folyamata
A TDD három fázisból áll: 
- a tesztírásból (piros fázis)
- az implementációból (zöld fázis)
- a kód „tökéletesítéséből”, azaz a refaktorálásból.

A refaktorálás végeztével újra az első fázis következik és ez így megy egészen addig, amíg az adott programrész el nem készül.

#### 3.2.1 A piros fázis
- cél: elbukó tesztek írása
- ebben a lépésben gyakorlatilag megtervezzük a viselkedéshez szükséges interfészt, azt, hogy milyen osztályokra és azokban milyen metódusokra van szükségünk. 
- a TDD rákényszerít minket, hogy ne az implementáló, hanem az osztályt használó szemszögéből tervezzük meg a kódunkat, aminek következtében az sokkal praktikusabb, „felhasználóbarátabb” lesz!
- az ideális teszt legfeljebb 4-5 sor
- ha a teszt elkészült akkor futtassuk a teljes tesztkészletet és bizonyosodjunk meg róla, hogy egyedül az utoljára megírt bukik el és az az elvárt módon lesz sikertelen. 

#### 3.2.2 A zöld fázis
- az implementáció írása. A cél, hogy a lehető legminimálisabb kóddal elégítsük ki az előbb elbukó tesztet.
- nem érdekes, hogyha a megoldás nem elegáns vagy még egy-két sebből vérzik — csak a teszt kizöldítésével foglalkozzunk
- ha túl sokáig tart az implementáció, akkor gondoljuk át újra, hogy pontosan mit is szerettünk volna csinálni, esetleg lépjünk egy lépéssel visszább. Nyugodtan kommenteljük ki az utolsó tesztet és próbáljuk a feladatot kisebb egységekre bontani

#### 3.2.3 A refaktorálás
- kód elegánsabbá, olvashatóbbá tétele
- következő tesztesetnek megágyazhatunk
- minden módosítás után futassuk le a teljes tesztkészletet
- teszteket is refaktorálni kell

#### 3.2.4 A ciklus ismétlése
- A refaktorálás végeztével egy újabb elbukó tesztet írunk, ezzel újrakezdve a TDD ciklusát

## 4 Egy egyszerű TDD példa
A tesztvezérelt fejlesztés bemutatásához a következőkben az egyik legnépszerűbb gyakorlófeladatot, a FizzBuzz kata-t fogjuk megoldani.

A feladat a következő: készítsünk egy osztályt, ami 1 és 100 között kiírja az összes számot, azonban minden hárommal osztható helyett „Fizz”-t, minden öttel osztható helyett „Buzz”-t és minden három és öttel osztható szám helyett „FizzBuzz”-t jelenít meg.

### 4.1 Az első lépések
Az első gondolatunk talán az, hogy a feladat pontos leírása alapján fel tudnánk vázolni, hogy hogyan is képzeljük el a végeredményt szolgáltató kódot. Az azonnal látszódik, hogy szó szerint véve a feladatot elég nagy lépést kellene tenni — konkrétan száz értéket ellenőrizhetnénk, amely ráadásul a feladat összes megszorítását tartalmazná. Ebből a gondolatmenetből viszont már adódik egy egyszerűbb lépés is: mi lenne, hogyha a feladatot megoldó metódusnak egy paraméterben átadnánk, hogy meddig írja ki a sorozatot — ekkor nem kellene az összes esetet vizsgálni, elég lenne kisebbekkel elkezdeni. Ez tényleg nem tűnik bonyolultnak: az első esetben 0 elemű, majd ezt követően az 1-et tartalmazó egy elemű sorozatot várnánk el, és így tovább haladhatunk a bonyolultabb megszorítások felé.

Írjuk is meg az első tesztünket a 0 értékre, ami bár túlságosan egyszerűnek tűnhet, azonban igen fontos lépés, hiszen ilyenkor formáljuk meg, hogy milyen interfészt is képzelünk el az adott szoftverrésznek:

kód #4:
```javascript
TestCase('FizzBuzz', {
    testDisplay_TakeZero_ReturnsEmptyList: function() {
        var fizzBuzz = new FizzBuzz();
        assertEquals('', fizzBuzz.display(0));
    }
});
```

Már ezen egyszerű teszt írása során is meg kellett hoznunk néhány igen fontos döntést: 
- mi legyen a megoldó osztály és metódus neve. 
- mi legyen a visszatérési érték típusa

Elkészült az első tesztünk, melyet futtatva természetesen hibát kapunk: nincs FizzBuzz nevű változónk. Itt az ideje, hogy elkészítsük az első implementációt. Most csupán annyi a dolgunk, hogy a lehető legegyszerűbb kódot írjuk meg, ami kielégíti a tesztet, semmiképp ne írjunk bele plusz tudást, egyelőre szükségtelen részeket.

kód #5:
```javascript
"use strict";
var FizzBuzz = function() ;
FizzBuzz.prototype = {
    display: function() {
        return '';
    }
};
```

Semmi másra nincs szükségünk, minthogy visszaadjuk az üres sztringet. Ennél egyszerűbb megoldás valószínűleg nincs is, és jelen pillanatban teljesen megfelel. A tesztünk szépen lefut, a zöld fázisba kerültünk, azaz elkezdhetünk refaktorálni. Mivel egyelőre nincs túl sok kódunk és nem is feltétlenül látjuk, hogy a következő lépést mi könnyítené meg, így egyszerűen hagyjuk így és írjuk meg a második tesztesetet.

kód #6:
```javascript
TestCase('FizzBuzz', {
    testDisplay_TakeZero_ReturnsEmptyList: function() {
        var fizzBuzz = new FizzBuzz();
        assertEquals('', fizzBuzz.display(0));
    },
    testDisplay_TakeOne_ReturnsTheFirstElement: function() {
        var fizzBuzz = new FizzBuzz();
        assertEquals('1', fizzBuzz.display(1));
    }
});
```

Piros fázisba érkeztünk, itt az ideje a teszt kizöldítésének, melyhez nincs másra szükség, mint a paraméter felhasználására:

kód #7:
```javascript
"use strict";
var FizzBuzz = function() ;
FizzBuzz.prototype = {
    display: function(lengthOfSequence) {
        if (!lengthOfSequence) {
            return '';
        }
        return '1';
    }
};
```

A tesztünk immáron zöld, így elkezdhetjük a kód refaktorálását. Az implementáció egyelőre nem tűnik olyannak, amihez hozzá kellene nyúlni — annál inkább a tesztek, ott ugyanis ordas problémára figyelhetünk fel: a FizzBuzz konstruálása duplikátumként szerepel. Nem jó gyakorlat a jövőre feltételezéseket tenni, viszont most egész biztosak lehetünk benne, hogy ezután is hasonló módon fogjuk konstruálni a sorozatot, így emeljük át ezt a műveletet a tesztkészlet setUp-jába:

kód #8:
```javascript
TestCase('FizzBuzz', {
    setUp: function() {
        this.fizzBuzz = new FizzBuzz();
    },
    testDisplay_TakeZero_ReturnsEmptyList: function() {
        assertEquals('', this.fizzBuzz.display(0));
    },
    testDisplay_TakeOne_ReturnsTheFirstElement: function() {
        assertEquals('1', this.fizzBuzz.display(1));
    }
});
```

### 4.2 Az üzleti logika kiemelése
Itt az idő, hogy továbbhaladjunk egy új iterációval, azaz írjunk egy elbukó tesztet. Értelemszerűen a két elemű lista következik, amely már jelentős mértékben módosítja a kimenetet, hiszen több számot is meg kell jeleníteni.

Az már előre konstatálható, hogy az implementáció nem lesz olyan egyszerű, mint eddig — adott esetben most megállhatnánk és úgy refaktorálhatnánk a kódot úgy, hogy a következő lépést már könnyebben tehessük meg. Ettől azonban most tekintsünk el, és próbáljuk meg, hátha néhány sorból is meg tudjuk oldani a feladatot.

A teszt bizonyára nem okoz nehézséget, egyetlen kérdésre kell csak választ adni: hogyan válasszuk el a lista elemeit? Mivel a specifikáció nem pontosított a kimenettel kapcsolatban, egyelőre használjuk a vesszőt szeparátorként:

kód #9:
```javascript
testDisplay_TakeTwo_ReturnsTheFirstTwoElement: function() {
    assertEquals('1,2', this.fizzBuzz.display(2));
}
```

Újra piros fázisba érkeztünk, írjuk is meg a tesztet kielégítő implementációt. Legegyszerűbbnek az tűnik, hogyha egy tömbbe pakoljuk a számokat addig amíg elérjük a kívánt szekvencia-méretet, végül a szeparátorral összekonkatenáljuk a tömb elemeit.

kód #10:
```javascript
FizzBuzz.prototype = {
    display: function(lengthOfSequence) {
        if (!lengthOfSequence) {
            return '';
        }
        var fizzBuzzSequence = [];
        for (var i=1; i<=lengthOfSequence; i++) {
            fizzBuzzSequence.push(i);
        }
        return fizzBuzzSequence.join(',');
    }
};
```

A teszt kielégült, nekiállhatunk a refaktorálásnak. Egyértelmű, hogy a display metódus kezd egy kissé túlterhelődni, próbáljuk meg egy kissé szétbontani:

kód #11:
```javascript
FizzBuzz.prototype = {
    SEPARATOR: ',',
    display: function(lengthOfSequence) {
        if (!lengthOfSequence) {
            return '';
        }
        return this._getSequenceUntil(lengthOfSequence).join(this.SEPARATOR);
    },
    _getSequenceUntil: function(length) {
        var fizzBuzzSequence = [];
        for (var i=1; i<=length; i++) {
            fizzBuzzSequence.push(i);
        }
        return fizzBuzzSequence;
    }
};
```

Mivel a JavaScriptben nem triviális privát osztálymetódusokat használni, így konvenció szerint alulvonással jelöljük azokat a függvényeket, amelyeket ilyen tulajdonságúnak gondolunk. Természetesen az óvatlan fejlesztőt semmi sem akadályozza meg abban, hogy az osztályon kívülről is meghívja a _getSequenceUntil-hoz hasonló metódusokat — azonban mindig gondoljunk arra, hogy az ilyen függvények nincsenek közvetlenül tesztekkel alátámasztva!

Az üres teszteset külön viselkedést vizsgál, de a számot visszaadók már ugyanazt az esetet valósítják meg. Érdemes lenne ezt a redundanciát megszüntetni azzal, hogy a hétköznapi számok megjelenítését vesszük egy tesztesetnek, vagyis összevonjuk az utolsó két tesztet. Rendben, a szándék megvan, már csak egy nevet kell találni a tesztnek. Első körben legyen:

kód #12:
```javascript
testDisplay_TakeFewerThanThree_ReturnsOnlyNumbers: function() {
    assertEquals('1', this.fizzBuzz.display(1));
    assertEquals('1,2', this.fizzBuzz.display(2));
}
```

Következő teszteset:

kód #13:
```javascript
testDisplay_TakeFirstFour_ReturnsFizzWhenDivisibleByThree: function() {
    assertEquals('1,2,Fizz,4', this.fizzBuzz.display(4));
}
```

Piros fázisban vagyunk, zöldítsük ki a tesztet. A legegyszerűbb megoldás, ha egy elágazást teszünk a ciklusba:

kód #14:
```javascript
FizzBuzz.prototype = {
    SEPARATOR: ',',
    display: function(lengthOfSequence) {
        if (!lengthOfSequence) {
            return '';
        }
        return this._getSequenceUntil(lengthOfSequence).join(this.SEPARATOR);
    },
    _getSequenceUntil: function(length) {
        var fizzBuzzSequence = [];
        for (var i=1; i<=length; i++) {
            if (i % 3 === 0) {
                fizzBuzzSequence.push('Fizz');
            } else {
                fizzBuzzSequence.push(i);
            }
        }
        return fizzBuzzSequence;
    }
};
```

A teszteket futtatva csupa zöldet kapunk — ámde a kezdeti örömöt hamar bánat kendőzheti, hogyha jobban megnézzük a kódjainkat. Mind a teszt, mind az implementáció elindult egy problémás irányba.

A tesztjeink egyre bonyolultabbá válnak, ráadásul az egyes tesztek a korábbiakra épülnek. Tegyük fel, hogy megírtuk az összes tesztet, lesz már egy legalább 5 elemű lista a Buzz-al és egy 15 elemű a FizzBuzz-al, ámde valamit elrontunk a számok kiírásával kapcsolatban. Ebben az esetben az összes tesztünk el fog bukni, hiszen mindegyikben építettünk arra, hogy a számokat jól jelenítjük meg. Sok esetben nem oldható meg, hogy a tesztek teljesen függetlenül vizsgálják a hozzájuk tartozó viselkedést, azonban — amennyire lehet — törekedni kell rá.

Az implementáció sincs jobb helyzetben. A _getSequenceUntil metódus most már túl sok felelősséget hordoz magában: nem csak a szekvencia végigszámolásáért felel, de az értékek helyes fordításáért is.

Természetesen osztályon belül is javíthatnánk a problémán, hogyha a felelősségeket több metódus között osztanánk fel — azonban maga az osztály még ebben az esetben is két jelentősen különböző feladatot látna el.

Szedjük össze, hogy mit is kellene megoldani: - A teszteket egyszerűbbé és függetlenebbé kellene tenni. - Valahogy el kellene választani a szekvencia-készítés és a számból FizzBuzz értékre való fordítást.

Ahogy a legtöbb problémára, így a fentiekre is megoldást jelent, ha viselkedéseket külön osztályokba szervezzük ki: maradjon a FizzBuzz-ban a szekvencia létrehozása, azonban hozzunk létre egy FizzBuzzTranslator osztályt, ami az adott szám lefordításáért felel.

Az a szerencsés eset áll fenn, hogy a zöld fázisban vagyunk, vagyis — jelen pillanatban bármit is csinálunk — ha a tesztek lefutnak, akkor nem rontottuk el a program működését. E védőháló alatt nyugodtan hozzunk létre egy új osztályt és egyszerűen emeljük ki a fordítást:

kód #15:
```javascript
var FizzBuzzTranslator = function() ;
FizzBuzzTranslator.prototype = {
    getValueOf: function(number) {
        if (number % 3 === 0) {
            return 'Fizz';
        }
        return number;
    }
};

// tesztelendő kód
var FizzBuzz = function() {
    this.translator = new FizzBuzzTranslator();
};
FizzBuzz.prototype = {
    SEPARATOR: ',',
    display: function(lengthOfSequence) {
        if (!lengthOfSequence) {
            return '';
        }
        return this._getSequenceUntil(lengthOfSequence).join(this.SEPARATOR);
    },
    _getSequenceUntil: function(length) {
        var fizzBuzzSequence = [];
        for (var i=1; i<=length; i++) {
            fizzBuzzSequence.push(this.translator.getValueOf(i));
        }
        return fizzBuzzSequence;
    }
};


```

Adósak maradtunk a tesztek átalakításával: a létrejött új osztályt is le kellene tesztelni. Mivel már elkészültek a teszteseteink így első körben emeljük át ezeket, ügyelve arra, hogy ne módosítsunk a jelentésükön — az új osztály interfészével, de ugyanazt vizsgálják:

kód #16:
```javascript
TestCase('FizzBuzzTranslator', {
    setUp: function() {
        this.translator = new FizzBuzzTranslator();
    },
    testGetValueOf_TakeFewerThanThree_ReturnsOnlyNumbers: function() {
        assertEquals('1', this.translator.getValueOf(1));
        assertEquals('1,2', this.translator.getValueOf(1) + ',' +
                            this.translator.getValueOf(2));
    },
    testGetValueOf_TakeFirstFour_ReturnsFizzWhenDivisibleByThree: function() {
        assertEquals('1,2,Fizz,4', this.translator.getValueOf(1) + ',' +
                                   this.translator.getValueOf(2) + ',' +
                                   this.translator.getValueOf(3) + ',' +
                                   this.translator.getValueOf(4));
    }
});
```

Amint az látható csak néhány módosítást eszközöltünk: - Ezúttal kézzel kellett előállítanunk a sorozatot. - Az üres bemenettel nem foglalkozik a fordító, így ez a teszt törölhető. - A tesztek neveiben módosítottuk a metódus nevét.

A fenti felesleges lépésnek tűnhet, azonban ne feledjük: a jó TDD kulcsa az apró lépésekben rejlik. Mindig a lehető legkevesebb módosítást végezzük és állandóan futtassuk a teszteseteinket. Egy bonyolultabb kód esetén sokat segíthet, ha betartjuk a fenti szabályt és először egy az egyben próbáljuk átemelni a teszteket.

Most, hogy megbizonyosodtunk arról, hogy minden rendben, az új osztályunk megfelelően működik, elkezdhetjük a saját tesztjeinek refaktorálását. Jelen pillanatban két viselkedést vizsgálunk, a számból számra, és a számból Fizz-re való fordítást, így erre írjunk két új tesztesetet. Ha ezek elkészültek — és lefut az összes teszt — akkor az áthozott két tesztet ki is törölhetjük, mivel az újak mellett redundánssá váltak. A végeredmény valahogy így fog kinézni:

kód #17:
```javascript
TestCase('FizzBuzzTranslator', {
    setUp: function() {
        this.translator = new FizzBuzzTranslator();
    },
    testGetValueOf_GivenSimpleNumber_ReturnsTheNumber: function() {
        [1,2,4,7,8,11,13,14].forEach(function(number) {
            assertEquals(number, this.translator.getValueOf(number));
        }, this);
    },
    testGetValueOf_GivenNumberDivisibleByThree_ReturnsFizz: function() {
        [3,6,9,12].forEach(function(number) {
            assertEquals('Fizz', this.translator.getValueOf(number));
        }, this);
    }
});
```

### 4.3 A végső simítások
Kicsit gondban lehetünk a következő lépéssel kapcsolatban: mihez írjunk tesztet? A fordítóhoz vagy a magasabb-szintű osztályhoz? A legjobb, hogyha nem bonyolítjuk túl a kérdést: a fordítóhoz tudunk a legkönnyebben tesztet írni és a feladat szempontjából is ez a legfontosabb rész jelenleg. Ettől függetlenül mintegy elérendő célként azért vázoljuk fel a végeredményt ellenőrző kódot, így ha végeztünk a FizzBuzzTranslator-al, akkor egyből láthatjuk, hogy minden rendben van-e.

Ha a megoldást végiggondoljuk, akkor könnyen rájöhetünk, hogy a FizzBuzz számok 15 elemenként ismétlődnek. Ha megvizsgáljuk az első 30 elemet, akkor a megszorításokat és az ismétlődést is ellenőrizzük, így elég ennyi:

kód #18:
```javascript
testDisplay_TakeFirst30_ReturnsFirst30FizzBuzzElements: function() {
    var expected = '1,2,Fizz,4,Buzz,Fizz,7,8,Fizz,Buzz,' +
                   '11,Fizz,13,14,FizzBuzz,16,17,Fizz,19,Buzz,' +
                   'Fizz,22,23,Fizz,Buzz,26,Fizz,28,29,FizzBuzz',
        result = this.fizzBuzz.display(30);
    assertEquals(expected, result);
}
```

A teszt tökéletesen megfelel a végső célként — mivel egyelőre messze vagyunk tőle, hogy működjön így most nyugodtan kommenteljük ki. Ez a módszer sokat segíthet a tervezésben, hiszen előre átgondolhatjuk a végeredmény összefüggéseit és azt, hogy esetleg milyen plusz interfészre van szükség a megoldáshoz.

Most újra zöld fázisban vagyunk, így megírhatjuk a következő elbukó tesztet, ezúttal már a fordító osztályba:

kód #19:
```javascript
testGetValueOf_GivenNumberDivisibleByFive_ReturnsBuzz: function() {
    [5,10].forEach(function(number) {
        assertEquals('Buzz', this.translator.getValueOf(number));
    }, this);
}
```

A tesztben semmi meglepő nincs, futtassuk is az összeset, így megbizonyosodva arról, hogy valóban a piros fázisban vagyunk. Az implementáció is meglehetősen egyszerű:

kód #20:
```javascript
FizzBuzzTranslator.prototype = {
    getValueOf: function(number) {
        if (number % 3 === 0) {
            return 'Fizz';
        }
        if (number % 5 === 0) {
            return 'Buzz';
        }
        return number;
    }
};
```

Itt az ideje a refaktorálásnak — adja is magát a FizzBuzzTranslator, hogy egy kissé beszédesebbé tegyük:

kód #21:
```javascript
FizzBuzzTranslator.prototype = {
    getValueOf: function(number) {
        if (this._isFizz(number)) return 'Fizz';
        if (this._isBuzz(number)) return 'Buzz';
        return number;
    },
    _isFizz: function(number) {
        return number % 3 === 0;
    },
    _isBuzz: function(number) {
        return number % 5 === 0;
    }
};
```

Már csak egyetlen teszteset van vissza, a FizzBuzz érték — mivel a fejlesztési folyamat során már — tudtunkon kívül — igen jól megágyaztunk neki, így nem nehéz a teszt és az implementációja sem:

kód #22:
```javascript
testGetValueOf_GivenNumberDivisibleByThreeAndFive_ReturnsFizzBuzz: function() {
    [15,30].forEach(function(number) {
        assertEquals('FizzBuzz', this.translator.getValueOf(number));
    }, this);
}
// És a kizöldítéshez szükséges implementáció:
var FizzBuzzTranslator = function() ;
FizzBuzzTranslator.prototype = {
    getValueOf: function(number) {
        if (this._isFizzBuzz(number)) return 'FizzBuzz';
        if (this._isFizz(number)) return 'Fizz';
        if (this._isBuzz(number)) return 'Buzz';
        return number;
    },
    _isFizz: function(number) {
        return number % 3 === 0;
    },
    _isBuzz: function(number) {
        return number % 5 === 0;
    },
    _isFizzBuzz: function(number) {
        return this._isFizz(number) && this._isBuzz(number);
    }
};
```

Fejezzük be a feladatunkat egy kis refaktorálással. A FizzBuzz tesztjei közül kivehetjük a redundánssá vált teszteseteket.

kód #23:
```javascript
TestCase('FizzBuzz', {
    setUp: function() {
        this.fizzBuzz = new FizzBuzz();
    },
    testDisplay_TakeZero_ReturnsEmptyList: function() {
        assertEquals('', this.fizzBuzz.display(0));
    },
    testDisplay_TakeFirst30_ReturnsFirst30FizzBuzzElements: function() {
        var expected = '1,2,Fizz,4,Buzz,Fizz,7,8,Fizz,Buzz,' +
                       '11,Fizz,13,14,FizzBuzz,16,17,Fizz,19,Buzz,' +
                       'Fizz,22,23,Fizz,Buzz,26,Fizz,28,29,FizzBuzz',
            result = this.fizzBuzz.display(30);
        assertEquals(expected, result);
    }
});

var ASSERT_SAME = 'assert_same';
TestCase('FizzBuzzTranslator', {
    setUp: function() {
        this.translator = new FizzBuzzTranslator();
    },
    testGetValueOf_GivenSimpleNumber_ReturnsTheNumber: function() {
        // Hagyjunk pár példát getValueOf használatára szem előtt
        // ezzel segítve a teszt dokumentációs jellegét
        assertEquals(1, this.translator.getValueOf(1));
        assertEquals(2, this.translator.getValueOf(2));
        [4,7,8,11,13,14].forEach(this.assertTranslation(ASSERT_SAME));
    },
    testGetValueOf_GivenNumberDivisibleBy3_ReturnsFizz: function() {
        [3,6,9,12].forEach(this.assertTranslation('Fizz'));
    },
    testGetValueOf_GivenNumberDivisibleBy5_ReturnsBuzz: function() {
        [5,10].forEach(this.assertTranslation('Buzz'));
    },
    testGetValueOf_GivenNumberDivisibleBy3And5_ReturnsFizzBuzz: function() {
        [15,30].forEach(this.assertTranslation('FizzBuzz'));
    },
    assertTranslation: function(translation) {
        var assertion = function(number) {
                assertEquals(
                    translation === ASSERT_SAME ? number : translation,
                    this.translator.getValueOf(number)
                );
            };
        return assertion.bind(this);
    }
});
```

## 5 Mikor jó egy teszt?
Fontos tisztában lenni azzal, hogy a TDD során elsősorban egységteszteket (unit teszt) próbálunk írni, azok minden szabályát betartva. Hogy ez mit jelent, talán Michael Feathers, az agilis fejlesztési módszertanok egyik úttörője és hangadója definiálta a legszemléletesebben: „Egységteszt az a teszt, ami gyors. Ha egy teszt lassú, akkor az nem egységteszt!”. Nem nevezhetőek unit tesztnek azok a tesztek, amelyek egy hálózaton keresztül más gépekkel kommunikálnak, amelyek használják a fájlrendszert vagy az adatbázist. Az ilyen tesztek lassúak, nehézkes definiálni őket és sokszor speciális környezeti beállításokat igényelnek. A TDD során az egyik legfontosabb szempont a tesztek egyszerűségének megőrzése.

Sokan tévesen úgy gondolják, hogy a különböző egységek (pl. osztályok) tesztjeinek teljesen függetlennek kell lenniük egymástól. Ez nem feltétlenül igaz, amíg a teszt gyors marad és azt a viselkedést vizsgálja, amire kíváncsiak vagyunk. Ha teljesen függetleníteni szeretnénk egymástól az osztályok tesztjeit akkor könnyedén beleesünk abba a csapdába, hogy az implementációt teszteljük és nem a viselkedést. Ez úgy fordulhat elő, hogy a vizsgálataink arra szorítkoznak, hogy A osztály meghívja-e B osztály adott metódusát, és használja-e a C osztályt. Ha így teszünk, akkor a refaktorálás egy kész rémálom lesz, egy egyszerű metódus átmozgatása egyik osztályból a másikba is problémás lehet — hiszen az implementációt ellenőrző tesztek mindegyikét módosítani kell.