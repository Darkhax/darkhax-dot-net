---
layout: post
title: Render Layers
description: Introduction to using render layers for entity rendering.
tags:
 - mc-mod-tutorial
---

Render layers are an aspect to entity rendering that was added back in 1.8. While layer renderers are often overlooked, they can be a really powerful tool when it comes to rendering entities. In vanilla render layers are used for everything from player capes to sheep wool. With a mod, you can add, remove and replace layers for any entity.

## Creating a layer
You can create a new render layer by making a new class that implements the `LayerRenderer` interface. This interface provides two methods, `doRenderLayer` which is used to handle your rendering, and `shouldCombineTextures` which is rarely used, and related to lighting and some other things you would not immediately expect. For this tutorial I have created a basic layer renderer that can be used for any living entity, and will render a rotating apple above it. 

```java
	@Override
	public void doRenderLayer(EntityLivingBase player, float limbSwing, float limbSwingAmount, float partialTicks, float ageInTicks, float netHeadYaw, float headPitch, float scale) {
		
		GlStateManager.pushMatrix();
		GlStateManager.rotate(180, 0, 0, 1);
		GlStateManager.scale(0.6, 0.6, 0.6);
		GlStateManager.rotate((ageInTicks) / 20.0F * (180F / (float)Math.PI), 0.0F, 1.0F, 0.0F);
		GlStateManager.translate(0, player.height - 0.3, 0);
		Minecraft.getMinecraft().getRenderItem().renderItem(new ItemStack(Items.APPLE), ItemCameraTransforms.TransformType.FIXED);
		GlStateManager.popMatrix();
	}
	
	@Override
	public boolean shouldCombineTextures() {
		
		return false;
	}
```

## Adding a layer
Adding a render player is surprisingly simple. You just need an instance of `RenderLivingBase` for the entity you want to modify. The way you go about doing this will be different based on what mob you are adding it to. 

If you want to add it to your own entity renderer, you can use `this.addLayer` in the constructor of your entity renderer. The following example is from the vanilla slime. 

```java
    public RenderSlime(RenderManager manager) {
        super(manager, new ModelSlime(16), 0.25F);
        this.addLayer(new LayerSlimeGel(this));
    }
```

If you want to add it to an entity you don't own, you can get the renderer during the init stage by using the class of the entity. This example shows how you can get the renderer for a sheep.
```java
		Render<Entity> renderer =  Minecraft.getMinecraft().getRenderManager().getEntityClassRenderObject(EntitySheep.class);
		
		if (renderer instanceof RenderLivingBase) {
			
			((RenderLivingBase<?>) renderer).addLayer(layer);
		}
```

If you want to add it to a player entity, a different method is required. Minecraft has support for different player renderers such as the Alex and Steve renderer.  and each  there is a special way to do that. Minecraft has support for different variations of the player model and each model has it's own renderer. The current models are `default` which is for Steve, and `slim` which is for Alex. You can get a specific player renderer by using this method. 

If you want to add a new layer renderer for players, a different aproach is required. Minecraft has support for different player render types such as the Alex and Steve models. You will need to add your layer to every player render type that you want to support. Typically this is just the Alex (slim) and Steve (default) models. Examples of 

```java
Minecraft.getMinecraft().getRenderManager().getSkinMap().get(type).addLayer(layer);
//Minecraft.getMinecraft().getRenderManager().getSkinMap().get("default").addLayer(layer);
//Minecraft.getMinecraft().getRenderManager().getSkinMap().get("slim").addLayer(layer);
```

Typically mods will just add support for the Alex and Steve models since non-vanilla player renderers may not be compatible with your specific render layer. If you want to add it to all player renders regardless you can use this code to loop through all of them. 

```java
		for (RenderPlayer playerRender : Minecraft.getMinecraft().getRenderManager().getSkinMap().values()) {
			
			playerRender.addLayer(layer);
		}
```

## Conclusion
I normally include references to various open source projects that relate to the subject of the tutorial. In the case of render layers, I had trouble finding public info on them beyond what is included in vanilla. If you made use of this tutorial, please send me screenshots of your project [on twitter](https://twitter.com/DarkhaxDev). If your project is open source, and demonstrates how to use render layers, I may edit this section to include your project as a reference!
