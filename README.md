## Curated list of useful SCM functions for everyday coding. Uses latest CLEO 5.1


### Uncategorized

* loadModel(modelId: int) - loads model by id
* getForecast(hours: int): int - returns weather type coming in {hours}
* setCarPlateText(car: int, text: string) - changes the text on car's number place
* replaceStringInFile(f: string, find: string, replace: string) - replaces string {find} in {f} with {replace}
* spawnCar(modelId: int): int - spawns a new car like a cheat and returns its handle
* isPointInsideGarage(x: float, y: float, z: float): logical - return true if point is located inside a garage
* getEntityPos(entity: int): float, float, float - returns XYZ coords of an entity (CEntity)
* clearBlipOnCharDeath(char: int) - makes the blip to disappear when {char} dies
* isOnMission() - checks if on mission flag is set
* setOnMission(state: int) - sets on mission flag
* getUserSettingsInt(settingId: int): int - returns an integer value of a particular configuration in the main menu. list of settings TBD
* setUserSettingsInt(settingId: int, value: int) - sets new value of a particular configuration in the main menu. list of settings TBD
* getUserSettingsFloat(settingId: int): float - returns an integer value of a particular configuration in the main menu. list of settings TBD
* setUserSettingsFloat(settingId: int, value: float) - sets new value of a particular configuration in the main menu. list of settings TBD

### Debug
* log(s: string) - adds a new entry in CLEO.log
* dbg() - pauses script execution until F5 is pressed
* dumpScriptVars() - writes a list of local variables (0@-31@) to CLEO.log
* viewScriptVars() - prints local variables on screen
* saveScreenToPng(f: string, left: int, top: int, w: int, h: int) - saves portion of screen to a png file
