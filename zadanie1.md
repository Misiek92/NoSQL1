#NoSQL Zadanie 1

## Spis treści
- [NoSQL Zadanie 1](#nosql-zadanie-1)
    - [Spis treści](#spis-treści)
    - [Zadania](#zadania)
        - [Zadanie 1a](#zadanie-1a)
        - [Zadanie 1b](#zadanie-1b)
        - [Zadanie 1c](#zadanie-1c)
        - [Zadanie 1d](#zadanie-1d)
        - [Przykładowe geospatial queries](#przykładowe-geospatial-queries)


## Zadania
### Zadanie 1a

W zadaniu został wykorzystany plik [Train.zip](https://www.kaggle.com/c/facebook-recruiting-iii-keyword-extraction/download/Train.zip)
Jego struktura to "Id", "Title", "Body", "Tags", nam jednak potrzebne jest przerobienie tego pliku do formatu, gdzie "Id" stanie się "_Id".
Dodatkowo w pliku znajdują się znaki nowej linii, które trzeba usunąć. Oba te problemy rozwiązuje skrypt [2unix.sh](https://github.com/nosql/aggregations-2/blob/master/scripts/wbzyl/2unix.sh) dostępny w repozytorium prowadzącego.

Do jego uruchomienia wykorzystałem następującą komendę
```
time ./2unix.sh Train.csv trainProcessed.csv
```

Czas:
```
real    14m49.281s
user    2m58.216s
sys     1m53.482s
```
| Type        | value           |
| ------------- |:-------------:|
| real      | 14m49.281s |
| user      | 2m58.216s     |
| sys | 1m53.482s      |

By uruchomić mongo wraz z storageEngine wiredtiger należy odpalić deamona z dodatkowym parametrem jak poniżej
```
mongod --storageEngine wiredtiger
```

Niestety próba zaimportowania pliku csv do mongo w trybie z wiredtiger wysypuje mi serwer po ok 300000 rekordów. Po kilku próbach, wyłączania programów by zwolnić więcej ramu etc. byłem jednak zmuszony do "standardowej" wersji serwera mongo.

By zaimportować rzeczony plik csv, użyłem komendy:
```
time mongoimport -d mongoTest -c train --type csv --file trainProcessed.csv --headerline
```
Czas:
```
imported 6034195 documents

real	19m28.634s
user	5m45.810s
sys	0m45.921s

```
| Type        | value           |
| ------------- |:-------------:|
| real      | 19m28.634s |
| user      | 5m45.810s     |
| sys | 0m45.921s      |

Statystyki:
```
db.train.stats()
{
	"ns" : "mongoTest.train",
	"count" : 6034195,
	"size" : 10678489296,
	"avgObjSize" : 1769,
	"numExtents" : 26,
	"storageSize" : 10832121824,
	"nindexes" : 1,
	"lastExtentSize" : 2146426864,
	"paddingFactor" : 1,
	"paddingFactorNote" : "paddingFactor is unused and unmaintained in 2.8. It remains hard coded to 1.0 for compatibility only.",
	"userFlags" : 1,
	"totalIndexSize" : 168973392,
	"indexSizes" : {
		"_id_" : 168973392
	},
	"ok" : 1
}
```

Jako, iż miałem problem z system-monitor (powszechny błąd natychmiastowego wyłączania się) skorzystałem z alternatywnej aplikacji "htop"

Zazwyczaj mongo korzystało z jednego rdzenia (niekoniecznie pierwszego)
![status monitor 1](https://cloud.githubusercontent.com/assets/1538320/5104747/77774fb8-6fe5-11e4-9d32-05594c984bf6.png "status monitor 1")

Zdarzały się jednak momenty próby rozłożenia obciążenia równomiernie:
![status monitor 3](https://cloud.githubusercontent.com/assets/1538320/5104800/e620136e-6fe5-11e4-9d28-425c66be80d6.png "status monitor 3")

Jak i dziwne przypadki, gdy rdzenie nie były praktycznie w ogóle wykorzystane
![status monitor 4](https://cloud.githubusercontent.com/assets/1538320/5104801/e7d617b2-6fe5-11e4-91db-2161191877b6.png "status monitor 4")




Przyszedł czas wykonać te same operacje, lecz dla bazy postgres. Do tego celu, trzeba wcześniej stworzyć nową tabelę:
```
CREATE TABLE train(id integer PRIMARY KEY NOT NULL,title varchar(256) NOT NULL,body text NOT NULL,tags varchar(256) NOT NULL);
```

A następnie skorzystać z poniższej komendy (tutaj importujemy "surowy" plik, nie jego przerobioną wersję).
```
COPY train(id,title,body,tags) FROM '/home/mateusz/Pulpit/nosql/Train.csv' WITH DELIMITER ',' CSV HEADER;
```

Czas wraz z ilością dodanych wierszy
```
COPY 6034195
Time: 1484708,929 ms
```
| Type        | value           |
| ------------- |:-------------:|
| Time      | 1484708,929 ms |
| W minutach      | 24,74 min    |

Screen z htop. Postgres raczej cały czas "jechał" na jednym rdzeniu.

![status monitor 2](https://cloud.githubusercontent.com/assets/1538320/5104799/e33cdb96-6fe5-11e4-9352-eebf1c3b33f5.png "status monitor 2")


Zestawienie MongoDB i Postgresql

| Type        | value           |
| ------------- |:-------------:|
| MongoDB      | 19.28 minut |
| W minutach      | 24,74 minuty    |

![porownanie mongo z postgre](https://cloud.githubusercontent.com/assets/1538320/5104902/11e35afa-6fe7-11e4-9895-784726b2c700.png "porownanie mongo z postgre")

###Zadanie 1b

Ilość wyników postgres zliczył nam już przy dodawaniu, tę samą wartość zwraca również zapytanie
```
SELECT count(*) FROM train;
```

Mongo również zwraca tę samą wartość:
```
db.train.count()
6034195
```

###Zadanie 1c

By zamienić kolumnę "tags" na tablicę w postgresie, możemy posłużyć się następującym zapytaniem:
```
ALTER TABLE train ALTER COLUMN tags TYPE TEXT[] using string_to_array(tags,' ');
```

Czas:
```
ALTER TABLE
Time: 405936,541 ms
```

Do sprawdzenia poprawności operacji wykorzystałem komendy:
```
SELECT tags FROM train WHERE id = 1;
tags                         
------------------------------------------------------
 {php,image-processing,file-upload,upload,mime-types}
(1 row)
```

```
SELECT tags[5] FROM train WHERE id = 1;
    tags    
------------
 mime-types
(1 row)
```

Do zliczenia ilości tagów skorzystałem z następującej komendy:
```
SELECT SUM(array_length(tags, 1)) FROM train;
   sum    
----------
 17409994
```

Ilość unikalnych tagów możemy zliczyć przez komendę
```
SELECT unnest(tags) FROM train GROUP BY unnest(tags);
(42061 rows)
```

W przypadku mongo, dla potrzeb tego zadania napisałem prostą funkcję, która wpierw zamienia stringa na arraya i za jednym ciosem zlicza ilość powtórzeń i wykrywa unikatowe wystąpienia tagów.

```
function operationsOnTags() {
	var database = db.train.find();

	var tags = [];
	var records = [];

	var countArray = 0;
	var countUniqueArray = 0;

	database.forEach(function(input) {
		var tagsArray = [];
		if(input.tags.constructor === String) {
			tagsArray = input.tags.split(" ");
		} else { 
			tagsArray.push(input.tags);
		}
		
		input.tags = tagsArray;
		countArray += tagsArray.length;

		for(var i = 0; i < tagsArray.length; i++) {
			if (tags.indexOf(tagsArray[i]) === -1) {
				countUniqueArray++;  
				tags.push(tagsArray[i]);
			}
		}
	});

	print("Liczba wszystkich tagów:" + countArray);
	print("W tym:" + countUniqueArray + " unikatowych");
}
```

Funkcję oczywiście wywołujemy poprzez
```
operationsOnTags();
```

Wynik:
```
Liczba wszystkich tagów:17409994
W tym:42061 unikatowych
```
| Type        | value           |
| ------------- |:-------------:|
| Łącznie      | 17409994 |
| Unikatowych      | 42061    |

Stosunek unikatowych tagów do duplikatów:
![tagi](https://cloud.githubusercontent.com/assets/1538320/5104903/135054c4-6fe7-11e4-9aba-1ad856a1967d.png "tagi")

###Zadanie 1d

Do tego zadania wykorzystałem [tę listę miast](https://github.com/mahemoff/geodata/blob/master/cities.geojson) stworzoną na podstawie [Wikipedii](http://en.wikipedia.org/wiki/List_of_cities_by_longitude). Udostępniam ją również na swoim repo [cities.json](https://github.com/Misiek92/NoSQL1/blob/master/cities.geojson).

Obiekt typue GeoJSON poddałem następującym operacjom:
usunąłem
```
{
	"type": "FeatureCollection",
	"features": [
```
```
]
}
```
Usunąłem przecinki między obiektami oraz zamieniłem "id" na "_id"


Import
```
mongoimport -d mongoTest -c cities < cities.json
```


Następnie stworzyłem skrypt który modyfikuje geojson do bardziej nam potrzebnej formy.
```
function fixToJson() {
	var database = db.cities.find();

	database.forEach(function (input) {
		if (!input.loc) {
			var newRow = {
				"_id": input._id,
				"loc": {
				"type": "Point",
				"coordinates": input.geometry.coordinates
				}
			}
			db.cities.remove({"_id": input._id});
			db.cities.insert(newRow);
		}
	});
	print("Zaaktualizowano");
}
```

Funkcję oczywiście wywołuję.

###Przykładowe geospatial queries

Do obrobienia json do geojson skorzystałem z [mongodb-geojson-normalize](https://www.npmjs.org/package/mongodb-geojson-normalize) ponieważ nie udało mi się dojść do tego, by w jq w obiekcie zagnieździć inny obiekt (błąd składni, w manualu nie znalazłem takiego case'a).

Gdybyśmy zaginęli u wybrzeży grenlandii, jakie miasta są najbliżej? [Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial1.geojson)
```
var punkt = {
	"type":"Point",
	"coordinates": [-43.742453, 59.870764]
}

db.cities.find({ loc: {$near: {$geometry: punkt}} }).limit(5)
{ "_id" : "Nuuk", "loc" : { "type" : "Point", "coordinates" : [ -51.75, 64.167 ] } }
{ "_id" : "Reykjav%C3%ADk", "loc" : { "type" : "Point", "coordinates" : [ -21.933, 64.133 ] } }
{ "_id" : "Stanley", "loc" : { "type" : "Point", "coordinates" : [ -57.85, 51.683 ] } }
{ "_id" : "Iqaluit", "loc" : { "type" : "Point", "coordinates" : [ -68.517, 63.75 ] } }
{ "_id" : "St. John%27s", "loc" : { "type" : "Point", "coordinates" : [ -52.7, 47.55 ] } }
```

Za pomocą [http://geojson.io/#map=2/20.0/0.0](http://geojson.io/#map=2/20.0/0.0) Stworzyłem pseuo kształt województwa pomorskiego i zamierzam sprawdzić, czy w mojej bazie znajdują się miejscowości w tym województwie. [Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial2.geojson)

```
db.cities.find({loc: {$geoWithin: {$geometry: {type: "Polygon", coordinates: [[[
              16.710205078125,
              54.56569261911192
            ],
            [
              16.85302734375,
              54.25238930276849
            ],
            [
              16.732177734375,
              54.204223304732196
            ],
            [
              16.94091796875,
              53.875202055794965
            ],
            [
              16.858520507812496,
              53.74871079689897
            ],
            [
              17.0233154296875,
              53.49784954396767
            ],
            [
              17.391357421875,
              53.49784954396767
            ],
            [
              17.490234375,
              53.621837552541365
            ],
            [
              17.6715087890625,
              53.56967636543387
            ],
            [
              18.072509765625,
              53.771442468556074
            ],
            [
              18.599853515625,
              53.64789400083556
            ],
            [
              18.753662109375,
              53.69020141273198
            ],
            [
              18.7701416015625,
              53.6120622350988
            ],
            [
              19.127197265625,
              53.589244357588655
            ],
            [
              19.335937499999996,
              53.81686890643213
            ],
            [
              19.4952392578125,
              53.784426471294296
            ],
            [
              19.588623046875,
              53.9560855309879
            ],
            [
              19.3524169921875,
              53.94315470224928
            ],
            [
              19.3853759765625,
              54.00776876193478
            ],
            [
              19.2205810546875,
              54.078728538670575
            ],
            [
              19.27001953125,
              54.29408771927571
            ],
            [
              19.66552734375,
              54.4029457476126
            ],
            [
              19.6270751953125,
              54.45407332522336
            ],
            [
              19.105224609375,
              54.35815677227375
            ],
            [
              18.819580078125,
              54.70558168515836
            ],
            [
              18.2318115234375,
              54.895564790773385
            ],
            [
              17.567138671875,
              54.88924640307589
            ],
            [
              16.710205078125,
              54.56569261911192
            ]]]}}}})

            { "_id" : "Gda%C5%84sk", "loc" : { "type" : "Point", "coordinates" : [ 18.667, 54.35 ] } }
```
            
Zapytanie powtórzyłem też, ale dla całej Polski [Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial3.geojson):
```
            db.cities.find({loc: {$geoWithin: {$geometry: {type: "Polygon", coordinates: [[[
              18.424072265625,
              54.92082843149136
            ],
            [
              17.05078125,
              54.80068486732233
            ],
            [
              14.21630859375,
              53.92375094101389
            ],
            [
              14.447021484374998,
              53.20603255157844
            ],
            [
              14.073486328125,
              52.81604319154934
            ],
            [
              14.589843749999998,
              52.556316065406556
            ],
            [
              14.974365234375,
              50.875311142200765
            ],
            [
              16.380615234375,
              50.61113171332364
            ],
            [
              16.259765625,
              50.33844888725473
            ],
            [
              16.677246093749996,
              50.099440987634985
            ],
            [
              17.138671875,
              50.38050249104245
            ],
            [
              17.86376953125,
              50.00067775723633
            ],
            [
              18.544921875,
              49.915861746597294
            ],
            [
              18.863525390625,
              49.532339195028115
            ],
            [
              19.072265625,
              49.38952445158216
            ],
            [
              19.489746093749996,
              49.56797785892715
            ],
            [
              19.962158203125,
              49.1888842152458
            ],
            [
              21.55517578125,
              49.42526716083716
            ],
            [
              22.8955078125,
              49.001843917978526
            ],
            [
              22.664794921874996,
              49.56085220619185
            ],
            [
              24.06005859375,
              50.604159488561
            ],
            [
              23.5986328125,
              52.07950600379697
            ],
            [
              23.170166015625,
              52.281601868071434
            ],
            [
              23.97216796875,
              52.76289173758374
            ],
            [
              23.543701171875,
              54.09806018306312
            ],
            [
              22.840576171875,
              54.43171285946844
            ],
            [
              19.75341796875,
              54.470037612805754
            ],
            [
              18.424072265625,
              54.92082843149136
            ]]]}}}})

            { "_id" : "Gda%C5%84sk", "loc" : { "type" : "Point", "coordinates" : [ 18.667, 54.35 ] } }
{ "_id" : "Krak%C3%B3w", "loc" : { "type" : "Point", "coordinates" : [ 19.933, 50.05 ] } }
{ "_id" : "Warsaw", "loc" : { "type" : "Point", "coordinates" : [ 21, 52.233 ] } }
```

[Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial4.geojson)
```
db.cities.find({ loc: {$near: {$geometry: punkt}} }).limit(5)
{ "_id" : "Valletta", "loc" : { "type" : "Point", "coordinates" : [ 14.5, 35.9 ] } }
{ "_id" : "Birkirkara", "loc" : { "type" : "Point", "coordinates" : [ 14.45, 35.883 ] } }
{ "_id" : "Naples", "loc" : { "type" : "Point", "coordinates" : [ 14.25, 40.833 ] } }
{ "_id" : "Tirana", "loc" : { "type" : "Point", "coordinates" : [ 19.817, 41.317 ] } }
{ "_id" : "Rome", "loc" : { "type" : "Point", "coordinates" : [ 12.5, 41.9 ] } }
> 
```

[Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial5.geojson)
```
db.cities.find({loc: {$geoIntersects: {$geometry: {type: "LineString", coordinates: [[2.6, -90],[2.6, 90]]}}}})
{ "_id" : "Porto-Novo", "loc" : { "type" : "Point", "coordinates" : [ 2.6, 6.5 ] } }
```


[Mapka](https://github.com/Misiek92/NoSQL1/blob/master/geospatial6.geojson)
```
db.cities.find({loc: { $geoWithin : {
...   $center : [[0, 0], 10],
... } } })
{ "_id" : "Yamoussoukro", "loc" : { "type" : "Point", "coordinates" : [ -5.283, 6.817 ] } }
{ "_id" : "Abidjan", "loc" : { "type" : "Point", "coordinates" : [ -4.017, 5.333 ] } }
{ "_id" : "Tamale", "loc" : { "type" : "Point", "coordinates" : [ -0.85, 9.4 ] } }
{ "_id" : "Accra", "loc" : { "type" : "Point", "coordinates" : [ -0.2, 5.55 ] } }
{ "_id" : "Lom%C3%A9", "loc" : { "type" : "Point", "coordinates" : [ 1.2, 6.133 ] } }
{ "_id" : "Cotonou", "loc" : { "type" : "Point", "coordinates" : [ 2.417, 6.35 ] } }
{ "_id" : "Porto-Novo", "loc" : { "type" : "Point", "coordinates" : [ 2.6, 6.5 ] } }
{ "_id" : "Lagos", "loc" : { "type" : "Point", "coordinates" : [ 3.383, 6.45 ] } }
{ "_id" : "Ibadan", "loc" : { "type" : "Point", "coordinates" : [ 3.883, 7.367 ] } }
{ "_id" : "S%C3%A3o Tom%C3%A9", "loc" : { "type" : "Point", "coordinates" : [ 6.683, 0.333 ] } }
{ "_id" : "Enugu", "loc" : { "type" : "Point", "coordinates" : [ 7.5, 6.45 ] } }
{ "_id" : "Malabo", "loc" : { "type" : "Point", "coordinates" : [ 8.767, 3.75 ] } }
{ "_id" : "Libreville", "loc" : { "type" : "Point", "coordinates" : [ 9.45, 0.383 ] } }
```







