# SDK Android Beta Documentation

# Bridgefy

## Overview

The Bridgefy Software Development Kit (SDK) is a state-of-the-art, plug-and-play package of awesomeness that will let people use your mobile app when they don’t have access to the Internet by using mesh networks.

Integrate the Bridgefy SDK into your Android and iOS app to reach the 3.5 billion people that don’t always have access to an Internet connection, and watch engagement and revenue grow!

**Website**. https://bridgefy.me/sdk/

 **Email**. contact@bridgefy.me **T**

**witter**. https://twitter.com/bridgefy 

**Facebook**. https://www.facebook.com/bridgefy

## Operation mode

All the connections are handled seamlessly by the SDK to create a mesh network. The size of this network depends on the number of devices connected and the environment as a variable factor, allowing you to join nodes in the same network or nodes in different networks.

![https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true](https://github.com/bridgefy/bridgefy-ios-sdk-sample/blob/master/img/Mobile_Adhoc_Network.gif?raw=true)

## Usage

### Start the SDK

The following code shows how to start the SDK (using your API key) and how to assign the delegate. By default the SDK starts using the **Standard propagation profile** mode

```swift
func start(withAPIKey apiKey: String,
              propagationProfile: PropagationProfile = .standard,
              delegate: BridgefyDelegate,
              verboseLogging: Bool = false) throws {
    }
```

The string **“apiKey”** represents a valid API key. An Internet connection is needed at least for the first time in order to validate the license.

To stop it, use the following code:

```swift
 func stop()

```

### Nearby peer detection

The following method is invoked when a peer has established connection:

```swift
func bridgefyDidConnect(withUserID userID: String)
```

**userID**: Identifier of the user that has established a connection.

When a peer is disconnected(out of range), the following method will be invoked:

```swift
func bridgefyDidDisconnect(fromUserID userID: String)
```

**userID**: Identifier of the disconnected user.

### Send data

The following method is used to send data using a transmission mode. This method returns the message id (**messageID**) to the client.

```swift
 func send(_ data: Data,
           using transmissionMode: TransmissionMode) throws -> String
```

**messageID**: Unique identifier related to the message.

**Transmission Modes**:

```swift
enum TransmissionMode {
    case p2p(userID: String)
    case mesh(userID: String)
    case broadcast
}
```

There are several modes for sending packets:

**p2p(userID: String)**: Sends the packet only when the receiver is in range. **mesh(userID: String)**: Sends the packet using mesh. It doesn’t need the receiver to be in range. **broadcast**: Sends a packet using mesh without a defined receiver. The packet is broadcast to all nearby users that are or aren’t in range.

If there is no error when sending the message, the following method will be received with the message id (**messageID**)

```swift
func bridgefyDidSend(_ messageID: String)
```

otherwise, the following method will be received

```swift
func bridgefyDidFailToSend(_ messageID: String)
```

### Direct and mesh transmission

Direct transmission is a mechanism used to deliver packets to a user that is nearby or visible (a connection has been detected).

Mesh transmission is a mechanism used to deliver offline packets even when the receiving user isn’t nearby or visible. It can be achieved taking advantage of other nearby peers; these receive the package, hold it, and forward to other peers trying to find the receiver.

A message can be transmitted using mesh transmission, direct transmission, or both. ### Receive Data When a packet has been received, the following method will be invoked:

```swift
func bridgefyDidReceive(_ data: Data,
                        with messageID: String,
                        through transmissionMode: TransmissionMode)
```

**data**: Received Data object **messageID**: Unique identifier related to the message. **transmissionMode**: The transmission mode used when sending the message

### Propagation Profiles

```swift
enum PropagationProfile {
    case standard
    case highDensityNetwork
    case sparseNetwork
    case longReach
    case shortReach
}
```

| **Profile** | **Hops limit** | **TTL(s)** | **Sharing Time** | **Maximum Propagation** | **Tracklist limit** |
|---|---|---|---|---|---|
| Standard | 100 | 86400 (1 d) | 15000 | 200 | 50 |
| High Density Network | 50 | 3600 (1 h) | 10000 | 50 | 50 |
| Sparse Network | 100 | 302400 (3.5 d) | 10000 | 250 | 50 |
| Long Reach | 250 | 604800 (7 d) | 15000 | 1000 | 50 |
| Short Reach | 0 | 0 | 0 | 0 | 0 |

- **Hops limit:** The maximum number of hops a message can get. Each time a message is forwarded, is considered a hop.
- **TTL:** Time to live, is the maximum amount of time a message can be propagated since its creation.
- **Sharing time:** The maximum amount of time a message will be kept for forwarding.
- **Maximum propagation:** The maximum number of times a message will be forwarded from a device.
- **Tracklist limit:** The maximum number of UUID’s stored in an array to prevent sending the message to a peer which already forwarded the message.
