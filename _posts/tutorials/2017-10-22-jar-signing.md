---
layout: post
title: Jar Signing
description: How to sign your MC mods.
tags:
 - mc-mod-tutorial
---

Jar signing is a process that allows you to put your signature on jar files. When a jar is signed, it allows things like Forge to verify that the jar file was actually created by you. While this aproach is by no means full proof, it's definetly a step in the right direction for improving the security within the community. Currently the only mods that need to be signed, as of [Forge's latest policy update](http://www.minecraftforge.net/forum/topic/58706-regarding-minecraft-112-and-policy-changes/), are core mods. Other mods can still be signed, and I would encourage everyone to try this.

## Gathering the pieces
To sign your jar, several things are needed. The first is a key to sign your files, and a keystore to hold your keys. If the Java Dev Kit has been installed properly on the system building the mod you can run the following command. After running the command, you will be asked to provide a password for the key, and then another password for the keystore. You also need an alias, which is basically the name of the key. Take note of the alias, keystore pass, key pass, and the location of the keystore.jks file as these will all be needed later.

```
keytool -genkey -alias YOUR_ALIAS_HERE -keyalg RSA -keysize 2048 -keystore keystore.jks
```

Now that you have all the private info, you need to get the public sha1 fingerprint. This is a simple hex string that will be added to your mod later to enable verification. This info can be obtained by running the following command. Note that if the certificate fingerprint (sha1) is printed with colons you will need to remove those, and if it has uper case characters you need to lowercase it. 

```
keytool -list -alias YOUR_ALIAS_HERE -keystore keystore.jks
```

## Putting things in place
At this point, you should have the key store file location, the alias, the keystore password, the key password, and the sha1 fingerprint noted. All of this data needs to be passed to the gradle environment. There are many ways to do this, and any of them will work as long as the private info is kept secured. The most common way to do this is to add the info to your global/home gralde properties file. If you're on a desktop it's usually in the `.gradle/` folder in your home directory, but if you're using jenkins on linux it's probably `/var/lib/jenkins/.gradle/gradle.properties`. The following entries should be added, make sure to edit it with your info though!

```
keyStore=THE_KEY_STORE_FILE_LOCATION
keyStoreAlias=YOUR_ALIAS_HERE
keyStorePass=THE_KEY_STORE_PASS
keyStoreKeyPass=THE_KEY_PASS
signSHA1=THE_SHA1_FINGERPRINT
```
Once these are set, you can access the values from a `build.gradle` script using `project.varname`. 

## Build script additions
Now that all the pieces are in place, it's trivial to put them to use. If you've been following along, you can copy the following code to the gradle script for your mod. It's very straight forward, just passing the info from before to the built in signing code. Once this has been done, building your mod as you do normally will sign it. 

```groovy
task signJar(type: SignJar, dependsOn: reobfJar) {

    // Skips if the keyStore property is missing.
    onlyIf {
        project.hasProperty('keyStore')
    }

    // findProperty allows us to reference the property without it existing. 
    // Using project.propName would cause the script to fail validation if 
    // the property did not exist. 
    keyStore = project.findProperty('keyStore')
    alias = project.findProperty('keyStoreAlias')
    storePass = project.findProperty('keyStorePass')
    keyPass = project.findProperty('keyStoreKeyPass')
    inputFile = jar.archivePath
    outputFile = jar.archivePath
}

// Runs this task automatically when build is ran. 
build.dependsOn signJar
```

## Mod code additions
Now that your jar is being signed, you need to set things up in your mod's code. The only required addition is adding `certificateFingerprint` to your @Mod annotation, and setting it to the sha1 fingerprint from earlier. The fingerprint is public, so you can just set it as a string directly. You can also use the same key and fingerprint for all your mods. 

You should keep in mind that if the key becomes compromised or you need to change it for some other reason, replacing that string in every mod you maintain could take a while. This is why I set the fingerprint as part of my build script. There are a few different ways to do this, but I just add it to the minecraft block.

```groovy
minecraft {

    version = "${version_minecraft}-${version_forge}"
    mappings = "${version_mcp}"
    runDir = 'run'
    
    replace '@VERSION@', project.version
    //Replaces all text that matches the left side with the right.
    replace '@FINGERPRINT@', project.findProperty('signSHA1')
    //Makes those replacement changes to the main mod class.
    //My mod class is set in my project's gradle.properties file.
    replaceIn "${mod_class}.java"
}
```

At this point, you've made all the changes you need to. However there is one other addition you can optionally add. Forge has a `FMLFingerprintViolationEvent`, which allows your mod to respond to a fingerprint violation. Using this to crash the game is discouraged, but printing out a message to the log is acceptable. This is the code I am using in all my mods. 

```java
    @EventHandler
    public void onFingerprintViolation(FMLFingerprintViolationEvent event) {
        
        LOG.warn("Invalid fingerprint detected! The file " + event.getSource().getName() + " may have been tampered with. This version will NOT be supported by the author!");
    }
```

Something to remember, is that this event is only fired if the verification failed, and it's fired BEFORE preInit has happened. Because this event is fired before preInit, it's important that you only access things that have been initialized, or else it will throw a crash. In some of my earlier tests I was initializing the logger in preInit, so make sure you check for that too!

## Final verification
To ensure that you did everything right, there are a few things you can look at. Inside your jar file, there should now be a MINECRAF.SF and MINECRAF.RSA file inside the META-INF folder. The MANIFEST.MF file should also have SHA256 digest info for every single file in your mod's jar. These files are used to verify the integrity of your jar. 

The second verification that you can do, is load the mod up in game and look at the `fml-client-latest.log` file. After all mods have been constructed, it will print out all of the mod signature data. If your file is listed in the valid signatures list you did it right! If it's not, then you may have to back track. Here is an example from my test instance log. 

```
 Mod signature data
  	Valid Signatures:
 		(e3c3d50c7c986df74c645c0ac54639741c90a557) FML	(Forge Mod Loader	8.0.99.99)	forge-1.12.1-14.22.0.2469-universal.jar
 		(e3c3d50c7c986df74c645c0ac54639741c90a557) forge	(Minecraft Forge	14.22.0.2469)	forge-1.12.1-14.22.0.2469-universal.jar
 		(d476d1b22b218a10d845928d1665d45fce301b27) bookshelf	(Bookshelf	2.1.433)	Bookshelf-1.12.1-2.1.433.jar
 		(d476d1b22b218a10d845928d1665d45fce301b27) gamestages	(Game Stages	1.0.61)	GameStages-1.12.1-1.0.61.jar
 		(d476d1b22b218a10d845928d1665d45fce301b27) dimstages	(Dimension Stages	1.0.14)	DimensionStages-1.12.1-1.0.14.jar
 		(d476d1b22b218a10d845928d1665d45fce301b27) orestages	(Ore Stages	1.0.5)	OreStages-1.12.1-1.0.5.jar
 		(d476d1b22b218a10d845928d1665d45fce301b27) tinkerstages	(Tinker Stages	1.0.11)	TinkerStages-1.12.1-1.0.11.jar
  	Missing Signatures:
 		minecraft	(Minecraft	1.12.1)	minecraft.jar
 		mcp	(Minecraft Coder Pack	9.19)	minecraft.jar
 		crafttweaker	(CraftTweaker2	4.0.5)	CraftTweaker2-1.12-4.0.5.jar
 		ctgui	(CT-GUI	1.0.0)	CraftTweaker2-1.12-4.0.5.jar
 		crafttweakerjei	(CraftTweaker JEI Support	2.0.0)	CraftTweaker2-1.12-4.0.5.jar
 		mantle	(Mantle	1.12-1.3.1.22)	Mantle-1.12-1.3.1.22.jar
 		tconstruct	(Tinkers' Construct	1.12-2.7.3.29)	TConstruct-1.12-2.7.3.29.jar
```

As you can see, all my mods have `d476d1b22b218a10d845928d1665d45fce301b27` by them, which is my signature at the time of writing this. If the signature is different or missing in a crash report, you will be able to tell that something is up! Forge will also throw an error in the log when your signature is violated. 

## Credits
If you would like to see another great tutorial on signing your mods, check out [Matthewprenger's post](https://gist.github.com/matthewprenger/9b2da059b89433a01c1c). Some examples of open source mods that make use of jar signing inclue [TechReborn](https://github.com/TechReborn/TechReborn), [Japanese Emoji Commands](https://github.com/proxysprojects/Japanese-Emoji-Commands), and [Ender Storage](https://github.com/TheCBProject/EnderStorage). You can also use [this](https://github.com/search?l=Java&o=desc&p=1&q=certificateFingerprint+fml&s=indexed&type=Code&utf8=%E2%9C%93) GitHub search link, to find even more examples.

I would like to give a quick shoutout to [Aeronic](https://github.com/Aeronica) and [lclc98](https://github.com/lclc98) for pointing out an error in an earlier versions of the tutorial which had an error. Previous versions of the tutorial accessed the properties using project.propName instsead of project.findProperty('propName') which would cause the the build script to fail validation in some cases. 
