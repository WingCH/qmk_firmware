# NuPhy Halo75 V2 加上 Apple Globe 鍵功能

## 問題：缺少 Globe 鍵

買了 NuPhy Halo75 V2 之後發現一個問題：它沒有 Apple 的 Globe 鍵帽。對於經常需要切換中英文輸入法的我來說，這個按鍵已經用習慣了，不想因為換鍵盤就改變操作習慣。

所以我想找個方法，在這把鍵盤上也能用 Globe 鍵的功能。

## 想法：合併 Control 和 Globe 功能

仔細想想，Control 鍵從來不會單獨使用，一定要配合其他按鍵（Ctrl+C、Ctrl+V 等）才有作用。而 Apple 的 Globe 鍵則是單獨按一下就能切換輸入法。

既然這兩個功能沒有衝突，我可以讓一個按鍵同時具備兩種功能：
- 單按 = Globe 鍵（切換輸入法）
- 長按 + 其他鍵 = Control 功能

## 實作方法：使用 QMK

QMK 是一個開源的鍵盤韌體，可以讓我們自定義按鍵功能。這個想法用 QMK 是可以實現的。

### 第一種方法：自己寫邏輯

最初我試著自己寫一套雙功能按鍵的邏輯，但程式碼很複雜，要處理各種按鍵時機問題。

### 第二種方法：使用 QMK 內建功能

後來發現 QMK 本身就有 Mod-Tap 功能。我可以用 `LCTL_T(KC_NO)` 做基礎，然後在偵測到單按時加入 Globe 的功能：

```c
bool process_record_kb(uint16_t keycode, keyrecord_t *record) {
    if (keycode == LCTL_T(KC_NO)) {
        if (record->tap.count && record->event.pressed) {
            // 偵測到 tap：發送 Globe 鍵
            host_consumer_send(0x029D);
            return false;
        } else if (record->tap.count && !record->event.pressed) {
            // Tap 釋放：取消 Globe 信號
            host_consumer_send(0);
            return false;
        }
        // Hold 的情況讓 QMK 自己處理 Control
        return true;
    }
    // 其他處理...
}
```

這個方案的好處：
- QMK 負責 tap/hold 偵測
- 我只需要處理 Globe 功能
- 程式碼比較簡單

## 實際設置方法

### 用 VIA 設置

對於不想自己編譯韌體的使用者，可以直接在 VIA 中設置：

1. 打開 VIA 並載入你的鍵盤配置
2. 進入 **SPECIAL** → **ANY**
3. 輸入 keycode：**0x2100**
4. 將它拖到左 Control 的位置

**為什麼是 0x2100？**

這個數字來自 QMK 的 keycode 計算：
- Mod-Tap 基礎值：`0x2000`
- 左 Control 修飾鍵：`0x0100`
- KC_NO（無鍵）：`0x00`
- 總計：`0x2000 + 0x0100 + 0x00 = 0x2100`

### 測試結果

設置完成後的體驗讓我非常滿意：

| 操作 | 結果 |
|------|------|
| 輕按 | 輸入法選單立即彈出 |
| 長按 + C | 標準的 Ctrl+C 複製功能 |
| 長按 + V | 標準的 Ctrl+V 貼上功能 |
| 長按後放開 | 只有 Control 功能，無多餘動作 |

## 使用心得

設置完成後的效果很不錯：

1. **保持原有習慣**：不需要重新學習新的按鍵方式
2. **功能沒有衝突**：單按和長按分別對應不同功能，使用起來很自然
3. **善用現有工具**：QMK 的 Mod-Tap 功能很成熟，比自己寫邏輯簡單很多

對於需要經常切換輸入法的人來說，這個小改動帶來的便利性還是很明顯的。

## 總結

這次的改造相對簡單，主要是利用 QMK 的現有功能來解決實際使用需求。

NuPhy Halo75 V2 + QMK 的 Mod-Tap 功能 = Globe/Control 雙功能鍵

如果你也有類似的需求，可以試試看這個方法。

---

**完整的程式碼和設置說明**可以參考我的 [GitHub repository](https://github.com/your-repo)。

如果你也有類似的經驗，歡迎分享討論！

#鍵盤 #QMK #客製化 #NuPhy #經驗分享
