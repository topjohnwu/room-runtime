# Room-Runtime
[![](https://jitpack.io/v/topjohnwu/room-runtime.svg)](https://jitpack.io/#topjohnwu/room-runtime)

This is a modified version of the AndroidX Room Persistent library, specifically the [room-runtime](https://android.googlesource.com/platform/frameworks/support/+/refs/heads/androidx-room-release/room/runtime/) component.

## Why

The goal is to allow full Proguard/R8 class obfuscation. In official room runtime, this proguard rule `-keep class * extends androidx.room.RoomDatabase` prevents any possible obfuscation for `RoomDatabase` classes. The reason why this rule is needed is because the generated `RoomDatabase` implementation will be found and created via classname reflection at runtime.

## Download

```gradle
android {
    compileOptions {
        sourceCompatibility JavaVersion.VERSION_1_8
        targetCompatibility JavaVersion.VERSION_1_8
    }
}
repositories {
    maven { url 'https://jitpack.io' }
}
dependencies {
    // Remove androidx.room:room-runtime and replace it with the following
    modules {
        module('androidx.room:room-runtime') {
            replacedBy('com.github.topjohnwu:room-runtime')
        }
    }
    implementation "com.github.topjohnwu:room-runtime:${vRoom}"
}
```

## Usage

In order to remove the usage of reflection, you will have to create a `RoomDatabase.Factory` to create database instances. Implement the factory and set it as shown below in static blocks of your `Application` or main `Activity`:

```java
Room.setFactory(clazz -> {
    switch (clazz) {
        case MyRoomDB1.class:
            return new MyRoomDB1_Impl();
        case MyRoomDB2.class:
            return new MyRoomDB2_Impl();
        default:
            return null;
    }
});
```

Or in Kotlin:

```kotlin
Room.setFactory {
    when(it) {
        MyRoomDB1::class.java -> MyRoomDB1_Impl()
        MyRoomDB2::class.java -> MyRoomDB2_Impl()
        else -> null
    }
}
```

Note that your factory class has to handle **ALL** `RoomDatabase` throughout your app. There might be additional `RoomDatabase` used in your dependencies (for example, `WorkManager`). To find out all possible `RoomDatabase` used in your application, the easiest way is to add this to your proguard rules: `-whyareyoukeeping class * extends androidx.room.RoomDatabase`, and build your project before switching to this implementation. It will print out all `RoomDatabase` classes in your project, and you can implement your `RoomDatabase.Factory` accordingly.
