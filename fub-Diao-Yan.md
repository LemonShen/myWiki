
## Getting Started
### [lead get 从zillow此类网站同步信息](http://www.followupboss.com/lead-providers/)
### 邮局服务，@homeplus邮箱
### gmail 收发 google日历同步
### 手机认证
### 消息提醒 sms tts 提醒（回拨功能？提醒agent有lead 然后按1回拨？）

## Basic training
### 即时通话 skype绑定？ 目测不需要skype 建议直接调用本机打电话

## Integrations （集成这块调研量大 不一定都做，等产品确认）
### 任务与google日历整合
### IDX 抓取、整合
### Mailchimp 通过电子邮件订阅 RSS 的在线工具（用户级别，可以一天发送15w邮件给其他人）
### egPlacester
### Better Voicemail 记录电话信息
### Mojo Sells
### Happy Grasshopper
### Generic Lead Parser Format
### Market Snapshot
### Piesync
### Salesdialers

## Following up your leads
### 拨打电话 ios可以直接调用iphone android也OK吧 产品给出了justremotephone
```android
在Android中，有两种方法拨打电话

Intent surf = new Intent(Intent.ACTION_DIAL, call);

Intent surf = new Intent(Intent.ACTION_CALL, call);

第一种方法调用手机系统的拨号界面，但是可以把号码自动填进去(上面的call变量)，这个方法需要用户手动点击通话电话才能打出去，不需要任何系统权限。

第二种方法可以不调用系统的拨号界面，也不需要用户确认，直接就拨打出电话，需要CALL_PHONE权限。你需要的就是这种方法。
```

```ios
IOS
调用打电话功能
[[UIApplicationsharedApplication] openURL:[NSURL URLWithString:@"tel://10086"]];
调用发短信功能

[[UIApplication sharedApplication]openURL:[NSURL URLWithString:@"sms://10000"]];
```

## Managing Account

## Pricing & Payments
