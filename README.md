## Curated list of useful SCM functions for everyday coding. Uses latest CLEO 5.1


### Uncategorized

* loadModel(modelId: int) - loads model by id
* getWeatherForecast(hours: int): int - returns weather type coming in {hours}
* setCarPlateText(car: int, text: string) - changes the text on car's number place
* replaceStringInFile(f: string, find: string, replace: string) - replaces string {find} in {f} with {replace}
* spawnCar(modelId: int): Car - spawns a new car like a cheat and returns its handle
* isPointInsideGarage(x: float, y: float, z: float): logical - return true if point is located inside a garage
* getEntityPos(entity: int): float, float, float - returns XYZ coords of an entity (CEntity)
* clearBlipOnCharDeath(char: int) - makes the blip to disappear when {char} dies
* isOnMission() - checks if on mission flag is set
* setOnMission(state: int) - sets on mission flag
* getUserSettingsInt(settingId: int): int - returns an integer value of a particular configuration in the main menu. list of settings TBD
* setUserSettingsInt(settingId: int, value: int) - sets new value of a particular configuration in the main menu. list of settings TBD
* getUserSettingsFloat(settingId: int): float - returns an integer value of a particular configuration in the main menu. list of settings TBD
* setUserSettingsFloat(settingId: int, value: float) - sets new value of a particular configuration in the main menu. list of settings TBD
* getDummyCoords(vehicle: Car, dummyId: int): float, float, float - returns XYZ of a particular dummy of the car

### Math

#### min
```lua
/// Returns the smallest of {a} and {b}
function min(a: int, b: int): int
    if a > b 
    then
        return b
    else
        return a
    end
end
```
#### minf
```lua
/// Returns the smallest of {a} and {b}
function minf(a: float, b: float): float
    if a > b 
    then
        return b
    else
        return a
    end
end
```
#### max
```lua
/// Returns the largest of {a} and {b}
function max(a: int, b: int): int
    if a < b 
    then
        return b
    else
        return a
    end
end
```
#### maxf
```lua
/// Returns the largest of {a} and {b}
function maxf(a: float, b: float): int
    if a < b 
    then
        return b
    else
        return a
    end
end
```

### Debug
#### log
```lua
/// Adds a new entry in CLEO.log
function log(s: string)
    debug_on
    write_debug s
    debug_off
end
```
#### dumpScriptVars
```lua
/// writes a list of local variables (0@-31@) to CLEO.log
:dumpScriptVars
    int buf = allocate_memory 512
    string_format {buffer} buf {format} "%d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d" {args} 0@ 1@ 2@ 3@ 4@ 5@ 6@ 7@ 8@ 9@ 10@ 11@ 12@ 13@ 14@ 15@ 16@ 17@ 18@ 19@ 20@ 21@ 22@ 23@ 24@ 25@ 26@ 27@ 28@ 29@ 30@ 31@
    log(buf)
    free_memory {address} buf
return
```
#### viewScriptVars
```
/// Prints local variables on screen
:viewScriptVars
    use_text_commands {state} true
    display_text_formatted {offsetLeft} 50.0 {offsetTop} 100.0 {format} "%d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d" {args} 0@ 1@ 2@ 3@ 4@ 5@ 6@ 7@ 8@ 9@ 10@ 11@ 12@ 13@ 14@ 15@ 16@ 17@ 18@ 19@ 20@ 21@ 22@ 23@ 24@ 25@ 26@ 27@ 28@ 29@ 30@ 31@
return
```
* saveScreenToPng(f: string, left: int, top: int, w: int, h: int) - saves portion of screen to a png file

#### viewPlayerCoords
```lua
/// Prints player coordinates
function viewPlayerCoords()
    float x, y, z
    
    use_text_commands {state} true
    x, y, z = get_char_coordinates $scplayer
    display_text_formatted {offsetLeft} 50.0 {offsetTop} 100.0 {format} "%.2f %.2f %.2f" {args} x y z
end
```
* viewEntityCoords3d(entity: int) - prints entity (CVehicle, CPed, CObject) coordinates above it
* reloadThisScript() - reload current script from disk

#### teleportToNearestCar
```lua
/// Teleports player to the nearest car
function teleportToNearestCar()
    float x, y, z
    float radius = 5.0
    x, y, z = get_char_coordinates $scplayer

    int handle = get_random_car_in_sphere_no_save_recursive {pos} x y z {radius} radius {findNext} false {skipWrecked} true
    while handle == -1
        radius += 5.0
        if radius > 100.0
        then
            return // no cars nearby
        end
        handle = get_random_car_in_sphere_no_save_recursive {pos} x y z {radius} radius {findNext} true {skipWrecked} true // get next
    end
    
    warp_char_into_car $scplayer {vehicle} handle
end
```
