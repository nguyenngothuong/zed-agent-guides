# Hướng dẫn Setup Autonomous Agent trong Zed

## 🎯 Mục tiêu

Thiết lập Zed Agent để hoạt động tự động hơn - tự lên kế hoạch, tự thực hiện, và tự sửa lỗi với ít sự can thiệp của user nhất có thể.

---

## ⚠️ Lưu ý quan trọng

**Zed hiện tại KHÔNG hỗ trợ AI chạy hoàn toàn tự động theo vòng lặp vô tận.**

Đây là workaround để agent hoạt động "tự động hơn", nhưng vẫn có giới hạn:
- Agent vẫn cần user gửi prompt ban đầu
- Agent có thể dừng và hỏi trong một số trường hợp
- Không có "background agent" chạy liên tục

---

## 📦 Các file cần thiết

### 1. Rules file: `AGENT_AUTONOMOUS.rules`

Đã tạo tại: `zed/AGENT_AUTONOMOUS.rules`

File này chứa instructions để agent:
- Tự lên kế hoạch trước khi làm
- Tự check diagnostics sau mỗi edit
- Tự sửa lỗi mà không hỏi user
- Chỉ dừng khi thực sự blocked

### 2. Settings configuration

Thêm vào `settings.json` của Zed:

```json
{
  "agent": {
    // Cho phép agent chạy tools mà không cần confirm
    "always_allow_tool_actions": true,
    
    // Thông báo khi agent xong (nếu bạn tab away)
    "play_sound_when_agent_done": true,
    "notify_when_agent_waiting": true,
    
    // Không yêu cầu modifier để gửi message
    "use_modifier_to_send": false,
    
    // Mở rộng edit card để thấy full diff
    "expand_edit_card": true,
    
    // Mở rộng terminal card để thấy output
    "expand_terminal_card": true
  }
}
```

---

## 🔧 Setup Step-by-Step

### Bước 1: Copy rules file vào project

Copy `AGENT_AUTONOMOUS.rules` vào thư mục gốc của project bạn muốn agent làm việc.

Hoặc rename thành một trong các tên được Zed hỗ trợ:
- `.rules`
- `AGENT.md`
- `CLAUDE.md`
- `GEMINI.md`

### Bước 2: Cập nhật settings.json

1. Mở Command Palette: `Cmd+Shift+P` (Mac) / `Ctrl+Shift+P` (Windows/Linux)
2. Tìm: `zed: open settings`
3. Thêm config từ section trên

### Bước 3: Tạo Custom Profile (Optional)

Để có đầy đủ tools cần thiết:

1. Mở Agent Panel
2. Click vào profile selector (góc dưới của message editor)
3. Chọn "Configure"
4. Tạo profile mới hoặc chỉnh sửa profile "Write"
5. Đảm bảo các tools sau được enable:
   - `thinking` ← **Quan trọng cho planning**
   - `diagnostics` ← **Quan trọng cho self-check**
   - `edit_file`
   - `read_file`
   - `grep`
   - `find_path`
   - `list_directory`
   - `terminal`
   - `fetch`

### Bước 4: Add rules vào default (Optional)

1. Mở Rules Library: `agent: open rules library`
2. Import file `AGENT_AUTONOMOUS.rules`
3. Click "Add to Default" để luôn áp dụng

---

## 💬 Cách sử dụng

### Prompt template cho autonomous work:

```
[MỤC TIÊU]
Implement feature X / Fix bug Y / Refactor Z

[YÊU CẦU]
- Yêu cầu cụ thể 1
- Yêu cầu cụ thể 2

[HƯỚNG DẪN]
Work autonomously:
1. Plan the implementation using thinking tool
2. Make all necessary changes
3. Check diagnostics after each file edit
4. Fix any errors you find
5. Run tests if available
6. Continue until complete with no errors

Do NOT stop to ask questions unless absolutely blocked.
Report progress periodically.
```

### Ví dụ cụ thể:

```
Refactor the authentication module to use JWT instead of sessions.

Requirements:
- Keep backward compatibility
- Add proper error handling
- Update related tests

Work autonomously - plan first, then implement step by step.
Check diagnostics after each file change and fix any errors.
Only ask me if you hit an unresolvable blocker.
```

---

## 📊 Tools Profile Config (settings.json)

Nếu muốn define profile trong settings:

```json
{
  "agent": {
    "profiles": {
      "Autonomous": {
        "tools": {
          "thinking": true,
          "diagnostics": true,
          "edit_file": true,
          "read_file": true,
          "grep": true,
          "find_path": true,
          "list_directory": true,
          "terminal": true,
          "fetch": true,
          "copy_path": true,
          "create_directory": true,
          "delete_path": true,
          "move_path": true,
          "now": true,
          "open": true,
          "web_search": true
        }
      }
    },
    "default_profile": "Autonomous"
  }
}
```

---

## 🚨 Troubleshooting

### Agent vẫn hay dừng hỏi?

1. Check `always_allow_tool_actions` đã set `true` chưa
2. Đảm bảo rules file được load (check trong Agent Panel settings)
3. Thêm explicit instruction trong prompt: "Do NOT ask for confirmation"

### Agent đi vào loop vô tận?

LLMs có thể bị stuck. Workarounds:
- Set limit trong prompt: "Make maximum 3 attempts to fix each error"
- Cancel và restart với clearer instructions
- Try different model (some are better at following instructions)

### Diagnostics tool không hoạt động?

1. Đảm bảo project có LSP configured
2. Check language server đang chạy
3. Một số languages cần thời gian để index

### Tools không có trong profile?

1. Mở Agent Panel
2. Click profile selector > Configure
3. Manually enable missing tools
4. Save profile

---

## 📚 References

- [Agent Panel Documentation](./src/ai/agent-panel.md)
- [Tools Documentation](./src/ai/tools.md)
- [Agent Settings](./src/ai/agent-settings.md)
- [Rules Documentation](./src/ai/rules.md)

---

## 🔄 Limitations

| Aspect | Status |
|--------|--------|
| True background execution | ❌ Not supported |
| Continuous loop until done | ❌ Not supported |
| Self-triggering new tasks | ❌ Not supported |
| Auto-planning from prompt | ✅ Supported via rules |
| Auto-fix after errors | ✅ Supported via rules |
| Skip confirmations | ✅ Supported via settings |

---

_Created for Zed Editor_
_Last Updated: 2025_