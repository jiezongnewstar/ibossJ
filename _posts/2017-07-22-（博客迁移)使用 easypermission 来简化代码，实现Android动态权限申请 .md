“动态权限申请”这个还是从6.0时代开始的，刚开始接触动态权限的小伙伴包括我，肯定是感觉很苦逼，因为要写一堆代码，才能把一些危险权限搞定。

easypermission的出现大大的简化了我们的操作，提升了效率，内部的封装使得我们的代码减少了很多，下面我们来看一下它的自我介绍：

简单的系统权限库，干掉6+++以上的系统权限问题。

github 地址：https://github.com/jiezongnewstar/easypermissions

方法一：

下面我们看下在Activity 和 Fragment 中的用法

public class MainActivity extends AppCompatActivity {
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
}


在权限回调的方法里加上这么一句就行：

EasyPermissions.onRequestPermissionsResult(requestCode, permissions, grantResults, this);


请求权限的方法

@AfterPermissionGranted(RC_CAMERA_AND_LOCATION)
private void methodRequiresTwoPermission() {
    String[] perms = {Manifest.permission.CAMERA, Manifest.permission.ACCESS_FINE_LOCATION};
    if (EasyPermissions.hasPermissions(this, perms)) {
        // Already have permission, do the thing
        // ...
    } else {
        // Do not have permissions, request them now
        EasyPermissions.requestPermissions(this, getString(R.string.camera_and_location_rationale),
                RC_CAMERA_AND_LOCATION, perms);
    }
}

方法二：

实现PermissionCallbacks接口

public class MainActivity extends AppCompatActivity implements EasyPermissions.PermissionCallbacks {

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

在某些情况下，如果没有某些权限，应用程序将无法正常运行。如果用户用“永远不要再问”选项否认这些权限，您将无法从用户请求这些权限，并且必须在应用程序设置中更改这些权限。你可以使用的方法easypermissions。somepermissionpermanentlydenied（…）显示一个对话框，在这种情况下，用户直接到系统为您的应用程序设置屏幕：

@Override
public void onPermissionsDenied(int requestCode, List<String> perms) {
    Log.d(TAG, "onPermissionsDenied:" + requestCode + ":" + perms.size());

    // (Optional) Check whether the user denied any permissions and checked "NEVER ASK AGAIN."
    // This will display a dialog directing them to enable the permission in app settings.
    if (EasyPermissions.somePermissionPermanentlyDenied(this, perms)) {
        new AppSettingsDialog.Builder(this).build().show();
    }
}

@Override
public void onActivityResult(int requestCode, int resultCode, Intent data) {
    super.onActivityResult(requestCode, resultCode, data);

    if (requestCode == AppSettingsDialog.DEFAULT_SETTINGS_REQ_CODE) {
        // Do something after user returned from app settings screen, like showing a Toast.
        Toast.makeText(this, R.string.returned_from_app_settings_to_activity, Toast.LENGTH_SHORT)
                .show();
    }
}




