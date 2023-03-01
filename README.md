# SDK Android Beta Documentation

> :bangbang: **The Bridgefy SDK** currently supports **P2P connections**, but we're adding **Mesh Networking, Broadcast, and Signal Protocol** encryption by mid-October! You can begin integrating the current version and add the updates as they're published.

## Overview

The Bridgefy Software Development Kit (SDK) is a state-of-the-art, plug-and-play package of awesomeness that will let people use your mobile app when they don’t have access to the Internet by using mesh networks.

Integrate the Bridgefy SDK into your Android and iOS app to reach the 3.5 billion people that don’t always have access to an Internet connection, and watch engagement and revenue grow!

**Website**. https://beta.bridgefy.me

**Email**. contact@bridgefy.me

**witter**. https://twitter.com/bridgefy

**Facebook**. https://www.facebook.com/bridgefy

## Operation mode

All the connections are handled seamlessly by the SDK to create a mesh network. The size of this network depends on the number of devices connected and the environment as a variable factor, allowing you to join nodes in the same network or nodes in different networks.

![https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true](https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true)

## Permissions
Android requires additional permissions declared in the manifest for an app to run a BLE scan since API 23 (6.0 / Marshmallow) and perform a Bluetooth Low Energy connection since API 31 (Android 12). These permissions currently assume scanning is only used when the App is in the foreground, and that the App wants to derive the user's location from Bluetooth Low Energy signal (on API >= 23). Below are a number of additions you can make to your `AndroidManifext.xml` for your specific use case.

#### Location permission for Bluetooth Low Energy Scanning
Bridgefy uses the `uses-permission-sdk-23` tag to require location only on APIs >= 23, require to scan for BLE peripherals and do not access location otherwise you can request only the required permissions by adding the following to your `AndroidManifest.xml`:
```xml
<uses-permission-sdk-23 android:name="android.permission.ACCESS_COARSE_LOCATION" android:maxSdkVersion="30" tools:node="replace" />
<uses-permission-sdk-23 android:name="android.permission.ACCESS_FINE_LOCATION" android:maxSdkVersion="30" tools:node="replace" />
```

#### Scan in the background and support APIs 29 & 30
You should add the following to your `AndroidManifest.xml`:
```xml
<uses-permission android:name="android.permission.ACCESS_BACKGROUND_LOCATION" android:maxSdkVersion="30" />
```
If you want to access the user's location in the background on APIs > 30, remove the `android:maxSdkVersion` attribute.

#### Location from BLE scanning in API >= 31

API 31 (Android 12) introduced new Bluetooth permissions. Bridgefy uses the `android:usesPermissionFlags="neverForLocation"` attribute on the `BLUETOOTH_SCAN` permission, which indicates scanning will not be used to derive the user's location, so location permissions are not required. If you need to locate the user with BLE scanning, use this instead, but keep in mind that you will still need `ACCESS_FINE_LOCATION`:
```xml
<uses-permission android:name="android.permission.BLUETOOTH_SCAN" tools:node="replace" />
```

#### Connect peripherals
You add the `BLUETOOTH_CONNECT` permission that Bridgefy requests in APIs >= 31:
```xml
<uses-permission android:name="android.permission.BLUETOOTH_CONNECT" />
```

#### Summary of available permissions
##### Scanning
A summary of available runtime permissions used for BLE scanning:

| from API | to API (inclusive) | Acceptable runtime permissions |
|:---:|:---:| --- |
| 18 | 22 | (No runtime permissions needed) |
| 23 | 28 | One of below:<br>- `android.permission.ACCESS_COARSE_LOCATION`<br>- `android.permission.ACCESS_FINE_LOCATION` |
| 29 | 30 | - `android.permission.ACCESS_FINE_LOCATION`<br>- `android.permission.ACCESS_BACKGROUND_LOCATION`\* |
| 31 | current | - `android.permission.BLUETOOTH_SCAN`<br>- `android.permission.ACCESS_FINE_LOCATION`\*\* |

\* Needed if [scan is performed in background](https://developer.android.com/about/versions/10/privacy/changes#app-access-device-location)

## Gradle
Bridgefy BETA version is available in our private repository to install it you must follow the instructions:
```kotlin
val bridgefy_beta_maven_url = "http://104.196.228.98:8081/artifactory/libs-snapshot-local"
val bridgefy_beta_username = "bridgefy-beta"
val bridgefy_beta_password = "g$thaP@ad8w=w=OT78_r"

allprojects {
  repositories {
    maven {
                url = java.net.URI(bridgefy_beta_maven_url)
                authentication {
                    create<BasicAuthentication>("basic")
                }
                credentials {
                    username = bridgefy_beta_username
                    password = bridgefy_beta_password
                }
                isAllowInsecureProtocol = true
            }
    }
}

/**
 * Declare dependencies
 * @see http://www.gradle.org/docs/current/userguide/userguide_single.html#sec:how_to_declare_your_dependencies
 */
dependencies {
  implementation (group = "me.bridgefy", name = "android-sdk", version = "0.3.0-SNAPSHOT", ext = "aar") { 
    isTransitive = true 
  }
}

```


# Usage
### Start the SDK

The following code shows how to start the SDK (using your API key) and how to assign the delegate. By default the SDK starts using the **Standard propagation profile** mode

```kotlin
    /**
     * Init bridgefy SDK
     *
     * @param bridgefyApiKey - API KEY
     * @param propagationProfile - Default PropagationProfile.Standard
     * @param delegate - The delegate object listens for all bridgefy actions.
     */
    fun init(
            bridgefyApiKey: String?,
            propagationProfile: PropagationProfile = PropagationProfile.Standard,
            delegate: BridgefyDelegate?
        )
```

The string **“bridgefyApiKey”** represents a valid API key. An Internet connection is needed at least for the first time in order to validate the license.
To stop it, use the following code:

```kotlin

    /**
     * Stop bridgefy sdk operations
     *
     */
    fun stop()

```

### Nearby peer detection

The following method is invoked when a peer has established connection:

```kotlin
    val delegate: BridgefyDelegate = object : BridgefyDelegate {
    
        override fun onConnected(userID: String) {
            ...
        }
    
    }
```

**userID**: Identifier of the user that has established a connection.
When a peer is disconnected(out of range), the following method will be invoked:

```kotlin
    val delegate: BridgefyDelegate = object : BridgefyDelegate {

        override fun onDisconnected(userID: String) {
            ...
        }
    
    }
```

**userID**: Identifier of the disconnected user.

### Send data

The following method is used to send data using a transmission mode. This method returns the message id (**messageID**) to the client.

```kotlin
    import me.bridgefy.commons.TransmissionMode
    // Bridgefy instance
    val bridgefy: Bridgefy
    
    // After start sdk, send ByteArray
    val messageId = bridgefy.send(
        "Sample text!".toByteArray(Charsets.UTF_8),
        TransmissionMode.P2P(userId)
    )
```

**messageID**: Unique identifier related to the message.

**Transmission Modes**:

```kotlin
sealed class TransmissionMode {
    /**
     * P2p
     *
     * @property receiver ID
     * @constructor Create empty P2p
     */
    data class P2P(val receiver: String) : TransmissionMode()
}
```

There are several modes for sending packets:

**P2P(val receiver: String)**: Sends the packet only when the receiver is in range.
**Mesh(val receiver: String)**: Sends the packet using mesh. It doesn’t need the receiver to be in range.
**Broadcast**: Sends a packet using mesh without a defined receiver. The packet is broadcast to all nearby users that are or aren’t in range.

If there is no error when sending the message, the following method will be received with the message id (**messageID**)

```kotlin
    val delegate: BridgefyDelegate = object : BridgefyDelegate {
        /**
         * On send
         *
         * @param messageID
         */
        override fun onSend(messageID: String) {
            ...
        }
    
    }
```

otherwise, the following method will be received

```kotlin
    val delegate: BridgefyDelegate = object : BridgefyDelegate {
        /**
         * On fail to send
         *
         * @param messageID
         */
        override fun onFailToSend(messageID: String) {
            ...
        }
    }
```

### Direct and Mesh transmission

Direct transmission is a mechanism used to deliver packets to a user that is nearby or visible (a connection has been detected).
Mesh transmission is a mechanism used to deliver offline packets even when the receiving user isn’t nearby or visible. It can be achieved taking advantage of other nearby peers; these receive the package, hold it, and forward to other peers trying to find the receiver.

A message can be transmitted using mesh transmission, direct transmission, or both.
### Receive Data When a packet has been received, the following method will be invoked:

```kotlin
    
    val delegate: BridgefyDelegate = object : BridgefyDelegate {
        /**
         * On receive
         *
         * @param data
         * @param messageID
         * @param transmissionMode
         */
        override fun onReceive(
            data: ByteArray,
            messageID: String,
            transmissionMode: TransmissionMode,
        ) {
            ...
        }   
    }

```

**data**: Received Data object

**messageID**: Unique identifier related to the message.

**transmissionMode**: The transmission mode used when sending the message

### Propagation Profiles

```kotlin
/**
 * Propagation profile
 */
enum class PropagationProfile {
    Standard,
    HighDensityEnvironment,
    SparseEnvironment,
    LongReach,
    ShortReach
}
```

| **Profile** | **Hops limit** | **TTL(s)** | **Sharing Time** | **Maximum Propagation** | **Tracklist limit** |
|---|---|---|---|---|---|
| Standard | 100 | 86400 (1 d) | 15000 | 200 | 50 |
| High Density Environment | 50 | 3600 (1 h) | 10000 | 50 | 50 |
| Sparse Environment | 100 | 302400 (3.5 d) | 10000 | 250 | 50 |
| Long Reach | 250 | 604800 (7 d) | 15000 | 1000 | 50 |
| Short Reach | 0 | 0 | 0 | 0 | 0 |

- **Hops limit:** The maximum number of hops a message can get. Each time a message is forwarded, is considered a hop.
- **TTL:** Time to live, is the maximum amount of time a message can be propagated since its creation.
- **Sharing time:** The maximum amount of time a message will be kept for forwarding.
- **Maximum propagation:** The maximum number of times a message will be forwarded from a device.
- **Tracklist limit:** The maximum number of UUID’s stored in an array to prevent sending the message to a peer which already forwarded the message.
