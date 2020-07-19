# Getting the battery level

```js
import { isIOS, isAndroid, Application } from "@nativescript/core";

/**
 * @returns the battery level (at least for iOS), as a number between 0 and 1.
 * @platform iOS, or Android API level 21.
 */
function getBatteryLevel(){
    if(isIOS){
        UIDevice.currentDevice.batteryMonitoringEnabled = true;
        const batteryLevel = UIDevice.currentDevice.batteryLevel;
        UIDevice.currentDevice.batteryMonitoringEnabled = false;
        return batteryLevel;
    } else if(isAndroid){
        const bm = Application.android.context.getSystemService(android.content.Context.BATTERY_SERVICE);
        // return (bm as android.os.BatteryManager).getIntProperty(android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY);
        return bm.getIntProperty(android.os.BatteryManager.BATTERY_PROPERTY_CAPACITY);
    }
    throw "Platform not yet supported!";
}
```

# Displaying the battery level in React NativeScript

```jsx
export function BatteryLevel(){
    const batteryLevel = getBatteryLevel();

    return (
        <flexboxLayout
            flexGrow={1}
            justifyContent={"center"}
            alignItems="center"
            width="100%"
            height="100%"
            backgroundColor="yellow"
        >
            <flexboxLayout width={100} height={25} backgroundColor="black">
                <flexboxLayout width={batteryLevel} backgroundColor="green">

                </flexboxLayout>
            </flexboxLayout>
        </flexboxLayout>
    );
}
```
