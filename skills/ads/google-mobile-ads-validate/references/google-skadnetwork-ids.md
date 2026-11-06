# 已实现所有 Google SKAdNetwork ID

-   **按以下步骤操作**：
    -   [ ] 判断项目是 Android、iOS 还是 Unity。
    -   [ ] 如果是 Android，或 Unity 仅面向 Android，标记为 **N/A**。
    -   [ ] 如果是 iOS，检查 `Info.plist` 中的 `SKAdNetworkItems` 数组。
    -   [ ] 如果是 Unity 面向 iOS，检查 `GoogleMobileAdsSKAdNetworkItems.xml`。
    -   [ ] 运行下方的 Google SKAdNetwork ID 校验脚本，确认所有 Google SKAdNetwork ID 均已存在。如果无法使用 Python 运行时或终端工具，请直接将项目的 `SKAdNetworkItems` 与下方脚本中的 `GOOGLE_SKADNETWORK_IDS` 集合逐一比对。
-   **通过/失败标准**：
    -   **失败**：
        -   项目的 `SKAdNetworkItems` 中缺少任一 Google SKAdNetwork ID。
        -   `SKAdNetworkItems` 数组完全缺失。
    -   **N/A**：项目面向 Android，或 Unity 面向 Android。

## Google SKAdNetwork ID 校验脚本

使用以下 Python 脚本检查项目文件是否包含所有必需的 Google SKAdNetwork ID：

```python
import plistlib
import re
import sys

GOOGLE_SKADNETWORK_IDS = {
    "22mmun2rn5.skadnetwork",
    "2fnua5tdw4.skadnetwork",
    "2u9pt9hc89.skadnetwork",
    "3qcr597p9d.skadnetwork",
    "3qy4746246.skadnetwork",
    "3rd42ekr43.skadnetwork",
    "3sh42y64q3.skadnetwork",
    "4468km3ulz.skadnetwork",
    "44jx6755aq.skadnetwork",
    "47vhws6wlr.skadnetwork",
    "4dzt52r2t5.skadnetwork",
    "4fzdc2evr5.skadnetwork",
    "578prtvx9j.skadnetwork",
    "7ug5zh24hu.skadnetwork",
    "8c4e2ghe7u.skadnetwork",
    "8s468mfl3y.skadnetwork",
    "97r2b46745.skadnetwork",
    "9t245vhmpl.skadnetwork",
    "a2p9lx4jpn.skadnetwork",
    "c3frkrj4fj.skadnetwork",
    "c6k4g5qg8m.skadnetwork",
    "cp8zw746q7.skadnetwork",
    "cstr6suwn9.skadnetwork",
    "e5fvkxwrpn.skadnetwork",
    "f38h382jlk.skadnetwork",
    "gta9lk7p23.skadnetwork",
    "hs6bdukanm.skadnetwork",
    "k674qkevps.skadnetwork",
    "kbd757ywx3.skadnetwork",
    "kbmxgpxpgc.skadnetwork",
    "klf5c3l5u5.skadnetwork",
    "ludvb6z3bs.skadnetwork",
    "mlmmfzh3r3.skadnetwork",
    "n38lu8286q.skadnetwork",
    "p78axxw29g.skadnetwork",
    "ppxm28t8ap.skadnetwork",
    "s39g8k73mm.skadnetwork",
    "su67r6k2v3.skadnetwork",
    "t38b2kh725.skadnetwork",
    "tl55sbb4fm.skadnetwork",
    "uw77j35x4d.skadnetwork",
    "v4nxqhlyqp.skadnetwork",
    "v72qych5uu.skadnetwork",
    "v9wttpbfk9.skadnetwork",
    "vutu7akeur.skadnetwork",
    "wg4vff78zm.skadnetwork",
    "wzmmz9fp6w.skadnetwork",
    "y5ghdn5j9k.skadnetwork",
    "yclnxrl5pm.skadnetwork",
    "ydx93a7ass.skadnetwork",
}


def check_skadnetwork_ids(content: str) -> tuple[bool, set[str]]:
    """Extracts SKAdNetwork IDs and performs a set containment check."""
    found_ids = {
        match.lower()
        for match in re.findall(
            r"([a-z0-9]+\.skadnetwork)", content, re.IGNORECASE
        )
    }
    missing_ids = GOOGLE_SKADNETWORK_IDS - found_ids
    return len(missing_ids) == 0, missing_ids


def generate_missing_xml(missing_ids: set[str]) -> str:
    """Generates Info.plist XML dict entries for missing identifiers."""
    lines = []
    for mid in sorted(missing_ids):
        lines.append(
            f"  <dict>\n    <key>SKAdNetworkIdentifier</key>\n   "
            f" <string>{mid}</string>\n  </dict>"
        )
    return "\n".join(lines)


if __name__ == "__main__":
    target = sys.argv[1] if len(sys.argv) > 1 else None
    if target in ("-h", "--help"):
        print("Usage: python script.py [path/to/file]")
        print("Validates that all required Google SKAdNetwork IDs are present.")
        sys.exit(0)

    if target:
        try:
            with open(target, "rb") as f:
                try:
                    data = plistlib.load(f)
                    items = data.get("SKAdNetworkItems", [])
                    found_ids = {
                        item.get("SKAdNetworkIdentifier", "").lower()
                        for item in items
                        if isinstance(item, dict)
                    }
                    missing = GOOGLE_SKADNETWORK_IDS - found_ids
                    passed = len(missing) == 0
                except Exception:
                    f.seek(0)
                    content = f.read().decode("utf-8", errors="ignore")
                    passed, missing = check_skadnetwork_ids(content)
        except Exception as e:
            print(f"Error reading {target}: {e}", file=sys.stderr)
            sys.exit(2)
    else:
        if sys.stdin.isatty():
            print(
                "Error: No input provided. Provide a file path or pipe content"
                " into stdin.",
                file=sys.stderr,
            )
            print(
                "Usage: python script.py [path/to/file]",
                file=sys.stderr,
            )
            sys.exit(2)
        content = sys.stdin.read()
        passed, missing = check_skadnetwork_ids(content)

    if passed:
        print(
            f"PASS: All {len(GOOGLE_SKADNETWORK_IDS)} Google SKAdNetwork IDs"
            " are present."
        )
        sys.exit(0)
    else:
        print(f"FAIL: Missing {len(missing)} Google SKAdNetwork IDs:")
        for mid in sorted(missing):
            print(f"  {mid}")
        sys.exit(1)
```

### 添加缺失的标识符

解决缺失标识符的方法：

-   **iOS**：将每个缺失的 ID 添加到 `Info.plist`：

    ```xml
    <key>SKAdNetworkItems</key>
    <array>
      <dict>
        <key>SKAdNetworkIdentifier</key>
        <string>{missing_id}</string>
      </dict>
    </array>
    ```

-   **Unity**：将每个缺失的 ID 添加到 `GoogleMobileAdsSKAdNetworkItems.xml`：

    ```xml
    <SKAdNetworkItems>
      <SKAdNetworkIdentifier>{missing_id}</SKAdNetworkIdentifier>
    </SKAdNetworkItems>
    ```