# NuPhy Halo75 V2：打造專屬的 Mac Globe Dual-Role 鍵

## 為什麼要改？

我一直好鍾意 NuPhy Halo75 V2 呢塊板，不過深入研究 QMK firmware 嘅時候撞到兩個問題：

1. 官方 framework/VIA 入面找唔到 macOS 專用嘅「地球鍵」(Globe key)。
2. 左 Ctrl 獨立存在感低，想搵方法令佢 tap 時切換輸入法，但係長按仍然保留 Ctrl 修飾功能。

於是決定自己為 Halo75 V2 加兩個 custom keycode：

- `MAC_GLOBE`：純粹模擬 Apple 地球鍵。按一次即出輸入法選單。
- `MAC_GLOBE_CTRL`：tap＝地球鍵，hold＝Ctrl，兼容所有快捷鍵。

## 變更概覽

| 檔案 | 作用 |
|------|------|
| `keyboards/nuphy/halo75_v2/ansi/ansi.h` | 在 `enum custom_keycodes` 加入 `MAC_GLOBE`、`MAC_GLOBE_CTRL` |
| `keyboards/nuphy/halo75_v2/ansi/ansi.c` | 實作 dual-role 行為，同步處理 Consumer usage 0x029D 同 Ctrl |
| `keyboards/nuphy/halo75_v2/ansi/keymaps/via/NuPhy Halo75 via3.json` | VIA 顯示新 keycode |
| `keyboards/nuphy/halo75_v2/nuphy-halo75-v2-via.json` | 提供官方 VIA 設定檔 |

## QMK 實作重點

### Custom Keycode 定義

```c
enum custom_keycodes {
    ...
    MAC_GLOBE,
    MAC_GLOBE_CTRL,
    ...
};
```

使用 `QK_KB_0` 起手自增，確保 VIA 寫入時對應到正確 HID code。

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

### `MAC_GLOBE_CTRL`

核心邏輯：按落即時觸發 Globe；若之後配合其他鍵，就取消 Globe 信號再變成 Ctrl；放手時按照狀態收尾。

```c
static bool     mac_globe_ctrl_pressed    = false;
static bool     mac_globe_ctrl_mod_active = false;
static bool     mac_globe_ctrl_tapped     = false;

bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    // 偵測到第二粒鍵，就轉做 Ctrl
    if (mac_globe_ctrl_pressed && !mac_globe_ctrl_mod_active) {
        if (keycode != MAC_GLOBE_CTRL && record->event.pressed) {
            if (mac_globe_ctrl_tapped) {
                host_consumer_send(0);
                mac_globe_ctrl_tapped = false;
            }
            register_code(KC_LCTL);
            mac_globe_ctrl_mod_active = true;
        }
    }

    switch (keycode) {
        case MAC_GLOBE_CTRL:
            if (record->event.pressed) {
                mac_globe_ctrl_pressed    = true;
                mac_globe_ctrl_mod_active = false;
                mac_globe_ctrl_tapped     = true;
                host_consumer_send(0x029D); // 立即有 Globe 效果
            } else {
                if (mac_globe_ctrl_mod_active) {
                    unregister_code(KC_LCTL);
                } else if (mac_globe_ctrl_tapped) {
                    host_consumer_send(0); // 單純 tap：清走 Globe 信號
                }
                mac_globe_ctrl_pressed    = false;
                mac_globe_ctrl_mod_active = false;
                mac_globe_ctrl_tapped     = false;
            }
            return false;
    }
    return true;
}
```

> ✦ 為咗 tap 反應快，喺 pressed 已經先送出 0x029D。如果之後偵測到長按配合其他鍵，就先 `host_consumer_send(0)` 再 `register_code(KC_LCTL)`，變成標準 Ctrl + Key。

## VIA 支援

VIA JSON 裏面的 `customKeycodes` 加入兩個新項目：

```json
{
    "name": "Mac\nGlobe",
    "title": "Mac Globe"
},
{
    "name": "Mac\nGlbCtrl",
    "title": "Mac Globe Ctrl"
}
```

VIA 重新載入後，就可以喺 GUI 直接拖放呢兩粒鍵。

## 編譯與刷新

```bash
qmk compile -kb nuphy/halo75_v2/ansi -km via
```

刷寫生成的 `nuphy_halo75_v2_ansi_via.bin`，然後喺 macOS 測試：

| 操作 | 結果 |
|------|------|
| Tap `Mac Globe` | 即時彈出輸入法選單 |
| Tap `Mac Globe Ctrl` | 同上 |
| Hold `Mac Globe Ctrl` + C | Output Ctrl+C |
| Hold `Mac Globe Ctrl` 冇配合鍵 | 釋放 Globe 信號，無副作用 |

## 心得

- QMK 其實已經有 `MT()`、`LT()` 呢類 dual-role 宏，但 Consumer usage（如 0x029D）唔喺 Keyboard page，所以要自己寫 handler。
- 喺 pressed 時即刻送出 Globe usage，可以避免 tap 延遲；記住要為 hold 情況補返清除邏輯。
- 註解寫得清楚，日後維護會容易好多。

## 下一步

1. 將 `nuphy-halo75-v2-via.json` 納入版本控制，方便分享 VIA 預設。
2. 以同樣模式加更多 macOS 快捷，例如 Spotlight、Siri 等 Consumer usages。

如果你都想為自己嘅 Halo75 V2 加上真正有用嘅 Mac 功能，呢兩粒 custom keycode 係一個簡單而優雅嘅開始。
