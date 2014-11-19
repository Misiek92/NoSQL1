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

Trochę statystyk:
```
db.getglue.stats()
{
	"ns" : "mongoTest.getglue",
	"count" : 19831300,
	"size" : 18280980928,
	"avgObjSize" : 921,
	"numExtents" : 30,
	"storageSize" : 20038070176,
	"nindexes" : 1,
	"lastExtentSize" : 2146426864,
	"paddingFactor" : 1,
	"paddingFactorNote" : "paddingFactor is unused and unmaintained in 2.8. It remains hard coded to 1.0 for compatibility only.",
	"userFlags" : 1,
	"totalIndexSize" : 643606544,
	"indexSizes" : {
		"_id_" : 643606544
	},
	"ok" : 1
}
```

### Agregacje

Jako [MongoDB Driver](http://docs.mongodb.org/ecosystem/drivers/) Użyłem PyMongo

Na początku postanowiłem sprawdzić jakie "modelName" występują w ogóle i w jakiej ilości.

##### Javascript
```
> db.getglue.aggregate(
    { $group: {
        _id: "$modelName",
        total: { $sum: 1 }
    } },
    { $sort: { total: -1 } }
  )
{ "_id" : "tv_shows", "total" : 12258355 }
{ "_id" : "movies", "total" : 7572855 }
{ "_id" : null, "total" : 56 }
{ "_id" : "topics", "total" : 23 }
{ "_id" : "recording_artists", "total" : 11 }
```

##### PyMongo
```
>>> import pymongo
>>> client = pymongo.MongoClient("localhost", 27017)
>>> db = client.mongoTest
>>> db.name
u'mongoTest'
db.getglue.aggregate([
    { "$group": {
        "_id": "$modelName",
        "total": { "$sum": 1 }
    } },
    { "$sort": { "total": -1 } }
])
{u'ok': 1.0, u'result': [{u'total': 12258355, u'_id': u'tv_shows'}, {u'total': 7572855, u'_id': u'movies'}, {u'total': 56, u'_id': None}, {u'total': 23, u'_id': u'topics'}, {u'total': 11, u'_id': u'recording_artists'}]}
```
| modelName        | total           |
| ------------- |:-------------:|
| tv_shows      | 12258355 |
| movies      | 7572855     |
| null      | 56      |
| topics      | 23     |
| recording_artists      | 11     |

Zaintrygowany tymi "recording_artists" postanowiłem to sprawdzić...

##### Javascript
```
db.getglue.aggregate(
   { $match: {
	"modelName": "recording_artists"
   } },
   { $group: {
       _id: "$title",
       total: { $sum: 1 }
   } }
)
```


Wynik jednak nieco mnie rozczarował, bo jest niewiele mówiący
```
{ "_id" : "net@night", "total" : 4 }
{ "_id" : "Engadget", "total" : 2 }
{ "_id" : "Twit", "total" : 5 }
```

##### PyMongo
```
db.getglue.aggregate([
   { "$match": {
	"modelName": "recording_artists"
   } },
   { "$group": {
       "_id": "$title",
       "total": { "$sum": 1 }
   } }
])
{u'ok': 1.0, u'result': [{u'total': 4, u'_id': u'net@night'}, {u'total': 2, u'_id': u'Engadget'}, {u'total': 5, u'_id': u'Twit'}]}
```


| _id        | total           |
| ------------- |:-------------:|
| net@night      | 4 |
| Engadget      | 2     |
| Twit      | 5      |


Wracając jednak do tematu, podstawową rzeczą jaka może przyjść do głowy mając bazę filmową jest... Sprawdzenie np. który reżyser stworzył najwięcej filmów (top 3):

##### Javascript
```
db.getglue.aggregate(
   { $match: {
	"modelName": "movies"
   } },
   { $group: {
       _id: "$director",
       total: { $sum: 1 }
   } },
   { $sort: { 
	total: -1 
   } },
   { $limit: 3 }
)

{ "_id" : "steven spielberg", "total" : 108553 }
{ "_id" : "tim burton", "total" : 101732 }
{ "_id" : "bill condon", "total" : 97818 }
```

##### PyMongo
```
db.getglue.aggregate([
   { "$match": {
	"modelName": "movies"
   } },
   { "$group": {
       "_id": "$director",
       "total": { "$sum": 1 }
   } },
   { "$sort": { 
	"total": -1 
   } },
   { "$limit": 3 }
])
{u'ok': 1.0, u'result': [{u'total': 108553, u'_id': u'steven spielberg'}, {u'total': 101732, u'_id': u'tim burton'}, {u'total': 97818, u'_id': u'bill condon'}]}
```

| _id        | total           |
| ------------- |:-------------:|
| steven spielberg      | 108553 |
| tim burton      | 101732     |
| bill condon      | 97818      |

Wynik jak jednak widać jest nieco... niewiarygodny. Postanowiłem zatem powtórzyć zapytanie z wrażeniem, że dane filmy się mogą powtarzać.

##### Javascript
```
db.getglue.aggregate(
   { $match: {
	"modelName": "movies"
   } },
   { $group: {
       _id: {"director": "$director", "id": "$title"}
   } },
   { $group: {
       _id: "$_id.director",
       total: { $sum: 1 }
   } },
   { $sort: { 
	total: -1 
   } },
   { $limit: 3 }
)
{ "_id" : "not available", "total" : 1474 }
{ "_id" : "various directors", "total" : 54 }
{ "_id" : "alfred hitchcock", "total" : 50 }
```

##### PyMongo
```
db.getglue.aggregate([
   { "$match": {
	"modelName": "movies"
   } },
   { "$group": {
       "_id": {"director": "$director", "id": "$title"}
   } },
   { "$group": {
       "_id": "$_id.director",
       "total": { "$sum": 1 }
   } },
   { "$sort": { 
	"total": -1 
   } },
   { "$limit": 3 }
])
```

| _id        | total           |
| ------------- |:-------------:|
| not available      | 1474 |
| various directors      | 54     |
| alfred hitchcock      | 50      |

Jak widać, te dane są dużo sensowniejsze, mimo iż pierwsze 2 pozycje są de facto śmieciowe, to jednak wiemy, że w tej bazie najwięcej filmów nakręcił Alfred Hitchcock.


Interesowało mnie również to, jakiego filmu najczęściej szukali internauci. Jako miernik uznałem liczbę odwiedzin.

##### Javascript
```
db.getglue.aggregate(
   { $match: {
	"modelName": "movies"
   } },
   { $group: {
	_id: {"title": "$title", "visitCount": "$visitCount"}
   } },
   { $sort: { 
	visitCount: -1 
   } },
   { $limit: 3 }
)
{ "_id" : { "title" : "Sherlock Holmes and the Baker Street Irregulars" } }
{ "_id" : { "title" : "Stanley and Iris" } }
{ "_id" : { "title" : "Extreme Ice: Nova" } }
```

##### PyMongo
```
db.getglue.aggregate([
   { "$match": {
	"modelName": "movies"
   } },
   { "$group": {
	"_id": {"title": "$title", "visitCount": "$visitCount"}
   } },
   { "$sort": { 
	"visitCount": -1 
   } },
   { "$limit": 3 }
])
```
