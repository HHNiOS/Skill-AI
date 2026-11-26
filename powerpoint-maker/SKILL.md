---
name: powerpoint-maker
description: Tạo và chỉnh sửa file thuyết trình PowerPoint (.pptx) với phong cách đơn giản, hiện đại, đẹp mắt và sinh động bằng python-pptx. Hãy dùng skill này bất cứ khi nào người dùng nhắc đến slide, bài thuyết trình, PowerPoint, pptx, deck, pitch deck, báo cáo dạng slide, hoặc muốn chuyển nội dung (văn bản, số liệu, tài liệu) thành slide, kể cả khi họ không nói rõ định dạng file. Đảm bảo bố cục sạch, màu sắc hài hòa, không tràn chữ, không đè lên nhau.
---

# Skill: Tạo PowerPoint đơn giản – đẹp – sinh động – hiện đại

Mục tiêu: mỗi slide nhìn vào hiểu ngay, thoáng, có điểm nhấn, trông như do nhà thiết kế làm, không phải "slide chữ" nhàm chán.

---

## 1. Quy trình làm việc

1. **Hiểu yêu cầu:** chủ đề, đối tượng khán giả, mục đích (thuyết phục, báo cáo, giảng dạy), số slide mong muốn. Nếu người dùng không nói, tự chọn hợp lý (6–10 slide) và nêu giả định trong 1 dòng, không hỏi dồn.
2. **Lên dàn ý:** mỗi slide = 1 ý chính. Viết sẵn tiêu đề dạng *thông điệp* (ví dụ "Doanh thu tăng 32% nhờ kênh online") thay vì tiêu đề chung chung ("Doanh thu").
3. **Chọn bảng màu** (mục 3) và **phông chữ** (mục 4) rồi dùng nhất quán cho cả bài.
4. **Viết một script Python hoàn chỉnh** (`build_slides.py`) dùng `python-pptx`, có các hàm dựng sẵn cho từng kiểu slide.
5. **Chạy script, xuất `.pptx`**, kiểm tra theo checklist (mục 8), sửa nếu cần, rồi gửi file cho người dùng.

Cài đặt nếu thiếu: `pip install python-pptx --break-system-packages`

---

## 2. Nguyên tắc vàng

- **Ít chữ, nhiều không gian:** mỗi slide 1 ý chính, tối đa 4 gạch đầu dòng, mỗi dòng ≤ 12 từ. Không đoạn văn dài.
- **Khoảng trắng là sang trọng:** chừa lề, đừng lấp đầy mọi chỗ trống.
- **Một điểm nhấn mỗi slide:** một con số lớn, một icon, một khối màu, đừng làm mọi thứ đều nổi bật.
- **Nhất quán:** cùng màu, cùng phông, cùng vị trí tiêu đề trên mọi slide.
- **Sinh động bằng hình khối, không bằng hiệu ứng rối mắt:** dùng khối màu, hình tròn, thẻ bo góc, số liệu lớn, emoji/ký hiệu đơn giản thay cho ảnh khi không có ảnh thật.

---

## 3. Bảng màu (chọn 1 bộ, dùng xuyên suốt)

Chỉ dùng **1 màu nền + 1 màu chữ + 1 màu chính + 1 màu phụ nhấn**. Không dùng quá 4 màu.

| Phong cách | Nền | Chữ | Chính (Accent) | Phụ |
|---|---|---|---|---|
| **Xanh hiện đại** (mặc định) | `#F8FAFC` | `#0F172A` | `#2563EB` | `#F59E0B` |
| **Tối sang trọng** | `#0F172A` | `#F1F5F9` | `#38BDF8` | `#F472B6` |
| **Xanh lá tươi mát** | `#F7FEF9` | `#14301F` | `#10B981` | `#FB923C` |
| **Cam năng động** | `#FFFBF5` | `#2D1B0E` | `#F97316` | `#6366F1` |
| **Tím sáng tạo** | `#FAF8FF` | `#1E1033` | `#7C3AED` | `#EC4899` |

Quy tắc:
- Nền sáng → chữ tối. Nền tối → chữ sáng. Độ tương phản phải cao, dễ đọc từ xa.
- Dùng bản **nhạt** của màu chính (độ trong ~10–15%) cho nền thẻ/khung để tạo chiều sâu.
- Slide bìa và slide kết có thể đảo: nền màu chính, chữ trắng để tạo nhịp.

---

## 4. Phông chữ & cỡ chữ

- **Phông:** 1 phông cho cả bài. Ưu tiên `Segoe UI`, `Calibri`, hoặc `Arial` (có sẵn trên hầu hết máy). Có thể dùng phông đậm cho tiêu đề, phông thường cho nội dung.
- **Cỡ chữ (pt):**

| Thành phần | Cỡ | Ghi chú |
|---|---|---|
| Tiêu đề bìa | 44–54 | In đậm |
| Tiêu đề slide | 30–36 | In đậm, tối đa 2 dòng |
| Số liệu nổi bật | 60–96 | Màu chính, in đậm |
| Nội dung chính | 18–22 | Giãn dòng 1.2–1.3 |
| Chú thích / nguồn | 12–14 | Màu xám nhạt |

- Không bao giờ dùng chữ nhỏ hơn **12pt**.
- Căn trái cho đoạn chữ; căn giữa chỉ dùng cho bìa, số liệu lớn, trích dẫn.

---

## 5. Kích thước & lề (chống tràn chữ)

```python
from pptx import Presentation
from pptx.util import Inches, Pt

prs = Presentation()
prs.slide_width = Inches(13.333)   # 16:9
prs.slide_height = Inches(7.5)
blank = prs.slide_layouts[6]       # layout trống, tự dựng mọi thứ
```

- **Lề an toàn:** 0.7" trái/phải, 0.6" trên/dưới. Mọi khung chữ phải nằm trong vùng này.
- **Vùng nội dung:** rộng tối đa `13.333 - 2*0.7 = 11.93"`.
- **Tiêu đề:** `Left 0.7"`, `Top 0.5"`, rộng 11.9", cao 1.1".
- **Nội dung:** bắt đầu từ `Top ≈ 1.9"`.
- Luôn bật `word_wrap = True` và đặt `auto_size` hợp lý; **ước lượng độ dài chữ** trước khi đặt vào khung (xem mục 8).

---

## 6. Các kiểu bố cục (luân phiên, không lặp 2 slide giống nhau liền kề)

1. **Bìa (Title):** nền màu chính hoặc nền tối, tiêu đề lớn bên trái, phụ đề nhỏ bên dưới, một hình tròn/khối màu trang trí ở góc.
2. **Mục lục / Chương (Section):** số thứ tự rất lớn (`01`, `02`) màu nhấn + tên chương.
3. **Hai cột:** bên trái nội dung ngắn (3–4 ý), bên phải khối màu chứa 1 con số/ý tóm tắt nổi bật.
4. **Thẻ (Cards):** 3–4 thẻ hình chữ nhật bo góc đặt ngang, mỗi thẻ gồm icon/số + tiêu đề ngắn + 1 dòng mô tả. Nền thẻ = màu chính nhạt.
5. **Số liệu lớn (Big number):** 1–3 con số khổng lồ kèm nhãn nhỏ. Rất hợp cho KPI.
6. **Dòng thời gian / Quy trình:** các bước nối nhau bằng đường kẻ, mỗi bước là hình tròn có số.
7. **So sánh:** 2 cột đối xứng (Trước/Sau, A/B) hoặc bảng gọn.
8. **Biểu đồ:** dùng chart gốc của `python-pptx` khi có số liệu, tối đa 1 biểu đồ mỗi slide, bỏ lưới thừa, ghi chú ngắn bên cạnh.
9. **Trích dẫn:** chữ lớn nghiêng, có thanh màu dọc bên trái.
10. **Kết (Closing):** thông điệp chính + lời kêu gọi hành động + thông tin liên hệ.

**Nhịp điệu gợi ý cho bài 8 slide:** Bìa → Vấn đề (số liệu lớn) → Giải pháp (2 cột) → Tính năng (thẻ) → Quy trình (timeline) → Kết quả (biểu đồ) → So sánh → Kết.

---

## 7. Mẫu code dựng sẵn

Luôn viết các hàm trợ giúp ngắn gọn rồi tái sử dụng:

```python
from pptx import Presentation
from pptx.util import Inches, Pt
from pptx.dml.color import RGBColor
from pptx.enum.shapes import MSO_SHAPE
from pptx.enum.text import PP_ALIGN, MSO_ANCHOR

# ---------- Giao diện ----------
BG      = RGBColor(0xF8, 0xFA, 0xFC)
TEXT    = RGBColor(0x0F, 0x17, 0x2A)
ACCENT  = RGBColor(0x25, 0x63, 0xEB)
ACCENT2 = RGBColor(0xF5, 0x9E, 0x0B)
SOFT    = RGBColor(0xE0, 0xE9, 0xFD)   # màu chính nhạt cho thẻ
MUTED   = RGBColor(0x64, 0x74, 0x8B)
FONT    = "Segoe UI"
MX, MY  = 0.7, 0.6                      # lề

prs = Presentation()
prs.slide_width, prs.slide_height = Inches(13.333), Inches(7.5)

def new_slide(bg=BG):
    s = prs.slides.add_slide(prs.slide_layouts[6])
    s.background.fill.solid()
    s.background.fill.fore_color.rgb = bg
    return s

def add_text(slide, text, x, y, w, h, size=20, bold=False, color=TEXT,
             align=PP_ALIGN.LEFT, anchor=MSO_ANCHOR.TOP, spacing=1.2):
    tb = slide.shapes.add_textbox(Inches(x), Inches(y), Inches(w), Inches(h))
    tf = tb.text_frame
    tf.word_wrap = True
    tf.vertical_anchor = anchor
    tf.margin_left = tf.margin_right = Inches(0.05)
    p = tf.paragraphs[0]
    p.alignment, p.line_spacing = align, spacing
    r = p.add_run()
    r.text = text
    r.font.size, r.font.bold, r.font.name = Pt(size), bold, FONT
    r.font.color.rgb = color
    return tb

def add_box(slide, x, y, w, h, fill=SOFT, shape=MSO_SHAPE.ROUNDED_RECTANGLE, radius=0.08):
    sh = slide.shapes.add_shape(shape, Inches(x), Inches(y), Inches(w), Inches(h))
    sh.fill.solid(); sh.fill.fore_color.rgb = fill
    sh.line.fill.background()            # bỏ viền
    sh.shadow.inherit = False            # bỏ bóng mặc định
    if shape == MSO_SHAPE.ROUNDED_RECTANGLE:
        sh.adjustments[0] = radius
    return sh

def add_bullets(slide, items, x, y, w, size=20):
    tb = slide.shapes.add_textbox(Inches(x), Inches(y), Inches(w), Inches(4))
    tf = tb.text_frame; tf.word_wrap = True
    for i, t in enumerate(items):
        p = tf.paragraphs[0] if i == 0 else tf.add_paragraph()
        p.line_spacing, p.space_after = 1.25, Pt(12)
        r = p.add_run(); r.text = "●  "          # chấm màu nhấn
        r.font.size, r.font.color.rgb, r.font.name = Pt(size - 6), ACCENT, FONT
        r2 = p.add_run(); r2.text = t
        r2.font.size, r2.font.color.rgb, r2.font.name = Pt(size), TEXT, FONT

# ---------- Bìa ----------
def slide_title(title, subtitle):
    s = new_slide(ACCENT)
    add_box(s, 9.6, -1.2, 5.5, 5.5, ACCENT2, MSO_SHAPE.OVAL)       # hình tròn trang trí
    add_box(s, 10.8, 4.8, 3.5, 3.5, RGBColor(0x1D,0x4E,0xD8), MSO_SHAPE.OVAL)
    add_text(s, title, MX, 2.4, 9, 1.8, 48, True, RGBColor(255,255,255), anchor=MSO_ANCHOR.BOTTOM)
    add_text(s, subtitle, MX, 4.4, 9, 1, 22, False, RGBColor(0xDB,0xEA,0xFE))
    return s

# ---------- Thẻ ----------
def slide_cards(title, cards):               # cards = [(icon, tiêu_đề, mô_tả), ...]
    s = new_slide()
    add_text(s, title, MX, MY, 11.9, 1.1, 32, True)
    n = len(cards); gap = 0.4
    w = (13.333 - 2*MX - gap*(n-1)) / n
    for i, (icon, head, desc) in enumerate(cards):
        x = MX + i*(w + gap)
        add_box(s, x, 2.1, w, 4.3, SOFT)
        add_box(s, x+0.35, 2.45, 0.9, 0.9, ACCENT, MSO_SHAPE.OVAL)
        add_text(s, icon, x+0.35, 2.45, 0.9, 0.9, 26, True, RGBColor(255,255,255),
                 PP_ALIGN.CENTER, MSO_ANCHOR.MIDDLE)
        add_text(s, head, x+0.3, 3.6, w-0.6, 0.8, 22, True)
        add_text(s, desc, x+0.3, 4.4, w-0.6, 1.8, 16, False, MUTED)
    return s

# ---------- Số liệu lớn ----------
def slide_big_number(title, number, label, note=""):
    s = new_slide()
    add_text(s, title, MX, MY, 11.9, 1.1, 32, True)
    add_text(s, number, MX, 2.2, 7, 2.4, 96, True, ACCENT, anchor=MSO_ANCHOR.MIDDLE)
    add_text(s, label, MX, 4.6, 7, 0.8, 24, True)
    add_text(s, note, MX, 5.4, 7, 1.2, 16, False, MUTED)
    add_box(s, 8.6, 1.9, 4.0, 4.6, SOFT)
    return s
```

Thêm các hàm tương tự cho `slide_two_column`, `slide_timeline`, `slide_chart`, `slide_closing`, rồi gọi lần lượt và `prs.save("ten_file.pptx")`.

**Biểu đồ gốc:**

```python
from pptx.chart.data import CategoryChartData
from pptx.enum.chart import XL_CHART_TYPE

cd = CategoryChartData()
cd.categories = ["Q1", "Q2", "Q3", "Q4"]
cd.add_series("Doanh thu", (120, 150, 190, 240))
gf = slide.shapes.add_chart(XL_CHART_TYPE.COLUMN_CLUSTERED,
        Inches(MX), Inches(1.9), Inches(8), Inches(4.8), cd)
ch = gf.chart
ch.has_legend = False
ch.plots[0].gap_width = 60
ch.plots[0].series[0].format.fill.solid()
ch.plots[0].series[0].format.fill.fore_color.rgb = ACCENT
ch.value_axis.has_major_gridlines = False
```

---

## 8. Kiểm tra trước khi giao (bắt buộc)

**Chống tràn chữ:**
- Ước lượng nhanh: ở cỡ 20pt, 1 inch chiều rộng chứa ~7–8 ký tự; ở 32pt ~4–5 ký tự. Nếu `số ký tự > (chiều rộng × ký tự/inch × số dòng cho phép)` thì **rút gọn chữ hoặc giảm cỡ** (không giảm dưới 14pt cho nội dung, 24pt cho tiêu đề).
- Tiêu đề dài hơn 2 dòng → viết lại ngắn hơn.
- Không để khung chữ vượt lề `0.7"` (trái/phải) hoặc `0.6"` (trên/dưới).

**Chống chồng lấn:** các khối không đè lên nhau trừ khi cố ý (chữ trên thẻ, chữ trên hình tròn).

**Kiểm tra bằng hình ảnh (nếu có LibreOffice):**
```bash
soffice --headless --convert-to pdf ten_file.pptx
pdftoppm -jpeg -r 60 ten_file.pdf slide
```
Xem từng ảnh để tìm: chữ tràn, chữ bị cắt, chữ khó đọc trên nền, khối lệch hàng. Sửa rồi xuất lại.

**Checklist cuối:**
- [ ] Tỷ lệ 16:9, đúng số slide yêu cầu
- [ ] Mỗi slide có 1 ý chính, ≤ 4 gạch đầu dòng
- [ ] Màu và phông nhất quán, đủ tương phản
- [ ] Không có 2 slide liền kề cùng kiểu bố cục
- [ ] Có ít nhất 1 điểm nhấn thị giác ở mỗi slide
- [ ] Không có chữ tràn, cắt, hoặc đè lên nhau
- [ ] Chính tả tiếng Việt có dấu đầy đủ, đúng

---

## 9. Điều nên tránh

- Slide toàn chữ, đoạn văn dài, chữ nhỏ dưới 14pt.
- Hơn 4 màu, màu gắt chói mắt, gradient rối rắm.
- Dùng layout mặc định có sẵn khung "Click to add title" bị bỏ trống.
- Căn lề lộn xộn, mỗi slide một vị trí tiêu đề khác nhau.
- Clip-art cũ, WordArt, bóng đổ nặng, viền dày.
- Chèn ảnh co giãn méo tỷ lệ.

---

## 10. Khi người dùng muốn chỉnh sửa

- **File có sẵn:** mở bằng `Presentation("file.pptx")`, giữ nguyên theme gốc, chỉ sửa phần được yêu cầu, không tự ý đổi toàn bộ phong cách.
- **Yêu cầu nhỏ** (đổi màu, sửa chữ, thêm slide): chỉnh trong script rồi xuất lại, không dựng lại từ đầu.
- Sau khi giao file, tóm tắt trong 1–2 câu: số slide, phong cách/màu đã chọn, và các giả định đã đặt ra.
