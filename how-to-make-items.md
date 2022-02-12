## How to make basic inert items

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
        ChiliPepperSeeds = register("chilipepper_seeds", new Item((new Item.Properties()).tab(Agricultured.CREATIVE_TAB)));
    }

    private static <T extends Item> T register(String name, T item) {
        ResourceLocation id = new ResourceLocation(Agricultured.MODID, name);
        item.setRegistryName(id);
        ForgeRegistries.ITEMS.register(item);
        return item;
    }
}
```

## Step 3: Subscribe to the Forge Registry Event for items (using the Forge event bus)

```java
@Mod.EventBusSubscriber(bus=Mod.EventBusSubscriber.Bus.MOD)
    public static class RegistryEvents {
        @SubscribeEvent
        public static void onItemRegister(final RegistryEvent.Register<Item> event) {
            ItemRegister.RegisterAll(event); // call your RegistarAll method when the event fires
        }
    }
```

