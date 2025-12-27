# Hướng dẫn tính năng Zed cho người dùng VSCode/Claude Code

Chào mừng bạn đến với Zed! Hướng dẫn này bao gồm mọi thứ bạn cần biết khi chuyển từ VSCode với Claude Code sang Zed.

---

## Mục lục

1. [Rules Files (Tương tự CLAUDE.md)](#rules-files)
2. [Agent Panel vs Claude Code](#agent-panel)
3. [Tool Profiles](#tool-profiles)
4. [Tổng quan tính năng AI](#ai-features)
5. [MCP Servers (Context Servers)](#mcp-servers)
6. [Khác biệt chính so với VSCode](#key-differences)
7. [Cài đặt quan trọng](#essential-settings)
8. [Phím tắt](#keyboard-shortcuts)
9. [Mẹo và thủ thuật](#tips-and-tricks)

---

## 1. Rules Files (Tương tự CLAUDE.md) {#rules-files}

**Có, `CLAUDE.md` hoạt động trong Zed!** Zed tự động nhận diện các file này ở thư mục gốc project:

| File | Độ ưu tiên | Mô tả |
|------|------------|-------|
| `.rules` | 1 | Format gốc của Zed |
| `.cursorrules` | 2 | Tương thích Cursor |
| `.windsurfrules` | 3 | Tương thích Windsurf |
| `.clinerules` | 4 | Tương thích Cline |
| `.github/copilot-instructions.md` | 5 | Format GitHub Copilot |
| `AGENT.md` / `AGENTS.md` | 6 | Hướng dẫn agent chung |
| `CLAUDE.md` | 7 | Format Claude Code ✅ |
| `GEMINI.md` | 8 | Format Gemini CLI |

### Cách Rules hoạt động trong Zed

- Rules được **tự động bao gồm** trong mọi cuộc hội thoại Agent Panel
- Bạn có thể có nhiều rules files và kết hợp chúng
- Dùng `@rule` trong Agent Panel để mention rules cụ thể

### Rules Library (Thư viện Rules)

Zed có **Rules Library** tích hợp để quản lý nhiều bộ rules:

1. Mở Agent Panel → Click menu `...` → Chọn `Rules...`
2. Hoặc dùng Command Palette: `agent: open rules library`
3. Tạo, chỉnh sửa và đặt rules mặc định

### Ví dụ file `.rules`

```text
# Hướng dẫn Project

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
- Duy trì >80% code coverage
```

---

## 2. Agent Panel vs Claude Code {#agent-panel}

| Tính năng | Claude Code (VSCode) | Zed Agent Panel |
|-----------|---------------------|-----------------|
| Giao diện chat | Terminal + Panel | UI Panel native |
| Tool calling | Qua Claude | Built-in + MCP |
| Chỉnh sửa file | Tools của Claude | Tool edit_file của Zed |
| Review code | Diff view | Inline diff + Review panel |
| Lịch sử | Lệnh `/resume` | ⚠️ Chỉ native agent* |
| Checkpoints | Dựa trên Git | Checkpoints tích hợp |
| Nhiều threads | Sessions riêng biệt | Một thread (tabs sắp có) |

> *External agents (Claude Code, Gemini CLI, Codex) chưa lưu lịch sử. [Issue #37074](https://github.com/zed-industries/zed/issues/37074)

### Mở Agent Panel

- **Phím tắt**: `Cmd+?` (Mac) / `Ctrl+?` (Windows/Linux)
- **Command Palette**: `agent: new thread`
- **Status Bar**: Click icon ✨ sparkles

### Các tùy chọn Agent trong Zed

Bạn có thể dùng các agent khác nhau:

1. **Zed Agent** (native) - Đầy đủ tính năng, hỗ trợ lịch sử
2. **Claude Code** - Qua ACP adapter
3. **Gemini CLI** - Qua ACP
4. **Codex** - Qua ACP
5. **Custom agents** - Qua settings hoặc extensions

Để chuyển đổi: Click nút `+` trong Agent Panel → Chọn loại agent

---

## 3. Tool Profiles {#tool-profiles}

Profiles kiểm soát tools nào AI có thể sử dụng. Điều này **khác với Claude Code** nơi tất cả tools luôn có sẵn.

### Profiles tích hợp

| Profile | Mô tả | Tools |
|---------|-------|-------|
| **Write** | Toàn quyền | Tất cả tools bật |
| **Ask** | Chỉ đọc | Không edit file, không terminal |
| **Minimal** | Chỉ chat | Không có tools |

### Các Tools có sẵn

#### Tools Đọc & Tìm kiếm
- `read_file` - Đọc nội dung file
- `grep` - Tìm kiếm nội dung file bằng regex
- `find_path` - Tìm files theo glob pattern
- `list_directory` - Liệt kê nội dung thư mục
- `diagnostics` - Lấy errors/warnings từ LSP
- `fetch` - Fetch URLs dưới dạng markdown
- `web_search` - Tìm kiếm web
- `now` - Lấy ngày/giờ hiện tại
- `thinking` - Suy luận nội bộ (không thực hiện action)

#### Tools Chỉnh sửa
- `edit_file` - Tạo hoặc sửa files
- `terminal` - Chạy lệnh shell
- `copy_path` - Copy files/thư mục
- `move_path` - Di chuyển/đổi tên files
- `delete_path` - Xóa files/thư mục
- `create_directory` - Tạo thư mục
- `open` - Mở files với app mặc định

### Tạo Custom Profiles

1. Agent Panel → Click tên profile (phía dưới) → `Configure`
2. Click `Add New Profile` hoặc fork từ profile có sẵn
3. Bật/tắt tools cụ thể
4. Save

Hoặc trong `settings.json`:

```json
{
  "agent": {
    "profiles": {
      "Profile Tùy Chỉnh": {
        "tools": {
          "thinking": true,
          "diagnostics": true,
          "edit_file": true,
          "read_file": true,
          "grep": true,
          "find_path": true,
          "terminal": true,
          "fetch": false,
          "web_search": false
        }
      }
    }
  }
}
```

---

## 4. Tổng quan tính năng AI {#ai-features}

Zed có **bốn tính năng AI chính**:

### 4.1 Agent Panel (Chat chính)

Trợ lý coding AI chính của bạn. Tương tự panel Claude Code.

- **Mở**: `Cmd+?` / `Ctrl+?`
- **Tính năng**: Dùng tools, edit file, chạy terminal, context mentions
- **Context**: Dùng `@` để mention files, symbols, threads, rules

### 4.2 Inline Assistant

Chỉnh sửa AI nhanh trực tiếp trong editor.

- **Mở**: `Cmd+Enter` / `Ctrl+Enter` (với selection)
- **Use case**: Refactor nhanh, giải thích code, sửa lỗi
- **Hỗ trợ**: Nhiều models cùng lúc (xem `inline_alternatives`)

### 4.3 Edit Prediction (Autocomplete)

Code completion AI khi bạn gõ.

- **Provider**: Zeta (model của Zed), Copilot, Supermaven, hoặc Codestral
- **Accept**: `Tab`
- **Modes**: `eager` (luôn hiển thị) hoặc `subtle` (giữ Alt để preview)

```json
{
  "features": {
    "edit_prediction_provider": "zed"  // hoặc "copilot", "supermaven", "codestral"
  },
  "edit_predictions": {
    "mode": "eager"  // hoặc "subtle"
  }
}
```

### 4.4 Commit Message Generation

Tạo git commit messages bằng AI.

- **Trigger**: Trong source control panel, click nút AI
- **Model**: Có thể cấu hình riêng với agent chính

---

## 5. MCP Servers (Context Servers) {#mcp-servers}

Zed hỗ trợ **Model Context Protocol** để mở rộng khả năng AI.

### Cài đặt MCP Servers

**Cách 1: Extensions**
- Command Palette → `zed: extensions`
- Filter theo "context-servers"
- Phổ biến: GitHub, Puppeteer, Brave Search, Linear

**Cách 2: Settings**

```json
{
  "context_servers": {
    "my-mcp-server": {
      "command": "npx",
      "args": ["-y", "@modelcontextprotocol/server-filesystem"],
      "env": {}
    }
  }
}
```

### Sử dụng MCP Tools

1. Cài đặt MCP server
2. Cấu hình API keys cần thiết
3. Kiểm tra status trong Agent Panel Settings (chấm xanh = active)
4. Mention tên server trong prompts hoặc tạo profile chỉ có MCP tools

### MCP + External Agents

- ✅ Claude Code hỗ trợ MCP từ Zed
- ✅ Codex hỗ trợ MCP từ Zed
- ❌ Gemini CLI KHÔNG hỗ trợ MCP của Zed (dùng config riêng của họ)

---

## 6. Khác biệt chính so với VSCode {#key-differences}

| Khía cạnh | VSCode + Claude Code | Zed |
|-----------|---------------------|-----|
| **Kiến trúc** | Electron | Native (Rust/Metal/Vulkan) |
| **Hiệu năng** | Tốt | Xuất sắc (nhanh hơn nhiều) |
| **Tích hợp AI** | Extension-based | Native first-class |
| **Extensions** | Hệ sinh thái lớn | Đang phát triển (format khác) |
| **Settings** | JSON + UI | JSON (settings.json) |
| **Keymaps** | JSON | JSON (keymap.json) |
| **Language Servers** | Extension cung cấp | Built-in + extensions |
| **Git Integration** | Extension (GitLens, etc.) | Built-in |
| **Terminal** | Tích hợp | Tích hợp |
| **Multi-cursor** | ✅ | ✅ (tương tự) |
| **Vim mode** | Extension | Built-in |

### Thiếu trong Zed (tính đến 2025)

- Một số extensions ngôn ngữ cụ thể (kiểm tra Zed extensions trước)
- Debugger cho một số ngôn ngữ (đang cải thiện)
- Một số tính năng Git nâng cao
- Remote SSH editing (đang phát triển)

---

## 7. Cài đặt quan trọng {#essential-settings}

### Agent Settings

```json
{
  "agent": {
    // Tự động chạy tools không cần xác nhận (như /auto mode của Claude)
    "always_allow_tool_actions": true,
    
    // Thông báo khi agent hoàn thành
    "play_sound_when_agent_done": true,
    "notify_when_agent_waiting": true,
    
    // Model mặc định
    "default_model": {
      "provider": "zed.dev",
      "model": "claude-sonnet-4"
    },
    
    // Model cho tóm tắt thread (có thể rẻ/nhanh hơn)
    "thread_summary_model": {
      "provider": "zed.dev",
      "model": "gpt-4-mini"
    },
    
    // Model cho commit messages
    "commit_message_model": {
      "provider": "zed.dev",
      "model": "gpt-4-mini"
    },
    
    // Hiển thị review controls trong single file buffers
    "single_file_review": true
  }
}
```

### Sử dụng API Keys riêng

```json
{
  "language_models": {
    "anthropic": {
      "api_key": "sk-ant-..."  // Hoặc dùng keychain
    },
    "openai": {
      "api_key": "sk-..."
    },
    "google": {
      "api_key": "..."
    }
  }
}
```

> **Mẹo**: Dùng `"api_key_from_env": "ANTHROPIC_API_KEY"` để đọc từ environment.

### Editor Settings (giống VSCode)

```json
{
  // Font
  "buffer_font_family": "JetBrains Mono",
  "buffer_font_size": 14,
  
  // UI
  "theme": "One Dark",
  "ui_font_size": 15,
  
  // Hành vi editor
  "tab_size": 2,
  "format_on_save": "on",
  "autosave": "on_focus_change",
  
  // Vim mode (nếu bạn muốn)
  "vim_mode": true,
  
  // Inlay hints (như VSCode)
  "inlay_hints": {
    "enabled": true,
    "show_type_hints": true,
    "show_parameter_hints": true
  }
}
```

---

## 8. Phím tắt {#keyboard-shortcuts}

### Phím tắt AI

| Hành động | Mac | Windows/Linux |
|-----------|-----|---------------|
| Mở Agent Panel | `Cmd+?` | `Ctrl+?` |
| Thread Agent mới | `Cmd+N` (trong panel) | `Ctrl+N` |
| Inline Assist | `Cmd+Enter` | `Ctrl+Enter` |
| Accept Edit Prediction | `Tab` | `Tab` |
| Toggle Agent Focus | `Cmd+Shift+A` | `Ctrl+Shift+A` |

### Phím tắt Editor (tương tự VSCode)

| Hành động | Mac | Windows/Linux |
|-----------|-----|---------------|
| Command Palette | `Cmd+Shift+P` | `Ctrl+Shift+P` |
| Mở file nhanh | `Cmd+P` | `Ctrl+P` |
| Đi tới Symbol | `Cmd+Shift+O` | `Ctrl+Shift+O` |
| Đi tới Definition | `F12` hoặc `Cmd+Click` | `F12` hoặc `Ctrl+Click` |
| Tìm trong Files | `Cmd+Shift+F` | `Ctrl+Shift+F` |
| Toggle Terminal | `Ctrl+`` ` | `Ctrl+`` ` |
| Split Editor | `Cmd+\` | `Ctrl+\` |
| Đóng Tab | `Cmd+W` | `Ctrl+W` |
| Multi-cursor | `Cmd+D` | `Ctrl+D` |

### Tùy chỉnh Keybindings

Chỉnh sửa `keymap.json`:

```json
[
  {
    "bindings": {
      "cmd-shift-a": "agent::ToggleFocus"
    }
  },
  {
    "context": "Editor",
    "bindings": {
      "cmd-shift-enter": ["assistant::InlineAssist", {"prompt": "Sửa code này"}]
    }
  }
]
```

---

## 9. Mẹo và thủ thuật {#tips-and-tricks}

### Mẹo 1: Dùng @mentions cho Context

Trong Agent Panel, gõ `@` để mention:
- `@file` - Thêm file làm context
- `@directory` - Thêm nội dung thư mục
- `@symbol` - Reference functions/classes cụ thể
- `@thread` - Reference cuộc hội thoại trước
- `@rule` - Include một rules file

### Mẹo 2: Review Changes trước khi Accept

Sau khi agent thay đổi:
1. Click "Review Changes" hoặc nhấn `Cmd+Shift+D`
2. Xem tất cả changes trong multi-buffer diff view
3. Accept/reject từng hunk hoặc tất cả changes

### Mẹo 3: Sử dụng Checkpoints

Mỗi message tạo một checkpoint. Nếu AI làm sai:
1. Tìm nút "Restore Checkpoint" trên message của bạn
2. Click để revert tất cả changes từ điểm đó

### Mẹo 4: Export Conversations

Để lưu cuộc hội thoại (workaround cho không có history):
1. Agent Panel → menu `...` → `Open as Markdown`
2. Lưu file markdown vào project của bạn

### Mẹo 5: Follow Agent

Click icon crosshair (góc dưới-trái Agent Panel) để theo dõi agent khi nó navigate và edit files.

### Mẹo 6: Sử dụng nhiều Models

So sánh outputs từ các models khác nhau:

```json
{
  "agent": {
    "inline_alternatives": [
      {"provider": "zed.dev", "model": "gpt-4o"},
      {"provider": "zed.dev", "model": "gemini-2.5-flash"}
    ]
  }
}
```

### Mẹo 7: Project-Specific Settings

Tạo `.zed/settings.json` trong thư mục gốc project cho settings riêng từng project, override settings global.

### Mẹo 8: Diagnostics Tool

Yêu cầu agent kiểm tra lỗi:
> "Chạy diagnostics trên toàn bộ project và sửa mọi errors"

Agent sẽ dùng LSP để tìm và sửa issues.

---

## Thẻ tham khảo nhanh

```
┌─────────────────────────────────────────────────────────┐
│  ZED AI - THAM KHẢO NHANH                               │
├─────────────────────────────────────────────────────────┤
│  Agent Panel.............. Cmd+? / Ctrl+?              │
│  Inline Assist............ Cmd+Enter / Ctrl+Enter      │
│  Command Palette.......... Cmd+Shift+P / Ctrl+Shift+P  │
│  Accept Prediction........ Tab                          │
│  Review Changes........... Cmd+Shift+D / Ctrl+Shift+D  │
├─────────────────────────────────────────────────────────┤
│  Context Mentions:                                      │
│    @file   @directory   @symbol   @thread   @rule      │
├─────────────────────────────────────────────────────────┤
│  Profiles:                                              │
│    Write (full)   Ask (read-only)   Minimal (no tools) │
├─────────────────────────────────────────────────────────┤
│  Rules Files (thứ tự ưu tiên):                          │
│    .rules > .cursorrules > CLAUDE.md > GEMINI.md       │
└─────────────────────────────────────────────────────────┘
```

---

## Tài nguyên

- [Tài liệu Zed](https://zed.dev/docs)
- [Zed Discord](https://discord.gg/zed)
- [Zed GitHub](https://github.com/zed-industries/zed)
- [Tham khảo phím tắt](https://zed.dev/docs/key-bindings)
- [Tổng quan tính năng AI](https://zed.dev/docs/ai/overview)

---

_Cập nhật lần cuối: 2025_
_Cho Zed Version: 0.200+_