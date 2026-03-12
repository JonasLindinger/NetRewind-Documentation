# Getting started

## Import
1. Import Netcode for GameObjects
2. Import the NetRewind package

## Set Up
1. Create a GameObject
2. Add NetRunner Component along with the NetworkManager and the Transport of your choice.
3. Assign your Transport and NetworkManager
4. Assign the Transport Layer Prefab. Choce the Prefab under the following path: "Assets/NetRewind/Prefabs/TransportLayer.prefab". Or choose your own Prefab, if you 100% know what your doing.
5. On Runtime, the NetRunner will automatically assign the NetworkTransport to the NetworkManager. It will also set the NetworkTickRate. So any changes you do in the editor with these attributes will be overwritten! Keep that in mind.
6. NetRewind uses conditional compilation. This means, that it is possible to create Dedicated Server builds, that only include code that the server actually needs. This also works for Hosts and clients.
So in order for NetRewind to work, you need to go to your Build Profiles (File/Build Profiles) and press Build Profile, if you don't have one already. I recommend creating 3 Profiles. One for Server, one for client and one for client with host capabilitys. For the Server Profile, i would recommend choosing a Dedicated Server Build. On the Profiles search for "Scripting Defines" and press "+". Type "Server" or "Client" depending on your build. For the Host Build (Client with hosting capabilitys), add "Server" and "Client" as seperate Definitions. Warning: The Definitions are case sensitive! After that select the build you want to develop (I would choose the Host, since it grants you the full controll while programming) and click Switch Profile in the bottom right corner. Your Project probably needs a few seconds to load the scripts.


## Start Programming

### Start Server
Now you can start programming. To start a server, client or host, don't call it via the NetworkManager. 

Call it like this:
```csharp
NetRewind.GetInstance().Run(RunType.Server);
```

### NetObject Info
Net objects are objects that are managed by NetRewind. So any object that should be managed by NetRewind should have ONE script that inherits by the NetObject class.

### Instantiating a NetObject
You can create/instantiate NetObjects like you would normally do, with Netcode for GameObjects.

# Scripts

## NetObject

Forbidden methods:
``` csharp
OnNetworkSpawn();
OnNetworkDespawn();
Update();
```

### Methods your allowed to use

``` csharp
HasInputForThisTick(uint tick);
```
Returns true, if the NetObject inherits from InputListener and the current input for this NetObject is the input for this tick.
If the NetObject isn't a InputListener, it returns false.

``` csharp
protected override void NetSpawn();
```
Is the replacement for `OnNetworkSpawn();`.

``` csharp
protected virtual void NetDespawn();
```
Is the replacement for `OnNetworkDespawn();`.

``` csharp
NetUpdate();
```
Is the replacement for `Update`.

``` csharp
ChangePredictionState(bool shouldBePredicted);
```
If this is true, the client will predict, run OnTick methods and compare the state with the server.
If this is false, the client won't touch this object and will apply the receaved server state no matter what.
The objects that receave the client input should always be predicted, otherwise there is no point of Client-Side prediction.

``` csharp
GetButton(string inputName);
```
This returns the input of the button. So when you registered a network input called "LMB", you read the input via this method.

``` csharp
GetVector2(string inputName);
```
This returns the input of the Vector2 input. So when you registered a network input called "Move", you read the input via this method. This returns the Vector2 of the client as a non normalized Vector2. So X and Y can have the following values: -1, 0 or 1. This is usefull when you want to sync basic keyboard information, that you might not want as a button. So you would use this for WASD for example. So when the user presses A and D at the same time, the y-value is 0 and you don't need to calculate it manually.

```csharp
GetData<T>();
```
This returns the extra input data, such as a y rotation of the player, if you decidecd to parse data along side the regular input. It returns your type `T`, since it automatically parses the sent IData into your T.

``` csharp
InputActions
```
! Client only code !
This links to a Dictionary<string, InputActions> and can only be used locally. So if you would want to make your player look around, you can go into your `NetUpdate` and access your input with this variable. After that you would make your player look and later parse the look rotation over the network for the server and other clients to know.

``` csharp
RegisterEvent(uint tick, IData data);
```
! Server only code !
This sends a event to every client along with some data.
An example for this would be shooting. You could register a shooting event and parse some shootingEventData if you wanted to.
This is the sender part. The receaver is the `OnEvent(IData eventData);`method.

``` csharp
protected override void OnEvent(IData eventData);
```
! Client only code !
Will be executet, when an event has been receaved.
This is the receaver part. The sender is the `RegisterEvent(uint tick, IData data);` method.

``` csharp
RunCodeInRollback(uint tickToRollbackTo, Action method);
```
! Server only code !
This code does Collider Rollback or really a complete rollback.
It rollsback EVERY NetObject it knows to the `tickToRollbackTo` Tick (ignoring the visuals), and executing the `method` you give it. After that it reverts to the current state. 
!!!This method will run, after the current tick has ended!!!

``` csharp
NetObject.RegisterInteraction(uint tick, NetObject obj1, NetObject obj2);
```
! Server only code !
This makes the Server move both objects into the same package, that gets send to the clients. So the clients can improve way more precise, if something on there end is off. So what i mean by that is: When a player interacts with a moving platform for example, the server can mark those two objects as interacing and will send both states in the same package to all clients for the next fiew seconds. So when a clients needs to reconcile, the chances of the clients reconciled state is higher, since he can reconstruct a GameState based on the given states and the states he knows locally.