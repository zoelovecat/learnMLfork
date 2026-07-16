---
name: compendium-study-log
description: >-
  Tracks and commits Maths CS AI Compendium study progress in learning/progress.md.
  Requires a 10-question comprehension quiz (70% pass) before marking a section completed
  and committing. Use when the user starts or ends a study session, asks to log learning
  progress, resume where they left off, commit study logs to git, take a section quiz,
  or mentions học xong / ghi log / tiếp tục học / đã học đến đâu / quiz.
---

# Compendium Study Log

Ghi log tiến độ học repo **maths-cs-ai-compendium**, cập nhật `learning/progress.md`, và commit lên git.

## Files

| File | Mục đích |
|------|----------|
| `learning/progress.md` | Single source of truth — resume pointer, checklist, session log |
| Section `.md` đang học | Nguồn để ra câu hỏi quiz |

## Bốn workflow

| Intent | Trigger (VN/EN) | Workflow |
|--------|-----------------|----------|
| **Start** | bắt đầu học, tiếp tục học, resume, học tiếp | [Start Session](#start-session) |
| **Log** | học xong, ghi log, log progress, kết thúc buổi học | [Log Session](#log-session) |
| **Quiz** | làm quiz, kiểm tra hiểu bài, test section | [Comprehension Quiz](#comprehension-quiz) |
| **Status** | đã học đến đâu, progress, xem log | [Show Status](#show-status) |

Nếu user vừa log vừa muốn push → commit trước, push sau (chỉ push khi user yêu cầu rõ).

---

## Start Session

1. Đọc `learning/progress.md`.
2. Lấy path từ mục **Resume Here**.
3. Xác nhận file tồn tại (folder dùng `chapter NN - topic/`, không phải `:`).
4. Trả lời ngắn gọn:
   - Section đang học / tiếp theo
   - Status hiện tại
   - Gợi ý 30 phút: đọc → tóm tắt 2 câu → coding task (nếu có)
   - Nhắc: muốn đánh dấu **completed** phải pass quiz 7/10
5. **Không commit** khi chỉ start.

---

## Comprehension Quiz

**Bắt buộc** trước khi `status: completed` và commit hoàn thành section.

### Khi nào chạy quiz

- User nói **học xong** / muốn **completed** → chạy quiz **trước**, chưa commit.
- User nói **làm quiz** / **kiểm tra hiểu bài** → chạy quiz độc lập (không commit nếu chưa qua gate).

`in_progress` hoặc `skipped` → **không** cần quiz; commit log trạng thái tạm được phép.

### Chuẩn bị

1. Xác định section file (từ user hoặc `learning/progress.md`).
2. **Đọc toàn bộ** file section đó — không hỏi từ nhớ chung chung.
3. Hỏi duration / notes / coding tasks song song nếu chưa có (ghi vào log sau).

### Ra đề

- **10 câu** xoay quanh nội dung section vừa đọc.
- Trộn loại: định nghĩa, intuition, công thức, ứng dụng ML, so sánh khái niệm, đọc code (nếu section có coding tasks).
- Độ khó vừa phải — kiểm tra **hiểu**, không cần chứng minh.
- Câu hỏi bằng **tiếng Việt** (thuật ngữ EN giữ nguyên khi cần).
- **Không** lộ đáp án khi hỏi.

### Cách hỏi

**Ưu tiên:** dùng `AskQuestion` — mỗi lần 1 câu, 4 lựa chọn (1 đúng, 3 nhiễu hợp lý).

Nếu không có `AskQuestion`: hỏi tuần tự trong chat, user trả lời `1`/`2`/`3`/`4` hoặc chữ cái.

Theo dõi nội bộ: `correct=0`, `total=10`, danh sách câu sai (không ghi đáp án vào progress trước khi chấm xong).

### Chấm & ngưỡng

- **Pass:** ≥ **7/10** (70%).
- Sau mỗi câu: chỉ nói đúng/sai; câu sai → giải thích ngắn + trích ý từ section.
- Hết 10 câu → báo điểm `X/10`.

### Kết quả quiz

**PASS (≥7/10):**
1. Cho phép `status: completed`.
2. Tiếp tục [Log Session → Cập nhật progress](#cập-nhật-learningprogressmd) với quiz score.
3. Commit.

**FAIL (<7/10):**
1. **Không** tick checklist, **không** đổi Resume Here sang section kế.
2. **Không** commit `completed`.
3. Gợi ý ôn: liệt kê chủ đề câu sai + trỏ lại đoạn trong section file.
4. Đặt `status: in_progress` nếu user muốn ghi log buổi học (commit `in_progress` được, không cần pass quiz).
5. Mời **làm lại quiz** sau khi ôn.

User có thể retake ngay hoặc session khác — mỗi lần retake: đọc lại section, **ra 10 câu mới** (không lặp y hệt).

---

## Log Session

Thu thập từ user hoặc context:

- Section file path (bắt buộc)
- `completed` | `in_progress` | `skipped`
- Duration (phút) — hỏi nếu chưa có
- Notes — 1–3 câu
- Coding tasks — `done` | `partial` | `none`

### Gate trước commit

```text
status == completed  →  BẮT BUỘC pass quiz (≥7/10) trước
status == in_progress | skipped  →  bỏ qua quiz, commit được
```

Nếu user yêu cầu `completed` mà chưa quiz → chuyển sang [Comprehension Quiz](#comprehension-quiz), **dừng** — chưa sửa progress, chưa commit.

### Cập nhật `learning/progress.md`

1. **Session Log** — entry mới **trên cùng**:

```markdown
### YYYY-MM-DD — [Section title]

- **Section:** `chapter NN - topic/SS. name.md`
- **Status:** `completed` | `in_progress` | `skipped`
- **Duration:** N phút
- **Notes:** ...
- **Coding tasks:** done | partial | none
- **Quiz:** X/10 (pass) | X/10 (fail) | skipped
```

- `completed` → bắt buộc có `Quiz: X/10 (pass)` với X ≥ 7
- `in_progress` / `skipped` → `Quiz: skipped` hoặc `X/10 (fail)` nếu vừa thi

2. **Checklist** — tick `[x]` chỉ khi `completed` + quiz pass.

3. **Trạng thái hiện tại** — cập nhật bảng.

4. **Resume Here:**
   - `completed` → section kế (cùng chapter, sort `00.`, `01.`, …; hết chapter → đầu chapter tiếp)
   - `in_progress` → giữ section hiện tại
   - `skipped` → giữ section, ghi chú trong Notes

5. **Cập nhật lần cuối** — hôm nay.

### Tìm section tiếp theo

```text
1. List *.md trong folder chapter hiện tại, sort theo prefix số
2. Lấy file sau file vừa complete
3. Hết chapter → chapter folder "chapter (N+1) - *", file đầu tiên
```

### Git commit

```bash
git add learning/progress.md
git status
git diff --staged
```

Commit message:

```text
study(log): chNN sSS <section-slug> — <completed|in_progress|skipped>

<1 câu tóm tắt notes>. Quiz: X/10 (chỉ khi completed).
```

- `section-slug`: tên file bỏ prefix số, lowercase, hyphen
- Chỉ stage `learning/progress.md` trừ khi user yêu cầu thêm
- **Không push** trừ khi user nói push / lên github
- Không empty commit

### Sau commit

Báo user: section đã log, Resume Here, commit hash, quiz score (nếu có).

---

## Show Status

1. Đọc `learning/progress.md`.
2. Hiển thị: Resume Here, status table, 3 session gần nhất (kèm quiz score), checklist chương.
3. Không commit.

---

## Quy ước

- **Không tự đánh dấu hoàn thành.** Chỉ tick `[x]` khi `completed` + quiz pass.
- **Không commit `completed` nếu quiz <7/10** — không ngoại lệ.
- Mở file để xem ≠ đã học.
- **Track:** `Web dev → ML (30 phút/ngày)` — giữ trừ khi user đổi.
- **Phase:** Ch 01–05 → Phase 1; Ch 06 → Phase 2; Ch 07+ → Phase 3+.
- Folder: `chapter NN - topic/` (dấu `-`).

## Ví dụ

**User:** "Học xong vector spaces, 30 phút"

→ Đọc `01. vector spaces.md` → quiz 10 câu → user 8/10 → `completed` → commit `study(log): ch01 s01 vector-spaces — completed`

**User:** "Học xong vector spaces" → quiz 5/10

→ Không `completed`, không commit pass → gợi ý ôn → hỏi ghi log `in_progress` hoặc retake quiz

**User:** "Học dở, 20 phút, chưa xong"

→ `in_progress`, quiz skipped → commit được

**User:** "Tiếp tục học"

→ Resume Here, nhắc quiz 7/10 trước khi hoàn thành section
