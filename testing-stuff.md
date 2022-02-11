### How to test online functionality with both a client and a server
Writing stuff involving client-server net communication is important for most mods. Your mod is probably not that interesting if it doesn't need to do this.

If you want to test the client-server communication side of your mod, you can start two simultaneous debug processes by clicking on ‘runServer’ followed by ‘runClient’.
Once your server starts up, you can join it with your client as you would an actual server. Click ‘Add Server’ and set the address to ‘localhost’.

**Important**: Make sure to set `online-mode=false` in your project's `server.properties` file inside the ‘Run’ folder before starting up, otherwise when you try to connect your client will get an ‘Invalid Session’ connection refusal.
