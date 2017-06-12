---
layout: post
title: "Developer Settings in iPhone"
description: ""
category: iOS
tags: []
---
{% include JB/setup %}

### 如何开启 Developer 选项

> To see the Developer settings, you must provision the device for development and plug it into your Mac while Xcode or Instruments is running. If your device hasn’t been provisioned yet, see Configuring Your Xcode Project for Distribution and Launching Your App on Devices. Even after the device has been provisioned, the Developer setting disappears when the device is rebooted or powered off. To restore the setting, reconnect the device to Xcode or Instruments.
 
### Logging

- Energy
- Networking

Logging 选项中包含以上电量和网络两个选项，开启选项并点击 Start Recording 之后，正常使用你所要测试的 APP，当结束测试时，返回 Logging 界面，点击 Stop Recording。

接着，将手机连接到 Mac，打开 Instruments，选择 Energy Diagnostics，选中 File > Import Logged Data from Device 即可查看到之前记录的数据信息。

[查看详细步骤](https://developer.apple.com/library/content/documentation/Performance/Conceptual/EnergyGuide-iOS/MonitorEnergyWithInstruments.html#//apple_ref/doc/uid/TP40015243-CH33-SW11)

### Enable UI Automation

启用该选项用于执行 UITest，Xcode 8 中 Automation 项已经从 Instruments 中被移除，由 XCUITest 替代。

[Testing with Xcode](https://developer.apple.com/library/content/documentation/DeveloperTools/Conceptual/testing_with_xcode/chapters/09-ui_testing.html)  
[WWDC2015](https://developer.apple.com/videos/play/wwdc2015/406/)  
[Testing with UI Automation](https://code.tutsplus.com/tutorials/introduction-to-ios-testing-with-ui-automation--cms-22730)  
[Xamarin UITest](https://developer.xamarin.com/guides/testcloud/uitest/)

### Network Link Conditioner

模拟网络状况

[Simulate Bad Network](https://www.natashatherobot.com/simulate-bad-network-ios-simulator/)


### PassKit Testing

用于接入 PassKit 功能时辅助测试

[Introduction to PassKit](https://developer.xamarin.com/guides/ios/platform_features/introduction_to_passkit/)  
[Passbook Tutorial](https://oleb.net/blog/2013/02/passbook-tutorial/)