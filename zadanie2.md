Zadanie 2 NoSQL
===============

Do tego zadania skorzystałem z pliku [GetGlue](http://getglue-data.s3.amazonaws.com/getglue_sample.tar.gz) dostępnego na stronie z zadaniami.

Po pobraniu i rozpakowanie pliku sprawdziłem, w jakim stanie jest owy plik, by wiedzieć co dalej robić. Pomogła mi z tym komenda
```
head -n3 getglue_sample.json
```

Jak widać, plik ma dość przyjazną strukturę:
```
{"comment": "", "hideVisits": "false", "modelName": "tv_shows", "displayName": "", "title": "Criminal Minds", "timestamp": "2008-08-01T06:58:14Z", "image": "http://cdn-1.nflximg.com/us/boxshots/large/70056671.jpg", "userId": "areilly", "private": "false", "source": "http://www.netflix.com/Movie/Criminal_Minds_Season_1/70056671", "version": "2", "link": "http://www.netflix.com/Movie/Criminal_Minds_Season_1/70056671", "lastModified": "2011-12-16T19:41:19Z", "action": "Liked", "lctitle": "criminal minds", "objectKey": "tv_shows/criminal_minds", "visitCount": "1"}
{"comment": "", "hideVisits": "false", "modelName": "movies", "displayName": "", "title": "Vicky Cristina Barcelona", "timestamp": "2008-08-13T23:04:10Z", "image": "http://cdn-5.nflximg.com/us/boxshots/large/70097585.jpg", "userId": "shgoda", "private": "false", "director": "woody allen", "source": "http://www.netflix.com/Movie/Vicky_Cristina_Barcelona/70097585", "version": "2", "link": "http://www.netflix.com/Movie/Vicky_Cristina_Barcelona/70097585", "lastModified": "2011-12-16T19:41:19Z", "action": "Liked", "lctitle": "vicky cristina barcelona", "objectKey": "movies/vicky_cristina_barcelona/woody_allen", "visitCount": "1"}
{"comment": "", "modelName": "movies", "displayName": "", "title": "Stardust", "timestamp": "2008-09-03T05:00:03Z", "image": "http://ia.media-imdb.com/images/M/MV5BMjE3ODcxMDUxMV5BMl5BanBnXkFtZTcwNjQyMTU1MQ@@._V1._SX100_SY129_.jpg", "userId": "brianhaddad", "private": "false", "director": "matthew vaughn", "source": "", "version": "1", "link": "http://www.imdb.com/title/tt0486655/", "lastModified": "2011-12-16T19:39:33Z", "action": "Liked", "lctitle": "stardust", "objectKey": "movies/stardust/matthew_vaughn"}
```

Zatem można przystąpić do importu
```
time mongoimport -d mongoTest -c getglue < getglue_sample.json
```

Czas:
```
real	34m38.754s
user	20m3.843s
sys	0m48.572s

```
| Type        | Value           |
| ------------- |:-------------:|
| real      | 34m38.754s |
| user      | 20m3.843s      |
| sys | 0m48.572s      |

Sprawdzenie poprawności:
```
db.getglue.findOne()
{
	"_id" : ObjectId("546c6bf47c5a0b652721b407"),
	"comment" : "",
	"hideVisits" : "false",
	"modelName" : "movies",
	"displayName" : "",
	"title" : "Vicky Cristina Barcelona",
	"timestamp" : "2008-08-13T23:04:10Z",
	"image" : "http://cdn-5.nflximg.com/us/boxshots/large/70097585.jpg",
	"userId" : "shgoda",
	"private" : "false",
	"director" : "woody allen",
	"source" : "http://www.netflix.com/Movie/Vicky_Cristina_Barcelona/70097585",
	"version" : "2",
	"link" : "http://www.netflix.com/Movie/Vicky_Cristina_Barcelona/70097585",
	"lastModified" : "2011-12-16T19:41:19Z",
	"action" : "Liked",
	"lctitle" : "vicky cristina barcelona",
	"objectKey" : "movies/vicky_cristina_barcelona/woody_allen",
	"visitCount" : "1"
}
```

