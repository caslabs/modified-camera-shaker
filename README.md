# modified-camera-shaker
Original roblox-ts package for Crazyman32's Roblox port of "EZ Camera Shake" by authors: https://github.com/osyrisrblx/rbxts-camera-shaker

## Note
This modified version of the camera-shaker library introduces the capability to create multiple independent CameraShaker instances. While this adds more flexibility in creating and controlling individual camera shake effects, please be aware that this feature is experimental. It may have potential performance impacts, especially when a large number of instances are active concurrently. Each instance also consumes additional memory, and potential for unintended interactions between instances has increased.

I encourage you to use this feature judiciously and monitor your game's performance and behavior closely when using multiple CameraShaker instances. Please feel free to report any issues or unexpected behavior, as your feedback will be valuable in refining and improving this feature.

## Modifications 
This modification was made to fit specific use cases and might not be suitable for all scenarios. As with any tool, I recommend testing extensively to ensure it fits your particular needs and does not adversely affect your game's performance.


Modified the original CameraShaker function to create a unique CameraShaker instance for each camera shake effect in Roblox, which employs render priority for effect sequence and a callback function for camera updates, with an added counter to keep track of the number of instances created. Modified code here:
```lua 
local counter = 0

function CameraShaker.new(renderPriority, callback)
  assert(type(renderPriority) == "number", "RenderPriority must be a number (e.g.: Enum.RenderPriority.Camera.Value)")
  assert(type(callback) == "function", "Callback must be a function")

  counter = counter + 1

  local self = setmetatable({
    _running = false;
    _renderName = "CameraShaker" .. counter;
    _renderPriority = renderPriority;
    _posAddShake = v3Zero;
    _rotAddShake = v3Zero;
    _camShakeInstances = {};
    _removeInstances = {};
    _callback = callback;
  }, CameraShaker)

  return self
end
```

This provides additional utility for developers when managing multiple instances, and of course for several reasons:

**Debugging and Monitoring:** By associating a unique ID (via the counter) with each instance, developers can more easily debug, track, and monitor individual CameraShaker instances. They can identify which instances are active, their state, and when they are created or destroyed.
```TS
let explosionShaker: CameraShaker = new CameraShaker(Enum.RenderPriority.Camera.Value, shakeCallback);

// Let's say at some point in your game, you want to debug and print all the active CameraShaker instances.
// Assuming you have some function that gets all active CameraShakers.
let activeShakers = getAllActiveCameraShakers();

for (let shaker of activeShakers) {
  print("Active CameraShaker ID: " + shaker.getRenderName());  // getRenderName() is a method that should return _renderName
}
```


**Instance Management:** When working with multiple CameraShaker instances, it may be necessary to interact with them individually, for example, to update their parameters or to stop/start them at different times. The unique ID can help manage these instances in an ordered manner.
## You can create multiple instances!
```TS
let smallExplosionShaker: CameraShaker = new CameraShaker(Enum.RenderPriority.Camera.Value, shakeCallback);
let largeExplosionShaker: CameraShaker = new CameraShaker(Enum.RenderPriority.Camera.Value, shakeCallback);

```


**Concurrency and Synchronization**: In a scenario where multiple effects are happening concurrently, unique identifiers can help ensure that operations on one instance do not inadvertently affect others.

```TS
// Assume an event causes both a small and large explosion at the same time
onExplosionEvent(() => {
  let smallExplosionShaker: CameraShaker = new CameraShaker(Enum.RenderPriority.Camera.Value, shakeCallback);
  let largeExplosionShaker: CameraShaker = new CameraShaker(Enum.RenderPriority.Camera.Value, shakeCallback);

  // You can control the start and stop of each CameraShaker independently even though they were created at the same time
  smallExplosionShaker.start();
  wait(1);  // Wait for 1 second
  largeExplosionShaker.start();
  
  wait(2);  // Wait for 2 seconds
  smallExplosionShaker.stop();
  wait(3);  // Wait for 3 seconds
  largeExplosionShaker.stop();
});
```

In the original camera-shaker, you can only define one instance, limiting the flexibility and diversity of camera shake effects, as it's unable to handle simultaneous or independent shake events, as all the camera shakes are tied to a single render step event listener. Each CameraShaker instance can have its own unique render step function, identified by the unique _renderName (which includes the counter value), which allows for multiple independent camera shakes, each with its own unique rendering behavior.

## My Custom Presets
**CameraShaker.Presets.Walk**\
A mild, rhythmic shake mimicking the natural movement of a player's viewpoint during walking. The shake is subtle but consistent, providing a gentle sway to the camera. It is typically applied once, but can be continually applied in a loop while the player's avatar is in a walking state to simulate ongoing movement.

**CameraShaker.Presets.Sprint**\
A more pronounced, rapid shake effect designed to replicate the intensity of sprinting. This preset creates a more sporadic and high-frequency camera shake that conveys the urgency and speed associated with sprinting. Like the Walk preset, it is generally applied once, but it can be looped during sprinting actions for a more immersive experience.
