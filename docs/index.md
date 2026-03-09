# Getting started

## 1. Import
1. Import Netcode for GameObjects
2. Import the NetRewind package

## 2. Set Up
1. Create a GameObject
2. Add NetRunner Component along with the NetworkManager and the Transport of your choice.
3. Assign your Transport and NetworkManager
4. Assign the Transport Layer Prefab. Choce the Prefab under the following path: "Assets/NetRewind/Prefabs/TransportLayer.prefab". Or choose your own Prefab, if you 100% know what your doing.
5. On Runtime, the NetRunner will automatically assign the NetworkTransport to the NetworkManager. It will also set the NetworkTickRate. So any changes you do in the editor with these attributes will be overwritten! Keep that in mind.
6. NetRewind uses conditional compilation. This means, that it is possible to create Dedicated Server builds, that only include code that the server actually needs. This also works for Hosts and clients.
So in order for NetRewind to work, you need to go to your Build Profiles (File/Build Profiles) and press Build Profile, if you don't have one already. I recommend creating 3 Profiles. One for Server, one for client and one for client with host capabilitys. For the Server Profile, i would recommend choosing a Dedicated Server Build. On the Profiles search for "Scripting Defines" and press "+". Type "Server" or "Client" depending on your build. For the Host Build (Client with hosting capabilitys), add "Server" and "Client" as seperate Definitions. Warning: The Definitions are case sensitive! After that select the build you want to develop (I would choose the Host, since it grants you the full controll while programming) and click Switch Profile in the bottom right corner. Your Project probably needs a few seconds to load the scripts.


## 3. Start Programming

### Start Server
Now you can start programming. To start a server, client or host, don't call it via the NetworkManager. 

Call it like this:
``` c#
NetRewind.GetInstance().Run(RunType.Server);
```

### NetObject Info
Net objects are objects that are managed by NetRewind. So any object that should be managed by NetRewind should have ONE script that inherits by the NetObject class.

### Instantiating a NetObject
You can create/instantiate NetObjects like you would normally do, with Netcode for GameObjects.

