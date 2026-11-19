# 红果净化

红果免费短剧（`com.phoenix.read`）的 LSPosed 模块：去广告 + 会员/锁集本地放行。

模块包名：`io.github.bancheng97.hongguo`　作者：半城

作者：半城

## 功能

| 分组 | 开关 | 作用 |
|---|---|---|
| 广告 | 广告 SDK 拦截 | 穿山甲 / 优量汇的 `load*` `request*` `show*` 直接判失败 |
| 广告 | 回调失败通知 | 主动回调 `onNoAd`，界面不会卡在加载中 |
| 广告 | 业务层广告 | 红果自己的 `loadXxxAd` / `showXxxAd` 一起拦 |
| 会员 | 会员判定 | `isVip` / `isSvip` / `isMember` 一类布尔方法返回真 |
| 会员 | 会员显示全开 | 顺带改写 VIP 类型、等级、到期时间 |
| 解锁 | 锁集放行 | `isLock` / `needPay` 判定为已解锁 |
| 解锁 | 激励视频跳过 | 看完回调直接判成功 |
| 解锁 | 剧集字段硬刷 | 反射改剧集对象的锁字段（最激进） |
| 高级 | 用户模型补丁 | `UserInfo` / `VipInfo` 字段刷成永久会员 |
| 高级 | 安全模式 | 只注入不装规则，用于排查闪退 |
| 高级 | 调试日志 | 命中规则写进 LSPosed 日志 |

## 原理

不写死混淆后的类名（写死的基本一周就废），而是：

1. hook `ClassLoader.loadClass`，目标包里的类一加载就入队；
2. 后台线程按**成员名分词**匹配规则（`isShowAd` → `[is, show, ad]`）；
3. 命中就 `XposedBridge.hookMethod` 改返回值，或反射改数据模型字段。

规则名不随混淆变化，所以同一个包能跨红果 7.x 各版本用。

## 使用

1. 安装 APK，LSPosed 里启用模块，作用域勾选「红果免费短剧」；
2. 强制停止红果，重开；
3. 设置界面入口：LSPosed 管理器 → 模块 → 红果净化 → 打开设置（桌面无图标）；
   备用入口 `am start -a android.intent.action.VIEW -d hongguo-hook://settings`。

## 配置

设置界面里的开关会写两份：
`/sdcard/Android/data/com.phoenix.read/files/hongguo_hook.conf`（需要「所有文件访问」）
与本模块 `SharedPreferences("hg_prefs")`（零权限，hook 用 XSharedPreferences 读）。
手动改 conf 也支持 `extra_prefix` / `skip=类#方法` / `force=类#方法=值`。

## 边界

广告可以在客户端彻底灭掉；会员/锁集改的是本地判定与模型，付费集视频地址由服务端签名下发，
服务端硬校验时只能做到界面放行。激励视频跳过走本地回调，成功率最高。

## 支持

issues：本仓库
