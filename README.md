NoSQL Zadanie 1
===============

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

```
CREATE TABLE train(id integer PRIMARY KEY NOT NULL,title varchar(256) NOT NULL,body text NOT NULL,tags varchar(256) NOT NULL);
```

```
COPY train(id,title,body,tags) FROM '/home/mateusz/Pulpit/nosql/Train.csv' WITH DELIMITER ',' CSV HEADER;
```

```
COPY 6034195
Time: 1484708,929 ms
```


ALTER TABLE train ALTER COLUMN tags TYPE TEXT[] using string_to_array(tags,' ');

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




Wszystkich tagów: 17409994
Różnych tagów: 42048



time mongoimport -d mongoTest -c train --type csv --file trainProcessed.csv --headerline
