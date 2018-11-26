---
layout: post
title: Manipulating Loot Tables
description: How to interact with loot tables.
tags:
 - mc-mod-tutorial
---

Loot tables are a relatively new feature in Minecraft. These tables handle the various pools of loot that are obtainable in the game. While normal users would interact with loot tables through a resource pack, mod authors have the `LootTableLoadEvent` which allows for complete manipulation of the loot pools. This event is fired every time a table is loaded. Tables are loaded during world load, and there are multiple tables. For this reason, you should try to keep your use of the event as light weight as possible. 

For the first part of this tutorial, we will review how to add items to the vanilla dungeon chest loot pool. Since this event will be fired once for every table loaded, the first thing that should be done is checking if the pool being loaded is the target loot pool. You can get the name of the current loot pool by calling `getName` on the `LootTableLoadEvent`. A list of all vanilla loot tables can be found in `LootTableList`, which allows for very easy checking. The following is an example of a check that is looking for the simple dungeon chest pool. 

```java
@SubscribeEvent
public void onLootTablesLoaded(LootTableLoadEvent event) {
 
    if (event.getName().equals(LootTableList.CHESTS_SIMPLE_DUNGEON)) {
         
    }
}
```

Once we are satisfied that the table being loaded is the target table, we can begin manipulating the loot pools. Pools are used as sub categories of a loot table. In cases such as the dungeon, there are three different pools. `main` which holds all of the rare loot, like horse armor and golden apples, `pool1` which holds the uncommon loot, like pumpkin seeds and bread, and `pool2` which holds the garbage, like rotten flesh. These names and purposes will likely be different for other tables. In this tutorial we will be adding cookies as a garbage loot, so we will be targeting `pool2`. Pools can be retrieved by calling `getTable().getPool("name")` on the `LootTableLoadEvent`. One thing to keep in mind, it is possible for other mods to delete loot pools, so a null check should be added for safety. 

```java
@SubscribeEvent
public void onLootTablesLoaded(LootTableLoadEvent event) {
 
    if (event.getName().equals(LootTableList.CHESTS_SIMPLE_DUNGEON)) {
 
        final LootPool pool2 = event.getTable().getPool("pool2"); 
        if (pool2 != null) {
 
        }
    }
}
```

At this point, we can now begin to add the new entry to the loot pool through the `addEntry` method. In this case we will be adding a `LootEntryItem`, which has a few constructor arguments. The first argument is the item, which is `Items.COOKIE` in this case, although other items can also be used. The second argument is the weight of the entry. For those unfamiliar with weighted randoms, it's just a different way to calculate the % chance of the outcome happening. An example formula for % chance is `(weight / totalWeight) * 100`. For this example I will use 10, which is about average for this pool. The third argument is quality, which is used to increase/decrease the odds of the outcome. This is almost never used in vanilla, and is mostly set to 0. The fourth parameter is an array of LootFunction, these allow for code to be applied to the entry after it has been selected. We will get into those more a bit later, so an empty array can be used for now. The fifth argument is an array of LootCondition, they allow for the entry to only be selected under certain conditions. We will again save this for later and use an empty array. The last argument is the name for the entry. Names must be unique, and the standard representation is "MODID:NAME". For this case I will be using `"tutorial:cookies"`. Once finished, the code will look something like this. 

```java
@SubscribeEvent
public void onLootTablesLoaded(LootTableLoadEvent event) {
 
    if (event.getName().equals(LootTableList.CHESTS_SIMPLE_DUNGEON)) {
 
        final LootPool pool2 = event.getTable().getPool("pool2");
 
        if (pool2 != null) {
 
            // pool2.addEntry(new LootEntryItem(ITEM, WEIGHT, QUALITY, FUNCTIONS, CONDITIONS, NAME));
            pool2.addEntry(new LootEntryItem(Items.COOKIE, 10, 0, new LootFunction[0], new LootCondition[0], "loottable:cookie"));
        }
    }
}
```

That was the final step to adding an entry to a loot pool. You should now have cookies, or whatever item you specified showing up in dungeon chests. Now to get into the two arrays we passed over. Firstly we have `LootCondition`s, which allow something to happen only if a condition is met. For example `RandomChance` is a vanilla condition, which will only pass true a specified % of the time. Adding `new RandomChance(0.50f);` to the conditions would only be true 50% of the time. The other thing we skipped earlier are `LootFunction`s. These allow for code to be executed when the entry is selected. Their arguments are dependant on the function, but nearly all of them accept an array of `LootCondition` which will prevent execution if not met. An example of a vanilla loot function is `SetCount` which accepts an array of `LootCondition` and a `RandomValueRange`. The value range is simply a wrapper class for ranged values. Adding `new SetCount(new LootCondition[0], new RandomValueRange(1, 5))` would result in 1-5 cookies being generated by the entry. 

All loot tables in the game follow these same guide lines, so this tutorial can easily be adapted to other tables and pools. Things like mob drops and fishing loot now use the pool system, so it is possible to apply this tutorial to them as well. A full list of the vanilla pools can be found [here](http://tutorials.darkhax.net/pages/loot_table_data/). I would highly recommend using it while editing loot pools. 
