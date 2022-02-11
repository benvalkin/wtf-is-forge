## All about Net Calls

Net calls with Forge aren't actually that scary. Forge has a special `SimpleChannel` class which abstracts away most of the bad stuff (the alternative is to use Netty as is).

Just look at the following examples to see how it's done.

**Note**: this is a very basic example designed to get you started quickly. For better understanding, consult the official Forge docs.

### A simple example

```java
public class TestPacket // this is your custom packet class. it will be sent to-and-from servers
{
    private static final Logger LOG = LogManager.getLogger();

    public String message; // the data we want to transfer, in this case a simple string

    public TestPacket(String message)
    {
        this.message = message;
    }
    public TestPacket(FriendlyByteBuf buffer) // this method controls how the packet is decoded upon arrival. (you might notice it's just a constructor that takes in a FriendlyByteBuf
    {
        this.message = buffer.readUtf();
    }
    public void Encode(FriendlyByteBuf buffer) // this method controls how the packet is encoded for sending.
    {
        buffer.writeUtf(this.message);
    }
    public void Process(Supplier<NetworkEvent.Context> ctx) // this method controls how the packet is processed by the SERVER.
    {
        ctx.get().enqueueWork(()->
        {
            LOG.info("PACKET RECEIVED :D");
            LOG.info("  From: " + ctx.get().getSender().getScoreboardName());
            LOG.info("  Message from the packet: " + this.message);
        });
        ctx.get().setPacketHandled(true);
    }
}
```
```java
public class Networker {
    private static final Logger LOG = LogManager.getLogger();
    private static final String PROTOCOL_VERSION = "1";

    // make a static SimpleChannel class member and initialize it like this. For more info on what each part does, consult the Official Forge docs.
    private static final SimpleChannel INSTANCE = NetworkRegistry.newSimpleChannel(
            new ResourceLocation("agricultured", "main"),
            () -> PROTOCOL_VERSION,
            PROTOCOL_VERSION::equals,
            PROTOCOL_VERSION::equals
    );

    // here is where you tell your mod to start listening for your custom packet. Make sure you call INSTANCE.registerMessage at some point when your mod starts up
    public static void RegisterMessages()
    {
        INSTANCE.registerMessage(
                0,
                TestPacket.class,
                TestPacket::Encode,
                TestPacket::new,
                TestPacket::Process
        );

    }
    // send a custom packet to the server
    public static void SendToServer(TestPacket test)
    {
        INSTANCE.sendToServer(test);
    }
}
```
Now, all that's left to do is send a packet to the server in some way or another. Here, the packet is send when the client joins.
```java
@Mod.EventBusSubscriber(modid = "agricultured", bus = Mod.EventBusSubscriber.Bus.FORGE, value = Dist.CLIENT)
public class TestHandler {
    @SubscribeEvent
    public static void drawLast(ClientPlayerNetworkEvent.LoggedInEvent event) {
        System.out.println("Player logged in on client");
        Networker.SendToServer(new TestPacket("HELLO :D :D"));
    }
}
```

