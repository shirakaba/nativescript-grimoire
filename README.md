# Background

## What is this?

These are snippets of JavaScript, assuming NativeScript bindings to the iOS native APIs, that allow one to invoke native iOS functionality. 📲

Equivalent (or similar) Swift code is provided where available. NativeScript is based on Obj-C interfaces, but I show Swift because I don't have any familiarity with Obj-C 🤷‍♂️

You can try these out by building a new NativeScript project. But I myself use [NS:IDE](https://github.com/shirakaba/nside), my open-source NativeScript run-time IDE.

<p align="center">
    <a href="https://twitter.com/intent/follow?screen_name=LinguaBrowse">
        <img src="https://img.shields.io/twitter/follow/LinguaBrowse.svg?style=social&logo=twitter">
    </a>
</p>

## How do you discover these snippets?

First, I write the desired native code in Swift. Then I try rewriting that same code line-by-line my [NativeScript Playground for iOS](https://shirakaba.github.io/NSIDE/ios/index.html), which provides intellisense based on the incredible [`tns-platform-declarations`](https://github.com/NativeScript/NativeScript/tree/master/tns-platform-declarations/ios/objc-x86_64). If I'm ever confused by the typings, I refer to the official Apple API documentation (ideally with 'Objective-C' selected as the language, which maps better to the NativeScript bindings) helps a lot here.

You can of course also start a TypeScript project of your own and use those platform declarations to get Intellisense. I just use NS:IDE to skip the source-rebuilding step (NS:IDE allows you to test the code at run-time).

## Invaluable docs

* https://docs.nativescript.org/core-concepts/ios-runtime/types/ObjC-Classes
* https://docs.nativescript.org/core-concepts/ios-runtime/how-to/ObjC-Subclassing
* https://github.com/NativeScript/NativeScript/tree/master/tns-platform-declarations/ios/objc-x86_64

# Examples

Now all moved into individual `README.md` files under folders of a corresponding name at the root of this repository. For example, [`accelerometer/README.md`](https://github.com/shirakaba/nativescript-grimoire/blob/master/accelerometer/README.md).
