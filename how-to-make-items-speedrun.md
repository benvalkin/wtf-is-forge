## How to make basic items (speedrun)
This will show you how to create basic inert items that don't do anything as quick and easy as possible.

## Step 1: Make a Creative Tab for your mod
This is necessary because all your items need to be registered with some sort of Creative Mode tab.
```java
import net.minecraft.world.item.CreativeModeTab;
import net.minecraft.world.item.ItemStack;
import net.minecraft.world.item.Items;

public class Agricultured
{
    public static final String MODID = "agricultured";
    // set your mod's creative tab here
    public static final CreativeModeTab CREATIVE_TAB = new CreativeModeTab("agricultured") {
        @Override
        public ItemStack makeIcon() {
            //return new ItemStack()
            return new ItemStack(Items.BAKED_POTATO); // uses the Baked Potato icon for the creative tab
        };
    };
```

## Step 2: make an ItemRegister class

This is a manager class that will be responsible for registering all your mod's new items when it starts up. 

**Important**: As you might have seen in the official Forge docs, you must always register items using `ForgeRegistries.ITEMS.register()`. If you don't, referencing items will result in undefined behaviour (probably crashes).

You can do something like:

```java
import com.uncreatedmc.agricultured.items.FoodData;
import net.minecraft.resources.ResourceLocation;
import net.minecraft.world.item.Item;
import net.minecraftforge.event.RegistryEvent;
import net.minecraftforge.registries.ForgeRegistries;

public class ItemRegister
{
    public static Item ChiliPepperSeeds; // declare your items as public static members

    public static void RegisterAll(RegistryEvent.Register<Item> itemRegister)
    {
        // items go here
        
        // "chilipepper_seeds" is the ID of the item. In this case, you can spawn it in using "agricultured:chilipepper_seeds"
        ChiliPepperSeeds = register("chilipepper_seeds", new Item((new Item.Properties()).tab(Agricultured.CREATIVE_TAB)));
        
        // copy and paste the above line to add more items.
    }

    private static <T extends Item> T register(String name, T item) {
        ResourceLocation id = new ResourceLocation(Agricultured.MODID, name);
        item.setRegistryName(id);
        ForgeRegistries.ITEMS.register(item); // this method registers the item. Do not forget to do this for all new items.
        return item;
    }
}
```

## Step 3: Subscribe to the Forge Registry Event for items (using the Forge event bus)

```java
// this will probably be done in your main mod class
@Mod.EventBusSubscriber(bus=Mod.EventBusSubscriber.Bus.MOD)
    public static class RegistryEvents {
        @SubscribeEvent
        public static void onItemRegister(final RegistryEvent.Register<Item> event) {
            ItemRegister.RegisterAll(event); // call your RegistarAll method when the event fires
        }
    }
```

## Step 4: The resources part (texture, model and translations)
Your resources folder should look something like this. If it does not look like this, make sure it does. Your file structure should be the same as this screenshot (you can ignore the stuff about Blocks if you're not using going to be using any block items):

![image](https://user-images.githubusercontent.com/69928007/153730274-205aeff1-9c4a-40e9-bb33-faf3c2d4d6ba.png)

#### Translations
In your translations file (`en_us.json`), add an entry with the key `item.yourmodid.youritemid`. Make sure it uses your correct mod ID and the item ID that you used to register your item above. The value of this entry should be the fancy English name you want to see in-game.

Also, if you haven't already, add a translation for your **mod name** with the key `itemGroup.yourmodid`. Otherwise your mod's creative tab will show "itemGroup.yourmodid". Your translations file should like something this:

![image](https://user-images.githubusercontent.com/69928007/153730522-63d2ae9d-ccae-4f9e-8bb7-f4e0e04599d8.png)

#### Model
This part is custom for block items and custom model items, but you still need a .json file for basic items like sticks and food. Again, make sure the mod ID and item ID are the ones you used to register your item above.

![image](https://user-images.githubusercontent.com/69928007/153730462-8546cea3-3a20-4c59-950e-e483befc9083.png)

#### Texture
The easiest part of all, make a a texture for your item (typically 16x16 pixels for vanilla-style) and drag it into `resources\assets\yourmodid\textures\items`.

![image](https://user-images.githubusercontent.com/69928007/153730624-8433863e-ef77-47cd-9c68-223b3c305be9.png)

## That's pretty much it, now you can test your item.
It should look like this if you did everything correctly. Notice the translations are all working.

![image](https://user-images.githubusercontent.com/69928007/153730786-91bd7e8f-cd2a-4a84-a4e7-0104fc9ac0ac.png)







