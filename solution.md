Nota: para eliminarlo todo: `MATCH (n) DETACH DELETE n`

# 1) Para poder trabajar con los datos en Neo4j, lo primero que debemos hacer es importar los datos.
El orden es importante que de sigua el siguiente: routes.dat, airport.dat y airline.dat

## routes.dat
```neo4j
LOAD CSV FROM 'https://raw.githubusercontent.com/marivalle/openflights/master/data/routes.dat' AS line WITH line
MERGE (airline:Airline { airlineId: line[1] })
MERGE (sourceAirport:Airport { airportId: line[3] })
MERGE (destinationAirport:Airport { airportId: line[5] })
CREATE (sourceAirport) -[route:ROUTE_TO { airlineId:airline.airlineId}] -> (destinationAirport)
```

## airports.dat
```neo4j
LOAD CSV FROM 'https://raw.githubusercontent.com/marivalle/openflights/master/data/airports.dat' AS line WITH line
MERGE (airport:Airport { airportId: line[0] })
	ON CREATE SET
		airport.name = line[1],
		airport.latitude = line[6],
		airport.longitude = line[7],
		airport.altitude = line[8]
	ON MATCH SET
		airport.name = line[1],
		airport.latitude = line[6],
		airport.longitude = line[7],
		airport.altitude = line[8]
MERGE (city:City { name: line[2] })
MERGE (country:Country { name: line[3] })
MERGE (city) -[:IN]-> (country)
CREATE (airport) -[:MAIN_CITY]-> (city)
```

## airlines.dat
```neo4j
LOAD CSV FROM 'https://raw.githubusercontent.com/marivalle/openflights/master/data/airlines.dat' AS line WITH line
MERGE (airline:Airline { airlineId: line[0] })
	ON CREATE SET
		airline.name = line[1],
		airline.alias = line[2],
		airline.callsign = line[5]
	ON MATCH SET
		airline.name = line[1],
		airline.alias = line[2],
		airline.callsign = line[5]
MERGE (country:Country { name: line[6] })
CREATE (airline) -[:INCORPORATED_INTO]-> (country)
```

# 2) Antes de continuar verifica que se hayan importado bien los datos revisando las cantidades
## count `ROUTE_TO`
```neo4j
MATCH ()-[r:ROUTE_TO]->()
RETURN count(r) as count
```
66765,
el resultado es correcto

## Count `MAIN_CITY`
```neo4j
MATCH ()-[r:MAIN_CITY]->()
RETURN count(r) as count
```
7697,
el resultado es correcto

## count `IN`
```neo4j
MATCH ()-[r:IN]->()
RETURN count(r) as count
```
7106,
el resultado es correcto

## count `INCORPORATED_INTO`
```neo4j
MATCH ()-[r:INCORPORATED_INTO]->()
RETURN count(r) as count
```
6162,
el resultado es correcto

## count `Airport`
```neo4j
MATCH (airport:Airport)
RETURN count(airport)
```
7803,
el resultado es correcto

## count `Airline`
```neo4j
MATCH (airline:Airline)
RETURN count(airline)
```
6162,
el resultado es correcto

## count `City`
```neo4j
MATCH (c:City)
RETURN count(c)
```
6956,
el resultado es correcto

## count `Country`
```neo4j
MATCH (c:Country)
RETURN count(c)
```
317,
el resultado es correcto

# 3) Muestra los nombres de los aeropuertos que hay en London
```neo4j
MATCH (airport:Airport) -[:MAIN_CITY]-> (:City { name: "London" })
RETURN airport.name
```
- London City Airport
- London Luton Airport
- London Gatwick Airport
- London Airport
- Madison County Airport
- London-Corbin Airport/Magee Field
- London Heathrow Airport
- London Heliport
- London Stansted Airport

# 5) Si estamos en Barcelona, ¿con qué aerolíneas podemos ir a Londres de forma directa?
> Nota: Se tradujo Londres a London para tener resultados positivos
```neo4j
MATCH (:City { name: "Barcelona" } ) <-[:MAIN_CITY]- (:Airport) -[r:ROUTE_TO]-> (:Airport) -[:MAIN_CITY]-> (:City { name: "London" } )
RETURN r.airlineId
```
Con las siguientes aerolíneas podemos ir directos de Barcelona a Londres *London*:
- 2439
- 1355
- 2822
- 1355
- 3737
- 2297
- 24
- 2822
- 2297
- 220
- 4296

# 6) Desde Santa Clara, ¿a cuantas ciudades puedo ir haciendo 1 escala (= dos rutas)?
```neo4j
MATCH (:City {name: "Santa Clara"} ) <-[:MAIN_CITY]- (:Airport) -[:ROUTE_TO*2]-> (:Airport) -[:MAIN_CITY]-> (c:City)
RETURN count(c)
```
Total de 1827 ciudades distintas haciendo 1 escala.

# 8) Muestra el país que tenga más cantidad de aeropuertos
```neo4j
MATCH (airport:Airport) -[:MAIN_CITY]-> (:City) -[:IN]-> (country:Country)
RETURN country.name, count(airport) AS count
ORDER BY count DESC
LIMIT 1
```
El país con más aeropuertos es United States.

# 9) Muestra las 5 ciudades origen con más cantidad de rutas, incluyendo la cantidad en el resultado
```neo4j
MATCH (:Airport) <-[r:ROUTE_TO]- (:Airport) -[:MAIN_CITY]-> (city:City)
RETURN city.name, count(r) AS count
ORDER BY count DESC
LIMIT 5
```
Las 5 ciudades con más cantidad de rutas, incluyendo el resultado son:
|#|city     |count  |
|-|---------|-------|
|1|London   |1230   |
|2|Atlanta  |915    |
|3|Paris    |724    |
|4|Chicago  |695    |
|5|Shanghai |616    |

# 10) Lista todos los aeropuertos que contengan la palabra International en su nombre
```neo4j
MATCH (airport:Airport)
WHERE airport.name =~ '.*International.*'
RETURN airport.name
```
Una breve muestra de los 897 que contienen la palabra International:
- Sochi International Airport
- Domodedovo International Airport
- Belgorod International Airport
- Heydar Aliyev International Airport

# 11) Muestra la ruta más corta para ir de Barcelona a Jersey
```neo4j
MATCH path = shortestPath ( (:City { name: "Barcelona" } ) -[*]- (:City { name: "Jersey" } ))
RETURN path
```
La ruta más corta es: Barcelona -> 1218 -> 488 -> 499 -> Jersey

# 12) ¿Cuantas rutas necesito hacer como mínimo para ir de Barcelona a Jersey, si queremos pasar primero por Montreal?
Si consideramos ruta como viaje, con solo una ruta tendrás suficiente.

# 13) De los distintos aeropuertos de Londres, ¿Cual es el que está más cerca del centro de Londres (Latitud: 51.5072, Longitud: -0.1275)?
```neo4j
MATCH (airport:Airport) -[:MAIN_CITY]-> (:City { name: "London" })
WITH airport, point({longitude: toFloat(airport.longitude), latitude: toFloat(airport.latitude)}) AS airportPoint, point({longitude: -0.1275, latitude: toFloat("51.5072") }) AS londonCenter
RETURN airport.name, distance(airportPoint, londonCenter) AS distance
ORDER BY distance ASC
LIMIT 1
```
El aeropuerto de Londres que más cerca esta del centro de la misma es London Heliport.
