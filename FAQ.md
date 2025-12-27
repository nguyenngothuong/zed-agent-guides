# FAQ - Câu hỏi thường gặp về Zed

## Mục lục

1. [Settings là gì? Có mấy loại?](#settings)
2. [Rules files là gì?](#rules-files)
3. [Sự khác biệt giữa Settings và Rules](#settings-vs-rules)
4. [Các file quan trọng trong Zed](#important-files)

---

## 1. Settings là gì? Có mấy loại? {#settings}

Zed có **hai loại settings**:

### 1.1 Global Settings (Cài đặt toàn cục)

| Thuộc tính | Chi tiết |
|------------|----------|
| **Vị trí** | `%APPDATA%\Zed\settings.json` (Windows) |
| | `~/.config/zed/settings.json` (Linux) |
| | `~/Library/Application Support/Zed/settings.json` (macOS) |
| **Phạm vi** | Toàn bộ Zed, tất cả projects |
| **Mục đích** | Cài đặt mặc định cho mọi project |

**Ví dụ nội dung:**
```json
{
  "theme": "One Dark",
  "buffer_font_size": 14,
  "agent": {
    "always_allow_tool_actions": true,
    "default_model": {
      "provider": "zed.dev",
      "model": "claude-sonnet-4"
    }
  }
}
```

### 1.2 Project Settings (Cài đặt từng project)

| Thuộc tính | Chi tiết |
|------------|----------|
| **Vị trí** | `<project>/.zed/settings.json` |
| **Phạm vi** | Chỉ project chứa file này |
| **Mục đích** | Override global settings cho project cụ thể |

**Ví dụ:** Project A dùng tab size 2, Project B dùng tab size 4:

```json
// Project A: .zed/settings.json
{
  "tab_size": 2
}

// Project B: .zed/settings.json
{
  "tab_size": 4
}
```

### Thứ tự ưu tiên

```
Project Settings (.zed/settings.json)
         ↓ override
Global Settings (%APPDATA%\Zed\settings.json)
         ↓ override
Zed Default Settings
```

---

## 2. Rules Files là gì? {#rules-files}

Rules files là **hướng dẫn cho AI** (prompt), KHÔNG phải cấu hình Zed.

### Các file được Zed nhận diện (theo thứ tự ưu tiên)

| Thứ tự | File | Mô tả |
|--------|------|-------|
| 1 | `.rules` | Format gốc của Zed |
| 2 | `.cursorrules` | Tương thích Cursor |
| 3 | `.windsurfrules` | Tương thích Windsurf |
| 4 | `.clinerules` | Tương thích Cline |
| 5 | `.github/copilot-instructions.md` | Format GitHub Copilot |
| 6 | `AGENT.md` / `AGENTS.md` | Hướng dẫn agent chung |
| 7 | `CLAUDE.md` | Format Claude Code ✅ |
| 8 | `GEMINI.md` | Format Gemini CLI |

### Vị trí đặt file

- Đặt ở **thư mục gốc** của project
- Zed tự động đọc và include vào mọi cuộc hội thoại Agent Panel

### Ví dụ nội dung `.rules`

```text
# Project Guidelines

## Code Style
- Dùng TypeScript strict mode
- Ưu tiên functional components trong React
- Luôn thêm JSDoc comments cho public APIs

## Kiến trúc
- Tuân theo clean architecture
- Logic nghiệp vụ đặt trong /src/domain
- UI components đặt trong /src/components

## Testing
- Viết tests cho tất cả features mới
- Dùng React Testing Library cho component tests
```

### Rules Library

Zed có **Rules Library** tích hợp để quản lý nhiều bộ rules:

1. Mở Agent Panel → Click menu `...` → Chọn `Rules...`
2. Hoặc dùng Command Palette: `agent: open rules library`
3. Tạo, chỉnh sửa và đặt rules mặc định

---

## 3. Sự khác biệt giữa Settings và Rules {#settings-vs-rules}

| Khía cạnh | Settings | Rules |
|-----------|----------|-------|
| **Loại file** | JSON | Text/Markdown |
| **Mục đích** | Cấu hình Zed | Hướng dẫn AI |
| **Ảnh hưởng** | Editor, UI, tools | Hành vi của AI |
| **Vị trí** | `.zed/settings.json` | `.rules`, `CLAUDE.md`, etc. |
| **Ví dụ** | Theme, font size, models | "Viết code theo style X" |

### So sánh trực quan

```
┌─────────────────────────────────────────────────────────────┐
│                        ZED CONFIGURATION                     │
├─────────────────────────────┬───────────────────────────────┤
│        SETTINGS             │           RULES               │
│   (Cấu hình Zed Editor)     │    (Hướng dẫn cho AI)         │
├─────────────────────────────┼───────────────────────────────┤
│ • Theme, fonts              │ • Code style guidelines       │
│ • Tab size, format on save  │ • Architecture patterns       │
│ • AI models, providers      │ • Testing requirements        │
│ • Tool profiles             │ • Documentation style         │
│ • Keybindings               │ • Language preferences        │
├─────────────────────────────┼───────────────────────────────┤
│ File: settings.json         │ File: .rules, CLAUDE.md, etc. │
│ Format: JSON                │ Format: Plain text/Markdown   │
└─────────────────────────────┴───────────────────────────────┘
```

---

## 4. Các file quan trọng trong Zed {#important-files}

### Tổng quan cấu trúc

```
Zed Configuration Files
│
├── Global (áp dụng mọi nơi)
│   ├── %APPDATA%\Zed\settings.json     ← Cài đặt toàn cục
│   └── %APPDATA%\Zed\keymap.json       ← Phím tắt toàn cục
│
├── Project (áp dụng từng project)
│   ├── .zed/settings.json              ← Cài đặt project
│   └── .zed/tasks.json                 ← Tasks của project
│
└── AI Rules (hướng dẫn cho AI)
    ├── .rules                          ← Format Zed
    ├── CLAUDE.md                       ← Format Claude Code
    ├── GEMINI.md                       ← Format Gemini
    └── AGENT.md                        ← Format chung
```

### Chi tiết từng file

| File | Vị trí | Mục đích |
|------|--------|----------|
| `settings.json` (global) | `%APPDATA%\Zed\` | Theme, font, models, agent mặc định |
| `keymap.json` (global) | `%APPDATA%\Zed\` | Phím tắt tùy chỉnh |
| `settings.json` (project) | `.zed/` | Override settings cho project |
| `tasks.json` | `.zed/` | Định nghĩa tasks (build, test, etc.) |
| `.rules` | Project root | Hướng dẫn AI cho project |
| `CLAUDE.md` | Project root | Hướng dẫn AI (format Claude) |

---

## Câu hỏi thường gặp khác

### Q: Tôi có file CLAUDE.md từ Claude Code, có dùng được không?
**A:** Có! Zed tự động nhận diện và sử dụng `CLAUDE.md`.

### Q: Làm sao để AI tự động chạy không cần confirm?
**A:** Thêm vào settings.json:
```json
{
  "agent": {
    "always_allow_tool_actions": true
  }
}
```

### Q: Project settings có override global settings không?
**A:** Có. Project settings luôn được ưu tiên hơn global settings.

### Q: Rules có áp dụng cho Inline Assistant không?
**A:** Có, rules được include trong context của cả Agent Panel và Inline Assistant.

### Q: Làm sao để xem settings đang áp dụng?
**A:** Dùng Command Palette: `zed: open settings` hoặc `zed: open local settings`

---

## Tài nguyên thêm

- [Zed Documentation](https://zed.dev/docs)
- [Agent Panel Guide](https://zed.dev/docs/ai/agent-panel)
- [Rules Documentation](https://zed.dev/docs/ai/rules)
- [Settings Reference](https://zed.dev/docs/configuring-zed)

---

_Cập nhật lần cuối: 2025_