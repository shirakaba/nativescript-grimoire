# `AVCaptureDevice`

Toggle the torch to 'on' (if off), or vice versa. Requires the device to have a torch of course, so won't do anything in the simulator.

Video here: https://twitter.com/LinguaBrowse/status/1088548651247513602

## JS

```js
function toggleTorch(){
    const device = AVCaptureDevice.defaultDeviceWithMediaType(AVMediaTypeVideo);
    if(device === null) return;
    if(!device.hasTorch) return;
    
    try {
        device.lockForConfiguration();

        if(device.torchMode == AVCaptureTorchMode.On){
            device.torchMode = AVCaptureTorchMode.Off;
        } else {
            try {
                device.setTorchModeOnWithLevelError(1.0);
            } catch(error){
                console.error("Could not set torch mode to 'on'.", error);
            }
        }
        device.unlockForConfiguration();
    } catch(error) {
        console.error("Torch could not be used.", error);
    }
}

toggleTorch();
```

## Swift

From: https://stackoverflow.com/questions/27207278/how-to-turn-flashlight-on-and-off-in-swift

```swift
import AVFoundation

func toggleTorch() {
    guard let device = AVCaptureDevice.default(for: AVMediaType.video) else { return }
    guard device.hasTorch else { return }

    do {
        try device.lockForConfiguration()

        if device.torchMode == .on {
            device.torchMode = .off
        } else {
            do {
                try device.setTorchModeOn(level: 1.0)
            } catch {
                print(error)
            }
        }

        device.unlockForConfiguration()
    } catch {
        print("Torch could not be used")
    }
}

toggleTorch()
```
