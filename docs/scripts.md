# Scripts

## NetObject

### Interfaces
ITick
This interface brings the `Tick(uint tick)` method. This method will be triggered automatically and should be the only way you update your objects. In here you will execute every method that needs a tick variable or really your entire code, that needs to run on the client and the server. If some code is exclusive for the client, execute it in `NetUpdate`. This method will be executet on every machine. No matter if the machine owns this client, predicts this client or is the server. So you will need to check if the NetObject is predicted, server, input owner... if you don't want the method to run on every machine, which is likely. If you just want a object, that is synced by the Server (inherits from `IStateHolder`) and the server changes the state of that object via an event or something like that, this method isn't required, which is why I made it optional.

IStateHolder
The IStateHolder makes the NetObject statefull. This means, that the NetObject syncs the state over the network.
The `GetCurrentState()` is for NetRewind to store the state and send it. The `UpdateState(IState state)` is for non predicted objects, that wait for an update from the server. The `ApplyState(IState state)` is for stuff like reconciliation and collider rollback. The A`ApplyPartialState(IState statem, uint part)` is for partial correction. So when a gun has everything in sync with the server, except the ammo count for example, you could return a number > 1 in the IState's `Compare` method, which will execute this ApplyPartialState method. So if you return a 2, the game won' reconcile. But it will trigger this ApplyPartialState method and parse the 2 as the part variable. So you could catch the 2 and update the ammo count to the correct amount for example. This allows for correction, that doesn't necessaryly mean that the entire game has to be corrected.

IInputListener
The interface will make it possible for this object to receave client input. So if this object needs to move around or shoot based on the client input, this needs to be implemented.
The InputListener brings some variables, which you will just implement without EVERY using them direcly! You will use `GetButton`, `GetVector2` or `GetData<T>`. To check if the NetObject even has the input for this tick, use the `HasInputForThisTick` method. The methods mentioned, should ONLY be used in the OnTick of the `Tick` interface. The method also brings the `NetInputOwnerUpdate`, which is really just a method like the `NetUpdate`, which runs before the `NetUpdate` method. But it only runs, is the client is the InputOwner. So no server, just locally for your client input. So if you want to implement looking, which doesn't really need to be approved by the server, you can do it in here and read the local input via the `InputActions` variable, which is a default variable of the NetObject.

IInputDataSource
This interface should be used, if you want to parse extra input or data, that the client decides and the server should know. This could also be normal input, that isn't a button / not supported by the default input sending code. So if you wanted to sync the rotation of the player camera, which doesn't really need to be approved by the server, so it's fine if the autority over that is on the client, you could parse the Vector2 rotation via a custom struct that inherits from `IData`. This struct just needs to be returned by the `OnInputData` method. You can than read this struct via the `GetData<T>` method, which requires the `IInputListener` interface.

### Don't use
``` csharp
OnNetworkSpawn();
OnNetworkDespawn();
Update();
```
!!! Don't use any other method you might be able to access without knowing what your doing. Many methods just have to be public for other scripts to be able to access them. If your unsure or are a more advanced user, you can just try to call some methods i didn' list and if it works, it works, if it doesn't it doesn't :) !!!

### You can use

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