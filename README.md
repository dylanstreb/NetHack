# Autoadjust inventory patch

This patch is for NetHack 3.7, for the current in-development version.
As it is still in development the patch will inevitably become out-of-sync
over time.
A patch for NetHack 3.6.7 is also available. This is located in a separate branch.

The purpose of this patch is to add the ability to automatically adjust specific items to specific letters via options.

A "rule" is a letter, an optional flag, and a string to match against. This works via simple text matching, not regex. If the rule text matches a new object's text, it will be automatically moved to the associated letter. If that letter is already taken, the old object will be swapped out.

The text matching will take the known information into account. An unidentified potion of gain level will not be matched by "gain level". But it is possible to match "smoky" potions, as long as the appearance is known.

Examples:
```
OPTIONS=autoadjust:e/key
OPTIONS=autoadjust:^a/weapon
OPTIONS=autoadjust:~f/unicorn horn
OPTIONS=autoadjust:w/bag
OPTIONS=autoadjust:-w/bag of tricks
OPTIONS=autoadjust:!y/food
```

From top to bottom:

* Upon picking up a new key, automatically move it to letter `e`
* The letter `a` will always contain a weapon, unless the inventory is full
* The letter `f` will always contain a unicorn horn. If the inventory fills up, new items will be assigned to `#`
* Auto-adjust bags to letter `w` ...
* ... unless that bag is a **known** bag of tricks
* Never ever put food in letter `y`; if the inventory is otherwise full the new food will be assigned to `#`

The "type flag" is always a single letter. Anything other than `^~!-` will be treated as the default behavior of adjusting without reserving the slot.

If the new item would be adjusted to an occupied letter but the object in that slot is also a match (e.g. you pick up a second weapon while already possessing one in `a`), nothing will happen.

## Limitations
There is no priority system, so even though "dagger" is a more exact match, it has the same priority as "weapon".

It is possible to use multiple letters for a single text match. The item will not always end up in the first matching letter.

Using too many reserved letters or forbidden items can give you items stored in the `#` slot, normally reserved for cursed loadstones. Multiple `#` items does not break anything but it is very inconvenient. It is recommended to use very few reserved letters.

# Extended Commands
Two extended commands are added.

* `#itemmatchname` - Prints the "match text" for an item currently in inventory. This will respect all known flags (e.g. it won't reveal that a gray stone is a luckstone without formal identification). Use this to make it easier to create new match rules.
    * The `|` character is used as a delimiter between groups, e.g. to separate "long sword" from "weapon". 
* `#autoadjust` - Run the rules on all items currently in inventory. If items can be moved to better letters, this command will do so. It will follow the same rules as pickup.
    * This can be used when identifying items or if picking up two weapons and dropping the one that ended up in the reserved slot.
