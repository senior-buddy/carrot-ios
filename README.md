<p align="center">
<img src="https://github.com/carrot-ar/carrot-ios/wiki/resources/Carrot@2x.png" alt="Carrot" width="300">
</p>
<p align="center">
    <a href="https://travis-ci.org/carrot-ar/carrot-ios">
        <img src="https://travis-ci.org/carrot-ar/carrot-ios.svg?branch=master" />
    </a>
    <img src="https://img.shields.io/badge/Swift-4.0-orange.svg" />
</p>

Carrot is an easy-to-use, real-time framework for building applications with multi-device AR capabilities. It works using WebSockets, Golang on the server-side, and a unique location tracking system based on iBeacons that we aptly named The Picnic Protocol. Using Carrot, multi-device AR apps can be created with high accuracy location tracking to provide rich and lifelike experiences. To see for yourself, check out Scribbles, a multiplayer drawing application made with Carrot. You can see a demo video [here](https://www.youtube.com/watch?v=6EVtb0pJPgk) and the code [here](https://github.com/carrot-ar/scribbles-ios).

|    | 🗂 Table of Contents |
|:--:|----------------------
| ✨ | [Features](#features)
| 📋 | [To-Do](#to-do)
| 🥗 | [The Picnic Protocol](#the-picnic-protocol)
| 🌎 | [Sessions](#sessions)
| ✉️ | [Messages](#messages)
| 🎙 | [Sending Messages to Carrot](#sending-messages-to-carrot)
| 📨 | [Receiving Messages from Carrot](#receiving-messages-from-carrot)

## ✨ Features

## 📋 To-Do

## 🥗 The Picnic Protocol

## 🌎 Sessions

Sessions are central to Carrot apps. A session is responsible for two things:

1. Managing authentication via some underlying protocol.
2. Providing a clean interface to the WebSocket used to relay messages to/from the Carrot server.

There are two types of sessions in Carrot: `CarrotSession` and `CustomCarrotSession`. The difference between the two lies in the underlying protocol: 

| Session | Protocol |
| ---------------------- | ----------------
| `CarrotSession`        | `PicnicProtocol`
| `CustomCarrotSession`  | Custom protocol conforming to `SessionDriver`

### CarrotSession

### CustomCarrotSession

#### SessionDriver

## ✉️ Messages

Messages in Carrot are how information about events gets encoded and packaged for the server-side to broadcast to other clients in the same session. In Swift, they are represented by the `Message<T: Codable>` struct:

```swift
public struct Message<T: Codable> {
  public var transform: matrix_float4x4?
  public var object: T
}
```

The `transform` property can be used to encode information about the position, orientation, and scale of objects pertaining to these encoded events, just like the corresponding property on [ARAnchor](https://developer.apple.com/documentation/arkit/aranchor/2867981-transform) in ARKit.

The generic `object` parameter allows you, the developer, to package any `Codable` information within a `Message`. This works nicely not only with `Codable` primitives like `String`, `Bool`, and `Int` but also with custom `Codable` classes, structs, and enums for example. Let's take a look at how this works in practice.

#### 💡 Tip

Using `enum` cases with associated values, you can describe different types of events within the same type, as long as the associated values are `Codable`. This allows you to represent the placement of different basic geometry nodes in an `ARSKView` with only one type (the custom conformance to `Codable` has been omitted for brevity):

```swift
enum Event: Codable {
  case label(String)
  case sphere(Int)
}

let message: Message<Event> = ...
```

[Little Bites of Cocoa #318](https://littlebitesofcocoa.com/318-codable-enums) is a great tutorial on how to make an `enum` with associated values conform to `Codable`.

### Example

Let's say you want your app to communicate to other clients that you've placed an `SKLabelNode` somewhere in your `ARSKView`, for example. This would allow you to broadcast this information to other devices and build a multi-device AR experience instead of a just standard and boring one 😛

You'd be able to use the following `struct` to encode all the information about an `SKLabelNode`:

```swift
struct LabelEvent: Codable {
  var text: String
}
```

Now, you'd be able to construct a `Message<LabelEvent>` after you've created the `ARAnchor` that represents the position and orientation of this `SKLabelNode` that's been placed by the user:

```swift
let anchor = ARAnchor(transform: sceneView.session.currentFrame!.camera.transform)
let message = Message(transform: anchor.transform, object: LabelEvent(text: "🥕"))
// Send the message via my `CarrotSession`
```

Upon receiving the message, another client would be able to decode it and have all the information necessary to render the `SKLabelNode`. We'll go over both the sending and receiving of messages in the following sections.

## 🎙 Sending Messages to Carrot

## 📨 Receiving Messages from Carrot
