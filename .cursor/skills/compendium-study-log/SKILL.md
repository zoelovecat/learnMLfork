---
name: compendium-study-log
description: >-
  Tracks and commits Maths CS AI Compendium study progress in learning/progress.md.
  Use when the user starts or ends a study session, asks to log learning progress,
  resume where they left off, commit study logs to git, or mentions học xong / ghi log /
  tiếp tục học / đã học đến đâu.
---

# Compendium Study Log

Ghi log tiến độ học repo **maths-cs-ai-compendium**, cập nhật `learning/progress.md`, và commit lên git.

## Files

| File | Mục đích |
|------|----------|
| `learning/progress.md` | Single source of truth — resume pointer, checklist, session log |
| `llms.txt` | Tra cứu tên section khi cần |

## Ba workflow

Phát hiện intent từ user message:

| Intent | Trigger (VN/EN) | Workflow |
|--------|-----------------|----------|
| **Start** | bắt đầu học, tiếp tục học, resume, học tiếp | [Start Session](#start-session) |
| **Log** | học xong, ghi log, log progress, kết thúc buổi học | [Log Session](#log-session) |
| **Status** | đã học đến đâu, progress, xem log | [Show Status](#show-status) |

Nếu user vừa log vừa muốn push → commit trước, push sau (chỉ push khi user yêu cầu rõ).

---

## Start Session

1. Đọc `learning/progress.md`.
2. Lấy path từ mục **Resume Here**.
3. Xác nhận file tồn tại (folder dùng `chapter NN - topic/`, không phải `:`).
4. Trả lời ngắn gọn:
   - Section đang học / tiếp theo
   - Status (`in_progress` / `completed`)
   - Gợi ý 30 phút: đọc → tóm tắt 2 câu → coding task (nếu có)
5. **Không commit** khi chỉ start.

---

## Log Session

Thu thập từ user hoặc suy ra từ context (file đang mở, nội dung vừa thảo luận):

- Section file path (bắt buộc)
- `completed` | `in_progress` | `skipped`
- Duration (phút) — hỏi nếu chưa có
- Notes — 1–3 câu: hiểu gì, còn thắc mắc gì
- Coding tasks — `done` | `partial` | `none`

### Cập nhật `learning/progress.md`

1. **Session Log** — thêm entry mới **trên cùng** (dưới comment `<!-- Mỗi buổi học... -->`):

```markdown
### YYYY-MM-DD — [Section title]

- **Section:** `chapter NN - topic/SS. name.md`
- **Status:** `completed` | `in_progress` | `skipped`
- **Duration:** N phút
- **Notes:** ...
- **Coding tasks:** done | partial | none
```

2. **Checklist chương** — tick `[x]` nếu `completed`; `(đang học)` nếu `in_progress`.

3. **Trạng thái hiện tại** — cập nhật bảng Status/File.

4. **Resume Here** — set section tiếp theo:
   - `completed` → section kế trong cùng chapter (sort theo số file `00.`, `01.`, …)
   - Hết chapter → section đầu chapter kế (`chapter (N+1) - …`)
   - `in_progress` → giữ nguyên section hiện tại
   - `skipped` → giữ section đó trong Resume, ghi chú skipped trong Notes

5. **Cập nhật lần cuối** — ngày hôm nay.

### Tìm section tiếp theo

```text
1. List *.md trong folder chapter hiện tại, sort theo prefix số
2. Lấy file sau file vừa complete
3. Nếu không còn → list chapter folders matching "chapter NN - *", sort NN, lấy file đầu chapter tiếp
```

### Git commit

Chạy tuần tự (workspace = repo root):

```bash
git add learning/progress.md
git status
git diff --staged
```

Commit message (HEREDOC):

```bash
git commit -m "$(cat <<'EOF'
study(log): chNN sSS <section-slug> — <completed|in_progress|skipped>

<1 câu tóm tắt notes của user>
EOF
)"
```

- `section-slug`: tên file bỏ prefix số, lowercase, hyphen (vd. `norms-and-metrics`)
- Chỉ stage `learning/progress.md` trừ khi user yêu cầu thêm file khác
- **Không push** trừ khi user nói push / lên github / đẩy lên remote
- Nếu không có thay đổi → báo user, không tạo empty commit

### Sau commit

Báo user:
- Đã log section nào
- **Lần sau học:** path từ Resume Here
- Commit hash (nếu commit thành công)

---

## Show Status

1. Đọc `learning/progress.md`.
2. Hiển thị: Resume Here, status table, 3 session log gần nhất, tiến độ checklist chương hiện tại.
3. Không commit.

---

## Quy ước

- **Không tự đánh dấu hoàn thành.** Chỉ tick `[x]` khi user xác nhận rõ trong buổi log (`học xong`, `completed`). Mở file để xem ≠ đã học.
- **Track** mặc định user: `Web dev → ML (30 phút/ngày)` — giữ trong progress.md trừ khi user đổi.
- **Phase** map nhanh: Ch 01–05 → Phase 1; Ch 06 → Phase 2; Ch 07+ → Phase 3+.
- Folder chapter trên disk: `chapter NN - topic/` (dấu `-`, không `:`).
- User đang học **tuần tự** — không tự động nhảy chapter trừ khi user yêu cầu.

## Ví dụ

**User:** "Học xong norms and metrics, 30 phút, đã chạy coding tasks"

→ Status `completed` cho `chapter 01 - vectors/03. norms and metrics.md`
→ Resume Here → `chapter 01 - vectors/04. products.md`
→ Commit: `study(log): ch01 s03 norms-and-metrics — completed`

**User:** "Tiếp tục học"

→ Đọc Resume Here, mở `04. products.md`, nhắc mục tiêu 30 phút.

**User:** "Ghi log và push lên github"

→ Log workflow + `git push` sau commit thành công.
