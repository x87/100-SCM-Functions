## Curated list of useful SCM functions for everyday coding. Uses latest CLEO 5.1


### Uncategorized

* loadModel(modelId: int) - loads model by id
* getForecast(hours: int): int - returns future weather type in {hours}
* log(s: string) - adds a new entry in CLEO.log
* setCarPlateText(car: int, text: string) - changes the text on car's number place
* saveScreenToPng(f: string, left: int, top: int, w: int, h: int) - saves portion of screen to a png file
* replaceStringInFile(f: string, find: string, replace: string) - replaces string {find} in {f} with {replace}
* spawnCar(modelId: int): int - spawns a new car like a cheat and returns its handle
* isPointInsideGarage(x: float, y: float, z: float): logical - return true if point is located inside a garage
* getEntityPos(entity: int): float, float, float - returns XYZ coords of an entity (CEntity)
