# LocationUpdatesServiceSample
This is a demo app to get location updates constantly in background and if user enters in specific range of already specfied location then notify him like Geofencing.


## Add below dependency in your app level build.gradle file
```gradle
implementation 'com.google.android.gms:play-services-location:17.0.0'
implementation 'androidx.localbroadcastmanager:localbroadcastmanager:1.0.0'
```

## Register Service in Manifest file
```xml
<!-- Required for foreground services on P+. -->
<uses-permission android:name="android.permission.FOREGROUND_SERVICE" />
<service
            android:name=".LocationUpdatesService"
            android:enabled="true"
            android:exported="true" />
```

## Initialize Service in Activity class
```kotlin
 // A reference to the service used to get location updates.
 private var mService: LocationUpdatesService? = null;
 // Tracks the bound state of the service.
 private var mBound: Boolean = false
 
 // Monitors the state of the connection to the service.
    private var mServiceConnection: ServiceConnection = object : ServiceConnection {
        override fun onServiceConnected(name: ComponentName?, service: IBinder?) {
            val binder: LocationUpdatesService.LocalBinder = service as LocationUpdatesService.LocalBinder
            mService = binder.service
            mBound = true
            // Check that the user hasn't revoked permissions by going to Settings.

            mService?.requestLocationUpdates()

        }

        override fun onServiceDisconnected(name: ComponentName?) {
            mService = null
            mBound = false
        }
    }

    override fun onStart() {
        super.onStart()
        bindService(Intent(this, LocationUpdatesService::class.java), mServiceConnection, Context.BIND_AUTO_CREATE)
    }

    override fun onStop() {
        if (mBound) {
            // Unbind from the service. This signals to the service that this activity is no longer
            // in the foreground, and the service can respond by promoting itself to a foreground
            // service.
            unbindService(mServiceConnection)
            mBound = false
        }
        super.onStop()
    }
```

## Check Location Permissions and GPS Settings
```xml
 <uses-permission android:name="android.permission.ACCESS_FINE_LOCATION" />
 <uses-permission android:name="android.permission.ACCESS_COARSE_LOCATION" />
 <uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" />
```
```kotlin
private val MY_PERMISSIONS_REQUEST_LOCATION = 68
private val REQUEST_CHECK_SETTINGS = 129

 if (ContextCompat.checkSelfPermission(this, Manifest.permission.ACCESS_FINE_LOCATION) != PackageManager.PERMISSION_GRANTED) {
            ActivityCompat.requestPermissions(this@MainActivity, arrayOf(Manifest.permission.ACCESS_FINE_LOCATION), MY_PERMISSIONS_REQUEST_LOCATION)
        } else {
            Log.e("MainActivity:","Location Permission Already Granted")
            if (getLocationMode() == 3) {
                Log.e("MainActivity:","Already set High Accuracy Mode")
                initializeService()
            } else {
                Log.e("MainActivity:","Alert Dialog Shown")
                showAlertDialog(this@MainActivity)
            }
        }
        
 override fun onRequestPermissionsResult(requestCode: Int, permissions: Array<out String>, grantResults: IntArray) {
        when (requestCode) {
            MY_PERMISSIONS_REQUEST_LOCATION -> {
                if ((grantResults.isNotEmpty() && grantResults[0] == PackageManager.PERMISSION_GRANTED)) {
                    Log.e("MainActivity:","Location Permission Granted")
                    if (getLocationMode() == 3) {
                        Log.e("MainActivity:","Already set High Accuracy Mode")
                        initializeService()
                    } else {
                        Log.e("MainActivity:","Alert Dialog Shown")
                        showAlertDialog(this@MainActivity)
                    }
                } else {
                    // permission denied, boo! Disable the
                    // functionality that depends on this permission.
                }
                return
            }
        }
    }

    private fun getLocationMode(): Int {
        return Settings.Secure.getInt(getContentResolver(), Settings.Secure.LOCATION_MODE)
    }
    
    override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        if (requestCode == REQUEST_CHECK_SETTINGS) {
            initializeService()
        }
    }
```
