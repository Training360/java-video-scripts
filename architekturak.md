A `C:\architekturak` könyvtárba javasolt dolgozni.

Szükséges szoftverek:

* Docker Desktop
* DBeaver
* Postman

## Standalone konzolos alkalmazás

* Ábra
* Tudnivalók a https://github.com/Training360/architecture-public GitHub repository-ban
* Alkalmazás letöltés a `https://github.com/Training360/architecture-public/releases/download/1.0.0/LocationsCli-1.0.0.exe`
címről (telepítő csomag)
* Telepítés (telepítés kiemelése)
* Telepítésre kerül a `C:\Program Files\LocationsCli` könyvtárba.
* Parancssor (prompt)
* Alkalmazás futtatása (`LocationsCli`)
* Listázás, hozzáadás
* Megőrzi az adatokat kilépés után is
* Adattárolás lokálisan a `%HOMEPATH%\locations` könyvtárban
  * Bináris állomány
* Scriptelhető
  * Hozzunk létre egy `input.txt` állományt a következő tartalommal


```
2
Budapest
47.497912,19.040235
5
```

* Alkalmazás tartalmának listázása, futtatható és erőforrás állomány
* Uninstalláljuk


## Standalone alkalmazás grafikus felülettel

* Ábra
* Letöltés a https://github.com/Training360/architecture-public/releases/download/1.0.0/locations-win32-64x.zip címről.
* Ki kell tömöríteni
* Grafikus felületről indítható
* Intuitív felhasználói felület, listázás, hozzáadás
* Mentés, betöltés

## Központi adatbázis

* Ábra
* Docker telepítése
* Hozzuk létre az adatbázist a következő paranccsal

```shell
docker run -d -e MYSQL_DATABASE=locations -e MYSQL_USER=locations -e MYSQL_PASSWORD=locations -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 3306:3306 --name locations-dbclient-mariadb mariadb
```

* A `config.js` állomány átírása, hogy adatbázishoz kapcsolódjon. Ugyanaz az alkalmazás, különböző beállítással.
* Alkalmazás használata, listázás, hozzáadás
* DBeaver telepítése
* A `location` tábla bemutatása
* Sor hozzáadása, módosítása, ellenőrzés a felületen
* Felületen felvétel, ellenőrzés az adatbázisban
* Adatbázis leállítása

## SQL

* SQL konzol

```shell
docker exec -it locations-dbclient-mariadb mysql locations
```

* SQL utasítások

```sql
desc location;

select * from location;

insert into location(name, lat, lon) values ('Work2', 3, 3);

update location set name = 'Work3' where id = 3;

delete location where id = 3;

select * from location left join tag on location.id = tag.locationId;
```

* Ugyanez a DBeaverből is
* Tranzakciókezelés

```sql
update location set name = 'aa' where id = 19;

update tag set name = 'sa' where id = 6;
```

* Megmutatni először Rollbackkel, majd Committal (DBeaver Switch to manual commit)

## NoSQL adatbázisok

* Ábra
* MongoDB elindítása

```shell
docker run -d -p27017:27017 --name locations-mongo mongo
```

* Alkalmazás konfigurálása, elindítása
* Listázás, hozzáadás
* Csatlakozás parancssorból, dokumentum alapú adatbázis, JSON

```shell
docker exec -it locations-mongo mongo locations
```

```javascript
db.location.find()

db.location.insert({name: "Work", lat: 2, lon: 2})

db.location.update({_id: "5f8e8e8237d3c021af6da9b6"}, {$set: {name: "Work2"}})
```

* Mongo leállítása

## Többrétegű alkalmazások

* Ábra

```shell
docker network create locations-net

docker run -d -e MYSQL_DATABASE=locations -e MYSQL_USER=locations -e MYSQL_PASSWORD=locations -e MYSQL_ALLOW_EMPTY_PASSWORD=yes -p 3306:3306 --network locations-net --name locations-mariadb mariadb

docker run -d -e SPRING_DATASOURCE_URL=jdbc:mariadb://locations-mariadb/locations -p 8080:8080 --network locations-net --name=my-locations training360/locations
```

* Konfiguráció, hogy REST-en menjen a vastagkliens
* Listázás, hozzáadás
* Napló nézése

```shell
docker logs -f my-locations
```

## Webes alkalmazás

* Ábra
* Ellenőrizzük, hogy fut-e az adatbázis és az alkalmazás (`docker ps`)
* `http://localhost:8080/server/` (URL)
* Böngészőben F12
  * Változik az URL
  * Kérés, válasz, státusz, header, hivatkozott erőforrások, url paraméter (details), űrlap (új location felvétele)
* Parancssorból:

```shell
docker run --rm --network locations-net --rm curlimages/curl -L -v http://my-locations:8080/server
```

## Web formátumai: HTML és CSS

* `index.html`


```html
<!DOCTYPE html>
<html>
    <head>
        <title>Example</title>
    </head>
    <body>
        <h1>Example Page</h1>
     <p>This is an example page. See <a href="http://training360.com">Training360</a>!</p>        
    </body>
</html>
```

* Markup, hypertext
* Megnyitás böngészőben, renderelés, forrás nézet
* Struktúra és tartalom
* DOM fa
* DOM fa szerkesztése
* `styles.css`

```html
 <link rel="stylesheet" href="styles.css" />
 ```

 ```css
 h1 {
  color: red;
}
 ```

* F12
* Szerkeszthető

## Webes alkalmazás RIA felülettel

* Ellenőrizzük, hogy fut-e a webszerver: `docker ps`

```html
<script src="myscript.js" type="text/javascript"></script>
```

```javascript
onload = () => document.querySelector('#welcome-button').addEventListener('click', e => {alert('Hello World!')});
}
```

### 

* Ábra
* HTTP kérések
* `http://localhost:8080/`
* Listázás, létrehozás - ugyanazon az oldalon marad, csak egy része változik, nem változik az URL

## REST webszolgáltatások

* Ábra (GUI-t kikerülve)
* Swagger `http://localhost:8080/swagger-ui.html`
* Listázás, új felvétele, felületen megmutatni, hogy változott
* JSON felépítése
* Postman telepítése
* Generálás OpenAPI alapján
  * http://localhost:8080/v3/api-docs
* Listázás, új felvétele

### 

```shell
docker run -d --rm -p 8081:8080 rodolpheche/wiremock
```

* posttal fel kell küldeni egy stub mappinget (http://wiremock.org/docs/running-standalone/)

## Szerver alkalmazás webszolgáltatás interfésszel

