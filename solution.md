
diapositivas 87

```neo4j
LOAD CSV FROM 'https://raw.githubusercontent.com/marivalle/openflights/master/data/routes.dat' AS line WITH line LIMIT 50
MERGE (airline:Airline { airlineId: line[1] })
MERGE (sourceAirport:Airport { airportId: line[3] })
MERGE (destinationAirport:Airport { airportId: line[5] })
CREATE (sourceAirport) -[route:ROUTE_TO { airlineId:airline.airlineId}] -> (destinationAirport)
RETURN airline,sourceAirport,destinationAirport
```

Part 2
-  7697 relaciones de tipo `MAIN_CITY`
- 66765 relaciones de tipo `ROUTE_TO`
-  7106 relaciones de tipo `IN`
-  6162 relaciones de tipo `INCORPORATED_INTO`
```neo4j
MATCH (airline:Airline)
RETURN count(airline)
```
