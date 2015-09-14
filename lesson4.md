# Függvények és a futási környezet

- first class tulajdonság:
    - futási időben létrehozhatók
    - ők maguk is objektumok, a Function objektumtól származnak
    - Lehetnek saját property-jeik, sőt metódusaik is, melyek referenciát tartalmaznak a konstruktorukra.
    - Eltárolhatóak változóban.
    - Átadhatóak más függvényeknek paraméterként.
    - Szerepelhetnek visszatérési értékként.

## 1 Függvények létrehozása

- mindig van visszatérési érték (ha nincs return, akkor *undefined*)
- nem muszáj minden paramétert megadni (amit nem adunk meg, az *undefined*)

### 1.1 létrehozási módok:

- deklarációval (kód #1)
    - az egész kontextusban elérhető
    - kötelező nevet megadni

kód #1:
```
function name([param[, param[, ... param]]]) {
   statements
}
```

- függvénykifejezésként (kód #2 - kód #6)
    - ojektum-referenciaként jelenik meg
    - a név opcionális (anonym függvény)

kód #2:
```
function [name]([param[, param[, ... param]]]) {
   statements
}
```
kód #3
```
// Anonym függvény átadása egy változónak
var getHero = function() {
    return 'Superman';
};
```
kód #4
```
// Függvénykifejezést bárhol használhatunk, ahol egyébként
// objektumokat használhatunk, így pl. lehet visszatérési
// érték is:
var greetingFactory = function() {
    return function(hero) {
        return "Hi, I'm " + hero + "!";
    };
};
var greeting = greetingFactory();
assertEquals("Hi, I'm Superman!", greeting('Superman'));
```
kód #5
```
// Ha egy függvénykifejezést egy objektum property-jéhez 
// rendeljük, akkor az a függvény az objektum metódusa lesz
var greeting = {
        fromSuperman: function() {
            return this.say('Superman');
        },
        say: function(hero) {
            return "Hi, I'm " + hero + "!";
        }
    };
assertEquals("Hi, I'm Superman!", greeting.fromSuperman());
```
kód #6
```
// tömbök elemeiként vagy objektum tulajdonságaiként
var greetings = [
        function() { return 'Hey boy, welcome!'; },
        function() { return "Hi, I'm a superhero!" }
    ],
    superman = {
        greet: greetings[1]
    };
assertEquals("Hi, I'm a superhero!", greetings[1]());
assertEquals("Hi, I'm a superhero!", superman.greet());


```

- nyíl operátorral (kód #7) // ES6
    - anomym függvények
    - egysoros metódus esetén elhagyhatjuk a kapcsos zárójeleket, ez esetben a kifejezés egyben a visszatérési érték is lesz
    - amennyiben nincs paraméter akkor a nyíl operátor elé mindenképp ki kell írni egy üres paraméterlistát, egy paraméternél viszont elhagyhatóak a zárójelek!

kód #7:
```
([param[, param[, ... param]]]) => {
   statements
}
param => expression

var getHero = () => 'Superman';
var showHero = (hero, address) => {
                    return hero + ' from ' + address;
                };

// Tényleges használata lehet
var heroes = [
        'Superman',
        'Spiderman',
        'Sugarman'
    ];
// Ha össze szeretnénk számolni a hősök neveinek hosszát, akkor
// legegyszerűbb a map függvényt használni:
heroes.map(function(hero) { return hero.length; });

// A Fat arrow operátorral lényegesen expresszívebb kódot kapunk:
heroes.map( hero => hero.length );

```

- Function objektummal
    - a függvénytörzset akár sztringként is átadhatjuk
    - nem ajánlott a használata (böngészők nem mindig optimalizálják jól)

- Változók kontextusa
    - függvény-scope: a függvény törzsén belül bárhol elérhető
    - minden hivatkozott változó létezik, de amíg nincs értékadás, *undefined* lesz az értékük
    - függvényen belüli függvény esetén a belső függvény látja a külső függvény változóit, hiszen a belső függvény is a külső függvény scope-jában van. Ha belső függvényben definiálunk egy ugyanolyan nevű változót, azzal elfedjük a külső függvény változóját
    - a globális névtér a szkript betöltődésekor automatikusan lefut
    - ne szennyezzük a globális névteret

- Önkioldó függvény
    - szükség van valamilyen megoldásra ahhoz, hogy elszeparálhassuk a privát, csak az aktuális kódhoz tartozó változókat a globális névtértől
    - Az önkioldó függvényekkel a JavaScriptben natívan nem létező névtereket emulálhatjuk következő módon (kód #8):

kód #8:
```
(function() {
    var hero = 'Superman',
        greet = function() {
            return "Hi I'm " + hero;
        };
    greet();
}());
// Itt, az önkioldó függvényen kívül nem érhető el a hero
// változó és a greet metódus.
```

- Függvényargumentumok
    - minden paraméter értékként kerül átadásra, így csak a függvényen belül módosul, KIVÉVE az objektumokat, melyeket referenciaként adunk át. Ha egy objektum változik a függvényen belül, változni fog kívül is
    - ES6: már vannak default paraméterek pl.: function(name='Superman'), melynél ha *undefined* értéket adunk át függvényhíváskor, akkor a default érték lesz mérvadó. SŐŐŐŐT, default érték bármilyen kifejezés lehet, akár függvényhívás is
    - ES6: változó számú paraméter lehetősége pl.: function(name, ...datas) , ahol datas a névtelen paraméterek tömbje

- Callback függvények
    - érdemes opcionálisként kezelni, hiszen nem mindenki akar callback-et hívni // erre szebb megoldás a guard operátor: *callback && callback()*

kód #9:
```
var showHero = function(name, callback) {
    console.log("I'm " + name);
    if (callback) {
        callback();
    }
};
showHero('Superman', function() {
    console.log('Callback invoked!');
});
// => "I'm Superman"
// => "Callback invoked!"
```

- Closure
    - a JavaScript bármely változója elérhető addig, amíg egy referencia mutat rá. Amint megszűnik ez a kötés, úgy a futtatókörnyezet memóriafelszabadításért felelős egysége, a Garbage collector véglegesen törölheti a memóriából. Ezt a jelenséget closure-nek nevezzük
    - az alábbi kódban a *hero* addig él, míg van rá referencia (ez esetben amíg callback le nem fut). Lehet, hogy csak másodpercekkel később törlődik

kód #10:
```
var setHeroGreeting = function() {
    var hero = "Superman";
    $('#greet').bind('click', function() {
        alert(hero);
    });
};
setHeroGreeting();

// a $('#greet').unbind('click'); paranccsal nem mutat több referencia a hero-ra, így a Garbage collector törölheti

```

## 2 Függvény, mint objektum

- meghívható
- *new* kulcsszó: objektum/függvény klónozása
    - nem szokás explicit visszatérési értéket megadni, hiszen így is visszadja az aktuálisan létrehozott objektumot

kód #11:
```
var Hero = function(name) {
    this.name = name;
};
var superman = new Hero('Superman'),
    spiderman = new Hero('Spiderman');
assertEquals('Superman', superman.name);
assertEquals('Spiderman', spiderman.name);
```

### 2.1 Prototípus alapú öröklődés
- pszeudo-klasszikus objektum öröklődés
- A JavaScript objektumai futási időben, dinamikusan bővíthetőek, így ha egy objektum prototípusát bővítjük valamivel, akkor ez a módosítás az összes olyan objektumra hatással lesz, amelynek ő a prototípusa!
- _ (alulvonás) a tulajdonság neve előtt priváttá teszi a változót

kód #12:
```
var Hero = function(name) {
    this.name = name;
};
Hero.prototype.getName = function() {
    return this.name;
};
var Superman = function(name) {
    this.name = name;
};

// Ha csak a Hero-t adnánk át, akkor az ő 
// prototípusában található függvények nem 
// lennének elérhetőek a Superman számára, 
// mindenképp egy objektumpéldányra van szükségünk
Superman.prototype = new Hero; 
Superman.prototype.fly = function() {
    return "Woohooo I'm flying!";
};
var clark = new Superman('Clark Kent');
assertEquals('Clark Kent', clark.name);
assertEquals('Clark Kent', clark.getName());
assertEquals("Woohooo I'm flying!", clark.fly());
```

kód #13:
```
var Hero = function(name) {
    this.name = name;

    // Ez a metódus minden példány számára létre lesz hozva, ami
    // felesleges memóriafoglalással jár, holott az egyes példányok
    // osztozhatnának rajta.
    this.greet = function() { return 'Hi!'; }
}

// Ez a metódus minden objektum-példányban elérhető, azonban csak
// egyszer jön létre, a példányokban csupán referencia mentődik el
// rá. Ez a megoldás jóval memóriakímélőbb.
Hero.prototype.getName = function() {
    return this.name;
};
var hero = new Hero('Superman');
assertEquals('Superman', hero.getName());
assertEquals('Hi!', hero.greet());
```

- prototípus konstruktorának meghívása (call):

kód #14:
```
var Hero = function(name) {
    this.name = name;
};
var Superman = function(name) {
    Hero.call(this, name);
};
var clark = new Superman('Clark Kent');
assertEquals('Clark Kent', clark.name);
```

## 3 A this jelentése
A *this* arra az objektumra utal, aminek a kontextusában éppen meghívjuk, azonban lehetőségünk van ezt a viselkedést befolyásolni.

### 3.1 Globális kontextus
kód #15:
```
// A böngészőkben a globális objektum a window
assertEquals(this, window);
this.hero = 'Superman';
assertEquals('Superman', window.hero);
```

### 3.2 Függvény kontextus
Egy függvényen belül mindig a hívás helyétől és módjától függ a this értéke.
- egyszerű függvényhívás törzsében
    - Egyszerű függvényhívás esetén strict módban undefined lesz a this értéke, egyébként pedig referencia a globális objektumra.
    - Strict módban véletlenül sem tudjuk módosítani a globális scope-ot, használata ezért is erősen ajánlott.

kód #16:
```
var sillyHero = function() {
        return this;
    },
    betterHero = function() {
        "use strict";
        return this;
    };
assertEquals(window, sillyHero());
assertEquals(undefined, betterHero());
```

- objektum metódusában
    - ha egy függvényt egy objektum metódusaként hívunk meg, akkor a this értéke az az objektum lesz, amin a függvényt meghívtuk.
    - Figyeljünk oda arra, hogy egy metódusbeli belső függvény már egyszerű függvénydefiníciónak minősül, így az ott szereplő this értéke a globális objektum (strict módban pedig undefined) lesz! 

kód #17:
```
var hero = {
    name: 'Superman'
};
function getName() {
    return this.name;
};
// A függvényre mutató referenciát később adjuk hozzá a
// hero objektumhoz, ha így, metódusként hívjuk meg
// akkor a this az objektumra fog mutatni:
hero.getName = getName;
assertEquals('Superman', hero.getName());
assertEquals(undefined, getName());
```

- konstruktor függvények törzsében
    - ne írjuk felül a visszatérési értéket. nem kell return


### 3.3 Megadott kontextus

A JavaScript — a Function objektum prototípusán — három különböző metódust biztosít arra, hogy saját magunk állíthassuk be a this jelentését — más szóval, mi szabhassuk meg a függvény futási kontextusát.

- call és apply
    - mintha normálisan meghívnánk a függvényt

kód #18:
```
heroGreeting.call(superman, 'Kripton', "Hi there, I'm "));
heroGreeting.apply(superman, ['Kripton', "Hi there, I'm "]));
```

- bind 
    - Az ES5 előtti időkben sokszor a this értékét egy _this, that vagy self nevű változóba mentettünk el, hogy a callback metódusokban is elérhető legyen az előző scope. A bind segítségével erre nincs szükség:

```
getIntroductionHTML: function() {
    var formattedName = (function(tag) {
            return '<' + tag + '>' + this.name + '</' + tag + '>';
    }).bind(this);

    return '<div class="name">' +
              formattedName('span') +
           '</div>';
}
```

### 3.4 A fat arrow kontextus
Az ES6-ban bemutatkozó fat arrow operátor olyan függvényt hoz létre, melynek kontextusa mindig az a scope, amiben őt definiáltuk.

## 4 Generátor-függvények (ES6)
Hatékony eszközök olyan metódusok írására, melyek valamilyen állapotot őriznek és minden hívásra továbblépnek egy következő állapotba.