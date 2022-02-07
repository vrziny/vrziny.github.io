---
layout: single
title: Unity as a Libary for Android
date: 2022-02-07 10:56 +0900
category: Unity
---

Android 프로젝트에서 Unity를 Library 형태로 만드는 방법에 대해 기술합니다.

>**NOTE**
> 이 문서는 Android 12 (API Level 31) 기준으로 작성되었습니다.

D:\Projects\UaalProject이라는 폴더를 만듭니다.
이 폴더에 아래의 구조로 프로젝트를 구성할 것입니다.

```
- UaalProject
  - AndroidProject
  - UnityProject
```

Android Studio를 실행해서 New Project 선택 후 Basic Activity를 선택합니다.

```
Name: AndroidProject
Package name: com.example.androidproject
Save location: D:\Projects\UaalProject\AndroidProject
Language: Java or Kotlin
```

위와 같이 설정한 뒤 Finish를 선택해서 프로젝트를 생성합니다.

Unity Hub를 실행해서 새 프로젝트를 선택합니다.

```
에디터 버전: 2019.4.x (현시점 기준 아직 LTS)
템플릿: 3D (코어)
프로젝트 이름: UnityProject
위치: D:\Projects\UaalProject (Android Studio와는 약간 다름)
```

위와 같이 새 프로젝트를 설정한 뒤 프로젝트 생성을 선택합니다.

File에서 Build Settings를 선택하고 Add Open Scenes를 눌러줍니다.

Platform에서 Android를 선택하고 Switch Platform을 눌러줍니다.

Player Settings로 이동합니다.

Scripting Backend를 IL2CPP로 바꾸고, Target Architectures의 ARM64를 선택합니다.

Publishing Setting에서 Custom Main Mainfest도 선택해줍니다.

이제, Ctrl+S를 눌러 프로젝트를 저장합니다.

Project창에서 Assets에서 Plugins\Android 폴더를 생성하고 아래의 파일을 생성해줍니다.

파일 생성 방법은 Project창에서 Plugins\Android 폴더로 이동 후 우클릭해서 Show in Explorer를 선택한 다음 새로 만들기>텍스트 문서를 선택해서 원하는 편집기로 파일을 만든 후 Unity로 돌아오면 됩니다.

```java
// 파일명: D:\Projects\UaalProject\UnityProject\Assets\Plugins\Android\OverrideUnityActivity.java

package com.company.product;

import android.os.Bundle;
import android.widget.FrameLayout;
import com.unity3d.player.UnityPlayerActivity;

public abstract class OverrideUnityActivity extends UnityPlayerActivity
{
    public static OverrideUnityActivity instance = null;
    abstract protected void showMainActivity(String setToColor);
    @Override
    protected void onCreate(Bundle savedInstanceState)
    {
        super.onCreate(savedInstanceState);
        instance = this;
    }
    @Override
    protected void onDestroy() {
        super.onDestroy();
        instance = null;
    }
}
```

위에서 Custom Main Manifest를 선택하면 Plugins/Android에 AndroidManifest.xml 파일도 자동으로 생성되었을 것입니다.

아래와 같이 android:exported를 추가해줍니다.
```xml
<?xml version="1.0" encoding="utf-8"?>
<manifest
    xmlns:android="http://schemas.android.com/apk/res/android"
    package="com.unity3d.player"
    xmlns:tools="http://schemas.android.com/tools">
    <application>
        <activity android:name="com.unity3d.player.UnityPlayerActivity"
                  android:exported="true"
                  android:theme="@style/UnityThemeSelector">
            <intent-filter>
                <action android:name="android.intent.action.MAIN" />
                <category android:name="android.intent.category.LAUNCHER" />
            </intent-filter>
            <meta-data android:name="unityplayer.UnityActivity" android:value="true" />
        </activity>
    </application>
</manifest>
 ```

Build Settings로 이동해서 Export Project를 체크하고 Export를 눌러줍니다.

창이 뜨면 D:\Projects\UaalProject\UnityProject 하위에 androidBuild라는 폴더를 만들고 폴더 안으로 들어가서 폴더 선택을 선택합니다.

Android Studio로 돌아갑니다.

D:\Projects\UaalProject\UnityProject\androidBuild\gradle.properties 파일에 아래의 내용을 아래와 같이 업데이트합니다.

```
org.gradle.jvmargs=-Xmx4096M -Dfile.encoding=UTF-8
org.gradle.parallel=true
android.useAndroidX=true
```

D:\Projects\UaalProject\AndroidProject\app\build.gradle 파일에 아래의 내용을 추가합니다.

```groovy
android {
    ...
    defaultConfig {
        ndk {
            abiFilters 'armeabi-v7a', 'x86'
        }
    }
}
...
dependencies {
    ...
    implementation project(':unityLibrary')
    implementation fileTree(dir: project(':unityLibrary').getProjectDir().toString() + ('\\libs'), include: ['*.jar'])
}
```

D:\Projects\UaalProject\AndroidProject\settings.gradle 파일에 아래의 내용으로 업데이트합니다.

```groovy
pluginManagement {
    repositories {
        gradlePluginPortal()
        google()
        mavenCentral()
    }
}

include ':unityLibrary'
project(':unityLibrary').projectDir=new File('..\\UnityProject\\androidBuild\\unityLibrary')

dependencyResolutionManagement {
    repositoriesMode.set(RepositoriesMode.FAIL_ON_PROJECT_REPOS)
    repositories {
        google()
        mavenCentral()
        flatDir {
            dirs "${project(':unityLibrary').projectDir}/libs"
        }
    }
}
rootProject.name = "AndroidProject"
include ':app'

```

gradle 파일 중 아무거나 선택해서 Sync Now를 선택합니다. 문제가 없으면 빌드가 성공했다는 로그가 나올 것입니다. 문제가 생기면 아래 사이트를 참고해서 다시 한번 잘 설정해주세요.

https://github.com/Unity-Technologies/uaal-example/blob/master/docs/android.md

Android Studio에서 app/java/com.example.androidproject를 선택해서 하위에 UnityMainActivity라는 클래스를 생성하고 아래의 내용으로 채워줍니다.

Kotlin
```kotlin
package com.example.androidproject

import android.content.Intent
import com.company.product.OverrideUnityActivity

class MainUnityActivity : OverrideUnityActivity() {
    override fun showMainActivity(setToColor: String) {
        val intent = Intent(this, MainActivity::class.java)
        intent.flags = Intent.FLAG_ACTIVITY_REORDER_TO_FRONT or
                Intent.FLAG_ACTIVITY_SINGLE_TOP
        intent.putExtra("setColor", setToColor)
        startActivity(intent)
    }
}
```
Java
```java
package com.example.androidproject;

import android.content.Intent;
import com.company.product.OverrideUnityActivity;

public class MainUnityActivity extends OverrideUnityActivity {
    @Override
    protected void showMainActivity(String setToColor) {
        Intent intent = new Intent(this, MainActivity.class);
        intent.setFlags(
                Intent.FLAG_ACTIVITY_REORDER_TO_FRONT |
                Intent.FLAG_ACTIVITY_SINGLE_TOP
        );
        intent.putExtra("setColor", setToColor);
        startActivity(intent);
    }
}
```

그리고 MainActivity 클래스를 선택해서 버튼을 누르면 Unity가 실행되도록 아래와 같이 수정합니다.

Kotlin
```kotlin
...
import android.content.Intent;
...
        ...
        binding.fab.setOnClickListener { view ->
            val intent = Intent(view.context, MainUnityActivity::class.java)
            intent.flags = Intent.FLAG_ACTIVITY_REORDER_TO_FRONT
            startActivityForResult(intent, 1)
        }
        ....
```
Java
```java
...
import android.content.Intent;
...
        ...
        binding.fab.setOnClickListener(new View.OnClickListener() {
            @Override
            public void onClick(View view) {
                Intent intent = new Intent(view.getContext(), MainUnityActivity.class);
                intent.setFlags(Intent.FLAG_ACTIVITY_REORDER_TO_FRONT);
                startActivityForResult(intent, 1);
            }
        });
        ....
```

Android Studio에서 app 하위의 res의 values의 strings.xml에 아래의 리소스를 추가해줍니다.

```xml
<resources>
    ...
    <string name="game_view_content_description">Game view</string>
<resources>
```

Android Studio에서 app 하위의 manifests를 선택해서 아래와 같은 activity를 추가해줍니다.

```xml
<manifest>
    <application>
        ...
        <activity
            android:label="@string/app_name"
            android:name=".MainUnityActivity"
            android:exported="true"
            android:screenOrientation="fullSensor"
            android:configChanges="mcc|mnc|locale|touchscreen|keyboard|keyboardHidden|navigation|orientation|screenLayout|uiMode|screenSize|smallestScreenSize|fontScale|layoutDirection|density"
            android:hardwareAccelerated="false"
            android:process=":Unity">
        </activity>
    </application>
</manifest>
```

이제 앱을 실행하고 Floating 버튼을 누르면 Unity가 라이브러리로부터 실행됩니다.
