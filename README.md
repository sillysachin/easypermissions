# EasyPermissions

EasyPermissions is a wrapper library to simplify basic system permissions logic when targeting
Android M or higher.

## Installation

EasyPermissions is installed by adding the following dependency to your `build.gradle` file:

```java
dependencies {
  compile 'pub.devrel:easypermissions:0.1.8'
}
```

## Usage

### Basic

To begin using EasyPermissions, have your Activity (or Fragment) implement the 
`EasyPermissions.PermissionCallbacks` and override the following methods:

```java
public class MainActivity extends AppCompatActivity
    implements EasyPermissions.PermissionCallbacks {

    @Override
    protected void onCreate(Bundle savedInstanceState) {
        super.onCreate(savedInstanceState);
        setContentView(R.layout.activity_main);
    }

    @Override
    public void onRequestPermissionsResult(int requestCode, String[] permissions, int[] grantResults) {
        super.onRequestPermissionsResult(requestCode, permissions, grantResults);
    
        // Forward results to EasyPermissions
        EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);
    }

    @Override
    public void onPermissionsGranted(int requestCode, List<String> list) {
        // Some permissions have been granted
        // ...
    }

    @Override
    public void onPermissionsDenied(int requestCode, List<String> list) {
        // Some permissions have been denied
        // ...
    }
}
```

### Request Permissions

The example below shows how to request permissions for a method that requires both
`CAMERA` and `CHANGE_WIFI_STATE` permissions. There are a few things to note:

  * Using `EasyPermissions#hasPermissions(...)` to check if the app already has the
    required permissions. This method can take any number of permissions as its final
    argument.
  * Requesting permissions with `EasyPermissions#requestPermissions`. This method
    will request the system permissions and show the rationale string provided if
    necessary. The request code provided should be unique to this request, and the method
    can take any number of permissions as its final argument.
  * Use of the `AfterPermissionGranted` annotation. This is optional, but provided for
    convenience. If all of the permissions in a given request are granted, any methods
    annotated with the proper request code will be executed. This is to simplify the common
    flow of needing to run the requesting method after all of its permissions have been granted.    
    This can also be achieved by adding logic on the `onPermissionsGranted` callback.

```java
    @AfterPermissionGranted(RC_CAMERA_AND_WIFI)
    private void methodRequiresTwoPermission() {
        String[] perms = {Manifest.permission.CAMERA, Manifest.permission.CHANGE_WIFI_STATE};
        if (EasyPermissions.hasPermissions(this, perms)) {
            // Already have permission, do the thing
            // ...
        } else {
            // Do not have permissions, request them now
            EasyPermissions.requestPermissions(this, getString(R.string.camera_and_wifi_rationale),
                    RC_CAMERA_AND_WIFI, perms);
        }
    }
```

### Required Permissions

In some cases your app will not function properly without certain permissions. If the user
denies these permissions with the "Never Ask Again" option, you will be unable to request
these permissions from the user and they must be changed in app settings. You can use the
method `EasyPermissions.checkDeniedPermissionsNeverAskAgain()` to display a dialog to the
user in this situation and direct them to the system setting screen for your app:

```java
    @Override
    public void onPermissionsDenied(int requestCode, List<String> perms) {
        Log.d(TAG, "onPermissionsDenied:" + requestCode + ":" + perms.size());

        // Handle negative button on click listener. Pass null if you don't want to handle it.
        DialogInterface.OnClickListener cancelButtonListener = new DialogInterface.OnClickListener() {
            @Override
            public void onClick(DialogInterface dialog, int which) {
                // Let's show a toast
                Toast.makeText(getContext(), R.string.settings_dialog_canceled, Toast.LENGTH_SHORT)
                        .show();
            }
        };

        // (Optional) Check whether the user denied permissions and checked NEVER ASK AGAIN.
        // This will display a dialog directing them to enable the permission in app settings.
        EasyPermissions.checkDeniedPermissionsNeverAskAgain(
                this,
                getString(R.string.rationale_ask_again),
                R.string.setting,
                R.string.cancel,
                cancelButtonListener,
                perms);
    }

    @Override
    public void onActivityResult(int requestCode, int resultCode, Intent data) {
        super.onActivityResult(requestCode, resultCode, data);

        // Do something after user returned from app settings screen. User may be
        // changed/updated the permissions. Let's check whether the user has some permissions or not
        // after returned from settings screen
        if (requestCode == EasyPermissions.SETTINGS_REQ_CODE) {
            boolean hasSomePermissions = EasyPermissions.hasPermissions(
                    getContext(), somePermissions
            );

            // Do something with the updated permissions
        }
    }
```
