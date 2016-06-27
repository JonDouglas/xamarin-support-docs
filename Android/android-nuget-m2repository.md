# Xamarin.Android Support Library NuGet m2repository manual download

### 1. Investigation

You may run into issues with downloading the `m2repository` when referencing a NuGet package of the Android Support Libraries / Google Play Services.

**Example error:**

`Download failed. Please download
https://dl-ssl.google.com/android/repository/android_m2repository_r25.zip
and put it to the
C:\Users\[Username]\AppData\Local\Xamarin\{SUPPORT LIBRARY NAME}\{VERSION NUMBER}
directory."`

**Mac Directory:** `/Users/[Username]/.local/share/Xamarin/`

**Windows Directory:** `C:\Users\[Username]\AppData\Local\Xamarin\`

### 2. Folder Contents

This example will be using Windows paths. This can be applied to either OS.

- Given the following:  `C:\Users\[Username]\AppData\Local\Xamarin\`

- A folder for each of the respective Android Support Libraries / Google Play Services will be shown.

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/bd8ae2cd-6dea-4f57-bb1a-c948c74a7fe0/03.08.2016-09.59.png)

- Each library should have a collection of versions:

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/b3fad7df-b96d-4a25-b22c-c2f04848c320/03.08.2016-10.00.png)

**Note:** In this example I'm showing all of the versions of `Android.Support.v4`

- We will then investigate the respective version we're interested in. We should see two folders inside, `content` and `embedded`:

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/d6e9c6f5-d6aa-41d6-9f1b-7f3d34eed4be/03.08.2016-10.00.png)

1. `content` - Contains the `m2repository`
2. `embedded` - Contains the respective `.aar` contents

### 3. Automatic Fix

- Delete the versioned library folder that is giving you errors:

**Mac Directory:** `/Users/[Username]/.local/share/Xamarin/{SUPPORT LIBRARY NAME}/{VERSION NUMBER}`

**Windows Directory:** `C:\Users\[Username]\AppData\Local\Xamarin\{SUPPORT LIBRARY NAME}\{VERSION NUMBER}`

- Rebuild your project (Which will kickoff a Build Task to re-download the library).

Note: You may already have this zip in your directory, but it could be partial or become corrupted. Please ensure you fully wipe the existing libraries as mentioned above.

### 4. Manual Fix

- Delete the versioned library folder that is giving you errors:

**Mac Directory:** `/Users/[Username]/.local/share/Xamarin/{SUPPORT LIBRARY NAME}/{VERSION NUMBER}`

**Windows Directory:** `C:\Users\[Username]\AppData\Local\Xamarin\{SUPPORT LIBRARY NAME}\{VERSION NUMBER}`

Note: You may already have this zip in your directory, but it could be partial or become corrupted. Please ensure you fully wipe the existing libraries as mentioned above.

There are two steps to manually fixing this error.

1. Adding the `m2repository` folder to the `/content` folder.
2. Adding the respective Android Support Library / Google Play Services `.aar` contents to the `/embedded` folder.


#### 1. Adding the m2repository to the /content folder

 Download the respective `m2repository` from google.

[https://dl-ssl.google.com/android/repository/android_m2repository_r25.zip](https://dl-ssl.google.com/android/repository/android_m2repository_r25.zip)

**Note:** This version number will vary based on your error message.

- Extract that .zip to any directory. There should now be a `android_m2repository_r25` folder.
- Inside the `android_m2repository_r25` folder, we have a `m2repository` folder.
- Place the `m2repository` into the `{VERSION NUMBER}/content` folder

![](http://content.screencast.com/users/JDouglas18/folders/Snagit/media/69c43ed3-c590-4fa7-a3ba-3ff77f5e5440/03.08.2016-10.09.png)

#### 2. Adding the .aar contents to the /embedded folder

- Inside the `m2repository` folder, there is an .aar for the support library to be resolved. It can be found in the `com\android\support` directory:

**Example:**

`m2repository\com\android\support`

- There should be a `support-v4\{VERSION NUMBER}` which will contain the `.aar` file. 
- Extract the `.aar` and put the contents into the `embedded` folder.

**Example:**

- `m2repository\com\android\support\support-v4\23.1.1\support-v4-23.1.1` will contain items such as a `aapt`, `aidl`, `assets`, `libs`, `res`, `AndroidManifest.xml`, `annotations.zip`, and `classes.jar`. 
- Place all of the contents into the `{VERSION NUMBER}/embedded` folder.

### 5. New Manual Fix 

- Get the URL of the missing m2repository download
- Use a MD5 hash on the download URL
- Rename the file to {MD5HASH}.zip (Where MD5HASH is the hashed download URL)
- Place the new hashed .zip file in your Xamarin\zips directory