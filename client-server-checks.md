# Checking whether code is running on a client or server
This is typically done with an instance of the Level class. So it’s really just a matter of finding a way to obtain this.

It seems as though many classes such as Entity store a reference to the instance of Level that they’re in, so you can often just reference that especially when working with events.
