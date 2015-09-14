# 1 Primitív értékek:

- string
- numeric
- boolean
- null 
- undifined
- regularExpression

## 1.1 string

- nincs string interpoláció (változó a szövegben)

## 1.2 numeric

- csak lebegőpontos számok (float)
- Infinity, -Infinity, NaN, egyértelmű számértékek
- csak 10-es számrendszerben dolgozik, egyéb mást átkonvertál
- IEEE 754 lebegőpontos számábrázolás

## 1.3 boolean

- false-t (falsy-k) jelentenek:
    + false
    + null
    + undefined
    + 0
    + NaN
    + ""
- true minden más, ami nem a falsy listából való (még ha üresek is)

## 1.4 egyéb

- undefined
    + változók alapértéke
    + ha nincs a függvénynek visszatérési értéke
- null
- requralExpression

# 2 Típusok

- typeof operátor csak a következő típusokat azonosítja:
    + number
    + undefined
    + object
    + boolean
    + string
    + function
    + xml
- minden másra object-et ad
- automatikus típuskonverziót kerüljük (number összeadása stringgel)

## 2.1 Number

- numeric értékű lesz
- isNaN(), isFinite(), typeof number === 'number'
- minden művelet előtt számmá konvertálja az operandusokat, kivéve az összeadást, mivel az megegyezik a string konkatenációval, ami erősebb precedenciájú, így a számot fogja szöveggé konvertálni
- parseInt() // ES5-öt NEM támogató böngészőkben detektálni próbálja a string számrendszerét, ezért második paraméterként érdemes megadni a 10-et, mint számrendszer azonosítót
- parseFloat()
- Number objektumként használhatjuk:
    + szam.toPrecision() pl.: 12.toPrecision(4); // "12.00"
    + szam.toFixed() függvényeket 
- Math objektum: 
    + Math.abs()        // abszolút érték
    + Math.floor()      // lefelé kerekítés
    + Math.ceil()       // felfelé kerekítés
    + Math.random()     // véletlen szám 0 és 1 között; pl.: Math.ceil(Math.random()*10) - 1 és 10 között

## 2.2 String

- nincs különbség ' és " között (nincs string interpoláció)
- egyetlen operátora a konkatenáció +
- String objektumként használhatjuk:
    + 'szoveg'.trim()
    + 'szoveg'.toUpperCase()
    + 'szoveg'.length
    + 'szoveg'.trim()
    + 'szoveg'.replace('keresendo', 'csere')
    + 'szoveg'.search('keresendo') // index vagy -1
    + 'szoveg'.split(',') // tömböt csinál a szövegből

## 2.3 boolean

- operátorai: !(tagadás), &&(és), ||(vagy)
- GUARD operátor (láncolt &&): csak addig fut a kiértékelés, amíg igaz pl.: kony && konyv.repul && konyv.repul() - nem jut el a függvényhívásig, mert a konyv-nek nincs repul függvénye
- DEFAULT operátor (láncolt ||): addig fut a kiértékelés, amíg hamis pl.: var name = '' || 0 || "Attila"; // a name értéke "Attila" lesz

## 2.4 reqularExpression

- változó beemelése csak az objektum használatával működhet pl.: new RegExp('a ' + string);
- . - a sortörésen kívül bármilyen karakter
- ? - az előtte álló karakter maximum 1x szerepelhet
- \* - az előtte álló karakter akárhányszor szerepelhet
- \+ - az előtte álló karakter legalább egyszer szerepeljen
- {n}, {n, m} - n darab VAGY n és m között szerepelhet
- \ - speciális karakterek védése pl.: \.
- [] - adott helyen engedélyezett karakterek pl.: [mno], [0-9], [a-zA-Z]
, [^0-9] // negáció
- | - "vagy"-ot jelent pl.: /A|B/.test('A') // true
- ^ - a szöveg elejét jelöli
- $ - a szöveg végét jelöli
- /i kapcsoló - kis- és nagybetű érzékenység kikapcsolása pl.: /aABb/i
- /g kapcsoló - globális ellenőrzés, cseréknél fontos, különben csak az első találatot cserélné ki
- () csoportosítás *backreference* - pl.: /(<.+>(.+)<\/.+>)/.test('<strong>valami</strong>')
- (?:x) *egyszerű ellenőrzés* - x egy tetszőleges kifejezés, nem kapunk vissza backreference-t
- x(?=y) *előretekintő ellenőrzés* - megmondhatjuk, hogy egy karaktersor után pontosan milyen kifejezést várunk el, nem kapunk vissza backreference-t
- x(?!y) *negált előretekintő ellenőrzés* - pontosan ugyanúgy működik mint az előretekintő ellenőrzés, csak éppen azt ellenőrizzük, hogy x után semmiképp sem szerepelhet y
- rövidítések: 
    + \w - [A-Za-z0-9_]
    + \W - [^A-Za-z0-9_]
    + \d - [0-9]
    + \D - [^0-9]
    + \s - whitespace karakterekre illeszkedik
    + \S - bármilyen nem whitespace karakterekre illeszkedik
    + \t, \n, \r - tabulátor, sortörés, _carriage return_ karakterekre illeszkedik
- amire használjuk:
    + String.replace() - kicseréljük a szöveget a szövegben
    + String.match() - tömbben adja vissza az illeszkedő értékeket pl.: '06301223444'.match(/(\d)\1+/gi) // ['22', '444']
    + String.split() - String.match-hez hasonló, azonban nem közvetlenül szelektálhatunk ki elemeket a bementből, hanem az elválasztást definiálhatjuk a reguláris kifejezéssel pl.: '+36 (30) 481-9873'.split(/[ ()-]+/gi) // ["+36", "30", "481", "9873"]