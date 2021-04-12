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
MERGE (country:Country { name: line[4] })
CREATE (airline) -[:INCORPORATED_INTO]-> (country)
```




# Part 2
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
6093: ups
317,
el resultado es correcto
