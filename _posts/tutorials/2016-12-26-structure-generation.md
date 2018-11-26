---
layout: post
title: Structure Generation
description: Structure generation using the NBT schematic format.
tags:
 - mc-mod-tutorial
---

One of the lesser known features hidden within the 1.9 Minecraft update is the nbt based schematic format. This format is used by Mojang to handle world gen for [Igloos](http://minecraft.gamepedia.com/Igloo), [Ender Cities](http://minecraft.gamepedia.com/End_city) and [Fossils](http://minecraft.gamepedia.com/Generated_structures#Fossil). With the help of the Structure Block, players are able to create their own schematic files and some players have taken advantage of this to create new dungeons through [command blocks](https://www.youtube.com/watch?v=2UmPDfeqlDY)! Mods can also take advantage of this to make fairly efficient structure generation. 

The first step is to open up your game, and make a new super flat creative world. In this world, build the structure that you want to generate with your mod. Keep in mind that there is a limit to this method. Each structure file can only contain 32x32x32 blocks. While this likely wont be a probelm for most structure designs, if you plan to do something large like the [Woodland Mansion](http://minecraft.gamepedia.com/Woodland_mansion) you will need to write code which can combine multiple structures together. Once you have the structure built, you will need to grab a structure block so you can export the file. The command for that is `/give @p minecraft:structure_block`. Once you have the structure block placed near your structure, click on the `[D]` button to switch it to save mode. This mode will change the gui to have some new options such as the size of the structure, and position relative to the block. I would recommend turning on `Show invisible blocks` as well. Once you have the size and position, give your structure a name. This name should be all lower cased and use underscores as the seperating character. After that, hit the save button and you're file will be created. Structures are saved inside the world file, so look in `world/structures` as an nbt file. Once you have the structure file, add it to your mods resource folder. The correct location is `assets/modid/structures/`. Note that you can also add aditional sub folders to group similar structures together. 

Now for the fun part, which is writing a new `IWorldGenerator` to load your structure into the world. This is actually much simpler than it sounds. This interface gives your class the `generate` method which gives you access to the world context, and allows you to make changes to the world. The first thing to do in said method is to work out where you want to actually place the structure. In this tutorial I am just going to spawn the structure up in the sky, and add a little bit of random offset to the x and z position. The next step is to create your `PlacementSettings`. Simply make a new instance of this class, and then use it to set up how you want to place the structure. There are a lot of cool settings you can mess with, such as `setRotation` which allows you to rotate the structure around when it is placed. Right now your code should look something like this.

```java
    public class WorldGenFlower implements IWorldGenerator {
        
        @Override
        public void generate (Random random, int chunkX, int chunkZ, World world, IChunkGenerator chunkGenerator, IChunkProvider chunkProvider) {
            
            final BlockPos basePos = new BlockPos(chunkX * 16 + random.nextInt(16), 100, chunkZ * 16 + random.nextInt(16));
            final PlacementSettings settings = new PlacementSettings().setRotation(Rotation.NONE);
        }
    }

```

Next up we are going to load in our structure and generate it with the settings created before. To do this, you must create a Template object. The way to do this is hidden away bit fairly straight forward. The first step is to create a new `ResourceLocation` which points to the structure file you made earlier. Once you have that, you can call this to get the Template for it. 

```java
world.getSaveHandler().getStructureTemplateManager().getTemplate(world.getMinecraftServer(), RESOURCE_LOCATION_HERE);
```

The template offers a few methods for doing math relative to the structure, however the thing we want right now is the `addBlocksToWorld` method. Simply pass this method the world instance, position and placement settings and the template will be generated in the world. Here is what it should look like now. 

```java
    
    public class WorldGenFlower implements IWorldGenerator {
        
        private static final ResourceLocation FLOWER = new ResourceLocation("libtest:flower");
        
        @Override
        public void generate (Random random, int chunkX, int chunkZ, World world, IChunkGenerator chunkGenerator, IChunkProvider chunkProvider) {
            
            final BlockPos basePos = new BlockPos(chunkX * 16 + random.nextInt(16), 100, chunkZ * 16 + random.nextInt(16));
            final PlacementSettings settings = new PlacementSettings().setRotation(Rotation.NONE);
            final Template template = world.getSaveHandler().getStructureTemplateManager().getTemplate(world.getMinecraftServer(), FLOWER);
            
            template.addBlocksToWorld(world, basePos, settings);
        }
    }
```

And lastly, we have to register our world generator. This can be done by calling `GameRegistry.registerWorldGenerator(new YourGenerator(), WEIGHT);` where the weight is used to determine the order in which structures generate. The term weight is a bit misleading in this situation as it refers to order of generation, and not necesarily how common it will be. The comonality of it can be changed by chaging the generate method to do nothing X% of the time. Now that the basics are down, you can modify and mess around with the code until you get your worldgen set up the way you want it to be. 

So, what about dynamic or random structure generation which can make changes to the template as it is generating. This is possible, howerver it is slightly more complex. Mojang has provided the `ITemplateProcessor` interface, which will allow every block in the template to be processed. Currently there is only one processor provided by Mojang and it is called `BlockRotationProcessor`. Despite the name, [it actually handles structural integrity](https://github.com/ModCoderPack/MCPBot-Issues/issues/323). This is used as the default processor, and allows for some interesting stuff. Basically, you give it a percentage as a float, and then every block in the template had that percent chance of not being removed from the template. Mojang uses this to randomly replace bone blocks with coal in their fossil generator, however it can also be added to make structures look old or broken. You can play around with this by adding `settings.setIntegrity(CHANCE);` to the code above. Note that changing the value in settings will only work if you use the default `addBlocksToWorld` method. 

So, processors may seem easy, however there is a catch. To use a custom processor you must change your `addBlocksToWorld` method to this one. `template.addBlocksToWorld(world, basePos, YOUR_PROCESSOR, settings, 2);`. The big issue here is that you can only have one processor at a time, and fixing that requires a bit more effort. For now I will just focus on writing a new processor though. This part is fairly straight forward just like the rest of it. You have a processBlock method which gets called for every non empty block in the structure. This method will allow you to change the block state or nbt data for that block. Bellow is an example of a processor which will randomly change the color of a colored block.

```java
public class TemplateProcessorColored implements ITemplateProcessor {

    private final BlockColored block;
    private final EnumDyeColor color;
    
    public TemplateProcessorColored(BlockColored block) {
        
        this.block = block;
        this.color = EnumDyeColor.values()[Constants.RANDOM.nextInt(EnumDyeColor.values().length)];
    }
    
    @Override
    public BlockInfo processBlock (World worldIn, BlockPos pos, BlockInfo blockInfoIn) {
        
        if (blockInfoIn.blockState.getBlock().equals(this.block))
            return new BlockInfo(blockInfoIn.pos, blockInfoIn.blockState.withProperty(BlockColored.COLOR, this.color), blockInfoIn.tileentityData);        
       
        return blockInfoIn;
    }
}
```

The last thing I will be covering in this tutorial is how other mods have been using this structure system. There are currently only a handful of mods which have worked with them, however most of them have all done something unique and interesting. The first example I would like to share is [RealFilingCabinet](https://github.com/bafomdad/realfilingcabinet/blob/f1254b80da17334e2abb85faa1389e18471f9f28/com/bafomdad/realfilingcabinet/world/TutorialGenerator.java) by bafomdad. He uses the structure format to bundle a small tutorial world with his mod. If you create a new flat world with the same name as the mod, you will be greated with a small build which explains how the mod works. 

The second great example I want to bring up is Vazkii's Quark mod and it's [Pirate Ships](https://github.com/Vazkii/Quark/blob/master/src/main/java/vazkii/quark/world/world/PirateShipGenerator.java). Vazkii makes use of many different elements to create a fully fledge dungeon structure, complete with pirate mobs, loot chests and other fancy things. 
