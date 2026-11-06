# 移动端 Safari（iOS）上的 Google IMA HTML5 SDK 指南

本指南介绍 Google 互动媒体广告（IMA）HTML5 SDK 支持移动端 Safari（iOS）的最佳实践。

## 基本设置

必须完成以下步骤：

*   检查并确保 HTML `<video>` 元素添加了 `playsinline` 属性。

*   添加播放按钮，由用户手势触发播放。

示例：

```html
<!-- 该视频将在 iOS 上尝试在其布局内播放 -->
<video playsinline controls></video>
```