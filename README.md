# Zed Agent Guides & Autonomous Setup

Bộ tài liệu hướng dẫn sử dụng và tối ưu hóa Zed Editor, đặc biệt tập trung vào các tính năng AI và Autonomous Agent.

## 📚 Mục lục tài liệu

### 1. 🤖 Autonomous Agent
- [**AUTONOMOUS_AGENT_SETUP.md**](./AUTONOMOUS_AGENT_SETUP.md)  
  Hướng dẫn setup Zed Agent để hoạt động tự động: tự lập kế hoạch, tự sửa lỗi, tự thực thi.
- [**AGENT_AUTONOMOUS.rules**](./AGENT_AUTONOMOUS.rules)  
  File rules mẫu để biến Zed Agent thành nhân viên tự động.

### 2. 📘 Hướng dẫn cho người mới
- [**ZED_FEATURES_GUIDE_FOR_VSCODE_USERS.md**](./ZED_FEATURES_GUIDE_FOR_VSCODE_USERS.md)  
  Dành cho người dùng chuyển từ VSCode/Claude Code: so sánh tính năng, phím tắt, settings.
- [**FAQ.md**](./FAQ.md)  
  Câu hỏi thường gặp về Settings, Rules files và cấu trúc project.

### 3. 🛠️ Best Practices
- [**RULES_BEST_PRACTICES.md**](./RULES_BEST_PRACTICES.md)  
  Cách viết file `.rules` hiệu quả để điều khiển AI.

### 4. 🔬 Nghiên cứu & Plan
- [**EXTERNAL_AGENTS_HISTORY_PLAN.md**](./EXTERNAL_AGENTS_HISTORY_PLAN.md)  
  Phân tích và kế hoạch để giải quyết vấn đề lưu lịch sử chat cho External Agents (Claude Code, Gemini).

---

## 🚀 Quick Start

1. Copy file `AGENT_AUTONOMOUS.rules` vào project của bạn (đổi tên thành `.rules`).
2. Thêm config vào `settings.json`:
   ```json
   "agent": {
     "always_allow_tool_actions": true
   }
   ```
3. Mở Agent Panel (`Cmd+?`) và bắt đầu với prompt:
   > "Work autonomously: Plan first, then execute step by step. Check diagnostics after each edit."

## 🤝 Đóng góp

Repo này được tạo ra để chia sẻ kiến thức với cộng đồng Zed Việt Nam. Mọi đóng góp đều được hoan nghênh!