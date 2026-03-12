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