# Enrollment with Okay SDK on Android

This is a simple guide on how to integrate and initiate enrollment with Okay SDK.

We will begin by creating a new Android project on Android Studio.

Before we begin enrollment with Okay on our Android app, we will need to add Okay as a dependency to our app level `build.gradle` file (i.e `app\build.gradle`)

```gradle

...

dependencies {
    implementation fileTree(dir: 'libs', include: ['*.jar'])
    implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
    implementation 'androidx.appcompat:appcompat:1.1.0'
    implementation 'androidx.core:core-ktx:1.1.0'
    implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
    implementation 'com.google.android.material:material:1.0.0'
    testImplementation 'junit:junit:4.12'
    androidTestImplementation 'androidx.test:runner:1.2.0'
    androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'

    // Okay dependency
    implementation 'com.okaythis.sdk:psa:1.1.0'
}

```

We will also be adding Okay's maven repository to your project level `build.gradle` file.

```gradle

allprojects {
    repositories {
        google()
        jcenter()
        maven {
            url 'https://dl.bintray.com/okaythis/maven'
        }
    }
}

```

We will also need to set up Firebase for our project. If you are not familiar with integrating Firebase messaging please check this [documentaion](https://firebase.google.com/docs/cloud-messaging/android/client) for more information as Okay SDK depends on it.

```gradle
    dependencies {
        implementation fileTree(dir: 'libs', include: ['*.jar'])
        implementation"org.jetbrains.kotlin:kotlin-stdlib-jdk7:$kotlin_version"
        implementation 'androidx.appcompat:appcompat:1.1.0'
        implementation 'androidx.core:core-ktx:1.1.0'
        implementation 'androidx.constraintlayout:constraintlayout:1.1.3'
        implementation 'com.google.android.material:material:1.0.0'
        testImplementation 'junit:junit:4.12'
        androidTestImplementation 'androidx.test:runner:1.2.0'
        androidTestImplementation 'androidx.test.espresso:espresso-core:3.2.0'
  
        // Okay dependency
        implementation 'com.okaythis.sdk:psa:1.1.0'

        // Firebase Dependency
        implementation 'com.google.firebase:firebase-messaging:20.0.0'
    }
```

We can now sync our app's gradle file to build.

## Init PSA SDK

In order for Okay SDk to work correctly we will need to sync the SDK with PSS. Initialization of the PSA should be done within our Application class, inside the `onCreate()` method.

We use the `PsaManager` class from Okay to initialize our PSA. We will be using two methods from the `PsaManager` class, the `init()` and `setPssAddress()` methods. The `init()` and `setPssAddress()`  method from the `PsaManager` class has the following structure

```java

  PsaManager psaManager = PsaManager.init(Context c, T extends ExceptionLogger)

  // The PSS_SERVER_ENDPOINT is the url address for our PSS service e.g "http://protdemo.demohoster.com"
  psaManager.setPssAddress(PSS_SERVER_ENDPOINT);
```

A typical illustration of how our `Application` class should be

```kotlin

 class OkayDemoApplication: Application {

    @Override fun onCreate() {

        super.onCreate();

        // PsaManager.init(this, T extends ExceptionLogger);
        // The second argument that is being passed to PsaManager.init() method, must implement the ExceptionLogger interface. So will be creating our exception logger called  CrashlyticsExceptionLogger which implements that interface (this class could use any crash logger we choose, but we have decided to use crashlytics for this demo).
        val psaManager = PsaManager.init(this, new CrashlyticsExceptionLogger());
        psaManager.setPssAddress("http://protdemo.demohoster.com");

    }

}

```

We will need to add our application class to our manifest file like so.

```xml

<?xml version="1.0" encoding="utf-8"?>
<manifest xmlns:android="http://schemas.android.com/apk/res/android"
          package="com.ogieben.okaydemo">

    <application
      android:name=".OkayDemoApplication"
      android:allowBackup="true"
      android:icon="@mipmap/ic_launcher"
      android:label="@string/app_name"
      android:roundIcon="@mipmap/ic_launcher_round"
      android:supportsRtl="true"
      android:theme="@style/AppTheme">

      ...
  
    </application>

```

The Okay SDK requires certain kinds of permissions to work properly. We will have to ask the users to grant these permissions before we proceed.  We can easily create these helper methods to handle permission resolution for us.

```kotlin

// PermissionHelper.kt

class PermissionHelper(private val activity: Activity) {

    val REQUEST_CODE = 204

    fun hasPermissions(ctx: Context, permission: Array<String>): Boolean = permission.all {
        ActivityCompat.checkSelfPermission(ctx, it) == PackageManager.PERMISSION_GRANTED
    }

    fun requestPermissions(permission: Array<String>) = ActivityCompat.requestPermissions(activity, permission, REQUEST_CODE)
}

```

The Okay SDK comes with a prepacked method `PsaManager.getRequiredPermissions()` that helps us fetch an array of all required permissions.

```kotlin

// MainActivity.kt

  val permissionHelper = PermissionHelper(activity)

  private fun checkPermissions() {
        // prepacked method
        val requiredPermissions = PsaManager.getRequiredPermissions()
        if (!permissionHelper.hasPermissions(this, requiredPermissions)) {
            permissionHelper.requestPermissions(requiredPermissions)
        }
    }

```

We can now use the `checkPermission()` method within our `MainActivity.kt`'s `onCreate` method to request for all permission.

```kotlin
// MainActivity.kt

 override fun onCreate(savedInstanceState: Bundle?) {
    super.onCreate(savedInstanceState)
    setContentView(R.layout.activity_main)
    setSupportActionBar(toolbar)

    checkPermissions()
  }

```

We will need our device token from Firebase to be able to finish our enrollment.

If we have firebase successfully setup, we could request for our device token using the code sample below. If you have note been able to setup Firebase please refer to this [documentaion](https://firebase.google.com/docs/cloud-messaging/android/client) as Okay requires this service to work correctly.

```kotlin
// MainActivity.kt
 private var preferenceRepo: PreferenceRepo = PreferenceRepo(context)

 private fun fetchInstanceId () {
        FirebaseInstanceId.getInstance().instanceId
            .addOnCompleteListener(OnCompleteListener { task ->
                if (!task.isSuccessful) {
                    Log.w("", "getInstanceId failed", task.exception)
                    Toast.makeText(this@MainActivity, "Error could not fetch token", Toast.LENGTH_LONG).show()
                    return@OnCompleteListener
                }
                val token = task.result?.token

                // save token to SharedPreference storage for easy retrieval
                preferenceRepo.persistAppPns(token.toString())
            })
    }

```

If all is good we can now invoke this method from our `onCreate()` method within our activity like so.

```kotlin
class MainActivity : AppCompatActivity() {
  
    override fun onCreate(savedInstanceState: Bundle?) {
        super.onCreate(savedInstanceState)
        setContentView(R.layout.activity_main)
        setSupportActionBar(toolbar)

        // request permissions
        checkPermissions()

        // fetch token
        fetchInstanceId()
    }

}

```

We can now proceed with our enrollment if all permissions has been granted and our appPns(also know as Firebase token) has been retrieved successfully.

```kotlin

 private fun beginEnrollment() {
  
    val appPns = preferenceRepo.getAppPns() // retrieve Firebase token from SharedPreference storage
    val pubPssB64 = "MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAxgyacF1NNWTA6rzCrtK60se9fVpTPe3HiDjHB7MybJvNdJZIgZbE9k3gQ6cdEYgTOSG823hkJCVHZrcf0/AK7G8Xf/rjhWxccOEXFTg4TQwmhbwys+sY/DmGR8nytlNVbha1DV/qOGcqAkmn9SrqW76KK+EdQFpbiOzw7RRWZuizwY3BqRfQRokr0UBJrJrizbT9ZxiVqGBwUDBQrSpsj3RUuoj90py1E88ExyaHui+jbXNITaPBUFJjbas5OOnSLVz6GrBPOD+x0HozAoYuBdoztPRxpjoNIYvgJ72wZ3kOAVPAFb48UROL7sqK2P/jwhdd02p/MDBZpMl/+BG+qQIDAQAB"
    val installationId = "9990"

    val spaEnroll = SpaEnrollData(appPns,
    pubPssB64,
    installationId,
    null,
    PsaType.OKAY)
    PsaManager.startEnrollmentActivity(this, spaEnroll)
   }

```

If the `beginErollment()` method was called successfully we will need a way to retrieve information from the enrollment Activity. So will override the `onActivityResult()` method within our `MainActivity.kt` class.

```kotlin
// MainActivity.kt

 override fun onActivityResult(requestCode: Int, resultCode: Int, data: Intent?) {
        super.onActivityResult(requestCode, resultCode, data)
        if (requestCode == PsaConstants.ACTIVITY_REQUEST_CODE_PSA_ENROLL) {
            if (resultCode == RESULT_OK) {
                //We should save data from Enrollment result, for future usage
                data?.run {
                    val resultData = PsaIntentUtils.enrollResultFromIntent(this)
                    resultData.let {
                        preferenceRepo.saveExternalID(it.externalId)
                    }
                    Toast.makeText(applicationContext,   "Successfully got this externalId " + resultData.externalId, Toast.LENGTH_SHORT).show()
                }
            } else {
                Toast.makeText(this, "Error Retrieving intent after enrollment", Toast.LENGTH_SHORT).show()
            }
        }
    }

```
