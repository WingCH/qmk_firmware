# NuPhy Halo75 V2：打造專屬的 Mac Globe Dual-Role 鍵

## 為什麼要改？

我一直好鍾意 NuPhy Halo75 V2 呢塊板，不過深入研究 QMK firmware 嘅時候撞到兩個問題：

1. 官方 framework/VIA 入面找唔到 macOS 專用嘅「地球鍵」(Globe key)。
2. 左 Ctrl 獨立存在感低，想搵方法令佢 tap 時切換輸入法，但係長按仍然保留 Ctrl 修飾功能。

於是決定自己為 Halo75 V2 加一個 custom keycode 以及採用 QMK 內建功能：

- `MAC_GLOBE`：純粹模擬 Apple 地球鍵。按一次即出輸入法選單。
- `LCTL_T(KC_NO)`：使用 QMK 內建的 Mod-Tap 功能，tap＝地球鍵，hold＝Ctrl，更穩定可靠。

## 變更概覽

| 檔案 | 作用 |
|------|------|
| `keyboards/nuphy/halo75_v2/ansi/ansi.h` | 移除了 `MAC_GLOBE_CTRL`，簡化 custom keycode 定義 |
| `keyboards/nuphy/halo75_v2/ansi/ansi.c` | 使用 QMK 內建 `LCTL_T(KC_NO)` 處理 dual-role，在 tap 時發送 Globe Consumer usage |
| `keyboards/nuphy/halo75_v2/ansi/keymaps/via/NuPhy Halo75 via3.json` | 移除了 Mac Globe Ctrl 項目，改用標準 QMK 功能 |
| `keyboards/nuphy/halo75_v2/nuphy-halo75-v2-via.json` | 提供官方 VIA 設定檔 |

## QMK 實作重點

### Custom Keycode 定義

```c
enum custom_keycodes {
    ...
    MAC_GLOBE,
    ...
};
```

使用 `QK_KB_0` 起手自增，確保 VIA 寫入時對應到正確 HID code。移除了 `MAC_GLOBE_CTRL`，改用 QMK 內建方案。

### `MAC_GLOBE`

非常直接：

```c
case MAC_GLOBE:
    if (record->event.pressed) {
        host_consumer_send(0x029D);
    } else {
        host_consumer_send(0);
    }
    return false;
```

只喺 pressed 時送出 Consumer usage 0x029D（macOS 的 Globe key），放開就清零。

### `LCTL_T(KC_NO)` 與 Globe 功能整合

改用 QMK 內建的 Mod-Tap 功能，當偵測到 tap 行為時發送 Globe Consumer usage：

```c
bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    // Handle LCTL_T(KC_NO) - the 0x2100 keycode for Globe/Ctrl dual function
    if (keycode == LCTL_T(KC_NO)) {
        if (record->tap.count && record->event.pressed) {
            // Tapped: Send Globe consumer key
            host_consumer_send(0x029D);
            return false;  // Prevent default processing
        } else if (record->tap.count && !record->event.pressed) {
            // Tap released: Cancel Globe consumer key
            host_consumer_send(0);
            return false;  // Prevent default processing
        }
        // For hold (no tap.count), let QMK handle the Ctrl modifier
        return true;
    }
    // ... 其他處理
}
```

> ✦ 使用 QMK 內建的 `LCTL_T` 功能更加穩定，QMK 會自動處理 tap/hold 的偵測。我們只需要在偵測到 tap 時發送 Globe usage 即可。

## VIA 支援

VIA JSON 裏面的 `customKeycodes` 只保留 Globe 鍵：

```json
{
    "name": "Mac\nGlobe",
    "title": "Mac Globe"
}
```

而 Globe/Ctrl 雙功能鍵則直接使用 QMK 內建的 `LCTL_T(KC_NO)` (keycode: 0x2100)，VIA 會顯示為 "LCtl_T(KC_NO)"。

## 編譯與刷新

```bash
qmk compile -kb nuphy/halo75_v2/ansi -km via
```

刷寫生成的 `nuphy_halo75_v2_ansi_via.bin`，然後喺 macOS 測試：

| 操作 | 結果 |
|------|------|
| Tap `Mac Globe` | 即時彈出輸入法選單 |
| Tap `LCTL_T(KC_NO)` | 同上 |
| Hold `LCTL_T(KC_NO)` + C | Output Ctrl+C |
| Hold `LCTL_T(KC_NO)` 冇配合鍵 | 只會有 Ctrl 功能，無 Globe 信號 |

## VIA 設置

要在 VIA 中使用 Globe/Ctrl 雙功能鍵：

1. 打開 VIA 並載入你的鍵盤配置
2. 進入 **SPECIAL** 頁面
3. 選擇 **ANY**
4. 輸入 keycode：**0x2100**
5. 拖拉到你想要的鍵位

### 0x2100 的計算方法

Keycode `0x2100` 來自 QMK 的 `LCTL_T(KC_NO)` 宏：

- `LCTL_T()` 創建一個以左 Ctrl 為修飾鍵的 Mod-Tap 鍵
- `KC_NO` (0x00) 是 tap 的 keycode（無鍵）
- QMK 的 Mod-Tap keycode 從 `0x2000` 開始
- 左 Ctrl 修飾鍵加 `0x0100`
- 因此：`0x2000` + `0x0100` + `0x00` = `0x2100`

## 心得

- 最初嘗試自定義實作 dual-role 邏輯，但後來發現 QMK 內建的 `LCTL_T` 加上自定義 tap 處理更加穩定可靠。
- 使用 `record->tap.count` 可以準確偵測 QMK 認定的 tap 行為，避免自己實作 tap/hold 偵測的複雜性。
- Consumer usage（如 0x029D）需要特別處理，但整合到 QMK 的 Mod-Tap 系統中效果更好。
- 程式碼變得更簡潔，維護性也更高。

## 下一步

1. 將 `nuphy-halo75-v2-via.json` 納入版本控制，方便分享 VIA 預設。
2. 以同樣模式加更多 macOS 快捷，例如 Spotlight、Siri 等 Consumer usages。

如果你都想為自己嘅 Halo75 V2 加上真正有用嘅 Mac 功能，呢個結合 QMK 內建功能的方案係一個簡單而優雅嘅開始。
