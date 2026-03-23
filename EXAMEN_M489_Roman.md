# EXAMEN · MÒDUL 489

## Programació de Dispositius Mòbils i Multimèdia

**Unitats Formatives:** RA2 i RA3  
**Curs:** 2n DAM · Videojocs  
**Data:** 23/03/2026
**Durada:** 2 hores  

**Alumne/a:** Roman Zyuzin Alenxandrov
**Grup:** 2DAM

---

## Posada en marxa de l'entorn

Consulta el fitxer **`README.md`** del projecte per a les instruccions completes d'instal·lació i arrencada (Node.js, servidor mock i Flutter).

---

> **Instruccions generals**
>
> - Respon cada pregunta en l'espai indicat (substitueix el text `[Escriu la teva resposta aquí]`).
> - Per a la part de codi, escriu directament en bloc `dart`. No és necessari que el codi compili, però ha de reflectir coneixement real de Flutter/Dart.
> - Tens el codi dels projectes **Cars** i **Phone** com a referència en el teu ordinador. **No pots accedir a internet** durant l'examen.
> - Desa el fitxer i lliura'l amb el nom: `EXAMEN_M489_[el_teu_nom].md`
> - Fes commit i push del .md modificat i de tots els arxius que hagis modificat

---

## BLOC 1 · ARQUITECTURA I CICLE DE VIDA *(RA 2)*

### Pregunta 1.1 – Comunicació entre Widgets *(12 punts)*

Al projecte **Cars**, el widget `CarsPage` gestiona el número de pàgina actual (`_currentPage`) i el passa a `CarsList1`. El widget `ButtonPanel` conté els botons "Anterior" i "Següent".

**a)** A `cars_page.dart`, el widget utilitza el mètode `setState` per gestionar la paginació. Explica:

- Quina és la funció de `setState` i per què cridar-lo fa que la UI es torni a dibuixar.
- Per quin motiu `_loadPage()` fa servir dos crides a `setState` separades (una a l'inici i una al final) en lloc d'una sola.

**Resposta:**

```
[La funció setState es crida quan el objecte state ha canviat, aixó li diu al programa que aquest widget s'ha de tornar a mostrar.
Perque la primera serveix per notificar que s'ha de mostrar visualment que esta cargant i la segona notifica que ja ha cargat.]
```

---

### Pregunta 1.2 – Cicle de vida d'un widget amb recursos *(13 punts)*

Al projecte **Camera**, el widget `CameraScreen` utilitza un `CameraController` per gestionar la càmera del dispositiu. Aquest controlador ocupa recursos del sistema (càmera, memòria) i cal alliberar-los correctament.

**a)** Quin mètode del cicle de vida de `State` s'usa a `CameraScreen` per alliberar el `CameraController` quan el widget és destruït? Escriu com es fa i explica per quina raó és imprescindible cridar-lo.

**Resposta:**

```
[El metode dispose, ho fa de forma CameraController.dispose() on cameracontroller pot ser qualsevol controlador. Ho fa per no gastar recursos de més i per si un altre metodo demana el permís per camara que no surti que ja esta en us]
```

---

**b)** El `CameraController` s'inicialitza de forma asíncrona a `initState()` i el resultat es guarda a `_initializeControllerFuture`. Respon les preguntes següents:

- Per quin motiu no es pot fer `await` directament a `initState()`?
- Quina millora aporta a l'usuari usar `FutureBuilder` en lloc de bloquejar el fil?
- Com treballen junts `_initializeControllerFuture` i `FutureBuilder`?

**Resposta:**

```
[Perque si fas el await al initState la aplicacio es quedara penjada esperant el await y no mostrara res per pantalla.
Perque amb el FutureBuilder podem modificar altres coses aixi com mostrar un text de que esta carregant o fins i tot animacions de carrega, si bloquejen el fil fins que no termini la pagina es queda congelada
Quan la pantalla es mostra el initializecontrollerfuture pot guardar l'estat actual del futurebuilder i saber que mostrar exactament depenent de l'estat.]
```

---

## BLOC 2 · COMUNICACIÓ, PERSISTÈNCIA I PROVES *(RA 2 — 35 punts)*

### Pregunta 2.1 – Consum d'API i robustesa *(18 punts)*

Analitza el mètode `getCarsPage(int page, int limit)` de `car_http_service.dart`.

Què passaria si el servidor de l'API trigués 60 segons a respondre? L'aplicació quedaria bloquejada per a l'usuari? Per què? Escriu com implementaries un *timeout* de 10 segons a la petició HTTP.

**Resposta:** En aquest cas el metode ja integra un timeout de 10 seconds, si l'api triga 60 segons, al segon 11 ja haura retornat un error en aquest cas de no resposta, 404.

```dart
// Escriu la modificació al getCarsPage aquí:
  Future<List<CarsModel>> getCarsPage(int page, int limit) async {
    final offset = (page - 1) * limit;
    final uri = _buildUri('/v1/cars', {'limit': '$limit', 'offset': '$offset'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }
```

---

### Pregunta 2.2 – Models de dades  *(17 punts)*

Analitza el constructor `factory CarsModel.fromMapToCarObject(Map<String, dynamic> json)` de `car_model.dart`.

**a)** Imagina que l'API retorna per error el camp `year` com a `String` en lloc d'`int` (per exemple, `"2021"` en lloc de `2021`). El codi actual fallaria. Escriu com resoldries el problema.

**Resposta:**

```
[Si nosaltres volem que el camp year sigue sempre un int, una vegada rebem el json i fem el mapa podem guardar as int, en aquest cas ja guarda year com int, el que podem fer es parcejar el json a string sempre i després a int de forma que entri String o int sempre surti int, pseudocodi a sota]
```
  factory CarsModel.fromMapToCarObject(Map<String, dynamic> json) {
    return CarsModel(
      id: json['id'] as int,
      year: int.tryParse(json['year'].toString()) as int,
      make: json['make'] as String? ?? '',
      model: json['model'] as String? ?? '',
      type: json['type'] as String? ?? '',
      city: json['city'] as String? ?? '',
      color: json['color'] as String? ?? '',
    );
  }
  
---

**b)** Al fitxer `class_model_test.dart`, el test utilitza un `const jsonString` amb un JSON escrit a mà en lloc de fer una petició real a l'API de RapidAPI. Explica per quin motiu és millor simular el JSON en un test unitari.

**Resposta:**

```
[Primer de tot s'ha d'entendre perque fem el test unitari, el test unitari serveix per probar el funcionament d'un widget en concret, el millor es aillar i limitar el errors que em poden donar, per comprobar l'integritat de l'api ja tinc el postman, a més a més es sobrecomplicarse perque només comprobem l'estructura d'un json que sempre te la mateixa estructura, si tenim molts test i una api es paga per request ens estalviem recursos.]
```

---

## BLOC 3 · IMPLEMENTACIÓ PRÀCTICA *(RA 3 — 30 punts)*

### Exercici – Widget de detall amb dades remotes

Imagina que volem crear una pantalla de detall per a cada cotxe del projecte Cars. Implementa el mètode `build` d'un widget `StatelessWidget` anomenat `CarDetailPage` que compleixi els requisits següents:

1. Rep un paràmetre `final CarsModel car` al constructor.
2. Mostra el **make** i el **model** del cotxe com a títol destacat (`Text` amb estil gran i negreta).
3. Mostra una **icona diferent** depenent del `type` del cotxe:
   - Si el `type` és `'SUV'`, mostra `Icons.directions_car`.
   - Per qualsevol altre tipus, mostra `Icons.car_rental`.
4. Afegeix un botó `ElevatedButton` que, quan es premi, mostri un `SnackBar` amb el text: `"Cotxe seleccionat: [make] [model]"`.

```dart
// Escriu el teu codi aquí:

Nota alumno: El siguiente widget esta hecho a mano por petición del profesor, con el indicativo de mostrar y demostrar los conocimientos lógicos en la materia, hay errores de sintaxis, la idea es que sea suficiente para ver la estructura general priorizando la creatividad y lógica por encima de memorizar y escupir.

class CarDetailPage extends StatelessWidget {
  final CarsModel car;

  const CarDetailPage({super.key, required this.car});

  @override
  Widget build(BuildContext context) {
    Scaffold{
      Appbar: appbar(title: "Lista Coches"),
      body: center{
        child: column{
          children: [
            text((car.make, car.model) style: bold),
            icon(car.type == "SUV" ? Icons.directions_car : Icons.car_rental),
            ElevatedButton(
              onPressed: -> show.snackbar(text("Cotxe seleccionat: " [$car.make] [$car.model] ));
              child: text("botó")
            )
          ]
        }
      }
    }
  }
}
```

---

**Ampliació (nivell Expert):** Afegeix un `FutureBuilder` que cridi al mètode `CarHttpService().getCarsPage(1, 5)` i mentre espera les dades mostri un `CircularProgressIndicator`. Quan les dades estiguin llestes, mostra un `ListView.builder` amb el `make` de cada cotxe. Si hi hagués un error, mostra un `Text` en color vermell amb el missatge de l'error.

```dart
//Escriu la teva ampliació aquí:
```

Alumno: Vale en este caso ya tenemos un Future en cars_page que se encarga de avisarme si la pagina esta cargando. Es la siguiente:

  Future<void> _loadPage() async {
    setState(() {
      _isLoading = true;
      _error = null;
    });
    try {
      final cars = await _service.getCarsPage(_currentPage, _itemsPerPage);
      setState(() {
        _cars = cars;
        _isLoading = false;
      });
    } catch (e) {
      setState(() {
        _error = e.toString();
        _isLoading = false;
      });
    }
  }

Entonces seria del estilo cambiarla para que currentpage y itemsperpage tenga los numeros del enunciado y luego hacer un
if(_isloading) {
  CircularProgressIndicator
} else if(_error == null){
  print(ListView.builder(car.make))
} else {
  Text(_error)
}

---

## BLOC 4 · EXTENSIÓ DEL SERVEI HTTP *(RA 2 — 10 punts)*

### Exercici 4.1 – Mètode parametritzat a `CarHttpService` *(10 punts)*

El servidor mock local té disponible un  endpoint de cerca:

```
GET http://localhost:8080/v1/cars/search?make=Toyota&model=Corolla
```

- El paràmetre `make` filtra per marca (coincidència parcial, insensible a majúscules).
- El paràmetre `model` filtra per model (coincidència parcial, insensible a majúscules).
- Tots dos paràmetres són opcionals: si no s'envien, retorna tots els cotxes.

Exemples vàlids:

- `/v1/cars/search?make=Toyota` → tots els Toyota
- `/v1/cars/search?model=X5` → tots els cotxes amb "X5" al model
- `/v1/cars/search?make=BMW&model=X` → BMW amb "X" al model

**Implementa** el mètode `getCarsByFilter` a la classe `CarHttpService` existent, seguint el mateix patrons que `getCarsPage`:

```dart
// Afegeix aquest mètode a car_http_service.dart:
```

Requisits:

1. Utilitza el mètode privat `_buildUri(String path, Map<String, String> queryParams)` ja existent.
2. Només afegeix els paràmetres `make` i/o `model` al mapa si el valor no és `null` ni buit (`isEmpty`).
3. Gestiona errors i timeout amb el mateix mecanisme que `getCarsPage`.

**Resposta:**

```dart
// Escriu aquí la teva implementació completa del mètode:
  Future<List<CarsModel>> getCarsByFilter(String make, int model) async {
    final uri = _buildUri('/v1/cars', {'make':'$make', '$model':'x'});

    final response = await http
        .get(uri, headers: _headers)
        .timeout(const Duration(seconds: 10));

    if (response.statusCode == 200) {
      return CarsModel.listFromJsonString(response.body);
    } else {
      throw Exception('Error ${response.statusCode}: ${response.body}');
    }
  }
```

---

## Resum de l'examen

| Bloc | RA | Punts màxims |
|------|----|:------------:|
| Bloc 1 – Arquitectura i Cicle de vida | RA 2 | 25 |
| Bloc 2 – Comunicació, Persistència i Proves | RA 2  | 35 |
| Bloc 3 – `CarDetailPage` (base) | RA 3 | 20 |
| Bloc 3 – Ampliació `FutureBuilder`  | RA 3 | 10 |
| Bloc 4 – Extensió del servei HTTP | RA 2 | 10 |
| **TOTAL** | | **100** |

---
