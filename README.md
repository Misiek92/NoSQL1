NoSQL Zadanie 1
===============

####Zadanie 1a

SELECT * W zadaniu został wykorzystany plik [Train.zip](https://www.kaggle.com/c/facebook-recruiting-iii-keyword-extraction/download/Train.zip)
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

####Zadanie 1b

Ilość wyników postgres zliczył nam już przy dodawaniu, tę samą wartość zwraca również zapytanie
```
SELECT count(*) FROM train;
```

Mongo również zwraca tę samą wartość:
```
db.train.count()
6034195
```

####Zadanie 1c

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

Próbę zliczenia unikalnych tagów w postgresie porzuciłem, bowiem jedyny pomysł jaki miałem, wiem z doświadczenia że by trwał bardzo długo. Pseudo kod, który planowałem stworzyć
```
i = 1 integer,
j = 0 integer,
table text[],
temp text[],
tempLength integer,
tempOne text,

count = select count(id) from train;

LOOP
SELECT tags from train where id = i INTO temp;
tempLength = select array_length(temp, 1);

LOOP
IF temp[j] != any table THEN
array_append(table, temp[j])
END IF;


IF j = tempLength THEN
 BREAK;
END IF;
END;


IF i >= count THEN
 BREAK;
END;
END;

return array_length(table, 1);
```

Sytuacja w mongo wygląda jednak zgoła inaczej :)

Dla potrzeb tego zadania napisałem prostą funkcję, która wpierw zamienia stringa na arraya i za jednym ciosem zlicza ilość powtórzeń i wykrywa unikatowe wystąpienia tagów.

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

Wynik może być tylko jeden:
```
Liczba wszystkich tagów: 17409994
W tym 42048 unikatowych
```

####Zadanie 1d

Do jego wykonania wykorzystam geojson'a z listą miast dostępną [pod tym adresem](https://raw.githubusercontent.com/mahemoff/geodata/master/cities.geojson). Lista ta została sporządzona na podstawie [Wikipedii](http://en.wikipedia.org/wiki/List_of_cities_by_longitude)



