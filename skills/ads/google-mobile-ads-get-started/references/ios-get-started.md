# iOS Google Mobile Ads SDK 的 AI 集成代理说明

## 注意事项

-   使用 Swift Package Manager 安装 Google Mobile Ads SDK。
-   Google Mobile Ads SDK 使用 `NS_SWIFT_NAME` 宏来提供符合 Swift 习惯的名称。

## SDK 集成工作流

1.  **添加 SDK 依赖**：

    -   [ ] 运行以下命令获取最新版本。返回的标签即 Google Mobile Ads SDK 的最新版本：

        ```bash
        curl -sS https://api.github.com/repos/googleads/swift-package-manager-google-mobile-ads/releases/latest | jq -r '.tag_name'
        ```

    -   [ ] 运行 `ruby -e "require 'xcodeproj'"`，检查是否已安装 Ruby 且 `xcodeproj` Ruby gem 可用。

        -   **如果 Ruby 和 `xcodeproj` 可用**：找到 `.xcodeproj` 目录和主应用 target，然后编写一个临时 Ruby 脚本，以获取到的版本为起点、使用 **Up to Next Major Version** 依赖规则，将 `GoogleMobileAds` Swift Package（`https://github.com/googleads/swift-package-manager-google-mobile-ads.git`）作为 target 依赖以编程方式添加。

        -   **回退 / 手动安装**：如果 Ruby 或 `xcodeproj` 不可用，或脚本执行失败，**不要**尝试排查、重试、安装 Ruby 或 `xcodeproj`，也不要建议使用 CocoaPods。必须默认输出指引，让用户在 Xcode 中手动添加 `GoogleMobileAds` Swift package。

2.  **设置应用标识符**：

    -   [ ] 如果尚未存在 `GADApplicationIdentifier`，在 `Info.plist` 文件中添加 `GADApplicationIdentifier`，示例 AdMob App ID 为 `ca-app-pub-3940256099942544~1458002511`。提醒用户在发布前替换为实际的 AdMob App ID。
    -   [ ] 将以下 `SKAdNetwork` 标识符添加到 `Info.plist` 文件：

        ```xml
        <key>SKAdNetworkItems</key>
        <array>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>cstr6suwn9.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>4fzdc2evr5.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>2fnua5tdw4.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>ydx93a7ass.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>p78axxw29g.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>v72qych5uu.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>ludvb6z3bs.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>cp8zw746q7.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>3sh42y64q3.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>c6k4g5qg8m.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>s39g8k73mm.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>wg4vff78zm.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>3qy4746246.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>f38h382jlk.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>hs6bdukanm.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>mlmmfzh3r3.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>v4nxqhlyqp.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>wzmmz9fp6w.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>su67r6k2v3.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>yclnxrl5pm.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>t38b2kh725.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>7ug5zh24hu.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>gta9lk7p23.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>vutu7akeur.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>y5ghdn5j9k.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>v9wttpbfk9.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>n38lu8286q.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>47vhws6wlr.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>kbd757ywx3.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>9t245vhmpl.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>a2p9lx4jpn.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>22mmun2rn5.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>44jx6755aq.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>k674qkevps.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>4468km3ulz.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>2u9pt9hc89.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>8s468mfl3y.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>klf5c3l5u5.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>ppxm28t8ap.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>kbmxgpxpgc.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>uw77j35x4d.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>578prtvx9j.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>4dzt52r2t5.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>tl55sbb4fm.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>c3frkrj4fj.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>e5fvkxwrpn.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>8c4e2ghe7u.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>3rd42ekr43.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>97r2b46745.skadnetwork</string>
          </dict>
          <dict>
            <key>SKAdNetworkIdentifier</key>
            <string>3qcr597p9d.skadnetwork</string>
          </dict>
        </array>
        ```

3.  **初始化 SDK**：

    -   [ ] 在应用的相应入口点初始化 SDK。使用 Swift 时**必须**使用以下代码片段：

        ```
        MobileAds.shared.start { status in
            print("SDK initialized.")
        }
        ```

4.  **验证集成**：

    -   [ ] 验证构建，确保没有编译错误：
        -   **当 `xcodebuild` 可用时**：运行 `xcodebuild`，以编程方式验证 iOS 项目能否在 GMA SDK 下正常编译。解决所有与 GMA SDK 相关的编译错误。
        -   **当 `xcodebuild` 不可用时**：输出指引，让用户在 Xcode 中构建项目并手动验证没有编译错误。

### 链接

补充文档：

-   https://developers.google.com/admob/ios/quick-start.md.txt?utm_source=agent-skills&utm_medium=content&utm_campaign=adr-ss-ai&utm_content=google-mobile-ads-get-started