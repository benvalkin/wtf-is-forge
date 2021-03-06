## All about Net Calls

Writing stuff involving client-server net communication is important for most mods. Your mod is probably not that interesting if it doesn't need to do this.

Net calls with Forge aren't actually that scary. Forge has a special `SimpleChannel` class which abstracts away most of the bad stuff (the alternative is to use Netty as is).

Look at the following examples to see how a basic networking system is done using the `SimpleChannel` class.

**Note**: this is a very basic example designed to get you started quickly. For better understanding, consult the official Forge docs.

## How do you test networking stuff?

If you want to test the client-server communication side of your mod, you can start two simultaneous debug processes by clicking on `runServer` followed by `runClient` in the IntelliJ Gradle tab (on the right). Once your server starts up, you can join it with your client as you would an actual server. Click ‘Add Server’ and set the address to ‘localhost’.

**Important**: Make sure to set `online-mode=false` in your project's `server.properties` file inside the ‘Run’ folder before starting up, otherwise when you try to connect your client will get an ‘Invalid Session’ connection refusal.

## Step 1: Making a custom packet class
### Client to Server packets
Here is a simple example of how you can make a custom packet class that will be sent from clients to the server. For the other way round, see the next sub-heading.

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

### Server to Client packets
Here is a simple example of how you can make a custom packet that will be send from the server to clients. It's the same as before aside from the Process() method, and the addition of a ClientPacketHandler class.

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

## Step 2: Registering and sending over packets

Here is how you actually send packet between clients and server, and how you register your custom packets so that you mod can listen out for them.
It can all be done with some sort of custom NetworkManager class that has a static instance of `SimpleChannel` as a member. Something like this:

```java
    public class Networker {
    private static final Logger LOG = LogManager.getLogger();
    private static final String PROTOCOL_VERSION = "1";

    // make a static SimpleChannel class member and initialize it like this.
    // To understand what all of the arguments mean, consult the Official Forge docs.
    private static final SimpleChannel INSTANCE = NetworkRegistry.newSimpleChannel(
            new ResourceLocation("agricultured", "main"),
            () -> PROTOCOL_VERSION,
            PROTOCOL_VERSION::equals,
            PROTOCOL_VERSION::equals
    );

    // here is where you tell your mod to start listening for your custom packet. Make sure you do this for all your custom packets at some point when your mod starts up.
    // If you don't register your packet with INSTANCE.registerMessage(), you will probably get some sort of errors about Unidentified Packets (or maybe nothing will happen)
    public static void RegisterMessages()
    {
        INSTANCE.registerMessage(
                420, // unique ID number. Can be anything, but make sure it's unique to your other Messages or there will probably be issues.
                TestServerPacket.class,
                TestServerPacket::Encode,
                TestServerPacket::new,
                TestServerPacket::Process
        );

        INSTANCE.registerMessage(
                421, // a different ID number
                TestClientPacket.class,
                TestClientPacket::Encode,
                TestClientPacket::new,
                TestClientPacket::Process
        );

    }
    // send your custom server packet to the server
    public static void SendToServer(TestServerPacket test)
    {
        INSTANCE.sendToServer(test);
    }

    // send your custom client packet to the server
    public static void SendToClientAnnoyingWay(ServerPlayer player, TestClientPacket packet) // annoying because it requires you to have an instance of ServerPlayer i think
    {
        INSTANCE.sendTo(packet, player.connection.getConnection(), NetworkDirection.PLAY_TO_CLIENT);
    }

    // some better methods that are more convenient
    public static void SendToClient(Supplier<ServerPlayer> player, TestClientPacket packet)
    {
        // Send to one player
        INSTANCE.send(PacketDistributor.PLAYER.with(player), packet);
    }
    public static void SendToAllClientsWithChunkLoaded(Supplier<LevelChunk> chunk, TestClientPacket packet)
    {
        // Send to all players tracking this level chunk
        INSTANCE.send(PacketDistributor.TRACKING_CHUNK.with(chunk), packet);
    }
    public static void SendToAllClients(TestClientPacket packet)
    {
        // Send to all clients connected to the server
        INSTANCE.send(PacketDistributor.ALL.noArg(), packet);
    }
}
```

## Step 3: Improving this system
You should now have a basic feel for how net calls work with Forge.

From here, you should:
1) Figure out how to encode more complex packets using `FriendlyByteBuf`
2) Make an `IPacket` interface for your custom packets to allow neater packet registration and handling
3) Register your packets using a for loop with incremented IDs (not individually as done above)
