
# Noke Mobile Library for Android #

The Nokē Mobile Library provides an easy-to-use and stable way to communicate with Nokē Devices via Bluetooth.  It must be used in conjunction with the Nokē Core API for full functionality such as unlocking locks and uploading activity.

![Nokē Core API Overview](https://imgur.com/vY2llC9.png)

## Requirements ##

This library is compatible with Android devices that support Bluetooth Low Energy (BLE) and are running Android version 4.4 or higher

## Installation ##

* The compat library may be found on Maven Central Repository.  Add it to your project by adding the following dependency:

```java
    compile 'com.noke.nokemobilelibrary:nokemobilelibrary:0.1.0'
```
	
* Once you've add the dependency to your project, add the Mobile API Key to your Android Manifest under the `<application>` header:
```xml
<meta-data android:name= "noke-core-api-mobile-key"
           android:value= "MOBILE_KEY_HERE"
            />
```

## Usage ##

### Setup ###

* The heart of the Noke Mobile Library is a service that when bound to an activity handles scanning for Noke Devices, connecting, sending commands, and receiving responses. To bind the NokeDeviceManagerService to an activity:
```java
private void initiateNokeService(){
        Intent nokeServiceIntent = new Intent(this, NokeDeviceManagerService.class);
        bindService(nokeServiceIntent, mServiceConnection, Context.BIND_AUTO_CREATE);
}
            
private ServiceConnection mServiceConnection = new ServiceConnection() {
        public void onServiceConnected(ComponentName className, IBinder rawBinder) {

            //Store reference to service
            mNokeService = ((NokeDeviceManagerService.LocalBinder) rawBinder).getService();

            //Register callback listener
            mNokeService.registerNokeListener(mNokeServiceListener);

            //Start bluetooth scanning
            mNokeService.startScanningForNokeDevices();

            if (!mNokeService.initialize()) {
                Log.e(TAG, "Unable to initialize Bluetooth");
            }
        }

        public void onServiceDisconnected(ComponentName classname) {
            mNokeService = null;
        }
    };
```

* Use the `NokeServiceListener` class to receive callbacks from the `NokeDeviceManagerService`:

```java
private NokeServiceListener mNokeServiceListener = new NokeServiceListener() {
        @Override
        public void onNokeDiscovered(NokeDevice noke) {

        }

        @Override
        public void onNokeConnecting(NokeDevice noke) {

        }

        @Override
        public void onNokeConnected(NokeDevice noke) {

        }

        @Override
        public void onNokeSyncing(NokeDevice noke) {

        }

        @Override
        public void onNokeUnlocked(NokeDevice noke) {

        }

        @Override
        public void onNokeDisconnected(NokeDevice noke) {

        }

        @Override
        public void onBluetoothStatusChanged(int bluetoothStatus) {

        }

        @Override
        public void onError(NokeDevice noke, int error, String message) {
            Log.e(TAG, "NOKE SERVICE ERROR " + error + ": " + message);
         }
        }
    };
```

### Scanning for Nokē Devices ###

* The `NokeDeviceManagerService` only scans for devices that have been added to the device array.
```java
//Add locks to device manager
NokeDevice noke1 = new NokeDevice("LOCK NAME", "XX:XX:XX:XX:XX:XX");
mNokeService.addNokeDevice(noke1);
```
**Note:** As of Android 8.0, Location Services **must** be enabled to scan for BLE devices.  If you're having trouble detecting devices, please ensure that your app has Location Permissions and that Location Services are turned on.

### Connecting to a Nokē Device ###

* When a Nokē Device is broadcasting and has been detected by the `NokeDeviceManagerService` the Nokē Device updates state to `Discovered`.  The service can then be used to connect to the Nokē Device.
```java
@Override
public void onNokeDiscovered(NokeDevice noke) {
    setStatusText("NOKE DISCOVERED: " + noke.getName());
    mNokeService.connectToNoke(noke);
}
```

### Unlocking a Nokē Device ###

* Once the Nokē device has successfully connected, the unlock process can be initialized.  Unlock requires sending a web request to a server that has implemented the Noke Core API (insert link here).  While some aspects of the request can vary, an unlock request will always contain:
    - **Mac Address** (`noke.getMac()`) - The bluetooth MAC Address of the Nokē device
    - **Session** (`noke.getSession()`) - A unique session string used to encrypt commands to the lock
* Both of these values can be read from the `noke` object *after* a successful connection

* Example:
```java
void requestUnlock(final NokeDevice noke, final String email)
    {
        Thread thread = new Thread(new Runnable() {
            @Override
            public void run() {
                try {
                    JSONObject jsonObject = new JSONObject();
                    jsonObject.accumulate("session", noke.getSession());
                    jsonObject.accumulate("mac", noke.getMac());
                    jsonObject.accumulate("email", email);

                    String url = serverUrl + "unlock/";
                    mDemoWebClientCallback.onUnlockReceived(POST(url, jsonObject), noke);
                } catch (Exception e) {
                    e.printStackTrace();
                }
            }
        });

        thread.start();
    }
```
* A successful unlock request will return a string of commands to be sent to the Nokē device.  These can be sent to the device by passing them to the `NokeDevice` object:
```java
currentNoke.sendCommands(commandString);
```
* Once the Nokē device receives the command, it will verify that the keys are correct and unlock.

### Uploading Activity Logs ###

The Nokē Mobile Library automatically uploads all responses from the Nokē device to the Nokē Core API for parsing.  Responses that contain activity logs are stored in the database and can be accessed using endpoints from the API.  Please see the Nokē Core API documentation for more details.

* The library is set to upload responses to the production API.  If you need to change this url for testing or other custom implementations, you can change the url using the 'NokeDeviceManagerService'

```java
mNokeService.setUploadUrl("NEW_URL_HERE");
```

## License

Nokē Mobile Library is available under the Apache 2.0 license. See the LICENSE file for more info.

Copyright © 2018 Nokē Inc. All rights reserved.
