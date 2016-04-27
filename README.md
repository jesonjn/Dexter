
this project if fork from  

![https://github.com/Karumi/Dexter]

and this is a demo for android proguard 


see the end 


![Karumi logo][karumilogo] Dexter [![Build Status](https://travis-ci.org/Karumi/Dexter.svg?branch=master)](https://travis-ci.org/Karumi/Dexter) [![Maven Central](https://maven-badges.herokuapp.com/maven-central/com.karumi/dexter/badge.svg)](https://maven-badges.herokuapp.com/maven-central/com.karumi/dexter) [![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-Dexter-green.svg?style=true)](https://android-arsenal.com/details/1/2804)
======


Dexter is an Android library that simplifies the process of requesting permissions at runtime.

Android Marshmallow includes a new functionality to let users grant or deny permissions when running an app instead of granting them all when installing it. This approach gives the user more control over applications but requires developers to add lots of code to support it.

The official API is heavily coupled with the ``Activity`` class.
Dexter frees your permission code from your activities and lets you write that logic anywhere you want.


Screenshots
-----------

![Demo screenshot][1]

Usage
-----

To start using the library you just need to initialize Dexter with a ``Context``, preferably your ``Application`` as it won't be destroyed during your app lifetime:

```java
public MyApplication extends Application {
	@Override public void onCreate() {
		super.onCreate();
		Dexter.initialize(context);
	}
}
```

Once the library is initialized you can start checking permissions at will. You have two options, you can either check for a single permission or check for multiple permissions at once.

###Single permission 
For each permission, register a ``PermissionListener`` implementation to receive the state of the request:

```java
Dexter.checkPermission(new PermissionListener() {
	@Override public void onPermissionGranted(PermissionGrantedResponse response) {/* ... */}
	@Override public void onPermissionDenied(PermissionDeniedResponse response) {/* ... */}
	@Override public void onPermissionRationaleShouldBeShown(PermissionRequest permission, PermissionToken token) {/* ... */}
}, Manifest.permission.CAMERA);
```

To make your life easier we offer some ``PermissionListener`` implementations to perform recurrent actions:

* ``EmptyPermissionListener`` to make it easier to implement only the methods you want.
* ``DialogOnDeniedPermissionListener`` to show a configurable dialog whenever the user rejects a permission request:

```java
PermissionListener dialogPermissionListener =
	DialogOnDeniedPermissionListener.Builder
		.withContext(context)
		.withTitle("Camera permission")
		.withMessage("Camera permission is needed to take pictures of your cat")
		.withButtonText(android.R.string.ok)
		.withIcon(R.mipmap.my_icon)
		.build();
Dexter.checkPermission(dialogPermissionListener, Manifest.permission.CAMERA);
```

* ``SnackbarOnDeniedPermissionListener`` to show a snackbar message whenever the user rejects a permission request:

```java
PermissionListener snackbarPermissionListener =
	SnackbarOnDeniedPermissionListener.Builder
		.with(rootView, "Camera access is needed to take pictures of your dog")
		.withOpenSettingsButton("Settings")
		.build();
Dexter.checkPermission(snackbarPermissionListener, Manifest.permission.CAMERA);
```

* ``CompositePermissionListener`` to compound multiple listeners into one:

```java
PermissionListener snackbarPermissionListener = /*...*/;
PermissionListener dialogPermissionListener = /*...*/;
Dexter.checkPermission(new CompositePermissionListener(snackbarPermissionListener, dialogPermissionListener, /*...*/), Manifest.permission.CAMERA);
```

###Multiple permissions
If you want to request multiple permissions you just need to do the same but registering an implementation of ``MultiplePermissionsListener``:

```java
Dexter.checkPermissions(new MultiplePermissionsListener() {
    @Override public void onPermissionsChecked(MultiplePermissionsReport report) {/* ... */}
    @Override public void onPermissionRationaleShouldBeShown(List<PermissionRequest> permissions, PermissionToken token) {/* ... */}
}, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

The ``MultiplePermissionsReport`` contains all the details of the permission request like the list of denied/granted permissions or utility methods like ``areAllPermissionsGranted`` and ``isAnyPermissionPermanentlyDenied``.

As with the single permission listener, there are also some useful implementations for recurring patterns:

* ``EmptyMultiplePermissionsListener`` to make it easier to implement only the methods you want.
* ``DialogOnAnyDeniedMultiplePermissionsListener`` to show a configurable dialog whenever the user rejects at least one permission:

```java
MultiplePermissionsListener dialogMultiplePermissionsListener =
	DialogOnAnyDeniedMultiplePermissionsListener.Builder
		.withContext(context)
		.withTitle("Camera & audio permission")
		.withMessage("Both camera and audio permission are needed to take pictures of your cat")
		.withButtonText(android.R.string.ok)
		.withIcon(R.mipmap.my_icon)
		.build();
Dexter.checkPermissions(dialogMultiplePermissionsListener, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

* ``SnackbarOnAnyDeniedMultiplePermissionsListener`` to show a snackbar message whenever the user rejects any of the requested permissions:

```java
MultiplePermissionsListener snackbarMultiplePermissionsListener =
	SnackbarOnAnyDeniedMultiplePermissionsListener.Builder
		.with(rootView, "Camera and audio access is needed to take pictures of your dog")
		.withOpenSettingsButton("Settings")
		.build();
Dexter.checkPermissions(snackbarMultiplePermissionsListener, Manifest.permission.CAMERA, Manifest.permission.READ_CONTACTS, Manifest.permission.RECORD_AUDIO);
```

* ``CompositePermissionListener`` to compound multiple listeners into one:

```java
MultiplePermissionsListener snackbarMultiplePermissionsListener = /*...*/;
MultiplePermissionsListener dialogMultiplePermissionsListener = /*...*/;
Dexter.checkPermissions(new CompositeMultiplePermissionsListener(snackbarMultiplePermissionsListener, dialogMultiplePermissionsListener, /*...*/), Manifest.permission.CAMERA, Manifest.permission.RECORD_AUDIO);
```

###Handling listener threads
If you want to receive permission listener callbacks on the same thread that fired the permission request, you just need to use the ``OnSameThread`` version of the single and multiple permissions methods.

* ``checkPermissionOnSameThread`` to request a single permission and receive callbacks in the thread that fired the request
* ``checkPermissionsOnSameThread`` to request multiple permissions and receive callbacks in the thread that fired the request

###Showing a rationale
Android will notify you when you are requesting a permission that needs an additional explanation for its usage, either because it is considered dangerous, or because the user has already declined that permission once.

Dexter will call the method ``onPermissionRationaleShouldBeShown`` implemented in your listener with a ``PermissionToken``. It's important to keep in mind that the request process will pause until the token is used, therefore, you won't be able to call Dexter again or request any other permissions if the token has not been used.

The most simple implementation of your ``onPermissionRationaleShouldBeShown`` method could be:

```java
@Override public void onPermissionRationaleShouldBeShown(PermissionRequest permission, PermissionToken token) {
		token.continuePermissionRequest();
  }
```

###Screen rotation
If your application has to support configuration changes based on screen rotation remember to add a call to ``Dexter`` in your Activity ``onCreate`` method as follows:

```java
  @Override protected void onCreate(Bundle savedInstanceState) {
    super.onCreate(savedInstanceState);
    setContentView(R.layout.sample_activity);
    Dexter.continuePendingRequestsIfPossible(permissionsListener);
  }
```

**IMPORTANT**: Remember to follow the [Google design guidelines] [2] to make your application as user-friendly as possible.

Add it to your project
----------------------

Include the library in your ``build.gradle``

```groovy
dependencies{
    compile 'com.karumi:dexter:2.2.2'
}
```

or to your ``pom.xml`` if you are using Maven

```xml
<dependency>
    <groupId>com.karumi</groupId>
    <artifactId>dexter</artifactId>
    <version>2.2.2</version>
    <type>aar</type>
</dependency>

```
Caveats
-------
* For permissions that did not exist before API Level 16, you should check the OS version and use *ContextCompat.checkSelfPermission*. See [You Cannot Hold Non-Existent Permissions](https://commonsware.com/blog/2015/11/09/you-cannot-hold-nonexistent-permissions.html).
* Don't call Dexter from an Activity with the flag `noHistory` enabled. When a permission is requested, Dexter creates its own Activity internally and pushes it into the stack causing the original Activity to be dismissed.

Do you want to contribute?
--------------------------

Feel free to add any useful feature to the library, we will be glad to improve it with your help.

Keep in mind that your PRs **must** be validated by Travis-CI. Please, run a local build with ``./gradlew checkstyle build`` before submiting your code.


Libraries used in this project
------------------------------

* [Butterknife] [3]
* [JUnit] [4]
* [Mockito] [5]

License
-------

    Copyright 2015 Karumi

    Licensed under the Apache License, Version 2.0 (the "License");
    you may not use this file except in compliance with the License.
    You may obtain a copy of the License at

       http://www.apache.org/licenses/LICENSE-2.0

    Unless required by applicable law or agreed to in writing, software
    distributed under the License is distributed on an "AS IS" BASIS,
    WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
    See the License for the specific language governing permissions and
    limitations under the License.

[1]: ./art/sample.gif
[2]: http://www.google.es/design/spec/patterns/permissions.html
[3]: https://github.com/JakeWharton/butterknife
[4]: https://github.com/junit-team/junit
[5]: https://github.com/mockito/mockito
[karumilogo]: https://cloud.githubusercontent.com/assets/858090/11626547/e5a1dc66-9ce3-11e5-908d-537e07e82090.png





Information:Gradle tasks [:sample:assembleRelease]
Observed package id 'add-ons;addon-google_apis-google-19' in inconsistent location 'E:\Android_IDE\sdk\add-ons\addon-google_apis-google-19-1' (Expected 'E:\Android_IDE\sdk\add-ons\addon-google_apis-google-19')
:dexter:preBuild
:sample:preBuild
:dexter:preBuild UP-TO-DATE
:sample:preBuild UP-TO-DATE
:dexter:preReleaseBuild
:sample:preReleaseBuild
:dexter:preReleaseBuild UP-TO-DATE
:sample:preReleaseBuild UP-TO-DATE
:dexter:compileReleaseNdk
:sample:checkReleaseManifest
:dexter:compileReleaseNdk UP-TO-DATE
:dexter:compileLint
:sample:preDebugBuild UP-TO-DATE
:dexter:copyReleaseLint
:sample:generateReleaseBuildConfig
:dexter:copyReleaseLint UP-TO-DATE
:dexter:checkReleaseManifest
:dexter:preDebugAndroidTestBuild UP-TO-DATE
:dexter:preDebugBuild UP-TO-DATE
:dexter:preDebugUnitTestBuild
:sample:generateReleaseBuildConfig UP-TO-DATE
:dexter:preDebugUnitTestBuild UP-TO-DATE
:sample:generateReleaseAssets
:dexter:preReleaseUnitTestBuild
:sample:generateReleaseAssets UP-TO-DATE
:dexter:preReleaseUnitTestBuild UP-TO-DATE
:sample:generateReleaseResValues
:dexter:prepareComAndroidSupportAnimatedVectorDrawable2320Library
:sample:generateReleaseResValues UP-TO-DATE
:sample:compileReleaseNdk UP-TO-DATE
:sample:prePackageMarkerForRelease
:dexter:prepareComAndroidSupportAnimatedVectorDrawable2320Library UP-TO-DATE
:dexter:prepareComAndroidSupportAppcompatV72320Library
:sample:processReleaseJavaRes UP-TO-DATE
:sample:validateExternalOverrideSigning
:dexter:prepareComAndroidSupportAppcompatV72320Library UP-TO-DATE
:dexter:prepareComAndroidSupportCardviewV72320Library UP-TO-DATE
:dexter:prepareComAndroidSupportDesign2320Library UP-TO-DATE
:dexter:prepareComAndroidSupportRecyclerviewV72320Library UP-TO-DATE
:dexter:prepareComAndroidSupportSupportV42320Library UP-TO-DATE
:dexter:prepareComAndroidSupportSupportVectorDrawable2320Library UP-TO-DATE
:dexter:prepareReleaseDependencies
:dexter:compileReleaseAidl UP-TO-DATE
:dexter:compileReleaseRenderscript UP-TO-DATE
:dexter:generateReleaseBuildConfig UP-TO-DATE
:dexter:generateReleaseAssets UP-TO-DATE
:dexter:mergeReleaseAssets UP-TO-DATE
:dexter:generateReleaseResValues UP-TO-DATE
:dexter:generateReleaseResources UP-TO-DATE
:dexter:mergeReleaseResources UP-TO-DATE
:dexter:processReleaseManifest UP-TO-DATE
:dexter:processReleaseResources UP-TO-DATE
:dexter:generateReleaseSources UP-TO-DATE
:dexter:compileReleaseJavaWithJavac UP-TO-DATE
:dexter:extractReleaseAnnotations UP-TO-DATE
:dexter:mergeReleaseProguardFiles UP-TO-DATE
:dexter:packageReleaseRenderscript UP-TO-DATE
:dexter:packageReleaseResources UP-TO-DATE
:dexter:processReleaseJavaRes UP-TO-DATE
:dexter:transformResourcesWithMergeJavaResForRelease UP-TO-DATE
:dexter:transformClassesAndResourcesWithSyncLibJarsForRelease UP-TO-DATE
:dexter:mergeReleaseJniLibFolders UP-TO-DATE
:dexter:transformNative_libsWithMergeJniLibsForRelease UP-TO-DATE
:dexter:transformNative_libsWithSyncJniLibsForRelease UP-TO-DATE
:dexter:bundleRelease UP-TO-DATE
:sample:prepareComAndroidSupportAnimatedVectorDrawable2320Library UP-TO-DATE
:sample:prepareComAndroidSupportAppcompatV72320Library UP-TO-DATE
:sample:prepareComAndroidSupportCardviewV72320Library UP-TO-DATE
:sample:prepareComAndroidSupportDesign2320Library UP-TO-DATE
:sample:prepareComAndroidSupportRecyclerviewV72320Library UP-TO-DATE
:sample:prepareComAndroidSupportSupportV42320Library UP-TO-DATE
:sample:prepareComAndroidSupportSupportVectorDrawable2320Library UP-TO-DATE
:sample:prepareDexterDexterUnspecifiedLibrary UP-TO-DATE
:sample:prepareReleaseDependencies
:sample:compileReleaseAidl UP-TO-DATE
:sample:compileReleaseRenderscript UP-TO-DATE
:sample:mergeReleaseAssets UP-TO-DATE
:sample:generateReleaseResources UP-TO-DATE
:sample:mergeReleaseResources UP-TO-DATE
:sample:processReleaseManifest UP-TO-DATE
:sample:processReleaseResources
:sample:generateReleaseSources
:sample:compileReleaseJavaWithJavac UP-TO-DATE
:sample:compileReleaseSources UP-TO-DATE
:sample:lintVitalRelease
:sample:transformResourcesWithMergeJavaResForRelease UP-TO-DATE
:sample:transformClassesAndResourcesWithProguardForRelease
ProGuard, version 5.2.1
Reading input...
Reading program jar [E:\Android_IDE\sdk\extras\android\m2repository\com\android\support\support-annotations\23.2.0\support-annotations-23.2.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\design\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\appcompat-v7\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\animated-vector-drawable\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\support-vector-drawable\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\recyclerview-v7\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\support-v4\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\support-v4\23.2.0\jars\libs\internal_impl-23.2.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\com.android.support\cardview-v7\23.2.0\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\classes.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\butterknife-7.0.1.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\glide-3.7.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\glide-okhttp3-integration-1.4.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\gson-2.5.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\hawk.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\locSDK_6.13.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\nineoldandroids-2.4.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\okhttp-3.0.1.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\okio-1.6.0.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\rxjava-1.1.1-SNAPSHOT.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\rxjava_android.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\exploded-aar\Dexter\dexter\unspecified\jars\libs\zxing.jar] (filtered)
Reading program jar [J:\TEMP\Dexter\sample\build\intermediates\transforms\mergeJavaRes\release\jars\2\1f\main.jar] (filtered)
Reading program directory [J:\TEMP\Dexter\sample\build\intermediates\classes\release] (filtered)
Reading library jar [E:\Android_IDE\sdk\platforms\android-23\android.jar]
Reading library jar [E:\Android_IDE\sdk\platforms\android-23\optional\org.apache.http.legacy.jar]
Note: duplicate definition of library class [android.net.http.SslError]
Note: duplicate definition of library class [android.net.http.SslCertificate]
Note: duplicate definition of library class [android.net.http.SslCertificate$DName]
Note: duplicate definition of library class [org.apache.http.conn.scheme.HostNameResolver]
Note: duplicate definition of library class [org.apache.http.conn.scheme.SocketFactory]
Note: duplicate definition of library class [org.apache.http.conn.ConnectTimeoutException]
Note: duplicate definition of library class [org.apache.http.params.HttpParams]
Note: there were 7 duplicate class definitions.
      (http://proguard.sourceforge.net/manual/troubleshooting.html#duplicateclass)
Initializing...
Note: the configuration refers to the unknown class 'com.google.vending.licensing.ILicensingService'
Note: the configuration refers to the unknown class 'com.android.vending.licensing.ILicensingService'
Note: the configuration refers to the unknown class 'fqcn.of.javascript.interface.for.webview'
Note: the configuration refers to the unknown class 'com.android.vending.licensing.ILicensingService'
Note: the configuration refers to the unknown class 'com.google.vending.licensing.ILicensingService'
Note: the configuration refers to the unknown class 'com.android.vending.licensing.ILicensingService'
Note: butterknife.internal.BindingClass calls 'Class.getEnclosingClass'
Note: com.google.gson.internal.$Gson$Types$ParameterizedTypeImpl calls 'Class.getEnclosingClass'
Warning:butterknife.internal.ButterKnifeProcessor: can't find superclass or interface javax.annotation.processing.AbstractProcessor
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.AbstractProcessor
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.ProcessingEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.Filer
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.JavaFileObject
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Modifier
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ElementKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeVariable
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.ArrayType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.DeclaredType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Types
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ExecutableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ElementKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ExecutableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ExecutableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.VariableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.DeclaredType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ElementKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.SourceVersion
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced field 'javax.annotation.processing.ProcessingEnvironment processingEnv' in program class butterknife.internal.ButterKnifeProcessor
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.ProcessingEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.Diagnostic$Kind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.Messager
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Elements
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.PackageElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.AnnotationMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.AbstractProcessor
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.DeclaredType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ExecutableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.VariableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.ProcessingEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.Filer
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.JavaFileObject
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Modifier
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ElementKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeKind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Types
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.SourceVersion
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.Diagnostic$Kind
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.Messager
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Elements
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.PackageElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.Diagnostic
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Elements
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.util.Types
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.Filer
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.ProcessingEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.tools.JavaFileObject
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Modifier
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeVariable
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.ArrayType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.DeclaredType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeVariable
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.annotation.processing.RoundEnvironment
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeVariable
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.VariableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.ExecutableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.VariableElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.DeclaredType
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.type.TypeMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.SourceVersion
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.TypeElement
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.AnnotationMirror
Warning:butterknife.internal.ButterKnifeProcessor: can't find referenced class javax.lang.model.element.Element
Warning:okio.DeflaterSink: can't find referenced class org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
Warning:okio.Okio: can't find referenced class java.nio.file.Files
Warning:okio.Okio: can't find referenced class java.nio.file.Path
Warning:okio.Okio: can't find referenced class java.nio.file.OpenOption
Warning:okio.Okio: can't find referenced class java.nio.file.Path
Warning:okio.Okio: can't find referenced class java.nio.file.OpenOption
Warning:okio.Okio: can't find referenced class org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
Warning:okio.Okio: can't find referenced class java.nio.file.Path
Warning:okio.Okio: can't find referenced class java.nio.file.OpenOption
Warning:okio.Okio: can't find referenced class java.nio.file.Path
Warning:okio.Okio: can't find referenced class java.nio.file.OpenOption
Warning:okio.Okio: can't find referenced class org.codehaus.mojo.animal_sniffer.IgnoreJRERequirement
Warning:rx.internal.schedulers.NewThreadWorker: can't find referenced method 'java.util.concurrent.ConcurrentHashMap$KeySetView keySet()' in library class java.util.concurrent.ConcurrentHashMap
Warning:rx.internal.schedulers.NewThreadWorker: can't find referenced class java.util.concurrent.ConcurrentHashMap$KeySetView
Note: android.support.v4.media.IMediaBrowserServiceCallbacksAdapterApi21: can't find dynamically referenced class android.service.media.IMediaBrowserServiceCallbacks
Note: android.support.v4.media.IMediaBrowserServiceCallbacksAdapterApi21: can't find dynamically referenced class android.content.pm.ParceledListSlice
Note: android.support.v4.media.IMediaBrowserServiceCallbacksAdapterApi21$Stub: can't find dynamically referenced class android.service.media.IMediaBrowserServiceCallbacks$Stub
Note: android.support.v4.media.ParceledListSliceAdapterApi21: can't find dynamically referenced class android.content.pm.ParceledListSlice
Note: android.support.v4.text.ICUCompatApi23: can't find dynamically referenced class libcore.icu.ICU
Note: android.support.v4.text.ICUCompatIcs: can't find dynamically referenced class libcore.icu.ICU
Note: android.support.v7.widget.DrawableUtils: can't find dynamically referenced class android.graphics.Insets
Note: com.baidu.location.b.d$a: can't find dynamically referenced class android.os.SystemProperties
Note: com.baidu.location.b.e: can't find dynamically referenced class android.os.storage.StorageVolume
Note: com.google.gson.internal.UnsafeAllocator: can't find dynamically referenced class sun.misc.Unsafe
Note: com.orhanobut.hawk.AesCbcWithIntegrity$PrngFixes: can't find dynamically referenced class org.apache.harmony.xnet.provider.jsse.NativeCrypto
Note: com.orhanobut.hawk.AesCbcWithIntegrity$PrngFixes: can't find dynamically referenced class org.apache.harmony.xnet.provider.jsse.NativeCrypto
Note: okhttp3.internal.Platform: can't find dynamically referenced class com.android.org.conscrypt.OpenSSLSocketImpl
Note: okhttp3.internal.Platform: can't find dynamically referenced class org.apache.harmony.xnet.provider.jsse.OpenSSLSocketImpl
Note: the configuration keeps the entry point 'android.support.design.widget.NavigationView { void setNavigationItemSelectedListener(android.support.design.widget.NavigationView$OnNavigationItemSelectedListener); }', but not the descriptor class 'android.support.design.widget.NavigationView$OnNavigationItemSelectedListener'
Note: the configuration keeps the entry point 'android.support.design.widget.Snackbar$SnackbarLayout { void setOnLayoutChangeListener(android.support.design.widget.Snackbar$SnackbarLayout$OnLayoutChangeListener); }', but not the descriptor class 'android.support.design.widget.Snackbar$SnackbarLayout$OnLayoutChangeListener'
Note: the configuration keeps the entry point 'android.support.design.widget.Snackbar$SnackbarLayout { void setOnAttachStateChangeListener(android.support.design.widget.Snackbar$SnackbarLayout$OnAttachStateChangeListener); }', but not the descriptor class 'android.support.design.widget.Snackbar$SnackbarLayout$OnAttachStateChangeListener'
Note: the configuration keeps the entry point 'android.support.design.widget.TabLayout { void setOnTabSelectedListener(android.support.design.widget.TabLayout$OnTabSelectedListener); }', but not the descriptor class 'android.support.design.widget.TabLayout$OnTabSelectedListener'
Note: the configuration keeps the entry point 'android.support.design.widget.TabLayout { void setupWithViewPager(android.support.v4.view.ViewPager); }', but not the descriptor class 'android.support.v4.view.ViewPager'
Note: the configuration keeps the entry point 'android.support.design.widget.TabLayout { void setTabsFromPagerAdapter(android.support.v4.view.PagerAdapter); }', but not the descriptor class 'android.support.v4.view.PagerAdapter'
Note: the configuration keeps the entry point 'android.support.v4.view.ViewPager { void setAdapter(android.support.v4.view.PagerAdapter); }', but not the descriptor class 'android.support.v4.view.PagerAdapter'
Note: the configuration keeps the entry point 'android.support.v4.view.ViewPager { void setOnAdapterChangeListener(android.support.v4.view.ViewPager$OnAdapterChangeListener); }', but not the descriptor class 'android.support.v4.view.ViewPager$OnAdapterChangeListener'
Note: the configuration keeps the entry point 'android.support.v4.view.ViewPager { void setOnPageChangeListener(android.support.v4.view.ViewPager$OnPageChangeListener); }', but not the descriptor class 'android.support.v4.view.ViewPager$OnPageChangeListener'
Note: the configuration keeps the entry point 'android.support.v4.widget.DrawerLayout { void setDrawerListener(android.support.v4.widget.DrawerLayout$DrawerListener); }', but not the descriptor class 'android.support.v4.widget.DrawerLayout$DrawerListener'
Note: the configuration keeps the entry point 'android.support.v4.widget.NestedScrollView { void setOnScrollChangeListener(android.support.v4.widget.NestedScrollView$OnScrollChangeListener); }', but not the descriptor class 'android.support.v4.widget.NestedScrollView$OnScrollChangeListener'
Note: the configuration keeps the entry point 'android.support.v4.widget.SlidingPaneLayout { void setPanelSlideListener(android.support.v4.widget.SlidingPaneLayout$PanelSlideListener); }', but not the descriptor class 'android.support.v4.widget.SlidingPaneLayout$PanelSlideListener'
Note: the configuration keeps the entry point 'android.support.v4.widget.SwipeRefreshLayout { void setOnRefreshListener(android.support.v4.widget.SwipeRefreshLayout$OnRefreshListener); }', but not the descriptor class 'android.support.v4.widget.SwipeRefreshLayout$OnRefreshListener'
Note: the configuration keeps the entry point 'android.support.v7.view.menu.ActionMenuItemView { void setItemInvoker(android.support.v7.view.menu.MenuBuilder$ItemInvoker); }', but not the descriptor class 'android.support.v7.view.menu.MenuBuilder$ItemInvoker'
Note: the configuration keeps the entry point 'android.support.v7.view.menu.ActionMenuItemView { void setPopupCallback(android.support.v7.view.menu.ActionMenuItemView$PopupCallback); }', but not the descriptor class 'android.support.v7.view.menu.ActionMenuItemView$PopupCallback'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActionBarContainer { void setTabContainer(android.support.v7.widget.ScrollingTabContainerView); }', but not the descriptor class 'android.support.v7.widget.ScrollingTabContainerView'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActionBarOverlayLayout { void setActionBarVisibilityCallback(android.support.v7.widget.ActionBarOverlayLayout$ActionBarVisibilityCallback); }', but not the descriptor class 'android.support.v7.widget.ActionBarOverlayLayout$ActionBarVisibilityCallback'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActionMenuView { void setPresenter(android.support.v7.widget.ActionMenuPresenter); }', but not the descriptor class 'android.support.v7.widget.ActionMenuPresenter'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActionMenuView { void setOnMenuItemClickListener(android.support.v7.widget.ActionMenuView$OnMenuItemClickListener); }', but not the descriptor class 'android.support.v7.widget.ActionMenuView$OnMenuItemClickListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActivityChooserView { void setActivityChooserModel(android.support.v7.widget.ActivityChooserModel); }', but not the descriptor class 'android.support.v7.widget.ActivityChooserModel'
Note: the configuration keeps the entry point 'android.support.v7.widget.ActivityChooserView { void setProvider(android.support.v4.view.ActionProvider); }', but not the descriptor class 'android.support.v4.view.ActionProvider'
Note: the configuration keeps the entry point 'android.support.v7.widget.ContentFrameLayout { void setAttachListener(android.support.v7.widget.ContentFrameLayout$OnAttachListener); }', but not the descriptor class 'android.support.v7.widget.ContentFrameLayout$OnAttachListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.FitWindowsFrameLayout { void setOnFitSystemWindowsListener(android.support.v7.widget.FitWindowsViewGroup$OnFitSystemWindowsListener); }', but not the descriptor class 'android.support.v7.widget.FitWindowsViewGroup$OnFitSystemWindowsListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.FitWindowsLinearLayout { void setOnFitSystemWindowsListener(android.support.v7.widget.FitWindowsViewGroup$OnFitSystemWindowsListener); }', but not the descriptor class 'android.support.v7.widget.FitWindowsViewGroup$OnFitSystemWindowsListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setAdapter(android.support.v7.widget.RecyclerView$Adapter); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$Adapter'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setRecyclerListener(android.support.v7.widget.RecyclerView$RecyclerListener); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$RecyclerListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setLayoutManager(android.support.v7.widget.RecyclerView$LayoutManager); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$LayoutManager'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setRecycledViewPool(android.support.v7.widget.RecyclerView$RecycledViewPool); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$RecycledViewPool'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setViewCacheExtension(android.support.v7.widget.RecyclerView$ViewCacheExtension); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$ViewCacheExtension'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setChildDrawingOrderCallback(android.support.v7.widget.RecyclerView$ChildDrawingOrderCallback); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$ChildDrawingOrderCallback'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setOnScrollListener(android.support.v7.widget.RecyclerView$OnScrollListener); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$OnScrollListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.RecyclerView { void setItemAnimator(android.support.v7.widget.RecyclerView$ItemAnimator); }', but not the descriptor class 'android.support.v7.widget.RecyclerView$ItemAnimator'
Note: the configuration keeps the entry point 'android.support.v7.widget.SearchView { void setOnQueryTextListener(android.support.v7.widget.SearchView$OnQueryTextListener); }', but not the descriptor class 'android.support.v7.widget.SearchView$OnQueryTextListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.SearchView { void setOnCloseListener(android.support.v7.widget.SearchView$OnCloseListener); }', but not the descriptor class 'android.support.v7.widget.SearchView$OnCloseListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.SearchView { void setOnSuggestionListener(android.support.v7.widget.SearchView$OnSuggestionListener); }', but not the descriptor class 'android.support.v7.widget.SearchView$OnSuggestionListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.SearchView$SearchAutoComplete { void setSearchView(android.support.v7.widget.SearchView); }', but not the descriptor class 'android.support.v7.widget.SearchView'
Note: the configuration keeps the entry point 'android.support.v7.widget.Toolbar { void setOnMenuItemClickListener(android.support.v7.widget.Toolbar$OnMenuItemClickListener); }', but not the descriptor class 'android.support.v7.widget.Toolbar$OnMenuItemClickListener'
Note: the configuration keeps the entry point 'android.support.v7.widget.ViewStubCompat { void setOnInflateListener(android.support.v7.widget.ViewStubCompat$OnInflateListener); }', but not the descriptor class 'android.support.v7.widget.ViewStubCompat$OnInflateListener'
Note: the configuration keeps the entry point 'butterknife.ButterKnife$Finder { ButterKnife$Finder(java.lang.String,int,butterknife.ButterKnife$1); }', but not the descriptor class 'butterknife.ButterKnife$1'
Note: the configuration keeps the entry point 'com.bumptech.glide.load.engine.executor.FifoPriorityThreadPoolExecutor$UncaughtThrowableStrategy { FifoPriorityThreadPoolExecutor$UncaughtThrowableStrategy(java.lang.String,int,com.bumptech.glide.load.engine.executor.FifoPriorityThreadPoolExecutor$1); }', but not the descriptor class 'com.bumptech.glide.load.engine.executor.FifoPriorityThreadPoolExecutor$1'
Note: the configuration keeps the entry point 'com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitionsComparator { int compare(com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions,com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions); }', but not the descriptor class 'com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions'
Note: the configuration keeps the entry point 'com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitionsComparator { int compare(com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions,com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions); }', but not the descriptor class 'com.google.zxing.datamatrix.detector.Detector$ResultPointsAndTransitions'
Note: the configuration keeps the entry point 'com.google.zxing.multi.qrcode.QRCodeMultiReader$SAComparator { int compare(com.google.zxing.Result,com.google.zxing.Result); }', but not the descriptor class 'com.google.zxing.Result'
Note: the configuration keeps the entry point 'com.google.zxing.multi.qrcode.QRCodeMultiReader$SAComparator { int compare(com.google.zxing.Result,com.google.zxing.Result); }', but not the descriptor class 'com.google.zxing.Result'
Note: the configuration keeps the entry point 'com.google.zxing.multi.qrcode.detector.MultiFinderPatternFinder$ModuleSizeComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration keeps the entry point 'com.google.zxing.multi.qrcode.detector.MultiFinderPatternFinder$ModuleSizeComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration keeps the entry point 'com.google.zxing.qrcode.decoder.Mode { int getCharacterCountBits(com.google.zxing.qrcode.decoder.Version); }', but not the descriptor class 'com.google.zxing.qrcode.decoder.Version'
Note: the configuration keeps the entry point 'com.google.zxing.qrcode.detector.FinderPatternFinder$CenterComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration keeps the entry point 'com.google.zxing.qrcode.detector.FinderPatternFinder$CenterComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration keeps the entry point 'com.google.zxing.qrcode.detector.FinderPatternFinder$FurthestFromAverageComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration keeps the entry point 'com.google.zxing.qrcode.detector.FinderPatternFinder$FurthestFromAverageComparator { int compare(com.google.zxing.qrcode.detector.FinderPattern,com.google.zxing.qrcode.detector.FinderPattern); }', but not the descriptor class 'com.google.zxing.qrcode.detector.FinderPattern'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ActivityInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ApplicationInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ComponentInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ConfigurationInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.FeatureGroupInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.FeatureInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.InstrumentationInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PackageInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PackageInstaller'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PackageItemInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PackageManager'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PackageManager$NameNotFoundException'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PathPermission'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PermissionGroupInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.PermissionInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ProviderInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ResolveInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.ServiceInfo'
Note: the configuration explicitly specifies 'android.content.pm.**' to keep library class 'android.content.pm.Signature'
Note: there were 6 references to unknown classes.
      You should check your configuration for typos.
      (http://proguard.sourceforge.net/manual/troubleshooting.html#unknownclass)
Note: there were 2 classes trying to access enclosing classes using reflection.
      You should consider keeping the inner classes attributes
      (using '-keepattributes InnerClasses').
      (http://proguard.sourceforge.net/manual/troubleshooting.html#attributes)
Note: there were 51 unkept descriptor classes in kept class members.
      You should consider explicitly keeping the mentioned classes
      (using '-keep').
      (http://proguard.sourceforge.net/manual/troubleshooting.html#descriptorclass)
Note: there were 19 library classes explicitly being kept.
      You don't need to keep library classes; they are already left unchanged.
      (http://proguard.sourceforge.net/manual/troubleshooting.html#libraryclass)
Note: there were 14 unresolved dynamic references to classes or interfaces.
      You should check if you need to specify additional program jars.
      (http://proguard.sourceforge.net/manual/troubleshooting.html#dynamicalclass)
Warning:there were 254 unresolved references to classes or interfaces.
         You may need to add missing library jars or update their versions.
         If your code works fine without the missing classes, you can suppress
         the warnings with '-dontwarn' options.
         (http://proguard.sourceforge.net/manual/troubleshooting.html#unresolvedclass)
Warning:there were 1 unresolved references to program class members.
         Your input classes appear to be inconsistent.
         You may need to recompile the code.
         (http://proguard.sourceforge.net/manual/troubleshooting.html#unresolvedprogramclassmember)
Warning:there were 1 unresolved references to library class members.
         You probably need to update the library versions.
         (http://proguard.sourceforge.net/manual/troubleshooting.html#unresolvedlibraryclassmember)
Warning:Exception while processing task java.io.IOException: Please correct the above warnings first.
:sample:transformClassesAndResourcesWithProguardForRelease FAILED
Error:Execution failed for task ':sample:transformClassesAndResourcesWithProguardForRelease'.
> java.io.IOException: Please correct the above warnings first.
Information:BUILD FAILED
Information:Total time: 13.136 secs
Information:1 error
Information:191 warnings
Information:See complete output in console
