# Xamarin.Android Installing APK as System Application

Ideally if you took a Release .apk, anything in the lib/ABI should go into the respective /system/lib folder.

For example with a File->New Android Project

If I created a Release Aligned .apk, I should be able to extract it and view the following:

`lib\armeabi-v7a`

Which has two .so files here:

`libmonodroid.so`

`libmonosgen-2.0.so`

You will need to ensure that both of these `.so` files are included in the `/system/lib` folder and then your application should install as a system application.