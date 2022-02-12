## Dealing with player classes
As with any game, there are dozen different player classes that do different things.

### `ServerPlayer`
`ServerPlayer` is the main class that the dedicated server uses to manage players.

#### How to get it
1) By casting a `Player` object (only works in server context)
```java
public ServerPlayer FromPlayer(Player player) { return (ServerPlayer) player; }
```
#### Some stuff you can do with it
1) Construct a `Supplier<ServerPlayer>` from it
```java
public Supplier<ServerPlayer> ToSupplier(ServerPlayer player)
{
    return new Supplier<ServerPlayer>() {
        @Override
        public ServerPlayer get() {
            return player;
        }
    };
}
```
