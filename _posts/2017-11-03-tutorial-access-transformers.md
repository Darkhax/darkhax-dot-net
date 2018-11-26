---
layout: post
title: Access Transformers
description: Access transformers allow acessing restricted fields and methods without reflection.
tags:
 - tutorial
 - minecraft
---

Access transformers are a fairly basic feature of Forge which allows a mod author to modify access restrictions. In simple terms, this allows you to turn a private field into a public one, or make a final field not so final. The same thing can be acomplished using reflection, however reflection can make your code ugly, and is slightly slower. This is why access transformers tend to be promoted over reflection.

To get started, you must first create an AT (Access Transformer) file. These files are named using your mod id, followed by `_at.cfg`. These files are usually created in `src/main/resources` of your mod workspace. Once you have created the file, you need to add some stuff to your build script to handle the AT file. 

The first thing to do to your `build.gradle` file is fairly easy. At the end of your `processResources` block add `rename '(.+_at.cfg)', 'META-INF/$1'`. This tells Gradle to take any AT files, and move them in to the `META-INF` folder, which is where they need to be when in the compiled environment. Lastly, we need the script to set some properties in the compiled manifest file. The last bit can be done by adding this block to your build script.

```
jar {

	manifest {
	
	    attributes 'FMLAT': 'MODID_at.cfg'
	}
}
```

For those not familar with gradle, this may seem a bit complicated at first, but it's fairly basic once you get the hang of it. If you would like an example build script, I have a modified version of the default which you can find [here](https://gist.githubusercontent.com/darkhax/e0db68585c6f72de23939e8bf69caa99/raw/8734e3c08ad7360b5011193f659aa9f3e63caff2/build.gradle).

Now that the Gradle stuff is out of the way, it's time to start messing around with the ATs. For this tutorial we will be looking at the `attackDamage` field in `ItemSword`. Traditionally this method is private, but with a simple AT it can be made public. The AT format is very simple, each line represents an AT entry, and every parameter is seperated using a space. The first parameter is what you want to do to the target, usually this is just making the field public. In such a case, you can write `public` here. The next parameter is the full class name. In this case it is `net.minecraft.item.ItemSword`. The last parameter is the Searge name for the field. In this case it is `field_150934_a`. These can be tricky to get if you don't have to the MCP bot. 

Most devs use the MCP Bot on [Esper IRC](https://esper.net/) for AT work. It can automatically output the AT input required to make a field public. If you are already on Esper, pm MCPBot_Reborn the message `!gf FIELDNAME` or `!gm METHODNAME` and the bot will give all the info about the method, including AT info, which you can copy and paste. The following is the AT that the bot gave me when I ran !gf attackDamage`

```
public net.minecraft.item.ItemSword field_150934_a # attackDamage
```

Additionally, if you want to remove final from a field, just add `-f` to the first parameter. 

```
public-f net.minecraft.item.ItemSword field_150934_a # attackDamage
```

Now that everything is in place, when you run `gradlew setupDecompWorkspace` or `gradlew build` it will detect the AT file, and handle it appropriately. If you are like me, and maintain a seperate project for the core workspace, you can simply put all your AT files in `src/main/resources` of that project, and run `gradlew setupDecompWorkspace` in there. You must run this command every time you make a change to your AT file. This will recreate the decompiled minecraft jar with your modifications aplied. 

I would recommend keeping extensive comments in your AT file. The # symbol can be used to specify something as a comment, preventing it from being read as part of the AT. It is considered good practice to write the MCP name of a method, along with sorting all ATs by class. 

If you are using a dependency which has ATs, you will want to add the dep AT flag. This is `useDepAts = true` and goes in the `minecraft` block. This tells ForgeGradle that your mod has dependencies which use access transformers.

The last thing you should know about ATs is that they can be a bit troublesome at times. There is currently a [bug in Forge Gradle](https://github.com/MinecraftForge/ForgeGradle/issues/312) which will fail a build if the version number for a dependency which uses ATs has updated. This is an annoying issue, but if you run the build again it will pass. 
