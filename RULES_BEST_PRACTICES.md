# Hướng dẫn viết file `.rules` hiệu quả cho Zed Agent

File `.rules` (hoặc các định dạng tương đương như `CLAUDE.md`, `.cursorrules`) đóng vai trò cực kỳ quan trọng trong việc định hướng hành vi của Zed Agent. Nó giống như một bản "System Prompt" tùy chỉnh cho từng dự án, giúp AI hiểu rõ bối cảnh, phong cách code và quy trình làm việc của bạn ngay từ đầu.

Dưới đây là các best practices để viết file `.rules` hiệu quả nhất.

---

## 1. Các định dạng file được hỗ trợ

Zed sẽ tự động tìm kiếm và sử dụng file hướng dẫn trong thư mục gốc của project theo thứ tự ưu tiên sau:

1. `.rules` (Khuyên dùng cho Zed)
2. `.cursorrules`
3. `.windsurfrules`
4. `.clinerules`
5. `.github/copilot-instructions.md`
6. `AGENT.md` hoặc `AGENTS.md`
7. `CLAUDE.md`
8. `GEMINI.md`

**Lời khuyên:** Nên sử dụng `.rules` nếu bạn chỉ dùng Zed, hoặc `.cursorrules`/`CLAUDE.md` nếu team của bạn sử dụng nhiều editor khác nhau.

---

## 2. Cấu trúc một file `.rules` chuẩn

Một file `.rules` tốt nên được viết bằng **Markdown** và chia thành các phần rõ ràng.

### Cấu trúc gợi ý:

```markdown
# [Tên Project] - Agent Guidelines

## 1. Vai trò & Mục tiêu
Mô tả ngắn gọn AI đóng vai trò gì (VD: Senior Frontend Engineer) và mục tiêu chính của project.

## 2. Tech Stack
Liệt kê các công nghệ sử dụng.
- Ngôn ngữ: TypeScript, Rust, Python...
- Framework: React, Next.js, Tokio...
- Libraries chính: TailwindCSS, Shadcn UI, Prisma...

## 3. Code Style & Conventions
Các quy tắc về coding style.
- Naming convention (camelCase, snake_case...)
- Cấu trúc thư mục
- Cách xử lý lỗi (Error handling)
- Type safety (VD: No `any` in TS)

## 4. Testing & Documentation
Yêu cầu về viết test và comment.
- Testing framework (Jest, Vitest...)
- Quy tắc viết docstring

## 5. Workflow (Quy trình làm việc)
Hướng dẫn AI cách tiếp cận vấn đề.
- Plan trước khi code
- Check diagnostics sau khi edit
- Không tự ý xóa code cũ nếu không chắc chắn
```

---

## 3. Best Practices (Thực hành tốt nhất)

### ✅ Nên làm (DO):

1.  **Cụ thể và Rõ ràng**: Thay vì nói "Viết code sạch", hãy nói "Sử dụng nguyên lý SOLID, tách nhỏ function không quá 20 dòng".
2.  **Cung cấp Ví dụ (Few-shot prompting)**: AI học rất nhanh qua ví dụ. Hãy đưa ra ví dụ về code "đúng" và code "sai" theo chuẩn của bạn.
3.  **Ưu tiên Khẳng định**: Hướng dẫn AI *làm gì* thay vì *không làm gì*.
    *   ❌ Không dùng `var`.
    *   ✅ Luôn sử dụng `const` hoặc `let`.
4.  **Cập nhật thường xuyên**: Khi project thay đổi (thêm thư viện mới, đổi kiến trúc), hãy cập nhật file `.rules`.
5.  **Dùng tiếng Anh (Khuyến khích)**: Mặc dù Zed hỗ trợ tiếng Việt, nhưng hầu hết các LLM (Claude, GPT-4) hiểu context kỹ thuật bằng tiếng Anh tốt hơn và chính xác hơn.

### ❌ Tránh (DON'T):

1.  **Quá dài dòng**: Đừng copy toàn bộ document của ngôn ngữ vào. Chỉ focus vào những gì đặc thù của project.
2.  **Mâu thuẫn**: Đảm bảo các quy tắc không đá nhau (VD: vừa yêu cầu code ngắn gọn, vừa yêu cầu comment mọi dòng).
3.  **Lỗi thời**: Rules cũ sẽ khiến AI hallucinate (ảo giác) về cấu trúc project.

---

## 4. Ví dụ mẫu (Template)

Dưới đây là một mẫu file `.rules` bạn có thể copy và tùy chỉnh:

```markdown
# Project Zed Extension - Guidelines

## Role
You are an expert Rust developer specializing in Zed extensions and GPUI framework.

## Tech Stack
- Rust (Latest stable)
- Zed Extension API
- GPUI (Zed's UI framework)

## Coding Standards
- **Async/Await**: Prefer `cx.spawn` for async tasks on the UI thread. Use `cx.background_executor()` for heavy computations.
- **Error Handling**: 
  - Never use `.unwrap()`. Use `?` or handle errors explicitly with `log::error!`.
  - Return `Result<T>` for fallible functions.
- **Imports**: Group imports by crate. STD first, then external crates, then internal modules.

## Zed Specifics
- When creating UI elements, implement `Render` trait.
- Use `gpui::Model` for state management.
- Always check `cx.context()` availability before using it in closures.

## Workflow Rules
1. **Analysis**: Before editing, read related files to understand the current implementation.
2. **Planning**: Use the `thinking` tool to outline your changes.
3. **Implementation**: Make small, incremental changes.
4. **Verification**: Run `diagnostics` after every edit to ensure no compilation errors.

## Example - UI Component
```rust
// Good
impl Render for MyView {
    fn render(&mut self, _cx: &mut ViewContext<Self>) -> impl IntoElement {
        div().child("Hello World")
    }
}
```
```

---

## 5. Mẹo nâng cao cho Zed

### Sử dụng `@rule`
Nếu bạn có nhiều dự án hoặc nhiều mảng kiến thức (Frontend, Backend, DevOps), bạn có thể tách ra thành nhiều file rules nhỏ (VD: `frontend.rules`, `backend.rules`) và dùng lệnh `@rule` trong khung chat để gọi đúng context cần thiết khi làm việc.

### Tự động hóa với `always_allow_tool_actions`
Kết hợp file `.rules` chặt chẽ với setting `always_allow_tool_actions: true`. Trong rules, hãy hướng dẫn AI:
> "You have permission to run tools. Always verify your changes using `diagnostics` tool immediately after editing a file."

Điều này giúp tạo ra một vòng lặp code -> check lỗi -> sửa lỗi bán tự động rất hiệu quả.