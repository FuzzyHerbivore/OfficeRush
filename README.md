[![Office Rush '95](./office_rush_logo.png)](https://s4g.itch.io/office-rush)

# Office Rush '95

**Office Rush '95 is a 3D, local multiplayer game, where your goal is to become employee of the month. Get more points than your opponent by working harder at fulfilling tasks, or by preventing your opponent from cashing in points by sabotaging them.**

This [Unreal](https://unrealenginge.com/) game was created as part of the third semester project at [S4G School for Games](https://www.school4games.net/) in 10 weeks, from December 2025 to February 2026.

[Download it from itch.io!](https://s4g.itch.io/barque-of-ra)


## Contributions

Engineering consisted of two people, my colleague Patric took care of all the mini-games, while I focused on the in-office gameplay like the interactions between players and NPCs, and systems around the doors and tasks.

Due to the fact that we used Blueprints to realize the game, many other aspects that engineers needed to at least be part of in previous projects were done by game designers and artists themselves.


## Noteworthy Aspects

### [Shove Mechanic](./Content/Features/Shoving/)

Players can shove each other to gain an advantage in the game, especially when managing to shove their opponent into one of the talkative NPCs standing around in the office (see [NPC stalling](./Content/Features/NPCs/)).

The implementation of the physical aspects of the shove is rather straight-forward, but the mechanic is an example of the architecture I followed:
- Actors implement a Blueprint Interface that defines the API of the desired feature to avoid tight coupling
- Actors delegate generalizable core functionality of features to Actor Components to increase reusability and reduce checkout conflicts in big Actor Blueprints
- Calls from other Actors go through the Actor that implements the interface and delegates to Actor Components (instead of calling methods in the AC directly) to allow for incorporating information from other features/components and react accordingly
- Proactive objects message reactive objects about intent, but the reactive object implements functionality, to allow for uniformity in interfaces while allowing for individuality in reactions

In the context of the shove mechanic, [BP_PlayerCharacter](./Content/Players/BP_PlayerCharacter.uasset) implements the `GetShoved` method required by the [BPI_Shovable](./Content/Features/Shoving/BPI_Shovable.uasset) interface. It checks for preconditions of other components, i.e. whether the Movement component reports the shoved Character is currently falling, and delegates the reaction to the [AC_Shovable](./Content/Features/Shoving/AC_Shovable.uasset) Actor Component if necessary.

The proactive [AC_Shove](./Content/Features/Shoving/AC_Shover.uasset) Actor Component contains the functionality to locate (using a SphereTrace) and message Characters that implement BPI_Shovable.


This pattern is repeated in several other features I implemented, the most complex of which is the [NPC Stalling](./Content/Features/NPCs/) mechanic, that requires event dispatchers to allow for VFX, animations and Input Action interactions while maintaining the separation of generic, stalling-specific concerns.


### [Inventory](./Content/Libraries/Inventory/)

In order to open/close doors or use task stations only when the Player has picked up the correct keycard or task before, I built an inventory system, that is admittedly overkill for this game, but kinda cool I think.

Sometimes state related to the Player is put into the PlayerCharacter in non-networked games. I tried to follow Unreal best-practices despite the game not using networking and created a custom [BP_PlayerState](./Content/Players/BP_PlayerStart.uasset) that uses the aforementioned combination of Interfaces, [BPI_Inventory](./Content/Libraries/Inventory/BPI_Inventory.uasset) and [BPI_Collectable](./Content/Libraries/Inventory/BPI_Collectable.uasset), and Actor Components, [AC_Inventory](./Content/Libraries//Inventory/AC_Inventory.uasset) for the Inventory functionality.

The inventory supports all kinds of Actors that implement BPI_Collectable, but allows for different limits on how many items of a 'category' (which is just a string) can be held at a time, and how to handle pickups when the limit is reached (see the Enumeration [E_CollectionBehavior](./Content/Libraries/Inventory/E_CollectionBehavior.uasset)).


In order to make Collectables and their associated [Authorization](./Content/Libraries/Authorization/) functionality easily manageable, I decided to use Data Tables. Again somewhat overkill for our game, but something that is apparently often used in Unreal Engine and would make life for our Designers easier if we had more items.

[Keycards](./Content/Features/Doors/BP_Keycard.uasset) and [Doors](./Content/Features/Doors/BP_Door.uasset) use [DT_Keycards](./Content/Features/Doors/DT_Keycards.uasset) for their configuration, while [Tasks](./Content/Features/Tasks/BP_Task.uasset) use [DT_Tasks](./Content/Features/Tasks/DT_Tasks.uasset). Those Data Tables are read during the Construction of the objects, so their effect is visible in the editor directly.
