## Dealing with player class
As with any game, there are dozen different player classes that do different things.

### `ServerPlayer`
`ServerPlayer` is the main class that the dedicated server uses to manage players.

Some stuff you can do with it:
1) Get it by casting a `Player`object when the in a server context
2) Construct a `Supplier<ServerPlayer>` from it:
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
