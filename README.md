## Curated list of useful SCM functions for everyday coding. 
Uses latest [CLEO 5.1](https://cleo.li) / [Sanny Builder 4](https://sannybuilder.com/)

Read more about functions in Sanny Builder: https://docs.sannybuilder.com/language/functions

# Table of Contents

## 1. Uncategorized Functions

- [AllocateString / DeallocateString](#allocatestring) - Creates and manages heap-allocated strings
- [GetCameraAngle](#getcameraangle) - Return camera rotation angle
- [GetEntityPos](#getentitypos) - Returns XYZ coords of an entity (CEntity)
- [GetEntityType](#getentitytype) - Returns entity type: 0-nothing, 1-building, 2-vehicle, 3-ped, 4-object, 5-dummy
- [IsOnMission](#isonmission) - Checks if on mission flag is set
- [IsThisEntityAVehicle](#isthisentityavehicle) - Checks if this entity is a vehicle
- [LoadModel](#loadmodel) - Loads model by id
- [ReplaceStringInFile](#replacestringinfile) - Replaces the first occurrence of a substring in a file
- [SetCarPlateText](#setcarplatetext) - Changes the text on car's number plate
- [SetOnMission](#setonmission) - Sets on mission flag
- [SpawnCar](#spawncar) - Spawns a new car like a cheat and returns its handle

## 2. Math Functions
- [Min](#min) - Returns the smallest of two integers
- [MinF](#minf) - Returns the smallest of two floats
- [Max](#max) - Returns the largest of two integers
- [MaxF](#maxf) - Returns the largest of two floats
- [ToRad](#torad) - Converts degrees to radians
- [ToDeg](#todeg) - Converts radians to degrees

## 3. Debug Functions

- [DumpScriptVars](#dumpscriptvars) - Writes local variables (0@-31@) to CLEO.log
- [Log](#log) - Adds a new entry in CLEO.log
- [ReloadThisScript](#reloadthisscript) - Reload current script from disk
- [TeleportToNearestCar](#teleporttonearestcar) - Teleports player to the nearest car
- [ViewPlayerCoords](#viewplayercoords) - Prints player coordinates
- [ViewScriptVars](#viewscriptvars) - Prints local variables on screen

### Uncategorized


#### GetEntityType
```lua
/// Returns entity type: 0-nothing, 1-building, 2-vehicle, 3-ped, 4-object, 5-dummy
function GetEntityType(address: int): int
    int type = read_memory_with_offset {address} address {offset} 0x36 {size} 1
    type &= 7
    return type
end
```

#### IsThisEntityAVehicle
```lua
/// Checks if this entity is a vehicle
function IsThisEntityAVehicle(address: int): logical
    int type = GetEntityType(address)
    return type == 2
end
```

#### GetCameraAngle

```lua
/// Return camera rotation angle
function GetCameraAngle(): float
    float x1, y1, z1, x2, y2, z2
    x1, y1, z1 = get_active_camera_coordinates
    x2, y2, z2 = get_active_camera_point_at
    y2 -= y1
    x2 -= x1
    float z = get_heading_from_vector_2d {x} x2 {y} y2
    return z
end
```

#### AllocateString

```lua
/// Creates a heap-allocated string
function AllocateString(s: string): int
    int len = get_text_length {text} s
    len++
    int buf = allocate_memory {size} len
    string_format buf "%s" s
    return buf
end

function DeallocateString(s: string)
    free_memory {address} s
end
```

#### LoadModel
```lua
/// Loads model by id
function LoadModel(modelId: int)
    request_model modelId
    while not has_model_loaded {modelId} modelId
        wait 0
    end
end
```
* GetWeatherForecast(hours: int): int - returns weather type coming in {hours}

#### SetCarPlateText
```lua
/// Changes the text on car's number place
function SetCarPlateText(vehicle: Car, plateText: string)
    int pVehicle = Memory.GetVehiclePointer(vehicle)
    int pCustomCarPlate = read_memory_with_offset {address} pVehicle {offset} 0x588 {size} 4
    if is_truthy pCustomCarPlate
    then
        RwTextureDestroy(pCustomCarPlate)
    end
    int plateTexture = CCustomCarPlateMgr_CreatePlateTexture(plateText, -1)
    write_memory_with_offset {address} pVehicle {offset} 0x588 {size} 4 {value} plateTexture

    function CCustomCarPlateMgr_CreatePlateTexture<cdecl, 0x6FDEA0>(text: string, plateType: int): int
    function RwTextureDestroy<cdecl, 0x7F3820>(texture: int)
end
```

```lua
/// Replaces the first occurence of {find_string} found in the file at {file_path} with {replace_string} in-place
export function ReplaceStringInFile(file_path: string, find: string, replace: string)
    int f

    // open file for reading
    if f = open_file file_path "rb"
    then

        int size = get_file_size f
        int source = allocate_memory size

        if read_block_from_file f size source
        then

            int p = strstr(source, find_string)
            if p > 0
            then
                // reopen file for writing
                close_file f
                f = open_file file_path "wb"

                int pFrom, count

                // copy all before found string
                pFrom = source
                count = p - pFrom
                trace "copy first %d bytes of file %s" count file_path
                write_block_to_file f count pFrom

                // copy new name
                pFrom = replace_string
                count = strlen(replace_string)
                trace "copy %d bytes of %s" count pFrom
                write_block_to_file f count pFrom

                // copy all after found string
                int last = source + size
                pFrom = strlen(find_string)
                pFrom += p
                count = last - pFrom
                trace "copy last %d bytes of file %s" count file_path
                write_block_to_file f count pFrom

            else
                trace "%s not found in %s" find_string file_path
            end
        else
            trace "can't read file %s" file_path
        end

        close_file f
        free_memory source

    else
        trace "file %s not found" file_path
    end
    /// Finds the first occurrence of a substring in a string and returns a pointer to it
    function strstr<cdecl, 0x822650>(str: string, substr: string): string
    /// Returns str length
    function strlen<cdecl, 0x718690>(str: string): int
end
```

#### SpawnCar
```lua
/// Spawns a new car like a cheat and returns its handle
function SpawnCar(modelId: int): int

    int pCheatCar = CCheat_VehicleCheat(modelId)
    // mark car as owned by player
    int flags = read_memory_with_offset {address} pCheatCar {offset} 0x428 {size} 4
    set_bit flags 17 // bHasBeenOwnedByPlayer=true
    write_memory_with_offset {address} pCheatCar {offset} 0x428 {size} 4 {value} flags
    Car hCheatCar = get_vehicle_ref {address} pCheatCar
    return hCheatCar

    /// Spawns a vehicle of this model in front of the player
    function CCheat_VehicleCheat<cdecl, 0x43A0B0>(vehicleModelId: int): int
end
```
* IsPointInsideGarage(x: float, y: float, z: float): logical - return true if point is located inside a garage

#### GetEntityPos
```lua
// Returns XYZ coords of an entity (CEntity)
function GetEntityPos(address: int): float, float, float
    int vec = read_memory_with_offset {address} address {offset} 0x14 {size} 4
    if vec > 0
    then
        vec = vec + 0x30 // matrix->pos
    else
        vec = address + 4 // entity->simpleCoors->pos
    end
    
    float x, y, z 
    x = read_memory_with_offset {address} vec {offset} 0x0 {size} 4
    y = read_memory_with_offset {address} vec {offset} 0x4 {size} 4
    z = read_memory_with_offset {address} vec {offset} 0x8 {size} 4
    return x, y, z
end
```
* ClearBlipOnCharDeath(char: int) - makes the blip to disappear when {char} dies

#### IsOnMission
```lua
/// Checks if on mission flag is set
function IsOnMission(): logical
    int address = -2221
    int offset = &0(address,1i) / 4
    int flag = &0(offset,1i)
    return flag <> 0
end
```

#### SetOnMission
```lua
/// Sets on mission flag
function SetOnMission(flag: int)
    int address = -2221
    int offset = &0(address,1i) / 4
    &0(offset,1i) = flag
end
```
* GetUserSettingsInt(settingId: int): int - returns an integer value of a particular configuration in the main menu. list of settings TBD
* SetUserSettingsInt(settingId: int, value: int) - sets new value of a particular configuration in the main menu. list of settings TBD
* GetUserSettingsFloat(settingId: int): float - returns an integer value of a particular configuration in the main menu. list of settings TBD
* SetUserSettingsFloat(settingId: int, value: float) - sets new value of a particular configuration in the main menu. list of settings TBD
* GetDummyCoords(vehicle: Car, dummyId: int): float, float, float - returns XYZ of a particular dummy of the car

### Math

#### Min
```lua
/// Returns the smallest of {a} and {b}
function Min(a: int, b: int): int
    if a > b 
    then
        return b
    else
        return a
    end
end
```
#### MinF
```lua
/// Returns the smallest of {a} and {b}
function MinF(a: float, b: float): float
    if a > b 
    then
        return b
    else
        return a
    end
end
```
#### Max
```lua
/// Returns the largest of {a} and {b}
function Max(a: int, b: int): int
    if a < b 
    then
        return b
    else
        return a
    end
end
```
#### MaxF
```lua
/// Returns the largest of {a} and {b}
function MaxF(a: float, b: float): float
    if a < b 
    then
        return b
    else
        return a
    end
end
```


#### ToRad
```lua
/// Converts degrees to radians
function ToRad(degrees: float): float
    float res = degrees * 0.0175 // PI / 180
    return res
end
```

#### ToDeg
```lua
/// Converts radians to degrees
function ToDeg(radians: float): float
    float res = radians * 57.2958 // 180 / PI
    return res
end
```


### Debug
#### Log
```lua
/// Adds a new entry in CLEO.log. Requires `LegacyDebugOpcodes = 1` in cleo\cleo_plugins\SA.DebugUtils.ini 
function Log(s: string)
    debug_on
    write_debug s
    debug_off
end
```
#### DumpScriptVars
```lua
/// writes a list of local variables (0@-31@) to CLEO.log
:DumpScriptVars
    int buf = allocate_memory 512
    string_format {buffer} buf {format} "%d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d" {args} 0@ 1@ 2@ 3@ 4@ 5@ 6@ 7@ 8@ 9@ 10@ 11@ 12@ 13@ 14@ 15@ 16@ 17@ 18@ 19@ 20@ 21@ 22@ 23@ 24@ 25@ 26@ 27@ 28@ 29@ 30@ 31@
    Log(buf)
    free_memory {address} buf
return
```
#### ViewScriptVars
```
/// Prints local variables on screen
:ViewScriptVars
    use_text_commands {state} true
    display_text_formatted {offsetLeft} 50.0 {offsetTop} 100.0 {format} "%d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d %d" {args} 0@ 1@ 2@ 3@ 4@ 5@ 6@ 7@ 8@ 9@ 10@ 11@ 12@ 13@ 14@ 15@ 16@ 17@ 18@ 19@ 20@ 21@ 22@ 23@ 24@ 25@ 26@ 27@ 28@ 29@ 30@ 31@
return
```
* SaveScreenToPng(f: string, left: int, top: int, w: int, h: int) - saves portion of screen to a png file

#### ViewPlayerCoords
```lua
/// Prints player coordinates
function ViewPlayerCoords()
    float x, y, z

    use_text_commands {state} true
    x, y, z = get_char_coordinates $scplayer
    set_text_wrapx {width} 640.0
    set_text_centre {state} true
    set_text_centre_size {width} 640.0
    display_text_formatted {offsetLeft} 320.0 {offsetTop} 20.0 {format} "%.2f %.2f %.2f" {args} x y z
end
```
* ViewEntityCoords3d(entity: int) - prints entity (CVehicle, CPed, CObject) coordinates above it

#### ReloadThisScript
```lua
/// Reload current script from disk
function ReloadThisScript()
    int buf = allocate_memory 256
    buf = get_script_filename -1 true
    stream_custom_script buf
    free_memory buf
    terminate_this_script
end
```

#### TeleportToNearestCar
```lua
/// Teleports player to the nearest car
function TeleportToNearestCar()
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

#### TeleportToMarker
```lua
/// Teleports player to the red target marker
function TeleportToMarker()
    float x, y, z
    if x, y, z = get_target_blip_coords
    then
        set_char_coordinates $scplayer {x} x {y} y {z} z
    end
end
```