---
name: imgui-ui-pro
description: Thiết kế giao diện ImGui đẹp, hiện đại cho app iOS (C++/ObjC++/Metal). Dùng khi tạo hoặc chỉnh menu ImGui, theme, widget custom, animation, hiệu ứng glass/glow/shadow. KHÔNG dùng khái niệm CSS/web; mọi thứ hiện thực bằng ImGuiStyle, Colors[], font TTF và ImDrawList.
---

# ImGui UI Pro (iOS / C++ / ObjC++)

## 0. Quy tắc nền tảng (đọc trước)

1. Đây là **ImGui immediate-mode**, không phải web. Không có CSS, blur, box-shadow, transition có sẵn. Mọi hiệu ứng phải tự vẽ bằng `ImDrawList` và tự animate bằng state + lerp.
2. Luôn làm theo thứ tự: **chọn style -> định nghĩa token -> áp theme -> font -> widget custom -> animation -> checklist**.
3. Không bao giờ để giao diện mặc định: font ProggyClean, border xám, màu xanh xám mặc định đều là lỗi.
4. File giao diện là `.mm` hoặc `.cpp`. Backend: `imgui_impl_metal` + `imgui_impl_ios` (hoặc binding của dự án).
5. Mọi kích thước đều qua `S(x)` (scale). Không hardcode pixel trần.
6. Chỉ dùng 1 style chính cho cả menu. Không trộn glass + neumorphism + brutalist.

## 1. Chọn phong cách

Hỏi người dùng hoặc suy ra từ mô tả. Nếu không nói rõ, mặc định **iOS Dark Minimal**.

| # | Style | Hợp với | Đặc trưng |
|---|---|---|---|
| 1 | **iOS Dark Minimal** | App tiện ích, tool | Nền đen/xám đậm, accent xanh, bo 12-16, không border, chữ sạch |
| 2 | **Liquid Glass** | Overlay, menu nổi | Panel trong mờ trắng 8-14%, viền sáng 1px, highlight trên đỉnh, shadow mềm |
| 3 | **Neon Cyber** | Game, mod menu | Nền tím đen, accent cyan/magenta, glow quanh viền, chữ sáng |
| 4 | **Aurora Gradient** | Menu thẩm mỹ cao | Nền gradient tím-xanh, accent gradient, card trong mờ |
| 5 | **Bento Card** | Dashboard, nhiều nhóm | Lưới card bo 16-20, nền card tách nhẹ, tiêu đề nhỏ viết hoa |
| 6 | **Soft Light (iOS Light)** | Giao diện sáng | Nền #F2F2F7, card trắng, shadow nhẹ, accent xanh |
| 7 | **Neumorphism Soft** | Điều khiển vật lý | Nền xám sáng, bóng sáng + bóng tối đối xứng, nổi/lõm |
| 8 | **Material 3 Tonal** | Giao diện kiểu Android | Surface có tint theo accent, pill button, ripple |
| 9 | **Brutalist Flat** | Cá tính mạnh | Viền đen dày 2-3, shadow cứng lệch, màu chói, góc vuông |
| 10 | **Terminal Hacker** | Công cụ dev | Đen + xanh lá/amber, monospace, góc vuông, chữ thường |

### Token từng style (hex RGB)

```
iOS Dark Minimal : bg 000000 | surface 1C1C1E | elevated 2C2C2E | sep 38383A | text FFFFFF | text2 8E8E93 | accent 0A84FF | ok 30D158 | warn FF9F0A | bad FF453A
Liquid Glass     : bg 0B1020->1B2A4A (gradient) | glass FFFFFF @10% | border FFFFFF @22% | text F5F7FF | text2 FFFFFF @60% | accent 64D2FF / 0A84FF
Neon Cyber       : bg 0B0B14 | surface 12121F | elevated 1A1A2E | border 2A2A4A | text E8F9FF | accent 00F0FF | accent2 FF2BD6 | ok 39FF88 | bad FF3B5C
Aurora Gradient  : bg 0F0C29->302B63->24243E | card FFFFFF @7% | text F4F1FF | accent 7F5AF0 -> 2CB67D | accent2 FF8906
Bento Card       : bg 0A0A0C | card 141417 | card2 1B1B20 | border 26262C | text EDEDF0 | text2 8A8A93 | accent 8B5CF6
Soft Light       : bg F2F2F7 | card FFFFFF | sep E5E5EA | text 000000 | text2 6C6C70 | accent 007AFF | ok 34C759 | bad FF3B30
Neumorphism Soft : bg E0E5EC | light FFFFFF | dark A3B1C6 | text 4A5568 | accent 6C63FF
Material 3 Tonal : bg 1C1B1F | surface 2B2930 | container 36343B | text E6E1E5 | accent D0BCFF | on-accent 381E72 | secondary CCC2DC
Brutalist Flat   : bg FFF8E7 | card FFFFFF | ink 000000 | accent FFD60A | accent2 FF5C8A | accent3 3DDC97
Terminal Hacker  : bg 0A0F0A | surface 0F1A0F | border 1F3A1F | text 33FF66 | text2 1F9E40 | warn FFB000 | bad FF4D4D
```

## 2. Design token chung (4pt grid)

```
Spacing   : 4, 8, 12, 16, 24, 32          (chỉ dùng các bậc này)
Radius    : sm 6 | md 10 | lg 14 | xl 20 | pill = height/2
Font size : caption 12 | body 15-16 | title 18-20 | header 24-28   (nhân scale)
Weight    : Regular cho body, Medium/Semibold cho tiêu đề, KHÔNG dùng bold giả
Touch     : vùng chạm tối thiểu 44pt, khoảng cách giữa 2 nút >= 8
Border    : 0 hoặc 1px mờ (alpha 6-20%). Không dùng viền xám đặc
Contrast  : chữ/nền >= 4.5:1, chữ phụ >= 3:1
Alpha rule: hover +6-8% sáng, pressed -6% tối, disabled alpha 40%
```

Font gợi ý: **Inter**, **SF Pro** (nếu được phép nhúng), **Poppins**, **Manrope**. Terminal: **JetBrains Mono**, **Fira Code**. Neon: **Orbitron** chỉ cho tiêu đề.

## 3. Nền tảng code (luôn có sẵn trong dự án)

```cpp
#include "imgui.h"
#include "imgui_internal.h"   // ShadeVertsLinearColorGradientKeepAlpha, ImLerp, ImSaturate

// ---------- Scale / màu ----------
static float g_scale = 1.f;                       // = [UIScreen mainScreen].scale hoặc tỉ lệ tuỳ chỉnh
inline float S(float v) { return v * g_scale; }

inline ImVec4 Hex(uint32_t rgb, float a = 1.f) {
    return ImVec4(((rgb >> 16) & 0xFF) / 255.f, ((rgb >> 8) & 0xFF) / 255.f, (rgb & 0xFF) / 255.f, a);
}
inline ImU32 U32(ImVec4 c) { return ImGui::ColorConvertFloat4ToU32(c); }
inline ImVec4 Alpha(ImVec4 c, float a) { c.w = a; return c; }
inline ImVec4 Mix(ImVec4 a, ImVec4 b, float t) {
    return ImVec4(a.x + (b.x - a.x) * t, a.y + (b.y - a.y) * t, a.z + (b.z - a.z) * t, a.w + (b.w - a.w) * t);
}

// ---------- Easing ----------
inline float EaseOutCubic(float t) { t = 1 - t; return 1 - t * t * t; }
inline float EaseInOut(float t)    { return t < .5f ? 4 * t * t * t : 1 - powf(-2 * t + 2, 3) / 2; }
inline float EaseOutBack(float t)  { const float c1 = 1.70158f, c3 = c1 + 1; t -= 1; return 1 + c3 * t * t * t + c1 * t * t; }

// ---------- Animation theo ID (state nằm trong ImGuiStorage) ----------
inline float Anim(ImGuiID id, float target, float speed = 14.f) {
    ImGuiStorage* st = ImGui::GetStateStorage();
    float v = st->GetFloat(id, target);
    v += (target - v) * ImMin(1.f, ImGui::GetIO().DeltaTime * speed);   // lerp mũ, mượt, không giật
    if (fabsf(target - v) < 0.001f) v = target;
    st->SetFloat(id, v);
    return v;
}
inline ImVec4 AnimColor(ImGuiID id, ImVec4 target, float speed = 14.f) {
    return ImVec4(Anim(id + 1, target.x, speed), Anim(id + 2, target.y, speed),
                  Anim(id + 3, target.z, speed), Anim(id + 4, target.w, speed));
}
```

## 4. Theme struct + áp vào ImGui

Mỗi style = 1 hàm điền `Theme`, rồi `ApplyTheme()` đẩy vào `ImGuiStyle`.

```cpp
struct Theme {
    ImVec4 bg, surface, elevated, border, text, text2, accent, accent2, ok, warn, bad;
    float  rWin, rCard, rCtrl;          // radius
    float  borderSize;                  // 0 hoặc 1
    bool   glass = false, glow = false, hardShadow = false;
};
static Theme T;

inline void ThemeIOSDark() {
    T = {};
    T.bg = Hex(0x000000); T.surface = Hex(0x1C1C1E); T.elevated = Hex(0x2C2C2E);
    T.border = Hex(0xFFFFFF, .08f); T.text = Hex(0xFFFFFF); T.text2 = Hex(0x8E8E93);
    T.accent = Hex(0x0A84FF); T.accent2 = Hex(0x64D2FF);
    T.ok = Hex(0x30D158); T.warn = Hex(0xFF9F0A); T.bad = Hex(0xFF453A);
    T.rWin = 16; T.rCard = 14; T.rCtrl = 10; T.borderSize = 0;
}
inline void ThemeGlass() {
    ThemeIOSDark();
    T.bg = Hex(0x0B1020); T.surface = Hex(0xFFFFFF, .10f); T.elevated = Hex(0xFFFFFF, .16f);
    T.border = Hex(0xFFFFFF, .22f); T.accent = Hex(0x64D2FF); T.text2 = Hex(0xFFFFFF, .60f);
    T.rWin = 22; T.rCard = 18; T.borderSize = 1; T.glass = true;
}
inline void ThemeNeon() {
    T = {};
    T.bg = Hex(0x0B0B14); T.surface = Hex(0x12121F); T.elevated = Hex(0x1A1A2E);
    T.border = Hex(0x2A2A4A); T.text = Hex(0xE8F9FF); T.text2 = Hex(0x7C86A8);
    T.accent = Hex(0x00F0FF); T.accent2 = Hex(0xFF2BD6);
    T.ok = Hex(0x39FF88); T.warn = Hex(0xFFD60A); T.bad = Hex(0xFF3B5C);
    T.rWin = 12; T.rCard = 10; T.rCtrl = 8; T.borderSize = 1; T.glow = true;
}
// Các style khác: điền Theme theo bảng token ở mục 1, đổi radius/flag cho đúng đặc trưng.
// Brutalist: rWin=rCard=rCtrl=0, borderSize=2, hardShadow=true. Terminal: radius 0-2, font mono.

inline void ApplyTheme() {
    ImGuiStyle& s = ImGui::GetStyle();
    s.WindowRounding = S(T.rWin);  s.ChildRounding = S(T.rCard);  s.FrameRounding = S(T.rCtrl);
    s.PopupRounding = S(T.rCard);  s.GrabRounding = S(T.rCtrl);   s.ScrollbarRounding = S(10);
    s.TabRounding = S(T.rCtrl);
    s.WindowPadding = ImVec2(S(16), S(16));  s.FramePadding = ImVec2(S(12), S(8));
    s.ItemSpacing = ImVec2(S(10), S(10));    s.ItemInnerSpacing = ImVec2(S(8), S(6));
    s.ScrollbarSize = S(6);  s.GrabMinSize = S(14);
    s.WindowBorderSize = T.borderSize; s.ChildBorderSize = T.borderSize; s.FrameBorderSize = 0; s.PopupBorderSize = T.borderSize;
    s.TouchExtraPadding = ImVec2(S(4), S(4));            // dễ chạm trên iOS
    s.WindowTitleAlign = ImVec2(.5f, .5f);

    ImVec4* c = s.Colors;
    c[ImGuiCol_Text]            = T.text;
    c[ImGuiCol_TextDisabled]    = T.text2;
    c[ImGuiCol_WindowBg]        = T.bg;
    c[ImGuiCol_ChildBg]         = T.surface;
    c[ImGuiCol_PopupBg]         = T.elevated;
    c[ImGuiCol_Border]          = T.border;
    c[ImGuiCol_FrameBg]         = T.elevated;
    c[ImGuiCol_FrameBgHovered]  = Mix(T.elevated, T.text, .06f);
    c[ImGuiCol_FrameBgActive]   = Mix(T.elevated, T.text, .10f);
    c[ImGuiCol_Button]          = T.accent;
    c[ImGuiCol_ButtonHovered]   = Mix(T.accent, ImVec4(1, 1, 1, 1), .12f);
    c[ImGuiCol_ButtonActive]    = Mix(T.accent, ImVec4(0, 0, 0, 1), .15f);
    c[ImGuiCol_Header]          = Alpha(T.accent, .18f);
    c[ImGuiCol_HeaderHovered]   = Alpha(T.accent, .28f);
    c[ImGuiCol_HeaderActive]    = Alpha(T.accent, .38f);
    c[ImGuiCol_SliderGrab]      = T.accent;
    c[ImGuiCol_SliderGrabActive]= Mix(T.accent, ImVec4(1, 1, 1, 1), .2f);
    c[ImGuiCol_CheckMark]       = T.accent;
    c[ImGuiCol_ScrollbarBg]     = ImVec4(0, 0, 0, 0);
    c[ImGuiCol_ScrollbarGrab]   = Alpha(T.text, .20f);
    c[ImGuiCol_Separator]       = Alpha(T.text, .08f);
    c[ImGuiCol_TitleBg]         = T.bg;  c[ImGuiCol_TitleBgActive] = T.bg;
}

inline void LoadFonts(const char* regular, const char* semibold) {
    ImGuiIO& io = ImGui::GetIO();
    ImFontConfig cfg; cfg.OversampleH = 3; cfg.OversampleV = 2; cfg.PixelSnapH = false;
    io.Fonts->AddFontFromFileTTF(regular,  S(16), &cfg);   // font 0: body
    io.Fonts->AddFontFromFileTTF(semibold, S(20), &cfg);   // font 1: title
    io.Fonts->AddFontFromFileTTF(semibold, S(12), &cfg);   // font 2: caption
    // ImGui 1.92+: dùng font động thay vì build theo size, PushFont(font, size).
}
```

## 5. Hiệu ứng hiện đại bằng ImDrawList

### 5.1 Shadow mềm (giả box-shadow)
```cpp
inline void Shadow(ImDrawList* dl, ImVec2 a, ImVec2 b, float r, float spread, ImVec4 col, ImVec2 off = ImVec2(0, 4)) {
    const int L = 10;
    for (int i = L; i >= 1; --i) {
        float t = (float)i / L, e = spread * t;
        float al = col.w * (1.f - t) * (1.f - t) * 0.5f;           // giảm dần ra ngoài
        dl->AddRectFilled(ImVec2(a.x - e + off.x, a.y - e + off.y), ImVec2(b.x + e + off.x, b.y + e + off.y),
                          U32(Alpha(col, al)), r + e);
    }
}
// Shadow cứng (Brutalist): dl->AddRectFilled(a + (4,4), b + (4,4), ink, 0); rồi vẽ card đè lên.
```

### 5.2 Glow (Neon): shadow màu accent, spread nhỏ, alpha cao
```cpp
inline void Glow(ImDrawList* dl, ImVec2 a, ImVec2 b, float r, ImVec4 col, float spread = 10) {
    Shadow(dl, a, b, r, S(spread), Alpha(col, .55f), ImVec2(0, 0));
}
```

### 5.3 Gradient bo góc (KHÔNG dùng AddRectFilledMultiColor vì nó không bo)
```cpp
inline void RoundGradient(ImDrawList* dl, ImVec2 a, ImVec2 b, float r, ImVec4 c0, ImVec4 c1, bool vertical = true) {
    int v0 = dl->VtxBuffer.Size;
    dl->AddRectFilled(a, b, IM_COL32_WHITE, r);
    ImVec2 p0 = a, p1 = vertical ? ImVec2(a.x, b.y) : ImVec2(b.x, a.y);
    ImGui::ShadeVertsLinearColorGradientKeepAlpha(dl, v0, dl->VtxBuffer.Size, p0, p1, U32(c0), U32(c1));
}
```

### 5.4 Glass panel (Liquid Glass)
Blur thật cần render nền ra texture rồi blur (Metal `MPSImageGaussianBlur`). Nếu chưa có, giả bằng lớp chồng:
```cpp
inline void GlassPanel(ImDrawList* dl, ImVec2 a, ImVec2 b, float r) {
    Shadow(dl, a, b, r, S(24), Hex(0x000000, .45f), ImVec2(0, S(10)));
    dl->AddRectFilled(a, b, U32(Hex(0x101828, .55f)), r);                       // lớp tối giả blur
    RoundGradient(dl, a, b, r, Hex(0xFFFFFF, .14f), Hex(0xFFFFFF, .04f));      // sáng dần từ trên xuống
    dl->AddRect(a, b, U32(Hex(0xFFFFFF, .22f)), r, 0, 1.f);                    // viền sáng
    dl->AddLine(ImVec2(a.x + r, a.y + 1), ImVec2(b.x - r, a.y + 1), U32(Hex(0xFFFFFF, .45f)), 1.f); // highlight cạnh trên
}
```
Mẹo: thêm nhiễu nhẹ (noise texture alpha 3%) để nhìn như kính nhám.

### 5.5 Nền Aurora
```cpp
inline void AuroraBG(ImDrawList* dl, ImVec2 a, ImVec2 b, float time) {
    dl->AddRectFilledMultiColor(a, b, U32(Hex(0x0F0C29)), U32(Hex(0x302B63)), U32(Hex(0x24243E)), U32(Hex(0x0F0C29)));
    ImVec2 c(a.x + (b.x - a.x) * (.3f + .1f * sinf(time * .4f)), a.y + (b.y - a.y) * .25f);
    for (int i = 6; i >= 1; --i)                                              // blob sáng mờ trôi chậm
        dl->AddCircleFilled(c, S(40.f * i), U32(Hex(0x7F5AF0, .04f)), 48);
}
```

### 5.6 Neumorphism (nổi/lõm)
```cpp
inline void Neu(ImDrawList* dl, ImVec2 a, ImVec2 b, float r, bool pressed) {
    float o = S(pressed ? 2 : 6);
    dl->AddRectFilled(ImVec2(a.x - o, a.y - o), ImVec2(b.x - o, b.y - o), U32(Hex(0xFFFFFF, pressed ? .3f : .9f)), r);
    dl->AddRectFilled(ImVec2(a.x + o, a.y + o), ImVec2(b.x + o, b.y + o), U32(Hex(0xA3B1C6, pressed ? .3f : .8f)), r);
    dl->AddRectFilled(a, b, U32(Hex(0xE0E5EC)), r);
}
```

## 6. Widget custom (copy rồi chỉnh theo Theme)

### 6.1 Toggle iOS (có animation, knob nảy nhẹ)
```cpp
inline bool Toggle(const char* label, bool* v) {
    ImGuiWindow* w = ImGui::GetCurrentWindow(); ImGuiID id = w->GetID(label);
    ImDrawList* dl = ImGui::GetWindowDrawList();
    float h = S(28), wd = S(48), r = h * .5f;
    ImVec2 p = ImGui::GetCursorScreenPos();
    ImGui::InvisibleButton(label, ImVec2(wd, h)); bool clicked = ImGui::IsItemClicked(); if (clicked) *v = !*v;
    float t = Anim(id, *v ? 1.f : 0.f, 16.f);
    ImVec4 col = Mix(Hex(0x39393D), T.ok, t);
    if (T.glow && *v) Glow(dl, p, ImVec2(p.x + wd, p.y + h), r, T.accent, 6);
    dl->AddRectFilled(p, ImVec2(p.x + wd, p.y + h), U32(col), r);
    float kx = p.x + r + (wd - h) * t;
    dl->AddCircleFilled(ImVec2(kx, p.y + r + S(1)), r - S(3) + S(1.5f) * ImGui::IsItemActive(), IM_COL32(0, 0, 0, 40), 24); // bóng
    dl->AddCircleFilled(ImVec2(kx, p.y + r), r - S(3) + S(1.5f) * ImGui::IsItemActive(), IM_COL32_WHITE, 24);
    ImGui::SameLine(); ImGui::AlignTextToFramePadding(); ImGui::TextUnformatted(label);
    return clicked;
}
```

### 6.2 Button gradient + hover + press scale
```cpp
inline bool GButton(const char* label, ImVec2 size) {
    ImGuiID id = ImGui::GetID(label); ImDrawList* dl = ImGui::GetWindowDrawList();
    ImVec2 p = ImGui::GetCursorScreenPos(); bool clk = ImGui::InvisibleButton(label, size);
    float hov = Anim(id, ImGui::IsItemHovered() ? 1.f : 0.f), act = Anim(id + 9, ImGui::IsItemActive() ? 1.f : 0.f, 22.f);
    float sh = act * S(2);                                       // lún nhẹ khi nhấn
    ImVec2 a(p.x + sh, p.y + sh), b(p.x + size.x - sh, p.y + size.y - sh);
    if (T.glow) Glow(dl, a, b, S(T.rCtrl), T.accent, 8 + 6 * hov);
    else if (!T.hardShadow) Shadow(dl, a, b, S(T.rCtrl), S(10), Alpha(T.accent, .35f), ImVec2(0, S(4)));
    RoundGradient(dl, a, b, S(T.rCtrl), Mix(T.accent2, T.accent, .0f + .2f * hov), T.accent);
    ImVec2 ts = ImGui::CalcTextSize(label);
    dl->AddText(ImVec2(a.x + (b.x - a.x - ts.x) * .5f, a.y + (b.y - a.y - ts.y) * .5f), U32(T.text), label);
    return clk;
}
```

### 6.3 Slider mảnh có fill + knob
```cpp
inline bool SliderF(const char* label, float* v, float mn, float mx, const char* fmt = "%.1f") {
    ImGuiID id = ImGui::GetID(label); ImDrawList* dl = ImGui::GetWindowDrawList();
    float w = ImGui::GetContentRegionAvail().x, h = S(32);
    ImVec2 p = ImGui::GetCursorScreenPos();
    ImGui::InvisibleButton(label, ImVec2(w, h));
    if (ImGui::IsItemActive()) *v = ImClamp(mn + (ImGui::GetIO().MousePos.x - p.x) / w * (mx - mn), mn, mx);
    float t = Anim(id, (*v - mn) / (mx - mn), 24.f), cy = p.y + h * .65f, tr = S(4);
    dl->AddRectFilled(ImVec2(p.x, cy - tr * .5f), ImVec2(p.x + w, cy + tr * .5f), U32(T.elevated), tr);
    RoundGradient(dl, ImVec2(p.x, cy - tr * .5f), ImVec2(p.x + w * t, cy + tr * .5f), tr, T.accent2, T.accent, false);
    float kr = S(8) + S(2) * ImGui::IsItemActive();
    dl->AddCircleFilled(ImVec2(p.x + w * t, cy), kr, IM_COL32_WHITE, 24);
    char buf[32]; snprintf(buf, 32, fmt, *v); ImVec2 ts = ImGui::CalcTextSize(buf);
    dl->AddText(p, U32(T.text), label); dl->AddText(ImVec2(p.x + w - ts.x, p.y), U32(T.text2), buf);
    return ImGui::IsItemActive();
}
```

### 6.4 Tab bar có indicator trượt (sidebar hoặc ngang)
```cpp
inline void Tabs(const char** names, int n, int* cur) {
    ImDrawList* dl = ImGui::GetWindowDrawList(); ImGuiID id = ImGui::GetID("tabs");
    float tw = ImGui::GetContentRegionAvail().x / n, th = S(40); ImVec2 p = ImGui::GetCursorScreenPos();
    float x = Anim(id, *cur * tw, 18.f);                                       // indicator trượt mượt
    dl->AddRectFilled(p, ImVec2(p.x + tw * n, p.y + th), U32(T.elevated), S(T.rCtrl + 2));
    dl->AddRectFilled(ImVec2(p.x + x + S(3), p.y + S(3)), ImVec2(p.x + x + tw - S(3), p.y + th - S(3)), U32(T.accent), S(T.rCtrl));
    for (int i = 0; i < n; ++i) {
        ImGui::SetCursorScreenPos(ImVec2(p.x + i * tw, p.y));
        if (ImGui::InvisibleButton(names[i], ImVec2(tw, th))) *cur = i;
        ImVec2 ts = ImGui::CalcTextSize(names[i]);
        dl->AddText(ImVec2(p.x + i * tw + (tw - ts.x) * .5f, p.y + (th - ts.y) * .5f),
                    U32(*cur == i ? T.text : T.text2), names[i]);
    }
    ImGui::SetCursorScreenPos(ImVec2(p.x, p.y + th + S(T.rCtrl)));
}
```

### 6.5 Card (nhóm điều khiển) + tiêu đề section
```cpp
inline void BeginCard(const char* title) {
    ImGui::PushStyleColor(ImGuiCol_ChildBg, T.glass ? ImVec4(0, 0, 0, 0) : T.surface);
    ImGui::BeginChild(title, ImVec2(0, 0), ImGuiChildFlags_AutoResizeY | ImGuiChildFlags_Borders, ImGuiWindowFlags_NoScrollbar);
    if (T.glass) { ImVec2 a = ImGui::GetWindowPos(); GlassPanel(ImGui::GetWindowDrawList(), a, ImVec2(a.x + ImGui::GetWindowWidth(), a.y + ImGui::GetWindowHeight()), S(T.rCard)); }
    ImGui::PushStyleColor(ImGuiCol_Text, T.text2); ImGui::TextUnformatted(title); ImGui::PopStyleColor();   // caption nhỏ, màu phụ
    ImGui::Dummy(ImVec2(0, S(4)));
}
inline void EndCard() { ImGui::EndChild(); ImGui::PopStyleColor(); }
```
Lưu ý: `AutoResizeY` trên child và vẽ nền glass trước nội dung có thể cần `dl->ChannelsSplit` hoặc vẽ ở frame sau dựa trên kích thước đã lưu. Nếu glitch, dùng chiều cao cố định.

### 6.6 Toast / thông báo trượt vào
Lưu `startTime`, `text`; vẽ bằng `GetForegroundDrawList()`; `t = EaseOutBack(clamp((now-start)/0.35))` để trượt từ trên xuống, tự tắt sau 2.5s với fade alpha.

### 6.7 Mở/đóng menu (quan trọng để "xịn")
Không bật/tắt tức thì. Dùng `float open = Anim(id, isOpen ? 1.f : 0.f, 12.f)`; `PushStyleVar(ImGuiStyleVar_Alpha, open)`; scale/trượt cửa sổ từ 0.96 -> 1.0 bằng `SetNextWindowPos` lệch theo `(1 - open) * S(12)`. Nếu `open < 0.01f` thì không gọi `Begin`.

## 7. Bố cục menu mẫu

```
Cửa sổ 360-420pt rộng (iPhone) | bo rWin | không title bar mặc định
 ├─ Header: tên app (font title) + nút đóng
 ├─ Tabs (segmented) 3-5 mục, tên ngắn
 ├─ Nội dung cuộn: BeginCard("NHÓM") -> Toggle / SliderF / GButton ...
 └─ Footer nhỏ: phiên bản (caption, text2)
```
- Mỗi card tối đa 5-7 điều khiển. Nhiều hơn thì tách card.
- Nhãn bên trái, điều khiển bên phải, căn thẳng hàng. Khoảng cách dọc đều nhau.
- Nút chính (accent) tối đa 1 nút mỗi màn. Nút phụ dùng nền `elevated`, chữ `text`.
- Trạng thái nguy hiểm (xoá, reset) dùng `T.bad`, đặt xa nút chính.

## 8. iOS riêng

- `g_scale = [UIScreen mainScreen].scale` (2 hoặc 3) hoặc chọn tay 1.0-1.4 theo mật độ muốn. Tạo font SAU khi biết scale, `io.FontGlobalScale = 1.f`.
- Chú ý **safe area**: lề trên/dưới tối thiểu `S(20)`/`S(34)` cho notch và home indicator.
- Cảm ứng: không dựa vào hover. Hiệu ứng hover vẫn viết, nhưng trạng thái *pressed* phải rõ.
- Cuộn bằng kéo: bật `io.ConfigWindowsMoveFromTitleBarOnly = true` để kéo nội dung không làm di chuyển cả cửa sổ.
- Haptic: gọi `UIImpactFeedbackGenerator` (ObjC++) khi toggle/chọn tab, rất đáng giá.
- Giới hạn FPS hợp lý (60, hoặc 120 với ProMotion), tránh vẽ shadow quá nhiều lớp mỗi frame: tối đa 8-10 lớp, cache khi có thể.

## 9. Quy trình cho agent khi được yêu cầu làm/sửa UI

1. Xác định style (mục 1). Nếu mơ hồ, chọn iOS Dark Minimal và nói rõ lựa chọn.
2. Viết/cập nhật `Theme` + `ApplyTheme()` + `LoadFonts()` trước, rồi mới viết widget.
3. Dùng lại helper ở mục 3-6, không tự viết lại bản kém hơn.
4. Mọi giá trị kích thước qua `S()`, mọi chuyển trạng thái qua `Anim()`.
5. Build, rồi tự đối chiếu checklist mục 10 và sửa trước khi báo xong.

## 10. Checklist trước khi giao

- [ ] Đã nạp font TTF, không còn ProggyClean
- [ ] Toàn bộ `ImGuiCol_*` quan trọng được set từ `Theme`
- [ ] Spacing theo bậc 4/8/12/16/24, radius thống nhất
- [ ] Có hover + pressed + disabled cho mọi điều khiển
- [ ] Có animation cho toggle, tab, mở/đóng menu (không bật tắt cứng)
- [ ] Chữ đủ tương phản, chữ phụ dùng `text2`
- [ ] Vùng chạm >= 44pt, không có 2 nút sát nhau
- [ ] Không trộn nhiều style, không quá 1 màu accent chính + 1 phụ
- [ ] Không có border xám mặc định, không có thanh scroll to
- [ ] Shadow/glow không quá 10 lớp mỗi phần tử, FPS ổn định

## 11. Lỗi hay gặp (tránh)

- Dùng `AddRectFilledMultiColor` rồi thắc mắc sao không bo góc: dùng `RoundGradient`.
- Đặt font size bằng `SetWindowFontScale` làm chữ mờ: hãy build font đúng size.
- Quên `PopStyleColor/PopStyleVar` làm lệch toàn bộ giao diện.
- Tạo font trong vòng lặp render (rất nặng).
- Hardcode màu rải rác thay vì lấy từ `T`.
- Dùng quá nhiều glow/gradient cùng lúc: nhìn rối. Chỉ nhấn vào 1-2 điểm chính.
