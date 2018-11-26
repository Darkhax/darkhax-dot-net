---
layout: post
title: Inter Mod Communication
description: Explaination of the IMC API.
tags:
 - mc-mod-tutorial
---

Inter mod communication, also called IMC is a feature of Forge, which allow mods to send messages to each other. Messages sent through IMC are limited in what types of data they can contain, however with this limitation comes many really nice benefits. One of these benefits is that the mods communicating typically don't need the other mod to be available when compiling using Gradle. Another benefit is that the mods should never directly touch, preventing issues where the player only has one of the mods installed. 

## Sending Messages
Sending a message is very simple. Keep in mind that every single mod has their own IMC API, and you should refer to their documentation above everything else. This tutorial is just covering the basics of how messages are sent. 

Messages are sent to a mod using the static method `FMLInterModComms.sendMessage(String, String, ?)`. The first parameter is the id of the mod you are sending the message to. The second parameter is the key, which is used as an identifier by the recieving mod, that tells it what to do with your message. The last parameter is the value you are sending. There are a few types of values, some examples are strings, nbt tags, item stacks, resource locations and functions. Messages can be sent whenever, however they are processed after init but before post init and late messages are ignored. 

```java
    FMLInterModComms.sendMessage("imc-receiver-mod", "info-message", "Hello World");
```

## Receiving Messages
Messages are received as a mod specific event. To start listening to messages, simply listen to the IMCEvent in your mod class. 

```java
@Mod(modid = "imc-receiver-mod", name = "IMC: Reciever", version = "1.0.0")
public class IMCRecieverMod {
    
    @EventHandler
    public void onMessageRecieved(IMCEvent event) {
        
    }
}
```

This event is fired after the init phase, but before the post init phase. All of the messages sent to your mod can be retrieved using the `IMCEvent#getMessages()` method. Each message has two key parts to it, the key and the value. The key is used to describe what system the message is for. For example, if your mod has a system where items could be blacklisted, the key might be `your-system-blacklist`. The value is the content of the message being sent. There are a few different types of messages that can be sent, like NBT, String, ResourceLocation and Function. For the sake of this tutorial, we will be checking if a info-message is being sent to the mod, and if it is we will print out the message. Please note that `IMCMessage#isStringMessage()` and `IMCMessage#getStringValue()` have variations for the other message types as well. 

```java
    @EventHandler
    public void onMessageRecieved(IMCEvent event) {
        
        for (IMCMessage message : event.getMessages()) {
            
            // The check is done this way to prevent null pointer exceptions with null keys.
            if ("info-message".equalsIgnoreCase(message.key) && message.isStringMessage()) {
                
                System.out.println(message.getStringValue());
            }
        }
    }
```

## Functional Messages
Function messages are a newer type of message. These messages are used to send Java 8 functions to a mod. When you send your message, you include a string which represents the function class. For a class to be valid, it must implement `java.util.function.Function`.