# Plan: Implement History Persistence cho External Agents trong Zed

> ⚠️ **EXISTING ISSUE FOUND**: Tính năng này đã được track tại GitHub!
> 
> **Issue chính**: [#37074 - Support history with external ACP agents](https://github.com/zed-industries/zed/issues/37074)
> - **Status**: Open (Feature Request)
> - **Labels**: `area:ai`, `area:ai/acp`, `area:ai/gemini`
> - **Votes**: 150+ 👍
> - **Created**: Aug 28, 2025
>
> **Related Issues (đã đóng, duplicate của #37074)**:
> - [#43525 - External Agents: no session history record](https://github.com/zed-industries/zed/issues/43525)
> - [#43378 - External Agents do not update thread titles](https://github.com/zed-industries/zed/issues/43378)
>
> **Related Discussions**:
> - [APC agents threads should be visible in history](https://github.com/zed-industries/zed/discussions) - Closed
> - [Support Claude Code History Selection in Zed](https://github.com/zed-industries/zed/discussions) - Closed
> - [I cannot see my previous history for claude code interactions](https://github.com/zed-industries/zed/discussions) - Closed

---

## 🚨 QUAN TRỌNG: Zed Team đang làm việc trên tính năng này!

### PRs đã bị đóng (Zed team muốn tự làm):

| PR | Tác giả | Nội dung | Lý do đóng |
|----|---------|----------|------------|
| [#42387](https://github.com/zed-industries/zed/pull/42387) | xipeng-jin | Concurrent Agent Chat with tabbed threads | **mikayla-maki (Zed)**: "we're cooking something up on this, stay tuned :D" |
| [#41874](https://github.com/zed-industries/zed/pull/41874) | aeroxy | History and recent conversations persistence per workspace | **benbrandt (Zed)**: "We are currently discussing how to make some of these history features work with ACP... I'm going to close for now, but we can revisit once we take a look at history for ACP agents + our agent." |
| [#36644](https://github.com/zed-industries/zed/pull/36644) | ConradIrwin | **MERGED** - "acp: Hide history unless in native agent" | Đây là PR **cố tình ẩn history** cho external agents! |

### Kết luận từ PRs:

1. **Zed team đang actively working** trên agent panel redesign
2. **Họ đã từ chối PRs từ community** với lý do "we're cooking something up"
3. **danilo-leal** (Nov 10): "We're still ironing out some bits of the design and should get to a more concrete roadmap super soon. You'll probably be able to find it in the form of a public channel notes in Zed."
4. **PR #36644** cho thấy history cho external agents bị **cố tình disable** - không phải chưa implement

### ⚡ Đề xuất hành động:

**OPTION 1**: Chờ Zed team release (họ nói "soon enough", "stay tuned")

**OPTION 2**: Fork và tự maintain (như wzulfikar đã làm - xem [wzulfikar/zed#8](https://github.com/wzulfikar/zed/pull/8))

**OPTION 3**: Comment trong issue #37074 hỏi cập nhật timeline

---

## 🎯 Tổng quan

**Mục tiêu**: Cho phép external agents (Claude Code, Gemini CLI, Codex) lưu và khôi phục lịch sử chat như Zed Agent (native).

**Độ khó**: Trung bình - Cao (Cần hiểu sâu về ACP protocol và Rust)

**Thời gian ước tính**: 2-4 tuần

---

## 📋 Phân tích hiện trạng

### Điều đã hoạt động (Native Agent)

1. `ThreadsDatabase` trong `crates/agent/src/db.rs` lưu threads vào SQLite
2. `HistoryStore` trong `crates/agent/src/history_store.rs` quản lý history entries
3. `NativeAgent::save_thread()` tự động lưu khi thread thay đổi

### Điều chưa hoạt động (External Agents)

1. `AcpConnection` (external agents) không có logic lưu thread
2. Không có cách serialize/deserialize ACP thread state
3. Khi resume thread, chỉ native agent có `open_thread()` method

Theo tài liệu tại `docs/src/ai/external-agents.md`:

> Note that some first-party agent features don't yet work with Claude Code: editing past messages, **resuming threads from history**, and checkpointing.
> We hope to add these features in the near future.

---

## 🔧 Giải pháp đề xuất

### Cách tiếp cận: **Lưu state ở phía Zed (Client-side persistence)**

Thay vì yêu cầu external agents implement save/load (sẽ cần thay đổi ACP protocol), ta lưu conversation state ở phía Zed:

```
┌─────────────────┐     ACP Protocol      ┌──────────────────┐
│   Zed Client    │◄─────────────────────►│  External Agent  │
│                 │                        │ (Claude/Gemini)  │
│  ┌───────────┐  │                        └──────────────────┘
│  │ Database  │  │  ← Lưu conversation history tại đây
│  └───────────┘  │
└─────────────────┘
```

---

## 📝 Implementation Steps

### Phase 1: Chuẩn bị & Nghiên cứu (1-2 ngày)

1. **Đọc kỹ ACP Protocol spec**:
   - https://agentclientprotocol.com
   - Hiểu các message types và session lifecycle

2. **Set up development environment**:

   ```sh
   # Build Zed from source
   cargo build --release
   ```

3. **Chạy tests hiện tại**:

   ```sh
   cargo test -p agent
   cargo test -p acp_thread
   ```

### Phase 2: Extend Database Schema (2-3 ngày)

**File cần sửa**: `crates/agent/src/db.rs`

1. **Thêm column để lưu agent type**:

   ```rust
   // Trong DbThreadMetadata, thêm:
   pub struct DbThreadMetadata {
       pub id: acp::SessionId,
       pub title: SharedString,
       pub updated_at: DateTime<Utc>,
       pub agent_type: AgentType,  // NEW: "native", "claude_code", "gemini", "codex", "custom"
   }

   pub enum AgentType {
       Native,
       ClaudeCode,
       Gemini,
       Codex,
       Custom(String),
   }
   ```

2. **Migration script cho existing database**

### Phase 3: Modify AcpThread to be Serializable (3-5 ngày)

**Files cần sửa**:

- `crates/acp_thread/src/acp_thread.rs`
- `crates/acp_thread/src/connection.rs`

1. **Thêm method `to_db()` cho AcpThread**:

   ```rust
   impl AcpThread {
       /// Serialize thread state for database storage
       pub fn to_db(&self, cx: &App) -> DbThread {
           DbThread {
               title: self.title.clone(),
               messages: self.entries.iter()
                   .map(|entry| entry.to_db_message())
                   .collect(),
               updated_at: Utc::now(),
               // ... other fields
           }
       }

       /// Restore thread from database
       pub fn from_db(db_thread: DbThread, connection: Rc<dyn AgentConnection>, cx: &mut Context<Self>) -> Self {
           // Reconstruct thread state
       }
   }
   ```

2. **Implement `AgentThreadEntry::to_db_message()`** cho mỗi loại entry

### Phase 4: Extend AgentConnection trait (2-3 ngày)

**File**: `crates/acp_thread/src/connection.rs`

```rust
pub trait AgentConnection {
    // Existing methods...

    // NEW: Support for resuming from saved state
    fn resume_from_history(
        self: Rc<Self>,
        db_thread: DbThread,
        project: Entity<Project>,
        cx: &mut App,
    ) -> Task<Result<Entity<AcpThread>>> {
        // Default implementation: not supported
        Task::ready(Err(anyhow!("Resume from history not supported")))
    }
}
```

### Phase 5: Implement Save Logic for External Agents (3-4 ngày)

**Files**:

- `crates/agent_servers/src/acp.rs`
- `crates/agent_ui/src/acp/thread_view.rs`

1. **Hook vào thread updates để auto-save**:

   ```rust
   // Trong AcpThreadView hoặc một wrapper
   impl AcpThreadView {
       fn observe_thread_changes(&mut self, cx: &mut Context<Self>) {
           // Khi thread thay đổi (new message, etc.)
           cx.observe(&self.thread, |this, thread, cx| {
               this.save_thread_to_history(thread, cx);
           }).detach();
       }

       fn save_thread_to_history(&self, thread: Entity<AcpThread>, cx: &mut Context<Self>) {
           let db_thread = thread.read(cx).to_db(cx);
           let database = ThreadsDatabase::connect(cx);
           let agent_type = self.get_agent_type();

           cx.spawn(async move |_, _| {
               database.await?.save_external_thread(id, agent_type, db_thread).await
           }).detach();
       }
   }
   ```

### Phase 6: Implement Resume Logic (3-4 ngày)

**Khi user click vào history entry của external agent**:

```rust
// Trong AgentPanel hoặc AcpThreadHistory
fn open_history_entry(&mut self, entry: HistoryEntry, window: &mut Window, cx: &mut Context<Self>) {
    match entry {
        HistoryEntry::AcpThread(metadata) => {
            let agent_type = metadata.agent_type;

            // 1. Load saved thread from database
            let db_thread = self.history_store.load_thread(metadata.id, cx);

            // 2. Get appropriate agent server
            let server = self.get_agent_server_for_type(agent_type, cx);

            // 3. Create new connection and restore state
            cx.spawn(async move |this, cx| {
                let db_thread = db_thread.await?;
                let connection = server.connect(...).await?;

                // 4. Create new thread with restored messages
                let thread = connection.resume_from_history(db_thread, project, cx).await?;

                // 5. Display in UI
                this.update(cx, |this, cx| this.show_thread(thread, cx))?;
                Ok(())
            }).detach();
        }
        // ...
    }
}
```

### Phase 7: UI Updates (2-3 ngày)

1. **Hiển thị agent type trong history list**
2. **Icon khác nhau cho mỗi agent type**
3. **Warning khi resume có thể không hoàn hảo**

### Phase 8: Testing (3-4 ngày)

1. **Unit tests**:

   ```rust
   #[gpui::test]
   async fn test_save_load_external_agent_thread(cx: &mut TestAppContext) {
       // Create thread with Claude Code
       // Send some messages
       // Save to database
       // Load from database
       // Verify messages are restored
   }
   ```

2. **Integration tests**
3. **Manual testing với mỗi external agent**

---

## 📁 Files cần thay đổi

| File                                       | Thay đổi                                  |
| ------------------------------------------ | ----------------------------------------- |
| `crates/agent/src/db.rs`                   | Thêm agent_type, migration                |
| `crates/agent/src/history_store.rs`        | Support external agent threads            |
| `crates/acp_thread/src/acp_thread.rs`      | `to_db()`, `from_db()` methods            |
| `crates/acp_thread/src/connection.rs`      | `resume_from_history()` trait method      |
| `crates/agent_servers/src/acp.rs`          | Implement resume logic                    |
| `crates/agent_ui/src/acp/thread_view.rs`   | Auto-save observer                        |
| `crates/agent_ui/src/acp/thread_history.rs`| UI for external agent history             |
| `crates/agent_ui/src/agent_panel.rs`       | Resume flow                               |

---

## ⚠️ Challenges & Considerations

### 1. **State không hoàn toàn khôi phục được**

- External agents có internal state mà Zed không biết
- **Giải pháp**: Khi resume, re-send conversation history như context mới

### 2. **ACP Protocol limitations**

- Có thể cần extend protocol để support proper session resumption
- **Giải pháp**: Fallback to "replay" mode nếu agent không support

### 3. **Tool call state**

- Pending tool calls có thể khó khôi phục
- **Giải pháp**: Chỉ lưu completed interactions

---

- Comment trong issue để confirm approach với maintainers
- Có thể họ đã có plans hoặc suggestions
=======
## 🚀 Quy trình đóng góp (Contributing)

### ⚠️ Cảnh báo: Zed team có thể từ chối PR

Dựa trên lịch sử PRs (#42387, #41874), Zed team đang tự develop tính năng này và **đã từ chối multiple PRs** từ community với lý do "we're cooking something up".

### Nếu vẫn muốn contribute:

### Bước 1: ~~Tạo GitHub Issue~~ → Issue đã tồn tại!

**KHÔNG CẦN tạo issue mới!** Issue đã được tạo và có 150+ votes:

👉 **[#37074 - Support history with external ACP agents](https://github.com/zed-industries/zed/issues/37074)**

### Bước 2: Hỏi trước khi code!

Comment trong issue để:
- Hỏi xem họ đã làm đến đâu
- Hỏi có accept contribution không
- Tránh lãng phí thời gian như xipeng-jin và aeroxy
=======

- Comment trong issue để confirm approach với maintainers
- Có thể họ đã có plans hoặc suggestions

### Bước 3: Fork & Branch

```sh
git fork zed-industries/zed
git checkout -b feature/external-agent-history-persistence
```

### Bước 4: Implement từng phase, mỗi phase 1 commit rõ ràng

### Bước 5: PR với:

- Clear description
- Screenshots/recordings
- Test evidence
- Sign CLA (Contributor License Agreement)
- Reference issue: `Closes #37074`

---

## 🔍 Thông tin từ GitHub Issues

### Issue #37074 - Summary
- **Reporter**: aexvir
- **Date**: Aug 28, 2025
- **Problem**: Threads created with Gemini agent via ACP are not saved to history
- **Steps to reproduce**:
  1. Create new thread with gemini agent
  2. Write something
  3. Create a new thread
  4. Expected: Thread from step 2 is in history
  5. Actual: Thread is nowhere to be found

### Community Impact
- 150+ upvotes cho thấy đây là tính năng được nhiều người cần
- Nhiều duplicate issues cho thấy users đang gặp vấn đề này thường xuyên

---

## 💡 Recommendations

1. **Bắt đầu nhỏ**: Implement cho 1 agent trước (recommend: native agent's save logic as reference)

2. **Đọc existing tests**: `cargo test -p agent` để hiểu expected behavior

3. **Join Zed Discord**: https://discord.gg/zed - để hỏi maintainers

4. ~~**Check existing issues**~~: ✅ Đã check - Issue #37074 đang open, chưa có ai được assign

5. **Rust experience**: Nếu chưa quen Rust, đây là project khá complex. Recommend làm quen với `gpui` framework trước.

6. **⚠️ Xem xét fork**: Nếu cần tính năng này ngay, consider fork như [wzulfikar](https://github.com/wzulfikar/zed) đã làm

---

## 📚 References

- [Zed Contributing Guide](../CONTRIBUTING.md)
- [ACP Protocol](https://agentclientprotocol.com)
- [External Agents Documentation](src/ai/external-agents.md)
- [Native Agent Implementation](../crates/agent/src/agent.rs)

---

## 📅 Status

- [ ] Phase 1: Research & Setup
- [ ] Phase 2: Database Schema
- [ ] Phase 3: AcpThread Serialization
- [ ] Phase 4: AgentConnection Extension
- [ ] Phase 5: Save Logic
- [ ] Phase 6: Resume Logic
- [ ] Phase 7: UI Updates
- [ ] Phase 8: Testing

---

## 🔗 Quick Links

| Resource | URL |
|----------|-----|
| Main Issue | https://github.com/zed-industries/zed/issues/37074 |
| PR #42387 (closed) | https://github.com/zed-industries/zed/pull/42387 |
| PR #41874 (closed) | https://github.com/zed-industries/zed/pull/41874 |
| wzulfikar's fork với tabs | https://github.com/wzulfikar/zed/pull/8 |
| ACP Protocol | https://agentclientprotocol.com |
| Zed Discord | https://discord.gg/zed |
| Contributing Guide | https://github.com/zed-industries/zed/blob/main/CONTRIBUTING.md |
| CLA | https://zed.dev/cla |

---

_Created: 2025_
_Last Updated: 2025_
_GitHub Issue: #37074 (150+ votes, Open)_

---

## 📊 Tóm tắt tình hình

| Aspect | Status |
|--------|--------|
| Issue tồn tại | ✅ #37074 (150+ votes) |
| Zed team aware | ✅ Đã xác nhận đang làm |
| Community PRs | ❌ Đã bị reject (2 PRs) |
| Timeline | ⏳ "soon enough", "stay tuned" |
| Workaround | 📝 Export as Markdown, hoặc fork |

**Khuyến nghị**: Chờ Zed team release hoặc fork nếu cần ngay.