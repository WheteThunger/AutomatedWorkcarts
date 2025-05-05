## Features

- Allows automating trains with NPC conductors
- Allows placing triggers to instruct conductors how to navigate
- Allows placing spawn points, which spawn workcarts with optional train wagons
- Allows disabling default underground workcart spawn points
- Allows configuring automated train engine force and speed
- Allows automated trains to honk at nearby players
- Optional default triggers for underground tunnels
- Optional map markers for automated trains and stops
- API for creating addon plugins

## Introduction

The primary feature of this plugin is to add "triggers" onto train tracks. A trigger is an invisible object which can detect when a train comes into contact with it, in order to perform various functions, including the following.

- Add a conductor to the train
- Change the speed, direction and track selection of the train
- Temporarily stop the train for a configurable period of time
- Run custom server commands
- Destroy the train

All of the above functions, except adding a conductor, only apply to trains that have a conductor, meaning that player-driven trains can pass through most triggers unaffected.

A trigger can also serve as a workcart spawn point, with an optional number of attached wagons and workcarts.

### How conductors work

Automating a train will add a conductor NPC to every workcart attached to that train, automatically dismounting any players currently driving them.

- Conductors are invincible
- All workcarts and wagons that are part of an automated train are invincible
- Automated workcarts do not require fuel
- Automated trains cannot be coupled to other trains, nor uncoupled

### How triggers work

Each trigger can have multiple properties, including direction (e.g., `Fwd`, `Rev`, `Invert`), speed (e.g, `Hi`, `Med`, `Lo`, `Zero`), track selection (e.g., `Default`, `Left`, `Right`), stop duration (in seconds), and departure speed/direction. When a conductor-driven train touches a trigger, its instructions may change according to the trigger properties. For example, if a train is currently following the instructions `Fwd`, `Hi`, `Left`, and then touches a trigger that specifies only track selection `Right`, the train instructions will change to `Fwd`, `Hi`, `Right`, causing the train to turn right at every intersection it comes to, until it touches a trigger that instructs otherwise.

### Types of triggers

#### Map specific triggers

- Can be placed anywhere on train tracks, above or below ground.
- Only apply to the map they were placed on, using world coordinates.
- Enabled via the `Enable map triggers` configuration option.
- Added with the `aw.addtrigger` or `awt.add` command.
- Saved in data file: `oxide/data/AutomatedWorkcarts/MAP_NAME.json`.
  - Note: The file name for non-procedural maps will exclude the wipe number so that you can re-use the triggers across force wipes.

#### Tunnel triggers

- Can be placed only in the vanilla train tunnels.
- Automatically replicate at all tunnels of the same type, using tunnel-relative coordinates.
- Enabled via the `Enable tunnel triggers` -> `*` options.
- Added with the `aw.addtunneltrigger` or `awt.addt` command.
- Saved in data file `oxide/data/AutomatedWorkcarts/TunnelTriggers.json`.

## Tutorials

### Tutorial: Automate all underground workcarts

The plugin provides default triggers which you can enable for underground tunnels. This setup should take only a few minutes.

1. Set `Enable tunnel triggers` -> `TrainStation` to `true` in the plugin configuration.
2. Set `Enable tunnel triggers` -> `VerticalIntersection` to `true` in the plugin configuration.
3. Reload the plugin.

This will place `Conductor` triggers on the vanilla workcart spawn points in the train station maintenance tunnels, which will automatically add conductors to the workcarts when they spawn. The workcarts will drive forward at max speed and turn left at all intersections. This will also place `Brake` triggers at the train stations, which will cause trains to automatically stop briefly near the elevators.

From the map perspective, each workcart will move in a counter-clockwise circle around an adjacent loop, or in a clockwise circle around the outer edges of the map, depending on which maintenance tunnel the workcart spawned in and the available nearby loops. This combination will cover most maps, allowing players to go almost anywhere by switching directions at various stops.

To see the triggers visually, grant the `automatedworkcarts.managetriggers` permission and run the `awt.show` command. For 60 seconds, this will show triggers at nearby tunnels. From here, you can add, update, move, and remove triggers to your liking. See the Commands section for how to manage triggers.

The following screenshot depicts a config with the following properties, with the trains having just spawned a moment ago.

- All default tunnel triggers enabled
- Vending machine and colored map markers enabled for trains, using dynamic route colors
- Colored map markers enabled for train stops, using dynamic route colors

![](https://raw.githubusercontent.com/WheteThunger/AutomatedWorkcarts/master/Images/DynamicRouteColors.png)

### Tutorial: Automate aboveground workcarts

Note: These instructions were originally written for custom maps and may not be suitable for procedurally generated aboveground tracks since trains spawn somewhat randomly. Instead, see the section below or creating spawn points.

1. If you are a map developer, please see the "Tips for map developers" section below. Designing the tracks sensibly will simplify setting up automated routes.
2. Carefully examine the tracks on your map to determine the route(s) you would like workcarts to use.
3. Grant yourself the `automatedworkcarts.managetriggers` permission.
4. Find a train spawn location where you would like trains to automatically receive conductors.
5. Aim at the track and run the command `awt.add Conductor Fwd Hi`. Any train that spawns on this trigger will automatically receive a conductor, and start driving forward at max speed.
6. Find a portion of track where you want automated workcarts to stop briefly.
7. Aim at the track and run the command `awt.add Brake Zero 15 Hi`. Any train that passes through this trigger will brake until stopping, wait for `15` seconds, then start moving the same direction at `Hi` speed.
8. Keep adding/editing triggers and spawning workcarts to refine the routes.

### Tutorial: Create spawn points

Workcarts can be spawned via spawn triggers. The following steps will walk you through an example to get you started.

1. Grant yourself the `automatedworkcarts.managetriggers` permission.
2. Aim somewhere on train tracks, and run the command `awt.add Workcart` (or `awt.addt Workcart` for a tunnel-relative trigger). A workcart should spawn there immediately (if there is sufficient space) but won't have a conductor.
3. If the workcart is facing the wrong direction, aim at the trigger while facing approximately the direction you want the workcart to face, then run `awt.rotate`.
4. To move the spawn point, aim where you want to move it to, and run `awt.move <id>`, where `<id>` should be replaced with the trigger id which you can see floating above the trigger (it will have a `#`).
5. To add wagons to the spawn point, aim at the trigger and run the command `awt.train Workcart WagonA WagonB WagonC Workcart`. This example command will add multiple wagons, with an additional workcart at the end. You can add as many wagons and workcarts as you want, in any order, as long as there is enough space for them on the tracks.
6. If you want to add a conductor to the train, aim at the trigger and run `awt.update Conductor Fwd Hi` then `awt.respawn`.

Note: If the train is destroyed, it will be respawned up to 30 seconds later. At most one train can be spawned by a given spawn point at a time. If you want multiple trains, you will need to create multiple spawn points.

## Permission

- `automatedworkcarts.toggle` -- Allows usage of the `aw.toggle` and `aw.resetall` commands.
- `automatedworkcarts.managetriggers` -- Allows viewing, adding, updating and removing triggers.

## Commands

### Toggle automation of individual workcarts

- `aw.toggle @<optional_route_name>` -- Toggles automation of the train you are looking at.
  - If the route name is specified, the train will respond to both global triggers (i.e., triggers that do not specify a route) and triggers assigned to that route. The train will ignore triggers assigned other routes.
  - If the route name is **not** specified, the train will respond only to global triggers.
  - The train will start driving according to `DefaultSpeed` and `Default track selection` in the plugin configuration.
- `aw.resetall` -- Resets all automated trains to normal. This removes all conductors. **Exception:** Trains spawned by conductor triggers will keep their conductors.

### Manage triggers

- `aw.showtriggers @<optional_route_name> <optional_duration_in_seconds>` (alias: `awt.show`) -- Shows all nearby triggers to the player for specified duration. Defaults to 60 seconds.
  - This displays each trigger's id, speed, direction, etc.
  - Triggers are also automatically shown for at least 60 seconds when using any of the other trigger commands or when manually automating a workcart.
  - When specifying a route name, global triggers and triggers matching that route will be shown with their default colors, but triggers for different routes will be colored gray.
- `aw.addtrigger <option1> <option2> ...` (alias: `awt.add`) -- Adds a trigger to the track position where you are aiming, with the specified options. Automated trains that pass through the trigger will be affected by the trigger's options.
  - Speed options: `Hi` | `Med` | `Lo` | `Zero`.
  - Direction options: `Fwd` | `Rev` | `Invert`.
  - Track selection options: `Default` | `Left` | `Right` | `Swap`.
  - Train car options: `Locomotive` | `Sedan` | `Workcart` | `WorkcartCovered` | `WagonA` | `WagonB` | `WagonC` | `WagonFuel` | `WagonLoot` | `WagonResource` | `Caboose`
  - Other options:
    - `Conductor` -- Adds a conductor to the train if not already present. A good place to put `Conductor` triggers is on workcart spawn locations, such as the vanilla spawns in the underground maintenance tunnels. This option can also be combined with the train car options to automatically spawn an automated train.
      - Note: Player-owned trains cannot receive conductors. Vanilla trains don't have owners, but most plugins that spawn vehicles for players will assign ownership by setting the `OwnerID` property of the vehicle.
    - `Brake` -- Instructs the train to brake until it reaches the designated speed. For example, if the train is going `Fwd_Hi` and enters a `Brake Med` trigger, it will temporarily go `Rev_Lo` until it slows down enough, then it will go `Fwd_Med`.
    - `Destroy` -- Destroys the train. This is intended primarily for testing and demonstrations, but it's also useful if you don't want to be bothered to make a more thoughtful track design.
    - `@<route_name>` -- Instructs the train to ignore this trigger if it's not assigned this route (replace `<route_name>` with the name you want).
      - If the trigger has the `Conductor` property and the train does not already have a conductor, it will be assigned this route.
    - `<chance>%` -- Makes the trigger have only a percent chance to affect a train. Does not affect the chance of a train receiving a conductor.
    - `Enabled` -- Enables the trigger.
    - `Disabled` -- Disables the trigger. Disabled triggers are ignored by trains and are colored gray.
- `aw.addtrunneltrigger <option1> <option2>` (alias: `awt.addt`) -- Adds a trigger to the track position where you are aiming.
  - Options are the same as for `aw.addtrigger`.
  - Must be in a supported train tunnel (one enabled in plugin configuration).
  - This trigger will be replicated at all train tunnels of the same type. Editing or removing one of those triggers will affect them all.
- `aw.updatetrigger <id> <option1> <option2> ...` (alias: `awt.update`) -- Adds or updates properties of the specified trigger, without removing any properties.
  - Options are the same as for `aw.addtrigger`.
- `aw.replacetrigger <id> <option1> <option2> ...` (alias: `awt.replace`) -- Replaces all properties of the specified trigger with the properties specified.
  - Options are the same as for `aw.addtrigger`.
  - This is useful for removing properties from a trigger (without having to remove and add it back) since `aw.updatetrigger` does not remove properties.
- `aw.movetrigger <id>` (alias: `awt.move`) -- Moves the specified trigger to the track position where the player is aiming.
- `aw.rotatetrigger <id>` (alias: `awt.rotate`) -- Rotates the specified trigger to where the player is facing. This is only useful for spawn triggers since it determines which way the train will be facing when spawned. This command will automatically respawn the train if the rotation has been reversed.
- `aw.removetrigger <id>` (alias: `awt.remove`) -- Removes the specified trigger.
- `aw.enabletrigger <id>` (alias: `awt.enable`) -- Enables the specified trigger. This is identical to `aw.updatetrigger <id> enabled`.
- `aw.disabletrigger <id>` (alias: `awt.disable`) -- Disables the specified trigger. This is identical to `aw.updatetrigger <id> disabled`. Disabled triggers are ignored by trains and are colored gray.
- `aw.settriggertrain <id> <wagon1> <wagon2> ...` (alias: `awt.train`) -- Assigns zero or more train cars to the trigger, replacing the current list of train cars, causing this trigger to automatically spawn a train with the specified workcarts and wagons, connected in order. The wagons will be automatically coupled to the workcart. Allowed values: `Locomotive`, `Sedan`, `Workcart`, `WorkcartCovered`, `WagonA`, `WagonB`, `WagonC`, `WagonFuel`, `WagonLoot`, `WagonResource`, `Caboose`. To remove all train cars, run the command without specifying any train cars.
- `aw.respawntrigger <id>` (alias: `awt.respawn`) -- Despawns and respawns the train for the specified trigger. When used at a tunnel trigger, trains will be respawned at all instances of the tunnel trigger.
- `aw.addtriggercommand <id> <command>` (alias: `awt.addcmd`) -- Adds the specified command to the trigger. which will be executed whenever a train passes through the trigger. You can use the magic variable `$id` in the command which will be replaced by the primary workcart's Net ID.
- `aw.removetriggercommand <id> <number>` (alias: `awt.removecmd`) -- Removes the specified command from the trigger. The command number will be 1, 2, 3, etc. and will be visible on the trigger info when using `aw.showtriggers`.

Tip: For the commands that update, move or remove triggers, you can skip the `<id>` argument if you are aiming at a nearby trigger.

### Command examples

Simple examples:

- `awt.add Lo` -- Causes the train to move in its **current direction** at `Lo` speed. Example: `Fwd_Hi` -> `Fwd_Lo`.
- `awt.add Fwd Lo` -- Causes the train to move **forward** at `Lo` speed, regardless of its current direction. Example: `Rev_Hi` -> `Fwd_Lo`.
- `awt.add Invert` -- Causes the train to **reverse direction** at its **current speed**. Example: `Fwd_Med` -> `Rev_Med`.
- `awt.add Invert Med` -- Causes the train to **reverse direction** at `Med` speed. Example: `Fwd_Hi` -> `Rev_Med`.
- `awt.add Brake Med` -- Causes the train to **brake** until it reaches `Med` speed. Example: `Fwd_Hi` -> `Rev_Lo` -> `Fwd_Med`.
- `awt.add Left` -- Causes the train to turn **left** at all future intersections.

Advanced examples:

- `awt.add Conductor Fwd Hi Left` -- Causes the train to automatically receive a **conductor**, to move **forward** at `Hi` speed, and to always turn **left**.
- `awt.add Brake Zero 10 Hi` -- Causes the train to **brake** to a **stop**, wait 10 seconds, then move `Hi` speed in the **same direction**.
- `awt.add Zero 20 Med` -- Causes the train to **turn off** its engine for `20` seconds (slowly rolling to a stop), then move the **same direction** at `Med` speed.

Route examples:

- `awt.add @Route1 Conductor Fwd Hi Left` -- Causes the train to automatically receive a **conductor**, to move **forward** at `Hi` speed, to always turn **left**, and to **ignore triggers assigned to other routes** (i.e., routes not named `Route1`).
- `awt.add @Route1 Left` -- Causes the train to turn **left** at all future intersections, **only** if the train is assigned the `Route1` route.

Update examples:

- `awt.update 60` -- Updates the trigger's stop duration to `60` seconds.
- `awt.update Left` -- Updates the trigger's track selection to `Left`.
- `awt.update Fwd Hi` -- Updates the trigger's direction to `Fwd`, and speed to `Hi`.
- `awt.update @Route2` -- Updates the trigger's route to `Route2`.
- `awt.update 60 Left Fwd Hi @Route2` -- Applies all of the above updates to the trigger at once.

Spawn examples:

- `awt.add Workcart` -- Creates a trigger that will spawn a workcart.
- `awt.train Workcart WagonA WagonB` -- Updates the trigger to spawn a workcart with one `WagonA` and one `WagonB` wagon coupled behind it.
- `awt.train Workcart WagonC WagonC` -- Updates the trigger to spawn a workcart with two `WagonC` wagons coupled behind it.
- `awt.train Workcart Workcart` -- Updates the trigger to spawn two workcarts coupled together.
- `awt.train Locomotive Workcart WagonA WagonB WagonC WagonFuel WagonLoot WagonResource Caboose WorkcartCovered` -- Updates the trigger to spawn a train with all possible train car types coupled behind it.

All-in-one examples:

- `awt.add Conductor Brake Zero 10 Fwd Hi Left Workcart WagonA WagonB` -- Creates a trigger that will spawn a workcart with one `WagonA` and one `WagonB` wagon coupled behind it. It will have a conductor, and will initially wait 10 seconds before driving forward at `Hi` speed. On return visits, it will brake to a stop and wait 10 seconds before continuing.

## Configuration

Default configuration:

```json
{
  "Play horn for nearby players in radius": 0.0,
  "Default speed": "Fwd_Hi",
  "Default track selection": "Left",
  "Bulldoze offending workcarts": false,
  "Destroy barricades instantly": false,
  "Enable map triggers": true,
  "Enable tunnel triggers": {
    "TrainStation": false,
    "BarricadeTunnel": false,
    "LootTunnel": false,
    "Intersection": false,
    "LargeIntersection": false,
    "VerticalIntersection": false
  },
  "Max conductors": -1,
  "Spawn triggers respect conductor limit": false,
  "Disable default tunnel workcart spawn points": false,
  "Trigger display distance": 150.0,
  "Conductor outfit": [
    {
      "ShortName": "jumpsuit.suit",
      "Skin": 0
    },
    {
      "ShortName": "sunglasses03chrome",
      "Skin": 0
    },
    {
      "ShortName": "hat.boonie",
      "Skin": 0
    }
  ],
  "Automated train engine overrides": {
    "assets/content/vehicles/sedan_a/sedanrail.entity.prefab": {
      "Enable engine overrides": false,
      "Override max speed": 44.0,
      "Override engine force": 10000.0
    },
    "assets/content/vehicles/trains/locomotive/locomotive.entity.prefab": {
      "Enable engine overrides": false,
      "Override max speed": 20.0,
      "Override engine force": 250000.0
    },
    "assets/content/vehicles/trains/workcart/workcart.entity.prefab": {
      "Enable engine overrides": false,
      "Override max speed": 21.0,
      "Override engine force": 33000.0
    },
    "assets/content/vehicles/trains/workcart/workcart_aboveground.entity.prefab": {
      "Enable engine overrides": false,
      "Override max speed": 18.0,
      "Override engine force": 30000.0
    },
    "assets/content/vehicles/trains/workcart/workcart_aboveground2.entity.prefab": {
      "Enable engine overrides": false,
      "Override max speed": 17.0,
      "Override engine force": 30000.0
    }
  },
  "Map markers": {
    "Train map markers": {
      "Map marker update interval seconds": 5.0,
      "Colored map marker": {
        "Enabled": false,
        "Color": "#00ff00",
        "Alpha": 1.0,
        "Radius": 0.05,
        "Use dynamic route color": false
      },
      "Vending map marker": {
        "Enabled": false,
        "Name": "Automated Train"
      }
    },
    "Train stop map markers": {
      "Display only while stop is reachable": false,
      "Colored map marker": {
        "Enabled": false,
        "Color": "#ff9900",
        "Alpha": 1.0,
        "Radius": 0.1,
        "Use dynamic route color": false
      },
      "Vending map marker": {
        "Enabled": false,
        "Name": "Train Stop"
      }
    },
    "Dynamic route colors": [
      "#ff0000",
      "#ff9900",
      "#ffff00",
      "#00ff00",
      "#0099ff",
      "#cc00ff",
      "#ffffff",
      "#777777"
    ]
  }
}
```

- `Play horn for nearby players in radius` -- Distance from nearby players required to automatically play the train horn. Set to `0.0` to disable the horn.
- `Default speed` -- Default speed to use when a train starts being automated.
  - Allowed values: `"Rev_Hi"` | `"Rev_Med"` | `"Rev_Lo"` | `"Zero"` | `"Fwd_Lo"` | `"Fwd_Med"` | `"Fwd_Hi"`.
  - This value is ignored if the train is on a trigger that specifies speed.
- `Default track selection` -- Default track selection to use when a train starts being automated.
  - Allowed values: `"Left"` | `"Default"` | `"Right"`.
  - This value is ignored if the train is on a trigger that specifies track selection.
- `Bulldoze offending workcarts` (`true` or `false`) -- While `true`, automated trains will destroy other non-automated trains in their path.
- `Destroy barricades instantly` (`true` or `false`) -- While `true`, automated trains will destroy train barricades instantly, though the train may still slow down.
  - Regardless of this setting, automated trains may destroy each other in head-on or perpendicular collisions.
- `Enable map triggers` (`true` or `false`) -- While `false`, existing map-specific triggers will be disabled, and no new map-specific triggers can be added.
- `Enable tunnel triggers` -- While `false` for a particular tunnel type, existing triggers in those tunnels will be disabled, and no new triggers can be added to tunnels of that type.
  - `TrainStation` (`true` or `false`) -- Self-explanatory.
  - `BarricadeTunnel` (`true` or `false`) -- This affects straight tunnels that spawn NPCs, loot, as well as barricades on the tracks.
  - `LootTunnel` (`true` or `false`) -- This affects straight tunnels that spawn NPCs and loot.
  - `Intersection` (`true` or `false`) -- This affects 3-way intersections.
  - `LargeIntersection` (`true` or `false`) -- This affects 4-way intersections.
- `Max conductors` -- The maximum number of automated trains allowed on the map at once. Set to `-1` for no limit. Note that having multiple automated workcarts on a single train will count as only one conductor. **Exception:** Trains spawned by conductor triggers are not subject to the conductor limit, nor do they count toward the conductor limit, unless `Spawn triggers respect conductor limit` is set to `true`.
- `Spawn triggers respect conductor limit` (`true` or `false`) -- While `true`, trains spawned by conductor triggers are subject to the conductor limit.
- `Disable default tunnel workcart spawn points` (`true` or `false`) -- While `true`, vanilla workcart spawn points at train station maintenance tunnels will be disabled, and any existing workcarts spawned by those spawn points will be destroyed when the plugin loads. This enables you to create custom spawn points in the same location, allowing you to use alternative train engines plus optional wagons.
- `Trigger display distance ` -- Determines how close you must be to a trigger to see it when viewing triggers (e.g., after running `awt.show`).
- `Conductor outfit` -- Items to use for the outfit of each conductor.
- `Automated train engine overrides` -- Configure engine settings for automated trains, by prefab path.
  - Notes:
    - Non-automated trains are not affected by these settings.
    - It is not currently possible to override engine settings for specific triggers or spawn points.
    - The `workcart` prefab is used for underground spawn points.
    - When you install the plugin, it will automatically discover and add train engine prefabs to the config. If you ever want to reset the values to vanilla, simply delete this section from the config and reload the plugin. The default values in the documentation were from a specific point in time, so they might not be the current vanilla values.
  - Each prefab has the following options.
    - `Enable engine overrides` (`true` or `false`) -- Whether to override engine settings for this prefab.
    - `Override max speed` -- The maximum speed of the train (probably in meters/second).
    - `Override engine force` -- The engine force of the train. This affects acceleration and braking deceleration.
- `Map markers`
  - `Train map markers` -- Configure map markers for automated trains.
    - `Map marker update interval seconds` -- The number of seconds between map marker updates. Updating the map markers periodically for many trains can impact performance, so you may adjust this value to trade off between accuracy and performance.
    - `Colored map marker`
      - `Enabled` (`true` or `false`) -- Whether to enable colored map markers for automated trains. Enabling this has a performance cost.
      - `Color` -- The marker color, using the hexadecimal format popularized by HTML.
      - `Alpha` (`0.0` - `1.0`) -- The marker transparency (`0.0` is invisible, `1.0` is fully opaque).
      - `Radius` -- The marker radius.
      - `Use dynamic route color` (`true` or `false`) -- While `true`, instead of using the specific color configured above, the color will be assigned according to the distinct route that each train is taking, determined by which unique set of triggers the train is predicted to interact with. The color will be assigned from the list of `Dynamic route colors`.
  - `Vending map marker`
    - `Enabled` (`true` or `false`) -- Whether to enable vending machine map markers for automated trains. Enabling this has a performance cost.
    - `Name` -- The name to display when hovering the mouse over the marker.
  - `Train stop map markers` -- Configure map markers for automated train stops. Any trigger with speed `Zero` is considered a stop.
    - `Display only while stop is reachable` (`true` or `false`) -- While `true`, train stops will only display on the map if it is predicted that a currently automated train will reach the trigger at some point. While `false`, all train stops will always be shown on the map.  
    - `Colored map marker`
      - `Enabled` (`true` or `false`) -- Whether to enable colored map markers.
      - `Color` -- The marker color, using the hexadecimal format popularized by HTML.
      - `Alpha` (`0.0` - `1.0`) -- The marker transparency (`0.0` is invisible, `1.0` is fully opaque).
      - `Radius` -- The marker radius.
      - `Use dynamic route color` (`true` or `false`) -- While `true`, the color will be assigned according to the distinct route that each train is taking, determined by which unique set of triggers the train is predicted to interact with. The color will be assigned from the list of `Dynamic route colors`. If multiple distinct train routes touch this trigger, the color for only one of those routes will be chosen.
    - `Vending map marker`
      - `Enabled` (`true` or `false`) -- Whether to enable vending machine map markers for train stops.
      - `Name` -- The name to display when hovering the mouse over the marker.
  - `Dynamic route colors` -- List of colors that can be applied to automated train and train stop map markers, if `Use dynamic route color` is enabled for them. If there are more distinct routes than colors, colors will be reused for multiple routes.

## Routes (advanced)

Routes are an advanced feature intended for complex aboveground tracks. Using routes allow trains to respond to only select triggers by ignoring triggers designated for other routes. This is useful for situations where multiple trains need to pass through shared tracks but exit those tracks in different directions.

#### Do I need routes?

Probably not. The routes feature allows solving some uses cases that simply aren't possible using only global triggers. However, many use cases that you might think you need routes for are actually achievable using global triggers.

Here are some example ways you can avoid needing to use the routes feature.

- Design the map to reduce track sharing to a minimum, and simplify shared track sections to remove unnecessary branches where you don't want automated trains to go.
- Place track selection triggers before shared track sections. These triggers should assign different track selections to trains coming from different source tracks. As long as the shared track sections do not require the trains to change tracks, this allows the trains to exit the shared tracks in different directions.
- Place track selection triggers with the `Swap` instruction to designate that a train should flip its track selection between `Left` and `Right`.

#### How to assign routes

A train can only be assigned a route when it receives a conductor. This can be done one of two ways.

- By running the `aw.toggle @<route_name>` command while aiming at the train. When this command adds a conductor, the train will be assigned the specified route.
- When a train receives a conductor via a trigger, if the trigger also has an assigned route, the train will be assigned that route.

#### How trains respond to triggers when using routes

- Triggers **with** an assigned route will affect **only** trains assigned the same route.
- Triggers **without** an assigned route will affect **all** trains.

## FAQ

#### Will this plugin cause lag?

This plugin's logic is optimized for performance and should not cause lag. However, trains moving along the tracks does incur some overhead, regardless of whether a player or NPC is driving them. Therefore, having many automated trains may reduce both client and server FPS. One way to address this is to limit the number of automated trains with the `Max conductors` configuration option.

#### Is this compatible with the Cargo Train Event plugin?

Generally, yes. However, if all trains are automated, the Cargo Train Event will never start since it needs to select an idle workcart, so it's recommended to limit the number of automated trains using the `Max conductors` configuration option, and/or to automate only specific trains based on their spawn location using triggers.

#### Is it safe to allow player trains and automated trains on the same tracks?

The **best practice** is to have separate, independent tracks for player vs automated trains. However, automated trains do have collision handling logic that makes them somewhat compatible with player tracks.

- When an automated train is rear-ended, if it's currently stopping or waiting at a stop, it will depart early.
- When an automated train collides with another train in front of it, its engine stops for a few seconds to allow the forward train to get some distance. This is especially useful for intersections since it allows one train to attempt passage while the other backs off.
- When two automated trains collide head-on, the slower one (or a random one if they are going the same speed) will explode.
- When an automated train collides with a non-automated train in a manner other than a rear-end, having the `Bulldoze offending workcarts` configuration option set to `true` will cause the non-automated train to be destroyed.

## Tips for map developers

- Design the map with automated routes in mind.
- Avoid dead-ends on the main routes.
  - While these can work, they may require additional effort to design automated routes, due to increased collision potential.
- Create parallel tracks throughout most of the map.
  - This allows trains to move in both directions, similar to the vanilla underground tracks.
- Create frequent alternate tracks.
  - Create alternate tracks at intended stop points to allow player-driven trains to easily pass automated trains at designated stops.
  - Create alternate tracks throughout the map to provide opportunities for player-driven trains to avoid automated trains.
- Create completely independent tracks.
  - For example, an outer circle and an inner circle.
  - This allows users of this plugin to selectively automate only specific areas, while allowing other areas to have player-driven trains, therefore avoiding interactions between the two.
- For underground tunnels, ensure each "loop" has at least two train stations (or other stops). This works best with the plugin's default triggers since it allows players to travel anywhere with automated trains by switching directions at various stops.
- If distributing your map, use this plugin to make default triggers for your users, and distribute the json file with your map.
  - The file can be found in `oxide/data/AutomatedWorkcarts/MAP_NAME.json`.

## Localization

## Developer API

#### API_AutomateWorkcart

```csharp
bool API_AutomateWorkcart(TrainEngine trainEngine)
```

Automates the specified train engine, along with all train engines attached to the same train. Returns `true` if successful, or if already automated. Returns `false` if it was blocked by another plugin.

#### API_StopAutomatingWorkcart

```csharp
void API_StopAutomatingWorkcart(TrainEngine trainEngine)
```

Stops automating the specified train engine, along with all train engines attached to the same train.

#### API_IsWorkcartAutomated

```csharp
bool API_IsWorkcartAutomated(TrainEngine trainEngine)
```

Returns `true` if the given train engine is automated, else `false`.

#### API_GetAutomatedWorkcarts

```csharp
TrainEngine[] API_GetAutomatedWorkcarts()
```

Returns an array of all train engines that are currently automated.

## Developer Hooks

#### OnWorkcartAutomationStart

```csharp
bool? OnWorkcartAutomationStart(TrainEngine trainEngine)
```

- Called when a train engine is about to become automated
- Returning `false` will prevent the train engine from becoming automated
- Returning `null` will result in the default behavior

#### OnWorkcartAutomationStarted

```csharp
void OnWorkcartAutomationStarted(TrainEngine trainEngine)
```

- Called after a train engine has become automated
- No return behavior

#### OnWorkcartAutomationStopped

```csharp
void OnWorkcartAutomationStopped(TrainEngine trainEngine)
```

- Called after a train engine has stopped being automated
- No return behavior
