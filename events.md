# Events

### All about events
Subscribing to events is pretty easy. The easiest way to do it is to have something like the following in your mod’s Main class:

```java
// simple non-static event
@SubscribeEvent
public void OnPlayerConnected(PlayerEvent.PlayerLoggedInEvent event)
{
   // do something with event
   // this listens for the player logging in on the SERVER
}
```

However, it is better practice to make a dedicated custom Handler class like this:

```java
// simple non-static event handler
public class PlayerJoinedHandler
{
   @SubscribeEvent
   public void OnPlayerConnected(PlayerEvent.PlayerLoggedInEvent event)
   {
      // do something with event
      // this listens for the player logging in on the SERVER
   }
}
```

You can also make event handler methods static. But, if you want to do this, you have to **register it** into Forge’s event bus with an annotation like this:

```java
// need this annotation to register the event. If this is missing, the event will never be detected
@Mod.EventBusSubscriber(modid = "agricultured", bus=Mod.EventBusSubscriber.Bus.FORGE, value = Dist.CLIENT)
public class PlayerJoinedHandler
{
   @SubscribeEvent
   public static void onPlayerJoined(ClientPlayerNetworkEvent.LoggedInEvent event)
   {
       // do something with event
       // this listens for the player logging in on the CLIENT
   }
}
```

You can replace the argument event’s type with one of forge’s many static event handler types. Most of their names end in ‘Event’ so just search for them.

It is important to keep in mind whether certain events are client-side or server-side. Subscribing to client-side events expecting it to run on your dedicated server will obviously not work. Some events (such as LoggedInEvent) have both client and server versions. Others seem to be settable with the annotation `@Mod.EventBusSubscriber(modid = "urmodid", bus=Mod.EventBusSubscriber.Bus.FORGE, value = Dist.CLIENT)` as used above.

