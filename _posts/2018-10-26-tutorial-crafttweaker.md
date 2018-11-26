---
layout: post
title: CraftTweaker Support
description: Basics for adding support for CraftTweaker in your mod.
tags:
 - tutorial
 - minecraft
---

CraftTweaker is a Minecraft mod that many modpack authors use to customize the game. CraftTweaker allows users to write scripts in ZenScript, which can access specific functions and variables exposed by CraftTweaker and the various addons that add support. This approach to tweaking the game provides modpack authors with a lot of benefits, and adding support for it in your mods can make them much more appealing to modpack authors. If your mod adds new machines or ways to craft items, or you want to add some special ways to configure your mod you should definitely consider adding support. 

## Getting Started
To get started with CraftTweaker support, you need to add it to your `build.gradle` file as a dependency. To do this you need to add the CraftTweaker maven to your repositories list, and then specify it as a dependency. To do this, open up your gradle file, and look for a section called `repositories` and a section called `dependencies`. Do **not** add this to the section that is inside the buildscript section, that is for gradle plugins, not project dependencies! If you do not have a repositories section you can add one. If you are still not sure you can find an example of it in action [here](https://github.com/Darkhax-Minecraft/ItemStages/blob/master/build.gradle). Once you have found of created these sections, add the following info to your script and save the script. Then run the `./gradlew setupDecompWorkspace eclipse|idea` command again.

```js
repositories {
    maven {
        url 'http://maven.blamejared.com'
    }
}

dependencies {
    // deobfCompile 'CraftTweaker2:CraftTweaker2-MC1120-Main:1.12-4.1.11.494'
    deobfCompile 'CraftTweaker2:CraftTweaker2-MC1120-Main:1.12-VERSION'
}
```

## Exposing Methods
Now that you have CraftTweaker added to your developer environment, you can start to add classes to the mod. I prefer to make a new "crt" package to hold all of my addon classes. To allow scripts to call methods in your mod, create a new class that will contain the methods you want to expose. At the top of your class add `@ZenRegister` annotation. This annotation helps CraftTweaker find and load your class. You also need to add the `@ZenClass("mods.modname.ClassName")` which is how scripts will reference your class.

**Bonus Tip:** If you are adding support for a separate mod, you can use the `@ModOnly("modid")` to only load your class when the specified mod is also loaded.

```java
@ZenRegister
@ZenClass("mods.yourmod.YourClass")
public class YourClass {
}
```

Now that this class is going to be loaded by CraftTweaker you can define methods and add the `@ZenMethod` annotation to expose this method to scripts. Keep in mind that the way you define your method will change how it is accessed and used in a script. For example if the method is static you can access the method anywhere, otherwise the user will need to create a new instance of the object. Below is a an example method that could be used from a script using `mods.yourmod.YourClass.someMethod("A String", 9001);`

```java
@ZenRegister
@ZenClass("mods.yourmod.YourClass")
public class YourClass {

    @ZenMethod
    public static void someMethod(String string, int number) {
        System.out.println("String: " + string + " Int: " + number);
    }
}
```

There are also some special types that you can use which have special behaviors to them. For example `IIngredient` can be used to represent a specific ItemStack, an ore dictionary entry, or even an item with wildcard values. This example will iterate through all of the items and print out their names. `<minecraft:dye:*>` will get all dye items, and `<ore:ingotCopper>` will get all copper ingots.

```java
@ZenRegister
@ZenClass("mods.yourmod.YourClass")
public class YourClass {

    @ZenMethod
    public static void someMethod(IIngredient input) {
        for (IItemStack item : input.getItems()) {
            System.out.println(item.getDisplayName());
        }
    }
}
```

CraftTweaker has a lot of interface based classes such as `IItemStack` which serve as an abstraction layer between their exposed methods and the vanilla game. This is done to ensure that changes to the base game have a minimal impact on the scripts of the users. These wrapper objects can be converted to their Minecraft specific counterparts using helper methods in the `CraftTweakerMC` class. For example you can convert an IItemStack into an ItemStack using `CraftTweakerMC.getItemStack(istack);`. These wrapper types may also provide some helper functions that their vanilla counterparts do not have.

## IAction 
An IAction is a way of representing a method call as an object. This provides the benefit of allowing errors to be traced back to a specific script line, and in the past they were used to handle features that could be loaded, unloaded, and reloaded. Actions also play an important role is logging craft tweaker specific information and reporting script input errors. It is considered best practice to redirect calls to a ZenMethod to an IAction. 

```java
    @ZenMethod
    public static void someMethod (IIngredient input) {
        CraftTweakerAPI.apply(new YourAction(input));
    }
```

I have found that the best way to handle errors with an action, or to express an error with the input from the script is to throw an exception in the `apply` method of your action. CraftTweaker will catch this exception and warn the player about it when they join a world. This will also give the player a line number referencing the part of the script that caused the error. 

## Expansions
Sometimes you may want to add additional methods to an existing class that you did not write. For example if you make a magic mod, it may be useful to add a method that allows a script to give/take mana from a player, or even check how much mana the player has. This is done similarly to exposing new methods, first you need a `@ZenRegister` annotation on your class. You also need to use the `@ZenExpansion` annotation which defines the class you want to expand. 

```java
@ZenRegister
@ZenExpansion("crafttweaker.player.IPlayer")
public class ExpansionExample {
}
```

Now that you are expanding the class, you can add in new methods. These methods should be static, and take an instance of the type you're expanding as the first parameter. Here is an example of a method that will remove mana from a player. The new method would be called from a script by getting an IPlayer and calling `player.removeMana(10);`. The player parameter is taken from the object the method is being called on.

```java
@ZenRegister
@ZenExpansion("crafttweaker.player.IPlayer")
public class ExpansionExample {

    @ZenMethod
    public static void removeMana(IPlayer player, int amount) {
        YourManaUtils.removeMana(CraftTweakerMC.getPlayer(player), amount);
    }
}
```

## Script Timing & Delayed Actions
Scripts are loaded and executed after the recipe registry event in Forge has fired. This means that items, blocks, and recipes will be registered and available, but enchantments, biomes, and entities are not guaranteed by Forge to be registered yet. This, along with many other reasons may mean that you need to delay things until your mod can safely handle this. The approach to how you delay things can vary, but the general idea is still the same. You simply take the input data, do what validation you can on it, and then store it in a list somewhere in your mod until you're ready to handle that data.

You may also want to load scripts earlier tha normal. You can find an example of how to do this in [ContentTweaker](https://github.com/The-Acronym-Coders/ContentTweaker) however this is fairly advanced so I wont be covering it here. 

## Credits & Additional Information

For more information on CraftTweaker you can read their official documentation [here](https://crafttweaker.readthedocs.io/en/latest/). You can also join the Discord server for the mod using [this](https://discord.blamejared.com/) invite link. You can find an example CraftTweaker addons [here](https://github.com/Darkhax-Minecraft/WailaStages/tree/master/src/main/java/com/jarhax/wailastages/compat/crt). For an example of a mod that has CraftTweaker support but does not specifically depend on CraftTweaker you can look [here](https://github.com/Darkhax-Minecraft/BadMobs).
