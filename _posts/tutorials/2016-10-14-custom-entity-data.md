---
layout: post
title: Custom Entity Data
description: Tracking custom entity data using capabilities.
tags:
 - mc-mod-tutorial
---

When making a mod, you may have the need to store extra data on an entities NBT. In the past this was acomplished using `IExtendedEntityProperties`, however Forge has replaced that system with their new capability system. Capabilities can do far more than just storing entity data, however this tutorial will be focusing on that aspect. In this tutorial I will be giving the entity a mana property. Before we get into the various elements of the system, you should know that this requires multiple classes to work. I personally prefer to make these inner classes, however that is not required. 

The first step in creating a storage capability is defining the methods which will be used to interact with your capability. This is done by creating a new interface, and adding bodyless methods to it. For this example I will need `getMana/setMana` methods, and `addMana/removeMana`for convenience. When finished, your interface should look something like this. 

```java
    public interface IManaHandler {

        int getMana();
        void setMana(int mana);
        void addMana(int mana);
        void removeMana(int mana);
    }
```

Now that you have the base interface, you need to create a capability field. This field is used to make reference checks to your capability. I tend to put all of these classes as inner classes, and then have this field at the highest level, although you can stick it anywhere. The `CapabilityInject` annotation will initialze this field with the correct stuff when forge loads the capability. 

```java
    @CapabilityInject(IManaHandler.class)
    public static final Capability<IManaHandler> CAPABILITY_MANA = null;
```

The second step is creating a default implementation of the earlier interface. This should be fairly simple, depending on what you're doing. In this example, we just need to create a new field, and write the logic for the methods defined earlier. This following is a basic implementation of the above interface. 

```java
    public class DefaultManaHandler implements IManaHandler {

        private int mana;
        
        @Override
        public int getMana () {
            return this.mana;
        }

        @Override
        public void setMana (int mana) {
            this.mana = mana;
        }

        @Override
        public void addMana (int mana) {
            this.mana += mana;
        }

        @Override
        public void removeMana (int mana) {
            this.mana -= mana;
            
            if (this.mana < 0)
                this.mana = 0;
        }
    }
```

This next step is where things start to become a bit more complex. We have to create a new class which implements `IStorage`. That interface gives us a `readNBT` and `writeNBT` method, which are used to handle reading and writign to nbt. Both methods will give you an instance of your earlier interface, which can be used to get/set data. In this example case, we only have to get/set the mana value. Here is another example. 

```java
    public class Storage implements Capability.IStorage<IManaHandler> {

        @Override
        public NBTBase writeNBT (Capability<IManaHandler> capability, IManaHandler instance, EnumFacing side) {
            
            final NBTTagCompound tag = new NBTTagCompound();           
            tag.setInteger("mana", instance.getMana());          
            return tag;
        }

        @Override
        public void readNBT (Capability<IManaHandler> capability, IManaHandler instance, EnumFacing side, NBTBase nbt) {
            
            final NBTTagCompound tag = (NBTTagCompound) nbt;
            instance.setMana(tag.getInteger("mana"));
        }
    }
```

The last part to writing our system is writing the provider class. This class is used by forge to connect the above code to the rest of the capability system. The `hasCapability` method is used by the rest of the system, to check if the provider handles a certain type of capability, you would just return whether or not the capability requested equals the capability field you created earlier in the tutorial. The `getCapability` method is similar, however it is used to create a new instance of the capability. Lastly there are the `serializeNBT/deserializeNBT` methods, which are similar to the `readNBT/writeNBT` mwthods from before. Here is a base example, there shouldn't be much to customize with it. 

```java
    public class Provider implements ICapabilitySerializable<NBTTagCompound> {
        
        IManaHandler instance = CAPABILITY_MANA.getDefaultInstance();

        @Override
        public boolean hasCapability(Capability<?> capability, EnumFacing facing) {
            
            return capability == CAPABILITY_MANA;
        }

        @Override
        public <T> T getCapability(Capability<T> capability, EnumFacing facing) {
            
            return hasCapability(capability, facing) ? CAPABILITY_MANA.<T>cast(instance) : null;
        }

        @Override
        public NBTTagCompound serializeNBT() {
            
            return (NBTTagCompound) CAPABILITY_MANA.getStorage().writeNBT(CAPABILITY_MANA, instance, null);
        }

        @Override
        public void deserializeNBT(NBTTagCompound nbt) {
            
            CAPABILITY_MANA.getStorage().readNBT(CAPABILITY_MANA, instance, null, nbt);
        }
    }
```

The last few parts of this tutorial are about tieing all the previous code together, and applying it. The first step for tieing this all together is registering the capability. This can be done by calling `CapabilityManager.INSTANCE.register` during the preInit phase. This method accepts a few parameters. The first is the class of the interface used. I will use `IManaHandler.class` for mine. The second is a new instance of the storage handler. `new Storage()` will work. The last parameter is the class of the default implementation. In my case I will use `DefaultManaHandler.class`. And that's it for registering the capability. 

```java
CapabilityManager.INSTANCE.register(IManaHandler.class, new Storage(), DefaultManaHandler.class);
```

The last step is attatching the capability to the entity. This is done using the `AttatchCapabilityEvent<Entity>` event. This event is fired when the entity is constructed, and allows us to add our capability to it. Since I only want `EntityPlayer`s to have mana, I will use `event.getObject` and check if it is an EntityPlayer. Once you have your conditions defined, you can use `event.addCapability` to apply a new instance of the provider we made earlier. This method has two parameters, the first is a `ResourceLocation` which should be specific to your mod, and the second is the new provider. Here is an example of how this should look. 

```java
    @SubscribeEvent
    public void attachCapabilities(AttachCapabilitiesEvent<Entity> event) {
        
        if (event.getObject() instanceof EntityPlayer)
            event.addCapability(new ResourceLocation("MODID", "NAME"), new Provider());
    }
```

If you followed the tutorial right, you should now be able to access the mana of a player, and use it however you like. The code for doing this is a bit long, so I tend to make wrapper methods to access it. The basic code The basic idea is that you can call `getCapability` on any entity, and you can get an instance of the interface you created. This method has two parameters, the first is for the capability field created earlier, and the second is for a direction you are trying to access. In this case the direction doesn't matter. It is mainly used for TileEntities. I tend to use `EnumFacing.DOWN` when it is not applicable. Here is an example of a basic getter method. 

```java
    public static IManaHandler getHandler(Entity entity) {

        if (entity.hasCapability(CAPABILITY_MANA, EnumFacing.DOWN))
            return entity.getCapability(CAPABILITY_MANA, EnumFacing.DOWN);
        return null;
    }
```

And that's it, you should now be able to interact with your mana system, and have save. You can find my full 1 class example code [here](https://gist.github.com/darkhax/8ea440d068a6616351197088e9a0d5bc) for further review.

But wait! There is more... If you are planning to store data on an `EntityPlayer`, this data will be lost when the player dies, or changes dimensions. To fix this, Forge has provided the `EntityPlayer.Clone` event. This event is fired when you would want to make your capability data persist. Simply grab the properties from the original entity, and apply them to the second one. This example shows the original mana being copied over to the new entity. 

```java
    @SubscribeEvent
    public void clonePlayer(PlayerEvent.Clone event) {
        
        final IManaHandler original = getHandler(event.getOriginal());
        final IManaHandler clone = getHandler(event.getEntity());
        clone.setMana(original.getMana());
    }
```
