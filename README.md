# Dynamic Caches 2.3.0 — Configuration reference

Author: Griffin.

Dynamic Caches places locked loot containers at saved points inside buildings. This guide explains every setting in Settings.json, Caches.json and Loot.json, followed by advanced examples you can adapt to your server. Building points apply to every instance of the same building class, including rotated copies. Loot is rolled once when a normal cache is opened.

| Section | Use it to configure |
| --- | --- |
| [Settings.json](#settingsjson) | Administrators, key drops, cache appearance, map-wide limits, locks, action durations and tool damage. |
| [Caches.json](#cachesjson) | Building classes, local positions/orientations, point chances and combined per-building limits. |
| [Loot.json](#lootjson) | Reusable item sets, independent set/item rolls, counts, quantities, health, attachments and key identity. |
| [Advanced examples](#advanced-configuration-examples) | Custom categories, paired/tools-only/class-based locks, mixed building categories, layered loot and key progression. |
| [Administrator commands](#administrator-commands) | Record, inspect, delete, clear and undo saved positions. |
| [Installation](#installation) | Required mods and first startup. |

## Configuration files and loading

The server uses `$profile:DynamicCaches`, under the profile directory chosen for your server. All three active files use `"Version": 1`. Use valid JSON: double-quoted strings, numeric values without quotes, true/false booleans, and no comments or trailing commas. The snippets in this guide identify the file and array/object to edit; fragments are not complete replacement files.

Missing Settings.json, Caches.json and Loot.json are created during server initialization, before the map scan or any administrator command. Existing valid files are retained. Fresh Settings/Loot include civilian, construction, firestation, hospital, military and police. Fresh Caches.json has no buildings or positions:

```json
{
  "Version": 1,
  "Buildings": []
}
```

Stop the server before manual JSON edits, then restart to load them. Chat position edits save immediately through a verified temporary file and Caches.json.bak. Existing gameplay caches keep their loot until restart; changed positions are used by the next inspection scan and normal spawning after restart. Invalid JSON, unknown referenced IDs, invalid classes or out-of-range settings prevent configuration loading/spawning; the server log identifies the problem.

Definitions link across files as follows:

| Define an ID here | Reference it here |
| --- | --- |
| Settings.Keys[].Id | Settings.Caches[].Lock.KeyId and Loot.ItemSets[].Items[].KeyId. |
| Settings.Caches[].Id | Caches.Buildings[].Categories[].CacheId and Loot.Caches[].CacheId. |
| Loot.ItemSets[].Id | Loot.Caches[].Sets[].SetId. |

Every Settings category needs exactly one Loot.Caches entry; Sets may be empty. A category can exist without any building points, but it cannot spawn until points are saved for it. Adding a point creates the placement category under a building class; it does not create the Settings/Loot definitions.

If a valid older Config.json exists, it supplies data for missing files and remains untouched. Otherwise missing files use bundled defaults. Existing active files are preserved except for the supported old category-name migration: medical_cache becomes hospital, firefighter_cache becomes firestation, and the other stock _cache IDs lose that suffix. Migration saves backups and retains recorded points, settings, keys and loot; old empty stock building placeholders are removed. Conflicting category IDs prevent saving migration. Key IDs such as medical_key and loot-set IDs such as medical_supplies retain their existing names.


## Setting conventions

An ID identifies a definition; a display name is the text players see. Use exact spelling/case for JSON references and DayZ class names. Keep key IDs stable because distributed keys retain their assigned ID. IDs and cache/key display names must be nonempty and at most 64 characters. IDs must be unique within their definition list; point IDs must be unique within a category at a building class.

Percent chances use `0` through `100`: `0` never passes and `100` always passes. Random minimum/maximum bounds are inclusive. Item counts and loot-set repetitions are integers; the maximum must be at least the minimum. Health is normalized: `1` means full health and `0.7` means 70% of maximum health. Quantity uses the item's native units, such as magazine rounds, stack count or liquid quantity.

The field tables describe every active JSON setting. Where an omitted-field default is stated, it is the value used when that field is absent; generated stock entries can explicitly set a different value. Keep identity/reference fields explicit. The stock tables below show bundled values.

## Settings.json

### File settings

| Setting | Instructions and valid values |
| --- | --- |
| `Version` | Schema version. Keep `1`. |
| `AdminSteamIDs` | Array of up to 100 Steam64 ID strings. `[]` grants command access to nobody. |
| `MaxKeysPerInfected` | Maximum successful mod key drops from one infected death, across key types. Integer `0`–`10`; default `1`. `0` disables infected key drops globally. Eligible key definitions are tested in random order until this limit is reached. |
| `Keys` | Array of key definitions, up to 100. Different key IDs may share the same physical `DC_Key` class. |
| `Caches` | Array of cache category definitions, up to 100. These IDs are used by addpos, placement categories and Loot.json. |

### Keys[]

| Setting | Instructions and valid values |
| --- | --- |
| `Id` | Unique logical key ID, for example `medical_key`. Refer to it from a lock's KeyId or a loot item's KeyId. |
| `DisplayName` | Player-visible name assigned to this key, for example `Hospital Cache Key`. |
| `ClassName` | Spawnable item class inheriting from `DC_Key`; stock value `DC_Key`. A custom class must also be installed on server/clients. Logical key identity is assigned by the mod when it drops or rolls the key. |
| `DropEnabled` | Enable this key's infected-death drop roll. Omitted default `false`; all six stock keys set `true`. Does not disable this key in Loot.json. |
| `AllInfected` | If `true`, any infected is eligible. If `false`, only InfectedClasses matches are eligible. Omitted default `false`. |
| `InfectedClasses` | Array of up to 100 infected class names. Matching includes descendants of those classes. Needed when DropEnabled=true and AllInfected=false; ignored as a filter when AllInfected=true. Every listed class must inherit from `ZombieBase`. |
| `DropChance` | Chance for this key to drop from an eligible infected, `0`–`100`. Omitted default `5`; stock values below. The global key cap can prevent additional successful rolls from being attempted. |

Key drops are rolled once per infected death, on the server. They are placed in the infected's inventory; if inventory creation fails, the mod attempts a ground drop with a 30-minute lifetime. Successful drops start at full health. Raw `DC_Key` items spawned through another system have no assigned logical KeyId unless that system assigns one; use this mod's infected drops or loot KeyId setting for ID-based locks.

### Caches[]

| Setting | Instructions and valid values |
| --- | --- |
| `Id` | Unique category ID, for example `hospital`. Caches.json and Loot.json must refer to this exact ID. |
| `DisplayName` | Player-visible cache name. Also supplies the lock's CacheName. |
| `ModelSize` | `small`, `wide`, or `""` for automatic selection. Small has 8×8 cargo; wide has 10×10. Automatic selection uses wide for firestation/police/military/construction and small for other IDs. |
| `Theme` | Skin name: `hospital`, `firefighter`, `police`, `military`, `construction`, `civilian`, or `""` for automatic selection. The category **firestation uses Theme `firefighter`**. Automatic selection maps known IDs to their stock skin; other IDs use civilian. |
| `Enabled` | Whether this category can spawn during normal gameplay. Default `true`. Show-all ignores this flag. |
| `MaxActive` | Maximum normal caches of this category across the entire map. Integer `0`–`500`; omitted default `10`, stock limits below. `0` disables normal spawning. Sum of configured MaxActive values must not exceed 1000, including disabled categories. Show-all ignores this limit. |
| `Lock` | Lock requirements, action timing and tool damage, described below. At least a usable key requirement or enabled nonempty tool list is required. |

### Caches[].Lock

| Setting | Instructions and valid values |
| --- | --- |
| `KeyClass` | Physical key item class. Default `DC_Key`. With KeyId set, it is refreshed from that key definition and matching uses the logical ID. With KeyId empty, matching requires this exact non-ruined item class. Set `""` with an empty KeyId for a tools-only lock. |
| `KeyId` | Logical key ID from Settings.Keys; `""` means use KeyClass matching instead. A nonempty ID takes precedence over generic class matching. |
| `KeyName` | Derived tooltip text: refreshed from the referenced key's DisplayName. With no KeyId, the mod clears this field and uses the KeyClass item's name in requirement text. Edit the key DisplayName to change an ID-based key's name. |
| `CacheName` | Derived cache name: refreshed from this cache's DisplayName. Edit DisplayName instead of this field. |
| `ToolClasses` | Array of up to 16 accepted exact item classes, for example `["Crowbar"]`. Descendant classes do not automatically match. Stock entries accept Crowbar. |
| `AllowTools` | Enable the tool-opening route. Default `true`. Set `false` for key-only opening. A tools-only lock needs true plus at least one ToolClasses entry. |
| `RequireKeyAndTool` | Default `false`: a matching key OR an enabled accepted tool can open. If `true`, both a usable key and accepted tool must be carried. Hold either one to use that route; the other is found in the player's inventory. Requires a configured key and enabled nonempty tool list. |
| `ConsumeKey` | Legacy compatibility field. The server forces it to `false`; it does not configure current key behavior. A used regular key is always ruined on successful opening, including paired key+tool opening. It is not deleted by this setting. The admin master key stays undamaged and reusable. |
| `ToolDamage` | Absolute damage applied to the tool used on successful opening, `0`–`1000` health points. `0` disables tool damage. Omitted default `15`; stock entries use `25`. Also applies to the carried tool when key+tool opening is required. |
| `KeySeconds` | Duration of the key-opening action, `0.5`–`300` seconds. Default `2`. |
| `ToolSeconds` | Duration of the tool-opening action, `0.5`–`300` seconds. Default `60`. |

Opening requires the chosen key/tool in the player's hands, not ruined, within 3m of the cache. When both are required, the other carried item must also be usable. Cancelling the action leaves the cache locked. Successful opening generates loot once, opens the lid and leaves the cache unlocked. Used regular mod keys cannot be repaired. Locked caches hide cargo; preview caches never offer an opening route. The admin key exception is described below.

### Admin master key

Spawn item class **`DC_AdminKey`** through your administrator item-spawning tool. Its displayed name is **Admin Cache Key**. Hold it in your hands, approach a normal cache within 3m and use **Open Cache**. It opens every category, including custom categories, tools-only locks and locks requiring both a key and tool. No matching KeyId or extra tool is needed. Opening takes that cache's KeySeconds duration and rolls loot normally once.

The admin key is indestructible and infinitely reusable: damage is disabled, successful opening never ruins/consumes it, and loading it after persistence restores full health and damage protection. It does not damage a carried tool when used. Regular keys and tools retain their configured behavior. Previews, stale caches and already-open caches cannot be opened again with it.

This is an item-based override: whoever holds the key can use it. AdminSteamIDs governs chat commands, not possession of this item. The admin key is not included in bundled infected drops or loot. It needs no Settings/Caches/Loot changes or logical key assignment to function.

### Bundled category and key values

All stock categories are enabled and accept Crowbar OR their assigned key. All use KeySeconds=2, ToolSeconds=60, ToolDamage=25, AllowTools=true and RequireKeyAndTool=false. Every stock category rolls its listed item set once with a 100% set chance; items inside have their own chances/counts.

| Category ID | DisplayName | ModelSize | Theme | MaxActive | KeyId | Loot SetId |
| --- | --- | --- | --- | --- | --- | --- |
| `civilian` | Civilian Cache | small | civilian | 20 | civilian_key | civilian_supplies |
| `construction` | Construction Cache | wide | construction | 12 | construction_key | construction_supplies |
| `firestation` | Firefighter Cache | wide | firefighter | 8 | firefighter_key | firefighter_supplies |
| `hospital` | Hospital Cache | small | hospital | 10 | medical_key | medical_supplies |
| `military` | Military Cache | wide | military | 6 | military_key | military_supplies |
| `police` | Police Cache | wide | police | 10 | police_key | police_supplies |

| Key Id | DisplayName | DropChance | Eligible infected |
| --- | --- | --- | --- |
| `civilian_key` | Civilian Cache Key | 1.5% | All infected (AllInfected=true). |
| `construction_key` | Construction Cache Key | 7% | ZmbM_ConstrWorkerNormal_Base, ZmbM_HeavyIndustryWorker_Base and descendants. |
| `firefighter_key` | Firefighter Cache Key | 8% | ZmbM_FirefighterNormal_Base and descendants. |
| `medical_key` | Hospital Cache Key | 6% | ZmbF_DoctorSkinny_Base, ZmbM_DoctorFat_Base and descendants. |
| `military_key` | Military Cache Key | 3% | ZmbM_SoldierNormal_Base and descendants. |
| `police_key` | Police Cache Key | 5% | ZmbM_PolicemanFat_Base, ZmbF_PoliceWomanNormal_Base, ZmbM_PolicemanSpecForce_Base and descendants. |

## Caches.json

### File and building settings

| Setting | Instructions and valid values |
| --- | --- |
| `Version` | Keep `1`. |
| `Buildings` | Array of building definitions, up to 500. Fresh default `[]`. Normally populated by target/addpos. |
| `Buildings[].BuildingClass` | Exact DayZ building class returned by the targeted object, for example `Land_City_Hospital`. Must inherit from `House` and be unique in Buildings. Points apply to every instance of this class. |
| `Buildings[].Categories` | Array of category placement definitions under this class, up to 100. Target creates an empty array. Adding the first point creates a category. Only one entry per CacheId is permitted under a class. |

### Buildings[].Categories[]

| Setting | Instructions and valid values |
| --- | --- |
| `CacheId` | Category ID from Settings.Caches, for example `hospital`. |
| `MaxPerBuilding` | Combined normal-cache limit for one physical building, across its applicable category definitions. Integer `1`–`100`; new categories default to `1`. When limits differ, the lowest applicable limit among normally enabled categories with MaxActive>0 wins, including rules with no positions. Example: hospital=1 and police=3 permit only one normal cache total at that building. Show-all ignores the limit. To disable a category, use Enabled/MaxActive in Settings rather than setting this to 0. |
| `Positions` | Array of possible points for this category at this class, up to 100. `[]` means no possible spawns for that entry. |

There can be at most 500 category placement definitions in total across all building classes, as well as the individual limits above.

### Buildings[].Categories[].Positions[]

| Setting | Instructions and valid values |
| --- | --- |
| `Id` | Nonempty point ID, up to 64 characters and unique within this category/class. Chat saves generate/reindex slot_1, slot_2, etc. This is not the global deletion number shown by list. |
| `Position` | Building-local vector encoded as a string, for example `"1.2 0.4 -2.5"`. Components are local X, Y (height), Z in metres, each within -100 to 100. Addpos captures the player's location relative to the selected building. Not map/world coordinates. |
| `Orientation` | Building-local yaw, pitch, roll in degrees, encoded as a string, for example `"90 0 0"`. Each component must be within -360 to 360. Addpos captures the player's full orientation; the mod combines it with each building's rotation when spawning. Default `"0 0 0"`. |
| `Chance` | Normal candidate-selection chance per point per matching building, `0`–`100`; default `100`. Passing this roll does not guarantee spawning if active/building limits, occupancy or collision checks prevent it. Show-all ignores this chance. |

This example shows one point; it is not inserted into fresh configurations automatically:

```json
{
  "Version": 1,
  "Buildings": [
    {
      "BuildingClass": "Land_City_Hospital",
      "Categories": [
        {
          "CacheId": "hospital",
          "MaxPerBuilding": 1,
          "Positions": [
            {
              "Id": "slot_1",
              "Position": "1.2 0.4 -2.5",
              "Orientation": "90 0 0",
              "Chance": 100
            }
          ]
        }
      ]
    }
  ]
}
```

Prefer recording coordinates in-game because building origins and floor heights differ. Normal spawning shuffles accepted candidates before applying counts. Caches use their collision bounds and five short floor probes to rest above nearby static floors, including upper storeys; their orientation is retained. Missing/uneven support and walls/objects/ceilings can reject a point. Allow sufficient width for the model and room for its lid to open. Floor search/clearance tolerances are built into the mod and are not JSON settings. Inspection scanning stops with an error above 200,000 candidate spawns.

## Loot.json

### File and category assignments

| Setting | Instructions and valid values |
| --- | --- |
| `Version` | Keep `1`. |
| `Caches` | Array of category-to-set assignments, up to 100. Every Settings.Caches category must have exactly one entry, even if its Sets array is empty. |
| `Caches[].CacheId` | Exact category ID from Settings.Caches. Unknown/duplicate IDs are rejected. |
| `Caches[].Sets` | Array of up to 30 loot-set roll definitions for this category. `[]` deliberately produces no loot. Each definition is tested independently when a normal cache first unlocks. |
| `ItemSets` | Array of reusable item-set definitions, up to 100. Different cache categories can use the same set. |
| `ItemSets[].Id` | Unique item-set ID used by SetId references, for example `medical_supplies`. |
| `ItemSets[].Items` | Array of item entries, up to 100 per set. Each entry is tested independently on every repetition of that set. |

### Caches[].Sets[]

| Setting | Instructions and valid values |
| --- | --- |
| `SetId` | Existing ItemSets.Id to roll. |
| `Chance` | Chance to activate this set assignment, `0`–`100`; default `100`. Tested once before its repetition count is chosen. |
| `MinRolls` | Minimum repetitions if the set chance passes. Integer `0`–`20`; default `1`. |
| `MaxRolls` | Maximum repetitions. Integer from MinRolls to `20`; default `1`. The repetition count is chosen inclusively between the two limits. |

Every repetition tests all item entries again. This is not a choice of one item from the set. For example, set Chance=50 and an item's Chance=50 give that item a 25% chance per opening when exactly one repetition is used, before count/cargo limits. Increasing MaxRolls can produce multiple copies.

### ItemSets[].Items[]

| Setting | Instructions and valid values |
| --- | --- |
| `ClassName` | Spawnable item, magazine or weapon class. Default `BandageDressing`. Referenced modded classes must be installed. Cache container classes themselves are rejected as loot. |
| `KeyId` | When ClassName is a mod key, assign its logical ID/display name from Settings.Keys. Default `""` assigns no identity. A nonempty value must reference an existing key definition whose ClassName exactly matches this item entry. |
| `Chance` | Chance to spawn this entry during each set repetition, `0`–`100`; default `100`. Tested once for the entry, then its count is chosen. |
| `MinCount` | Minimum number of copies when the item's chance passes. Integer `0`–`20`; default `1`. |
| `MaxCount` | Maximum copies, integer from MinCount to `20`; default `1`. Inclusive random count. A count of 0 spawns nothing. |
| `Quantity` | Fixed quantity when RandomQuantity=false. Range `-1`–`100000`; default `-1` leaves the item's initial quantity unchanged. Use integer rounds for magazines. Values are capped at the item's native capacity and ignored for items without quantity. |
| `RandomQuantity` | Default `false`: use Quantity. If true, choose quantity between QuantityMin/QuantityMax separately for every copy. |
| `QuantityMin` | Lower random quantity bound, `0`–`100000`; default `0`. With magazines, rounded upwards to the next integer. |
| `QuantityMax` | Upper random quantity bound, from QuantityMin to `100000`; default `1`. With magazines, rounded downwards to an integer; the resulting integer range must be nonempty. Quantities are capped at native capacity. Both bounds are still validated when random quantity is disabled. |
| `HealthMin` | Lower random health bound, or the fixed health when RandomHealth=false. Range `0.01`–`1`; omitted default `1`. Stock entries commonly use `0.7`. |
| `HealthMax` | Upper random health bound, from HealthMin to `1`; default `1`. Still validated when random health is disabled. |
| `RandomHealth` | Default `true`: independently choose each spawned copy's normalized health between HealthMin/HealthMax. If false, use HealthMin exactly. |
| `Attachments` | Array of up to 15 direct attachment class names, default `[]`. Each must be a spawnable class and fit an available attachment slot on the parent item. Each parent copy gets an attachment attempt for every listed class. No nested attachment definitions are supported. Incompatible/occupied slots are logged and skipped; parent quantity/health settings are not applied to attachments. |

The validator limits each category to at most 500 possible parent items per opening, computed from all referenced entries' MaxCount × their assignment's MaxRolls, even for low/zero-chance entries. Cargo capacity can be lower than this maximum. Items that cannot fit are logged and skipped; they do not spill onto the ground. Loot rolls once per cache opening, with no timed refill/relock during that server session.

Here is a complete item entry demonstrating random magazine quantity and fixed condition. Add it to a referenced set's Items array:

```json
{
  "ClassName": "Mag_STANAG_30Rnd",
  "KeyId": "",
  "Chance": 50,
  "MinCount": 1,
  "MaxCount": 2,
  "Quantity": -1,
  "RandomQuantity": true,
  "QuantityMin": 10,
  "QuantityMax": 30,
  "HealthMin": 0.8,
  "HealthMax": 1,
  "RandomHealth": false,
  "Attachments": []
}
```

## Adding a custom category

1. Copy a category definition in Settings.Caches and give it a new lower-case single-word Id and DisplayName. Select ModelSize, Theme, limits and lock behavior.
2. Reuse an existing KeyId, or add a matching definition to Settings.Keys. For tools-only opening, clear both KeyId and KeyClass and keep AllowTools=true with a nonempty ToolClasses array.
3. Add one Loot.Caches entry with CacheId matching the new category. Reuse existing SetIds, or add ItemSets with new unique IDs and item entries. An empty Sets array is valid for an intentionally empty cache.
4. Restart to load the definitions. Target a building and use `!dc addpos <newid>`; the first point creates that category under the targeted class. Run list/show-all to inspect, and hide-all before further edits.

Renaming a category requires updating Settings.Caches.Id, every matching Caches.Buildings[].Categories[].CacheId, and Loot.Caches[].CacheId. Renaming a loot set requires updating ItemSets.Id and every SetId reference. Renaming a key requires updating the key definition and all Lock.KeyId/loot-item KeyId references; already-distributed keys still carry their old ID, so preserving key IDs is preferable.

## Advanced configuration examples

Apply the examples independently unless an example explicitly refers to another. Keep existing entries and unique IDs: append new definitions, replace the stated existing entry/object, and avoid duplicate category assignments. All examples use Version 1 file schemas and existing vanilla/mod classes. Omitted fields use the defaults in the reference tables.

### 1. A custom category with a rare infected key

Add the key below to **Settings.json → Keys**, and the category below to **Settings.json → Caches**. The wrapper shows the two destination arrays; merge their entries into your existing file. The new armory category has a wide military skin and at most three normal caches across the map. Its key drops only from military infected and their descendants, with a 2% chance when that definition is tested. Ordinary tools cannot open it.

```json
{
  "Keys": [
    {
      "Id": "armory_key",
      "DisplayName": "Armory Cache Key",
      "ClassName": "DC_Key_Red",
      "DropEnabled": true,
      "AllInfected": false,
      "InfectedClasses": ["ZmbM_SoldierNormal_Base"],
      "DropChance": 2
    }
  ],
  "Caches": [
    {
      "Id": "armory",
      "DisplayName": "Armory Cache",
      "ModelSize": "wide",
      "Theme": "military",
      "Enabled": true,
      "MaxActive": 3,
      "Lock": {
        "KeyClass": "DC_Key_Red",
        "KeyId": "armory_key",
        "ToolClasses": [],
        "AllowTools": false,
        "RequireKeyAndTool": false,
        "ConsumeKey": false,
        "ToolDamage": 0,
        "KeySeconds": 5,
        "ToolSeconds": 60
      }
    }
  ]
}
```

Append this assignment to **Loot.json → Caches**. It reuses the bundled military_supplies item set, so no new ItemSets entry is needed:

```json
{
  "CacheId": "armory",
  "Sets": [
    { "SetId": "military_supplies", "Chance": 100, "MinRolls": 1, "MaxRolls": 1 }
  ]
}
```

Restart, use `!dc target` on a building, then `!dc addpos armory`. Without saved points, MaxActive=3 is only a cap and produces no caches. Use the explicit Theme: an unfamiliar category ID otherwise selects the civilian skin automatically.

The global MaxKeysPerInfected still applies. With a cap of 1, a successful earlier eligible key drop can prevent the armory roll from being attempted; 2% is not a guaranteed overall rate. To allow up to two successful keys on one infected, set **Settings.json → MaxKeysPerInfected** to 2. To make this key eligible on every infected, set AllInfected=true and InfectedClasses=[]; to stop only this key's infected drops, set DropEnabled=false. MaxKeysPerInfected=0 stops all infected key drops while keys in loot continue to work.

### 2. Require both a matching key and a carried tool

Replace the **Lock object of the military category in Settings.json** with this object. A usable military_key and Crowbar must both be carried. Holding the key gives a 5-second action; holding the Crowbar gives a 90-second action. Either successful route ruins the regular key and deals 40 absolute health points of damage to the Crowbar. Cancellation applies neither cost. The admin master key bypasses the paired requirement and remains pristine.

```json
{
  "KeyClass": "DC_Key",
  "KeyId": "military_key",
  "ToolClasses": ["Crowbar"],
  "AllowTools": true,
  "RequireKeyAndTool": true,
  "ConsumeKey": false,
  "ToolDamage": 40,
  "KeySeconds": 5,
  "ToolSeconds": 90
}
```

Setting RequireKeyAndTool=true with AllowTools=false, an empty ToolClasses list, or no key requirement is invalid. ToolDamage uses health points, not a percentage. Setting ConsumeKey=false does not prevent regular key ruin; it is a legacy field.

### 3. Tools-only caches with two accepted tools

Replace the **Lock object of the construction category in Settings.json**. Either exact Crowbar or SledgeHammer class can open it in 45 seconds, with 20 absolute health points of tool damage. Clearing both key fields removes the ordinary key route; clearing only KeyId would leave class-based matching enabled if KeyClass were nonempty.

```json
{
  "KeyClass": "",
  "KeyId": "",
  "ToolClasses": ["Crowbar", "SledgeHammer"],
  "AllowTools": true,
  "RequireKeyAndTool": false,
  "ConsumeKey": false,
  "ToolDamage": 20,
  "KeySeconds": 2,
  "ToolSeconds": 45
}
```

Each listed tool is an alternative; this does not require both tools. Tool subclasses are not automatically accepted. KeySeconds is still valid and used if an admin master key opens this cache.

### 4. Match a physical key class instead of a logical key ID

Replace the **Lock object of the civilian category in Settings.json**. Any non-ruined DC_Key_Blue opens it in 3 seconds, even if that item was spawned without a logical ID or was assigned a different one. Other regular key classes do not match. This requires no new Settings.Keys definition.

```json
{
  "KeyClass": "DC_Key_Blue",
  "KeyId": "",
  "ToolClasses": [],
  "AllowTools": false,
  "RequireKeyAndTool": false,
  "ConsumeKey": false,
  "ToolDamage": 0,
  "KeySeconds": 3,
  "ToolSeconds": 60
}
```

Use logical KeyId matching when two keys with the same physical class must open different categories. With a nonempty KeyId, the referenced key definition controls KeyClass/KeyName; manually changing those derived values will not change which logical key matches.

### 5. Mix categories and limit the total per building

This is **one entry in Caches.json → Buildings**, using the existing hospital and police category definitions. Merge Categories into an existing entry for this building class rather than duplicating BuildingClass. Replace these illustrative positions/orientations with points recorded in-game; their floor/clearance is not verified.

```json
{
  "BuildingClass": "Land_City_Hospital",
  "Categories": [
    {
      "CacheId": "hospital",
      "MaxPerBuilding": 2,
      "Positions": [
        { "Id": "slot_1", "Position": "1.2 0.4 -2.5", "Orientation": "90 0 0", "Chance": 100 },
        { "Id": "slot_2", "Position": "-1.5 0.4 2.0", "Orientation": "180 0 0", "Chance": 35 }
      ]
    },
    {
      "CacheId": "police",
      "MaxPerBuilding": 2,
      "Positions": [
        { "Id": "slot_1", "Position": "2.0 0.4 1.0", "Orientation": "0 0 0", "Chance": 50 }
      ]
    }
  ]
}
```

Each physical hospital can receive at most **two caches total**, not two per category. Reducing either applicable category's MaxPerBuilding to 1 reduces the combined cap to 1. Rules for disabled categories or categories with MaxActive=0 do not participate in the normal scan. An enabled rule can still constrain the building even if its Positions array is empty or none of its chances pass.

The three point chances are independent candidate rolls for each building instance. Passing a roll does not reserve a cache: accepted candidates are shuffled, then map-wide MaxActive, the combined building cap, existing occupancy and collision checks apply. A Chance=100 point can therefore remain empty. MaxActive counts caches across the map, including opened ones; opening does not free a slot for a replacement that session. There is no timed refill or periodic chance reroll.

Show-all ignores the random chances, Enabled and both count limits, but still skips points that fail floor/clearance checks. It creates temporary non-lootable previews; hide-all returns to normal gameplay.

### 6. Guaranteed supplies plus repeated bonus ammunition

In **Loot.json**, replace the existing military entry in Caches with the assignment below, and append both new ItemSets definitions. The wrapper identifies the arrays to merge; retain the other categories and existing sets.

```json
{
  "Caches": [
    {
      "CacheId": "military",
      "Sets": [
        { "SetId": "field_medical", "Chance": 100, "MinRolls": 1, "MaxRolls": 1 },
        { "SetId": "bonus_ammunition", "Chance": 40, "MinRolls": 1, "MaxRolls": 3 }
      ]
    }
  ],
  "ItemSets": [
    {
      "Id": "field_medical",
      "Items": [
        {
          "ClassName": "BandageDressing",
          "Chance": 100,
          "MinCount": 2,
          "MaxCount": 2,
          "Quantity": -1,
          "HealthMin": 1,
          "HealthMax": 1,
          "RandomHealth": false,
          "Attachments": []
        }
      ]
    },
    {
      "Id": "bonus_ammunition",
      "Items": [
        {
          "ClassName": "Mag_STANAG_30Rnd",
          "Chance": 50,
          "MinCount": 1,
          "MaxCount": 2,
          "Quantity": -1,
          "RandomQuantity": true,
          "QuantityMin": 10,
          "QuantityMax": 25,
          "HealthMin": 0.6,
          "HealthMax": 0.9,
          "RandomHealth": true,
          "Attachments": []
        }
      ]
    }
  ]
}
```

Provided cargo space is available, every opening attempts two pristine bandages. Separately, the ammunition set has a 40% activation chance. If activated, it repeats 1–3 times; on each repetition, the magazine entry independently has a 50% chance to attempt 1–2 magazines. Every magazine gets its own 10–25-round quantity and 60–90% health roll.

At most six magazines plus two bandages are attempted. Before cargo limits, the probability of at least one magazine attempt is about **28.33%**: 40% × the average of 50%, 75% and 87.5% for one, two and three repetitions. Multiplying 40% × 50%=20% would describe one repetition only. Set rolls are independent and can all pass; they are not weighted choices of one set/item.

The validator uses the worst-case count: 2×1 + 2×3 = 8 parent items. It ignores chances for this limit calculation. Keeping unused entries at Chance=0 does not exclude their MaxCount×MaxRolls contribution from the 500-item cap. Cargo overflow is logged and skipped without ground spill.

### 7. A weapon with direct attachments and fixed condition

Append this **item entry to the Items array of a referenced Loot.json item set**, such as military_supplies. Each successful 20% entry roll attempts one M4A1 at exactly 80% health, with the listed stock, handguard and optic attached.

```json
{
  "ClassName": "M4A1",
  "Chance": 20,
  "MinCount": 1,
  "MaxCount": 1,
  "Quantity": -1,
  "RandomQuantity": false,
  "QuantityMin": 0,
  "QuantityMax": 1,
  "HealthMin": 0.8,
  "HealthMax": 0.8,
  "RandomHealth": false,
  "Attachments": ["M4_OEBttstck", "M4_PlasticHndgrd", "M4_CarryHandleOptic"]
}
```

Attachments are direct class-name attempts on each parent copy. They do not use the parent's quantity/health settings, have no separate chance/count fields, and cannot contain nested attachments. To control magazine rounds explicitly, put a magazine in a separate loot item entry using the quantity settings from example 6. Quantity=-1 leaves initial quantity unchanged; it does not promise a loaded weapon or a filled item. RandomHealth=false uses HealthMin exactly; RandomQuantity=false uses Quantity. HealthMax and both quantity bounds must remain valid even when randomization is off.

### 8. Find the next cache key inside an earlier cache

Append this **item entry to Loot.json → ItemSets → medical_supplies → Items**. The existing hospital category rolls that set, giving a 10% key attempt per repetition. The spawned key is assigned military_key and its configured display name, so it opens the military category's ID-based lock.

```json
{
  "ClassName": "DC_Key",
  "KeyId": "military_key",
  "Chance": 10,
  "MinCount": 1,
  "MaxCount": 1,
  "Quantity": -1,
  "HealthMin": 1,
  "HealthMax": 1,
  "RandomHealth": false,
  "Attachments": []
}
```

For progression through cache loot only, set DropEnabled=false on the existing military_key definition in Settings.Keys. This stops its infected drops, not its loot entry. The example must use the exact physical ClassName from that key definition; changing it to DC_Key_Red without changing the definition is invalid. Omitting KeyId creates an unassigned key that does not match a military_key lock.

The regular key is ruined after successful opening, so subsequent military caches require another usable key. Do not include DC_AdminKey in progression loot: it is the reusable master key that bypasses all these locks.

### 9. Disable spawning, reserve a point, or produce an empty cache

| Goal | Edit | Result |
| --- | --- | --- |
| Stop one category without losing its positions | Settings.Caches entry: Enabled=false. | No normal spawning for it; show-all can still inspect its points. |
| Set a map-wide cap to zero | Settings.Caches entry: MaxActive=0. | No normal spawning for it. Existing JSON definitions remain available. |
| Reserve one saved point for later | Caches position: Chance=0. | Never selected normally; still visible in show-all if physical placement succeeds. |
| Keep a locked cache but roll no items | Its existing Loot.Caches entry: Sets=[]. | Cache can spawn/open with normal lock rules, but generates no loot. |
| Stop all infected key drops | Settings.MaxKeysPerInfected=0. | Keys can still be supplied through Loot.json. |

Manual changes take effect after restart; they do not remove already spawned caches during the running session. Keep a Loot.Caches assignment even for a disabled category. MaxPerBuilding=0 is invalid; use Enabled, MaxActive or point Chance instead.


## Administrator commands

Enter commands in ordinary chat. The server checks `AdminSteamIDs` against the authenticated sender. Commands and replies are private to the issuing administrator; spawned inspection caches are visible to nearby players.

| Command | Instructions |
| --- | --- |
| `!dc target` | Look at a building within 100m. Select it as your reference and save its class if absent, with no categories or positions added. |
| `!dc addpos <category>` | Stand where the cache should rest and face its intended direction. Create that category under the selected class if absent, or append to its existing points. Example: `!dc addpos hospital`. The category must already be defined in Settings/Loot. |
| `!dc list` | List all building classes, categories and positions privately, with local position/orientation/chance and one-based global numbers. Buildings/categories are alphabetically grouped. |
| `!dc delpos <number>` | Delete a point using the number from your latest list. Example: `!dc delpos 3`. Any successful edit by any administrator invalidates old list numbers; run list again. |
| `!dc clear all` | Clear all saved points across every category under your targeted building class. Stay within 100m of the target. Retain the class and its categories. This affects every building of that class. |
| `!dc undo` | Reverse the latest successful add, deletion or clear. Repeat to continue backwards. History is shared among administrators until server restart, and undo does not depend on your current target. Failed edits and empty clears do not add history; failed undo does not consume history. Targeting/read-only commands do not add history. |
| `!dc show all` | Scan all matching buildings and show configured points, bypassing Enabled, Chance and normal active/building limits. Physical floor/clearance checks still apply. Blocked points are skipped, counted in the reply and identified in the server log. |
| `!dc hide all` | Cancel a pending inspection and remove temporary preview caches. Normal caches retain their loot and unlocked state. |
| `!dc help` | Show command syntax only. |
| `!dc categories` | Show configured category IDs only. |

Each command runs only its own handler. Commands are case insensitive and accept only the argument counts shown. Unknown commands, bare `!dc`, missing arguments and extra arguments return `Command does not exist.` Valid commands with invalid category/position/target values return the relevant error. Use lower-case, single-word category IDs so they can be entered in chat.

Position edits and undo require a living administrator and completed scanning, with inspection hidden. Use `!dc hide all` before editing. Target selection and numbered lists are separate for each administrator and reset on reconnect; undo history stays with the running server. After undo, run `!dc list` again before deleting by number.

Example workflow: target a hospital, stand at the desired point, run `!dc addpos hospital`, and repeat at further points. Inspect with `!dc list` and `!dc show all`. Hide previews before adding/deleting/clearing/undoing. Temporary caches are named `Preview: <cache name>` and cannot be opened or looted. An existing gameplay cache at the same category/point represents that point without another preview.

## Installation

1. Compile the source with addon prefix DynamicCaches. Include config.cpp, scripts, assets and mod metadata. Enable model binarization and keep assets/models/model.cfg beside both editable model sources; their Geometry autocenter=0 preserves the bottom origin.
2. Install matching copies on server and clients. Load Community Framework and Dynamic Caches with `-mod=@CF;@DynamicCaches`. VPP and mission init.c changes are not required.
3. Start once to create missing profile configuration files. Stop, add your account's 17-digit Steam64 ID string to Settings.AdminSteamIDs, then restart. Replace this example with your own ID; an empty array authorizes nobody:

```json
{
  "AdminSteamIDs": ["76561198000000001"]
}
```

4. Record positions with target/addpos and inspect them with show-all. All three profile files are separate from this README in the mod root.


## Troubleshooting

| Symptom | Check |
| --- | --- |
| Commands have no reply | Correct 17-digit AdminSteamIDs strings; server restarted after changing Settings; matching addon and CF loaded on server/client. |
| `Command does not exist.` | Use exactly the command/argument count in the command table. Use help or categories explicitly. |
| Category does not exist | Exact lower-case category ID in Settings.Caches, matching Loot.Caches entry, and restart after manual definition edits. Category names and skin names are different settings. |
| No normal caches | Fresh Caches.json is empty. Add positions; check Enabled, MaxActive, Chance, combined MaxPerBuilding, occupied points, collision rejection and configuration errors in the log. |
| Some show-all points are missing | Check its blocked-placement count and the logged building/category/slot. Correct the saved location or select a model that fits. |
| Key does not work | Non-ruined key; matching assigned logical KeyId, or exact KeyClass when no logical ID is required. Check both carried requirements if RequireKeyAndTool=true. |
| Less loot than expected | Set Chance, repetitions, item Chance/counts, valid classes, quantity capacity and cargo space. Skipped item/attachment attempts are logged. |
| Cannot delete a number | Run list again after any edit/undo or reconnect. Numbers are global within that administrator's latest list, not point IDs. |
| Nothing to undo | No successful position edits remain in this server session, or the server has restarted. |
| Cannot edit during inspection | Hide all, then wait for scanning/spawning to finish. |

Back up the three profile JSON files before changing IDs or restructuring definitions. After manual edits, restart and check the server log, then inspect saved points with show-all and hide-all. Example coordinates illustrate the format; record suitable floors and clearance in-game before using them for normal spawning.
