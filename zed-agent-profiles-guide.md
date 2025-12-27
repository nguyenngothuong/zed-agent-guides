# Zed Agent Profiles - Hướng dẫn đầy đủ

## Profile trong Zed Agent là gì?

**Profile** là cách để **nhóm các tools** lại với nhau theo mục đích sử dụng. Nó giống như "chế độ" hoặc "vai trò" cho AI agent.

---

## 🎯 Tại sao cần Profile?

Mỗi lần bạn chat với AI, bạn muốn nó có **quyền khác nhau**:

- **Hỏi về code** → Chỉ cho phép ĐỌC, không cho SỬA
- **Viết code** → Cho phép SỬA FILE, CHẠY LỆNH
- **Chat thường** → Không cần tools gì cả

Profile giúp bạn **switch nhanh** giữa các chế độ này.

---

## 📦 3 Built-in Profiles

### 1. **Write** (Full Power)

Tools: TẤT CẢ
- Read: read_file, grep, find_path, list_directory, diagnostics, web_search, fetch
- Edit: edit_file, create_directory, delete_path, copy_path, move_path
- Execute: terminal, open

**Dùng khi**: Muốn AI viết code, sửa file, chạy lệnh
**Ví dụ**: "Tạo component React mới", "Fix bug này", "Chạy test"

---

### 2. **Ask** (Read-Only)

Tools: CHỈ ĐỌC
- read_file, grep, find_path, list_directory
- diagnostics, web_search, fetch, thinking

**Dùng khi**: Chỉ muốn hỏi, không cho AI sửa code
**Ví dụ**: "Code này làm gì?", "Tìm bug ở đâu?", "Giải thích function này"

---

### 3. **Minimal** (No Tools)

Tools: KHÔNG CÓ

**Dùng khi**: Chat thường, không liên quan code
**Ví dụ**: "Viết email", "Giải thích khái niệm", "Brainstorm ý tưởng"

---

## 🛠️ Danh sách Tools đầy đủ

### Read & Search Tools

| Tool | Chức năng | Profile mặc định |
|------|-----------|------------------|
| read_file | Đọc nội dung file | Write, Ask |
| grep | Search regex trong code | Write, Ask |
| find_path | Tìm file theo glob pattern | Write, Ask |
| list_directory | List files trong thư mục | Write, Ask |
| diagnostics | Xem errors/warnings | Write, Ask |
| web_search | Search Google/web | Write, Ask |
| fetch | Fetch URL content | Write, Ask |
| thinking | AI suy nghĩ trước khi trả lời | Write, Ask |
| now | Lấy ngày giờ hiện tại | Write, Ask |

### Edit Tools

| Tool | Chức năng | Profile mặc định |
|------|-----------|------------------|
| edit_file | Sửa file (replace text) | Write only |
| create_directory | Tạo folder mới | Write only |
| delete_path | Xóa file/folder | Write only |
| copy_path | Copy file/folder | Write only |
| move_path | Move/rename file | Write only |

### Execute Tools

| Tool | Chức năng | Profile mặc định |
|------|-----------|------------------|
| terminal | Chạy shell commands | Write only |
| open | Mở file với default app | Write only |

---

## 🎨 Custom Profiles - Config Examples

### Config trong settings.json

Location: C:\Users\ADMIN\AppData\Roaming\Zed\settings.json

```json
{
  "agent": {
    "default_profile": "write",
    "profiles": {
      "blog-writer": {
        "name": "Blog Writer",
        "tools": ["read_file", "grep", "web_search", "fetch", "thinking"]
      },
      "full-dev": {
        "name": "Full Dev",
        "tools": [
          "read_file", "edit_file", "terminal", "grep",
          "create_directory", "delete_path", "diagnostics"
        ]
      }
    }
  }
}
```

---

## 💡 Best Practices

1. **Dùng "Ask" khi học code mới** - Tránh AI sửa nhầm
2. **Dùng "Write" khi code** - Full power
3. **Tạo custom profiles cho workflows lặp lại**
4. **Minimal cho brainstorming** - Chat không cần tools

---

## 🔒 Bypass Permissions

### Setting: always_allow_tool_actions

```json
{
  "agent": {
    "always_allow_tool_actions": true
  }
}
```

- **true**: AI tự chạy không hỏi (giống Claude Code bypass)
- **false**: AI hỏi permission trước khi edit/run

---

## 📋 Tóm lại

- **Profile** = Nhóm tools theo mục đích
- **Write** = Full power (edit + run)
- **Ask** = Read-only (hỏi không sửa)
- **Minimal** = No tools (chat thường)
- **Custom** = Tự tạo combo tools riêng
