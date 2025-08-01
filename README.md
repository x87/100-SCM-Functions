# Curated List of Useful SCM Functions for Everyday Coding.
Uses latest [CLEO 5.1](https://cleo.li) / [Sanny Builder 4](https://sannybuilder.com/)

Read more about functions in Sanny Builder: https://docs.sannybuilder.com/language/functions

## Uncategorized Functions

- [AllocateString / DeallocateString](#allocatestring) - Creates and manages heap-allocated strings
- [GetCameraAngle](#getcameraangle) - Return camera rotation angle
- [SetFOV](#setfov) - Sets the field of view (FOV) for the camera
- [GetEntityPos](#getentitypos) - Returns XYZ coords of an entity (CEntity)
- [GetEntityType](#getentitytype) - Returns entity type: 0-nothing, 1-building, 2-vehicle, 3-ped, 4-object, 5-dummy
- [IsOnMission](#isonmission) - Checks if on mission flag is set
- [LoadModel](#loadmodel) - Loads model by id
- [ReplaceStringInFile](#replacestringinfile) - Replaces the first occurrence of a substring in a file
- [SetOnMission](#setonmission) - Sets on mission flag
- [ClearBlipOnCharDeath](#clearbliponchardeath) - Makes the blip to disappear when character dies
- [GetCLEOSDK](#getcleosdk) - Returns a pointer to a function exported from CLEO.asi
- [GetCLEOVersion](#getcleoversion) - Returns the version of the CLEO library
- [GetTimeScale](#gettimescale) - Returns current gameplay speed multiplier (set with set_time_scale)

## Vehicle Functions

- [IsThisEntityAVehicle](#isthisentityavehicle) - Checks if this entity is a vehicle
- [SpawnCar](#spawncar) - Spawns a new car like a cheat and returns its handle
- [SetCarPlateText](#setcarplatetext) - Changes the text on car's number plate
- [GetCarFlag](#getcarflag) - Returns a car flag
- [SetCarFlag](#setcarflag) - Sets a car flag
- [IsMissionCar](#ismissioncar) - Checks if a car is a mission car
- [OpenCarWindow](#opencarwindow) - Opens a car window (0-3)
- [CloseCarWindow](#closecarwindow) - Closes a car window (0-3)
- [IsCarWindowOpen](#iscarwindowopen) - Checks if car window (0-3) is open
- [GetCarDoorNodeId](#getcardoornodeid) - Returns car node id for a door. (left/right, front/rear) (0-LF,1-RF,2-LR,3-RR)

## Math Functions

- [Min](#min) - Returns the smallest of two integers
- [MinF](#minf) - Returns the smallest of two floats
- [Max](#max) - Returns the largest of two integers
- [MaxF](#maxf) - Returns the largest of two floats
- [ToRad](#torad) - Converts degrees to radians
- [ToDeg](#todeg) - Converts radians to degrees

## Debug Functions

- [DumpScriptVars](#dumpscriptvars) - Writes local variables (0@-31@) to CLEO.log
- [Log](#log) - Adds a new entry in CLEO.log
- [ReloadThisScript](#reloadthisscript) - Reload current script from disk
- [TeleportToNearestCar](#teleporttonearestcar) - Teleports player to the nearest car
- [TeleportToMarker](#teleporttomarker) - Teleports player to the red target marker
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

#### SetFOV

```lua
/// Sets the field of view (FOV) for the camera
function SetFOV(fov: float)
    write_memory_with_offset {address} 0xB6F028 {offset} 0xCB8 {size} 4 {value} fov
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

- GetWeatherForecast(hours: int): int - returns weather type coming in {hours}

#### ReplaceStringInFile

```lua
/// Replaces the first occurence of {find_string} found in the file at {filePath} with {replace_string} in-place
function ReplaceStringInFile(filePath: string, find: string, replace: string)
    int f

    // open file for reading
    if f = open_file filePath "rb"
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
                f = open_file filePath "wb"

                int pFrom, count

                // copy all before found string
                pFrom = source
                count = p - pFrom
                trace "copy first %d bytes of file %s" count filePath
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
                trace "copy last %d bytes of file %s" count filePath
                write_block_to_file f count pFrom

            else
                trace "%s not found in %s" find_string filePath
            end
        else
            trace "can't read file %s" filePath
        end

        close_file f
        free_memory source

    else
        trace "file %s not found" filePath
    end
    /// Finds the first occurrence of a substring in a string and returns a pointer to it
    function strstr<cdecl, 0x822650>(str: string, substr: string): string
    /// Returns str length
    function strlen<cdecl, 0x718690>(str: string): int
end
```

- IsPointInsideGarage(x: float, y: float, z: float): logical - return true if point is located inside a garage

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

- ClearBlipOnCharDeath(char: int) - makes the blip to disappear when {char} dies

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

#### GetCLEOSDK

```lua
/// Returns a pointer to a function exported from CLEO.asi
function GetCLEOSDK(name: string): optional int
    if int cleo = load_dynamic_library {fileName} "CLEO.asi"
    then
        if int func = get_dynamic_library_procedure {procName} name cleo
        then
            free_dynamic_library cleo
            return func
        else
            free_dynamic_library cleo
            return
        end
    end
end
```

#### GetCLEOVersion

```lua
/// Returns CLEO Library version, e.g. 0x50100000 for 5.1.0
function GetCLEOVersion(): optional int
    if GetVersion fn = GetCLEOSDK("_CLEO_GetVersion@0")
    then
        int version = fn()
        return version
    end
    return

    function GetVersion<stdcall>(): int
end
```

### Vehicles

#### IsThisEntityAVehicle

```lua
/// Checks if this entity is a vehicle
function IsThisEntityAVehicle(address: int): logical
    int type = GetEntityType(address)
    return type == 2
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

#### IsMissionCar

```lua
function IsMissionCar(handle: Car): logical
    int ptr = get_vehicle_pointer {handle} handle
    int createdBy = read_memory_with_offset {address} ptr {offset} 0x4A4 {size} 1 // CVehicle::m_nCreatedBy
    return createdBy == 2 // MISSION_VEHICLE
end
```

#### GetCarFlag

```lua
const VEHICLEFLAG_ISLAWENFORCER = 0               // Is this guy chasing the player at the moment
const VEHICLEFLAG_ISAMBULANCEONDUTY = 1           // Ambulance trying to get to an accident
const VEHICLEFLAG_ISFIRETRUCKONDUTY = 2           // Firetruck trying to get to a fire
const VEHICLEFLAG_ISLOCKED = 3                    // Is this guy locked by the script (cannot be removed)
const VEHICLEFLAG_ENGINEON = 4                    // For sound purposes. Parked cars have their engines switched off (so do destroyed cars)
const VEHICLEFLAG_ISHANDBRAKEON = 5               // How's the handbrake doing ?
const VEHICLEFLAG_LIGHTSON = 6                    // Are the lights switched on ?
const VEHICLEFLAG_FREEBIES = 7                    // Any freebies left in this vehicle ?
const VEHICLEFLAG_ISVAN = 8                       // Is this vehicle a van (doors at back of vehicle)
const VEHICLEFLAG_ISBUS = 9                       // Is this vehicle a bus
const VEHICLEFLAG_ISBIG = 10                       // Is this vehicle big
const VEHICLEFLAG_LOWVEHICLE = 11                  // Need this for sporty type cars to use low getting-in/out anims
const VEHICLEFLAG_COMEDYCONTROLS = 12              // Will make the car hard to control (hopefully in a funny way)
const VEHICLEFLAG_WARNEDPEDS = 13                  // Has scan and warn peds of danger been processed?
const VEHICLEFLAG_CRANEMESSAGEDONE = 14            // A crane message has been printed for this car already
const VEHICLEFLAG_TAKELESSDAMAGE = 15              // This vehicle is stronger (takes about 1/4 of damage)
const VEHICLEFLAG_ISDAMAGED = 16                   // This vehicle has been damaged and is displaying all its components
const VEHICLEFLAG_HASBEENOWNEDBYPLAYER = 17        // To work out whether stealing it is a crime
const VEHICLEFLAG_FADEOUT = 18                     // Fade vehicle out
const VEHICLEFLAG_ISBEINGCARJACKED = 19            //
const VEHICLEFLAG_CREATEROADBLOCKPEDS = 20         // If this vehicle gets close enough we will create peds (coppers or gang members) round it
const VEHICLEFLAG_CANBEDAMAGED = 21                // Set to FALSE during cut scenes to avoid explosions
const VEHICLEFLAG_OCCUPANTSHAVEBEENGENERATED = 22  // Is true if the occupants have already been generated. (Shouldn't happen again)
const VEHICLEFLAG_GUNSWITCHEDOFF = 23              // Level designers can use this to switch off guns on boats
const VEHICLEFLAG_VEHICLECOLPROCESSED = 24         // Has ProcessEntityCollision been processed for this car?
const VEHICLEFLAG_ISCARPARKVEHICLE = 25            // Car has been created using the special CAR_PARK script command
const VEHICLEFLAG_HASALREADYBEENRECORDED = 26      // Used for replays
const VEHICLEFLAG_PARTOFCONVOY = 27
const VEHICLEFLAG_HELIMINIMUMTILT = 28             // This heli should have almost no tilt really
const VEHICLEFLAG_AUDIOCHANGINGGEAR = 29           // sounds like vehicle is changing gear
const VEHICLEFLAG_ISDROWNING = 30                  // is vehicle occupants taking damage in water (i.e. vehicle is dead in water)
const VEHICLEFLAG_TYRESDONTBURST = 31              // If this is set the tyres are invincible
const VEHICLEFLAG_CREATEDASPOLICEVEHICLE = 32      // True if this guy was created as a police vehicle (enforcer, policecar, miamivice car etc)
const VEHICLEFLAG_RESTINGONPHYSICAL = 33           // Don't go static cause car is sitting on a physical object that might get removed
const VEHICLEFLAG_PARKING = 34
const VEHICLEFLAG_CANPARK = 35
const VEHICLEFLAG_FIREGUN = 36                     // Does the ai of this vehicle want to fire it's gun?
const VEHICLEFLAG_DRIVERLASTFRAME = 37             // Was there a driver present last frame ?
const VEHICLEFLAG_NEVERUSESMALLERREMOVALRANGE = 38 // Some vehicles (like planes) we don't want to remove just behind the camera.
const VEHICLEFLAG_ISRCVEHICLE = 39                 // Is this a remote controlled (small) vehicle. True whether the player or AI controls it.
const VEHICLEFLAG_ALWAYSSKIDMARKS = 40             // This vehicle leaves skidmarks regardless of the wheels' states.
const VEHICLEFLAG_ENGINEBROKEN = 41                // Engine doesn't work. Player can get in but the vehicle won't drive
const VEHICLEFLAG_VEHICLECANBETARGETTED = 42       // The ped driving this vehicle can be targeted, (for Torenos plane mission)
const VEHICLEFLAG_PARTOFATTACKWAVE = 43            // This car is used in an attack during a gang war
const VEHICLEFLAG_WINCHCANPICKMEUP = 44            // This car cannot be picked up by any ropes.
const VEHICLEFLAG_IMPOUNDED = 45                   // Has this vehicle been in a police impounding garage
const VEHICLEFLAG_VEHICLECANBETARGETTEDBYHS = 46   // Heat seeking missiles will not target this vehicle.
const VEHICLEFLAG_SIRENORALARM = 47                // Set to TRUE if siren or alarm active, else FALSE
const VEHICLEFLAG_HASGANGLEANINGON = 48
const VEHICLEFLAG_GANGMEMBERSFORROADBLOCK = 49     // Will generate gang members if NumPedsForRoadBlock > 0
const VEHICLEFLAG_DOESPROVIDECOVER = 50            // If this is false this particular vehicle can not be used to take cover behind.
const VEHICLEFLAG_MADDRIVER = 51                   // This vehicle is driving like a lunatic
const VEHICLEFLAG_UPGRADEDSTEREO = 52              // This vehicle has an upgraded stereo
const VEHICLEFLAG_CONSIDEREDBYPLAYER = 53          // This vehicle is considered by the player to enter
const VEHICLEFLAG_PETROLTANKISWEAKPOINT = 54       // If false shooting the petrol tank will NOT Blow up the car
const VEHICLEFLAG_DISABLEPARTICLES = 55            // Disable particles from this car. Used in garage.
const VEHICLEFLAG_HASBEENRESPRAYED = 56            // Has been resprayed in a respray garage. Reset after it has been checked.
const VEHICLEFLAG_USECARCHEATS = 57                // If this is true will set the car cheat stuff up in ProcessControl()
const VEHICLEFLAG_DONTSETCOLOURWHENREMAPPING = 58  // If the texture gets remapped we don't want to change the colour with it.
const VEHICLEFLAG_USEDFORREPLAY = 59               // This car is controlled by replay and should be removed when replay is done.
```

```lua
/// Returns a car flag
/// flagIdx is one of the VEHICLEFLAG_* constants
function GetCarFlag(handle: Car, flagIdx: int): logical
    int ptr = get_vehicle_pointer {handle} handle
    int bitIdx = flagIdx % 8

    int offset = 0x428 // CVehicle::m_nVehicleFlags
    flagIdx /= 8
    offset += flagIdx
    ptr += offset

    int flags = read_memory {address} ptr {size} 1 {vp} false

    is_bit_set {var_number} flags {bitIndex} bitIdx
end
```

#### SetCarFlag

```lua
/// Sets a car flag
/// flagIdx is one of the VEHICLEFLAG_* constants
function SetCarFlag(handle: Car, flagIdx: int, state: int)
    int ptr = get_vehicle_pointer {handle} handle
    int bitIdx = flagIdx % 8

    int offset = 0x428 // CVehicle::m_nVehicleFlags
    flagIdx /= 8
    offset += flagIdx
    ptr += offset

    int flags = read_memory {address} ptr {size} 1 {vp} false
    if
        state <> false
    then
        set_bit {var_number} flags {n} bitIdx
    else
        clear_bit {var_number} flags {n} bitIdx
    end
    write_memory {address} ptr {size} 1 {value} flags {vp} false
end
```

#### OpenCarWindow

```lua
/// Opens a car window (0-3)
function OpenCarWindow(vehicle: Car, window: int)
    int pCar = get_vehicle_pointer {handle} vehicle
    if int nodeId = GetCarDoorNodeId(window)
    then
        CVehicle_SetWindowOpenFlag(pCar, nodeId)
    end
    function CVehicle_SetWindowOpenFlag<thiscall,0x6D3080>(struct: int, nodeId: int)
end
```

#### CloseCarWindow

```lua
/// Closes a car window (0-3)
function CloseCarWindow(vehicle: Car, window: int)
    int pCar = get_vehicle_pointer {handle} vehicle
    if int nodeId = GetCarDoorNodeId(window)
    then
        CVehicle_ClearWindowOpenFlag(pCar, nodeId)
    end
    function CVehicle_ClearWindowOpenFlag<thiscall,0x6D30B0>(struct: int, nodeId: int)
end
```

#### IsCarWindowOpen

```lua
/// Checks if car window (0-3) is open
function IsCarWindowOpen(vehicle: Car, window: int): logical
    const ATOMIC_IS_DOOR_WINDOW_OPENED = 12
    int pCar = get_vehicle_pointer {handle} vehicle
    int clump = read_memory_with_offset {address} pCar {offset} 0x18 {size} 4
    if int nodeId = GetCarDoorNodeId(window)
    then
        int frame = CClumpModelInfo_GetFrameFromId(clump, nodeId)
        int atomic = read_memory_with_offset {address} frame {offset} 0x90 {size} 4
        atomic -= 8
        int flag = CVisibilityPlugins_GetUserValue(atomic)

        is_bit_set flag ATOMIC_IS_DOOR_WINDOW_OPENED
    end

    function CVisibilityPlugins_GetUserValue<cdecl,0x7323A0>(atomic: int): int {int16}
    function CClumpModelInfo_GetFrameFromId<cdecl,0x4C53C0>(clump: int {RpClump*}, id:int): int {RwFrame*}
end
```

#### GetCarDoorNodeId

```lua
/// Returns car node id for a door. (left/right, front/rear) (0-LF,1-RF,2-LR,3-RR)
function GetCarDoorNodeId(door: int): optional int
    switch door
    case 0
        return 10
    case 1
        return 8
    case 2
        return 11
    case 3
        return 9
    default
        return false
    end
end
```

- GetUserSettingsInt(settingId: int): int - returns an integer value of a particular configuration in the main menu. list of settings TBD
- SetUserSettingsInt(settingId: int, value: int) - sets new value of a particular configuration in the main menu. list of settings TBD
- GetUserSettingsFloat(settingId: int): float - returns an integer value of a particular configuration in the main menu. list of settings TBD
- SetUserSettingsFloat(settingId: int, value: float) - sets new value of a particular configuration in the main menu. list of settings TBD
- GetDummyCoords(vehicle: Car, dummyId: int): float, float, float - returns XYZ of a particular dummy of the car

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

- SaveScreenToPng(f: string, left: int, top: int, w: int, h: int) - saves portion of screen to a png file

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

- ViewEntityCoords3d(entity: int) - prints entity (CVehicle, CPed, CObject) coordinates above it

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

#### ClearBlipOnCharDeath

```lua
/// Clears the blip on character death
function ClearBlipOnCharDeath(handle: Char)
    int address = get_ped_pointer {char} handle
    int flags = read_memory_with_offset address {offset} 0x474 {size} 4
    set_bit {var_number} flags {bitIndex} 13
    write_memory_with_offset address {offset} 0x474 {size} 4 {value} flags
end
```

#### GetTimeScale

```lua
/// Returns current gameplay speed multiplier (set with set_time_scale)
function GetTimeScale(): float
    float speed = read_memory 0x00B7CB64 {size} 4 {vp} false
    return speed
end
```

## Credits

Seemann, Vital, Miran, OrionSR.
