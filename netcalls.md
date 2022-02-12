## All about Net Calls

Net calls with Forge aren't actually that scary. Forge has a special `SimpleChannel` class which abstracts away most of the bad stuff (the alternative is to use Netty as is).

Just look at the following examples to see how it's done.

**Note**: this is a very basic example designed to get you started quickly. For better understanding, consult the official Forge docs.

### CLIENT to SERVER packets
Here is a simple example of how you can make clients send packets to the server. For the other way round, see the next sub-heading.

```java
public class TestServerPacket // this is your custom packet class. it will be sent from clients to the server.
{
    private static final Logger LOG = LogManager.getLogger();

    public String message; // the data we want to transfer, in this case a simple string

    public TestServerPacket(String message) // basic constructor
    {
        this.message = message;
    }
    public TestServerPacket(FriendlyByteBuf buffer) // this method controls how the packet is decoded upon arrival. (you might notice it's just a constructor that takes in a FriendlyByteBuf)
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
            LOG.info("PACKET RECEIVED BY SERVER :D");
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
                420, // unique ID number. Can be anything, but make sure it's unique to your other Messages or there will probably be issues.
                TestServerPacket.class,
                TestServerPacket::Encode,
                TestServerPacket::new,
                TestServerPacket::Process
        );

    }
    // send a custom packet to the server
    public static void SendToServer(TestServerPacket test)
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

### SERVER to CLIENT packets
Here is a simple example of how you can make the server send packets to clients.

```java
public class TestClientPacket // another custom packet class. It's identical to the previous packet class aside from the Process() method.
{
    private static final Logger LOG = LogManager.getLogger();

    public String message;

    public TestClientPacket(String message)
    {
        this.message = message;
    }
    public TestClientPacket(FriendlyByteBuf buffer)
    {
        this.message = buffer.readUtf();
    }
    public void Encode(FriendlyByteBuf buffer)
    {
        buffer.writeUtf(this.message);
    }
    public void Process(Supplier<NetworkEvent.Context> ctx) // here is the difference
    {
        ctx.get().enqueueWork(()->
        {
            // this makes it so that ClientPacketHandler.process() is only run when on the client
            DistExecutor.unsafeRunWhenOn(Dist.CLIENT, () -> () -> ClientPacketHandler.Process(this, ctx));
        });
        ctx.get().setPacketHandled(true);
    }
}

// and lastly, our client-side packet handler class. Not sure why we need a custom class for this, but Forge says so
class ClientPacketHandler
{
    private static final Logger LOG = LogManager.getLogger();

    public static void Process(TestClientPacket packet, Supplier<NetworkEvent.Context> ctx) // controls how the packet is processed by the CLIENT.
    {
        LOG.info("PACKET RECEIVED ON CLIENT :D");
        //LOG.info("  From: " + ctx.get().getSender().getScoreboardName());
        LOG.info("  Message from the packet: " + packet.message);
    }
}
```
```java
        // probably don't need to mention this, but remember to add this to RegisterMethods()
        INSTANCE.registerMessage(
                421, // use a different ID. Forge recommends you assign IDs in a for loop to be safe
                TestClientPacket.class,
                TestClientPacket::Encode,
                TestClientPacket::new,
                TestClientPacket::Process
        );
```

