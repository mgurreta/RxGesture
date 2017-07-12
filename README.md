# RxGesture

[![Version](https://img.shields.io/cocoapods/v/RxGesture.svg?style=flat)](http://cocoapods.org/pods/RxGesture)
[![License](https://img.shields.io/cocoapods/l/RxGesture.svg?style=flat)](http://cocoapods.org/pods/RxGesture)
[![Platform](https://img.shields.io/cocoapods/p/RxGesture.svg?style=flat)](http://cocoapods.org/pods/RxGesture)

## Usage

![](Pod/Assets/demo.gif)

To run the example project, clone the repo, in the __Example__ folder open `RxGesture.xcworkspace`.

You _might_ need to run `pod install` from the Example directory first.

---

__RxGesture__ allows you to easily turn any view into a tappable or swipeable control like so:

```swift
view.rx.tapGesture().when(.recognized).subscribe(onNext: {_ in
   //react to taps
}).addDisposableTo(stepBag)
```

You can also react to more than one  gesture. For example to dismiss a photo preview you might want to do that when the user taps it, or swipes up or down:

```swift
view.rx.anyGesture(.tap(), .swipe([.up, .down])).when(.recognized).subscribe(onNext: {_ in
   //dismiss presented photo
}).addDisposableTo(stepBag)
```

`rx.gesture` is defined as `Observable<G>` where `G` is the actual type of the gesture recognizer so what it emits is the gesture recognizer itself (handy if want to call methods like `asLocation(in view:)` or `asTranslation(in view:)`)


#### On iOS, RxGesture supports:

```swift
view.rx.tapGesture()           -> ControlEvent<UITapGestureRecognizer>
view.rx.pinchGesture()         -> ControlEvent<UIPinchGestureRecognizer>
view.rx.swipeGesture(.left)    -> ControlEvent<UISwipeGestureRecognizer>
view.rx.panGesture()           -> ControlEvent<UIPanGestureRecognizer>
view.rx.longPressGesture()     -> ControlEvent<UILongPressGestureRecognizer>
view.rx.rotationGesture()      -> ControlEvent<UIRotationGestureRecognizer>
view.rx.screenEdgePanGesture() -> ControlEvent<UIScreenEdgePanGestureRecognizer>

view.rx.anyGesture(.tap(), ...)           -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.pinch(), ...)         -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.swipe(.left), ...)    -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.pan(), ...)           -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.longPress(), ...)     -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.rotation(), ...)      -> ControlEvent<UIGestureRecognizer>
view.rx.anyGesture(.screenEdgePan(), ...) -> ControlEvent<UIGestureRecognizer>
```

#### On macOS, RxGesture supports:

```swift
view.rx.clickGesture()         -> ControlEvent<NSClickGestureRecognizer>
view.rx.rightClickGesture()    -> ControlEvent<NSClickGestureRecognizer>
view.rx.panGesture()           -> ControlEvent<NSPanGestureRecognizer>
view.rx.pressGesture()         -> ControlEvent<NSPressGestureRecognizer>
view.rx.rotationGesture()      -> ControlEvent<NSRotationGestureRecognizer>
view.rx.magnificationGesture() -> ControlEvent<NSMagnificationGestureRecognizer>

view.rx.anyGesture(.click(), ...)         -> ControlEvent<NSGestureRecognizer>
view.rx.anyGesture(.rightClick(), ...)    -> ControlEvent<NSGestureRecognizer>
view.rx.anyGesture(.pan(), ...)           -> ControlEvent<NSGestureRecognizer>
view.rx.anyGesture(.press(), ...)         -> ControlEvent<NSGestureRecognizer>
view.rx.anyGesture(.rotation(), ...)      -> ControlEvent<NSGestureRecognizer>
view.rx.anyGesture(.magnification(), ...) -> ControlEvent<NSGestureRecognizer>
```


ℹ️ If you use a gesture recognizer alone, prefer the `view.rx.fooGesture()` syntax over `view.rx.anyGesture(.foo())` because it returns the concrete `UIGestureRecognizer` subclass and avoid you to cast it in `subscribe()`.


## Filtering State

By default, there is no filter on the state of the gesture recognizer. That means that you will always receive a first event with the initial state of the gesture recognizer (almost always `.possible`).

Here are the preferred states that can be used for each kind of gestures (__iOS__ and __macOS__):

Kind | States
---|---
`.tap()`, `.click()`, `.rightClick()`, `.swipe()`| `.recognized`
`.longPress()`, `.press()`, | `.began`
`.pan()`, `.pinch()`, `.rotation()`, `.magnification()`, `.screenEdgePan()` | `.began`, `.changed`, `.ended`

You usually filter the state using the `.when()` operator:
```swift
view.rx.tapGesture().when(.recognized)
view.rx.panGesture().when(.began, .changed, .ended)
```

If you are observing multiple gestures at once, you can use the `.when()` operator if you want to filter against the same state for __all__ gesture recognizers, or use the tuple syntax for individual filtering:

```swift
view.rx
	.anyGesture(.tap(), .swipe([.up, .down]))
	.when(.recognized)
	.subscribe(onNext: { gesture in
		// Called whenever a tap, a swipe-up or a swipe-down is recognized (state == .recognized)
	})
	.addDisposableTo(bag)
	
view.rx
	.anyGesture(
		(.tap(), when: .recognized),
		(.pan(), when: .ended)
	)
	.subscribe(onNext: { gesture in
		// Called whenever:
		// - a tap is recognized (state == .recognized) 
		// - or a pan is ended (state == .ended)
	})
	.addDisposableTo(bag)
```


__The demo app includes examples for all recognizers ➡️ [iOS](Example/RxGesture/ViewController.swift), [macOS](Example/RxGesture-OSX/ViewController.swift)__.

## Delegate customization
### Lightweight customization
Each gesture recognizer has a default RxGestureRecognizerDelegate.
For now it only provides a way to customize the result of `gestureRecognizer(_:shouldRecognizeSimultaneouslyWith:)` by using a policy:
```swift
view.rx.tapGesture(configure: { gestureRecognizer, delegate in 
    delegate.simultaneousRecognitionPolicy = .allow // (default value)
    // or
    delegate.simultaneousRecognitionPolicy = .forbid
    // or
    delegate.simultaneousRecognitionPolicy = .custom { gesture, otherGesture in
    	return otherGesture is UIPanGestureRecognized
	}
})
```
### Full customization
You can also replace the default delegate by your own, or remove it.
```swift
view.rx.tapGesture(configure: { [unowned self] gestureRecognizer, delegate in 
    gestureRecognizer.delegate = nil
    // or
    gestureRecognizer.delegate = self
	}
})
```

## Requirements

This library depends on both __RxSwift__ and __RxCocoa__.


## Installation

RxGesture is available through [CocoaPods](http://cocoapods.org). To install
it, simply add the following line to your Podfile:

```swift
pod "RxGesture"
```

## TODO

- [ ] add tests 
- [ ] make pr to RxCocoa to add native support for rx.event to `NSGestureRecognizer` and remove the implementation from this repo

## Thanks

Everyone in the RxSwift Slack channel 💯

## License

RxGesture is available under the MIT license. See the LICENSE file for more info.
