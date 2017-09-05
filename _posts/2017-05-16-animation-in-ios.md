---
layout: post
title: "Animation in iOS"
description: ""
category: iOS
tags: [animation]
---
{% include JB/setup %}

### View Animations vs Layer Animations


### View Animations


### Layer Animations

#### CABasicAnimation

动画不绑定到指定的 `Layer`，每次调用 `add(anim: CAAnimation>, forKey: String?)` 都会生成一份该动画的拷贝，并应用到当前 `Layer` 上。

动画本质上并不是改变 `View` 的真实状态，而只是改变 `View` 的一个缓存版本 **Presentation Layer**，当动画开始时，隐藏 `View`，当动画结束时，显示 `View`。

1. 如果当前视图所处的状态即动画的最终状态(如: 位置，透明度)，则需要指定 `fillMode = kCAFillModeBackwards` 在动画被添加到 `Layer` 后，不管真实的开始时间是什么，动画的第一帧都会立即执行，从而保证不会先看到视图的最终状态，然后再执行动画。
2. 如果设置视图的初始状态为动画的初始状态，在将动画添加到视图之后，需要设置视图的最终状态为动画的最终状态，以保证真实的视图状态发生变化。

当动画执行结束后，如果要保留动画的最后一帧在视图上，需要同时设置 `isRemovedOnCompletion = false` 和 `fillMode = kCAFillModeForwards` 或 `fillMode = kCAFillModeBoth`

> 由于上面提到动画的是 `Presentation Layer`，因此保留动画的最后一帧时，当前显示视图并不是真实的视图，因此失去交互性，无法进行如激活输入框等操作

#### CAAnimationDelegate

使用 `UIKit` 动画，你无法在创建动画之后，暂停、结束甚至访问它；而使用 **Core Animation**，你可以很容易的结束一个动画，甚至可以为动画设置一个委托，观察动画执行的状态。

可以通过设置 `delegate` 检测动画的开始和结束状态，由于动画可以被添加到多个 `Layer`，因此如果我们需要区分当前的 `Layer`，需要在动画被添加到 `Layer` 之前，通过如 `setValue(layer, forKey: "layer")` 的方式存储对当前 `Layer` 的引用，并在 `delegate` 对应的方法中获取引用。在动画执行的过程中无法对动画对象进行修改。

#### CAAnimationGroup

`CAAnimationGroup` 的属性设置会应用到组内的所有动画上， `animations` 是不可变的，需要一次即完全赋值。动画缓冲(Animation easing)包含缓入、缓出、缓入和缓出，如通过代码来实现缓入效果:

```swift
timingFunction = CAMediaTimingFunction(name: kCAMediaTimingFunctionEaseIn)
```
也可以通过 `CAMediaTimingFunction(controlPoints...)` 构造来自定义缓冲点。

> 缓入：由慢到快；缓出：由快到慢；缓入缓出：由慢到快再到慢；

#### More timing options

当 `repeatCount` 与 `autoreverse` 共同使用时，由于使用 `autoreverse` 会导致视图回到起点状态，可以通过设置 `repeatCount = 1.5` 的方式来实现动画停止在最终位置。可以通过设置动画的 `speed` 属性控制动画的速度，也可以直接设置 `Layer` 的 `speed` 属性来控制速度，但是注意：最终速度取决于 `Layer` 层级和动画上速度值的乘积。

#### Layer Springs

使用支持弹性的 **UIView.animate()** 方法时，你需要传入 `duration` 参数，`UIKit` 负责根据传入的时长控制其它的参数值，因此有时候这种方式的弹性动画会略显突兀。而使用 **Core Animation** 的 `CASpringAnimation` 类，你可以指定关于弹性配置的所有参数 `damping`(默认 10.0)、`mass`(默认 1.0)、`stiffness`(默认 100.0)、`initialVelocity`(默认 0.0)，最后将 `settlingDuration`(根据参数配置获取动画过程的时长) 值赋值给 `duration`。

想象着你在推一个人荡秋千，`initialVelocity`(初始速度) 值越大会让秋千上的人往前荡的更远，`mass`(质量) 的值越大会让秋千向后荡的更远，`stiffness`(刚度) 的值越大越使得秋千来回荡的速度更快，`damping`(阻尼)越大秋千来回荡的次数也就越少。

#### Keyframe Animations

**View keyframe animations** 可以让你结合不同的动画，动画不同的视图属性，以及允许动画之间有重叠或者间隔。 **Layer keyframe animations** 只能让你动画单个 `Layer` 的单个属性，而且中间不能有重叠或间隔。

一个上下摇摆的动画：

```swift
let wobble = CAKeyframeAnimation(keyPath: "transform.rotation")
wobble.duration = 0.25
wobble.repeatCount = 4
wobble.values = [0.0, -M_PI_4/4, 0.0, M_PI_4/4, 0.0]
wobble.keyTimes = [0.0, 0.25, 0.5, 0.75, 1.0]
view.layer.add(wobble, forKey: nil)
```

##### Animating struct values

以下是动画一个 `struct` 类型的 `CGPoint`, 你也可以动画 `CGSize`, `CGRect`, `CATransform3D`

```swift
let flight = CAKeyframeAnimation(keyPath: "position")
flight.duration = 12.0
flight.values = [CGPoint(x: 0.0, y: 0.0),
					CGPoint(x: 100.0, y: 50),
					CGPoint(x: 0.0, y: 100)]
flight.keyTimes = [0.0, 0.5, 1.0]
layer.add(flight, forKey: nil)
```
#### Shapes and Masks

`CAShapeLayer` 可用于 `Layer` 的遮罩层(Mask)，其 `path` 属性可使用 `UIBezierPath` 构造，且可动画该属性。

#### Gradient Animations

`CAGradientLayer` 使用 `mask` 属性，并动画其 `locations` 属性，实现 **slide to unlock** 效果。
