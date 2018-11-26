---
layout: post
title: Update Checker
description: Update checking using Forge's built in update checker.
tags:
 - mc-mod-tutorial
---

Ensuring that your players have the latest updates can be very important, especially when critical bugs or security issues have been patched. Historically mod have used the game chat to relay update information, however that quickly got annoying, especially when playing with multiple mods. Luckily forge provides a lightweight solution for this. 
 
The first step in using Forge's update checker system is to create a json file to contain all of your update info. Creating said file is fairly easy, however if your mod already exists and is published it will require some effort to get all of the required information. Below is an example update file I made for a hypothetical mod. While the format forge uses is very straight forward, there are some design details which need further explanation. For the sake of completion each element will be explained below the example. 
 
``` json 
{ 
        "homepage": "www.yoursite.com/yourmod", 
        "promos": { 
                "1.10.2-latest": "1.2.1.1",
                "1.10.2-recommended": "1.2.0.3",
                "1.9.4-latest": "1.1.0.3",
                "1.9.4-recommended": "1.1.0.1",
                "1.9-latest": "1.0.0.7",
                "1.9-recommended": "1.0.0.4"
        }, 
        "1.10.2": {
                "1.2.0.0": "Ported to 1.10.2",
                "1.2.0.3": "Rewrote networking code",
                "1.2.1.0": "Added experimental custom crafting system",
                "1.2.1.1": "Fixed a bug with experimental crafting system"
        }, 
        "1.9.4": {
                "1.1.0.0": "Ported to 1.9.4",
                "1.1.0.1": "Fixed some oversights with 1.9.4 porting",
                "1.1.0.3": "Added tacos. Yumm!"
        }, 
        "1.9": {
                "1.0.0.0": "Initial Release",
                "1.0.0.4": "Fixed a crash when crafting X",
                "1.0.0.5": "Added experimental JEI support",
                "1.0.0.7": "Fixed loading time issue with JEI support"
        } 
} 
``` 
 
`homepage` - This is used as the website or download location for your mod. Forge will not automatically download your mod, or give players the option to download it directly, however they will forward users to this link. If your mod is on CurseForge, it would be a good idea to put your project link here.  
 
`promos` - This section is used to define special versions of your mod. Each sub element should be named using a versionnumber-type format, where versionnumber is the version of Minecraft that is being targeted, and type is either `latest` or `recommended`. Latest is used to define the newest version for that version, while recommended is used to define the most stable release for that version. 
 
`1.X.X` - These blocks of the json are used to define every public version of your mod. The format is simple, the first block defines the version of Minecraft, and inside you have every version of your mod for that version followed by a changelog string. The changelog string is shown in the update checker to show the player what has changed. It is possible to use `\n` to define a line break in the changelog, it is not possible to format that in a sane way within your json format. I would highly recommend pointing the changelog to the changelog on Curse or GitHub if you have one of those.  
 
The next step in adding support for Forge's update checker is to upload your json file to the web so others can access it. If you don't have a place to upload it, I would highly recommend using [Gist](https://gist.github.com/). Once you have uploaded the file, get the URL for it. If you used Gist, the URL should look like this 

``` 
https://gist.githubusercontent.com/darkhax/486b52eabda471c7a04b1986ba134552/raw/update.json
``` 

Now that you have the URL, simply define it in your main mod class by adding `updateJSON = "your url here"` to the @Mod annotation parameters. It should look something like this.  
 
```java 
@Mod(modid = "yourid", name = "Mod Name", version = "1.2.2.1", updateJSON = "https://gist.githubusercontent.com/darkhax/486b52eabda471c7a04b1986ba134552/raw/update.json") 
``` 
 
After that, your mod should now use Forge's update checker, and whenever a new version is available a flashing icon will show up on the Mods button in the main menu. When the player clicks said button they will get a list of all the mods that have updates, along with changelogs for them. Once you have this done, the only thing you have to remember is to add new versions to this file every time you release a new one. [ArmorPlus](http://fdn.redstone.tech/TheDragonTeam/armorplus/update.json) and [Charset](https://gist.githubusercontent.com/Meow-J/fe740e287c2881d3bf2341a62a7ce770/raw/bf829cdefc84344d86d1922e2667778112b845b1/update.json) are both examples of mods which are currently using Forge's update checker. 
 
Keeping your versions file up to date can naturally be tedious, especially if you have a build server pushes many versions of the mod. This can be automated using Gradle, and when I find a decent setup for this I will expand this tutorial. If you already have a solution, or find one before I update this, please drop me the details so I can share it with others.
