# Reflection — Thành viên 5 (Calculator + Lead + Data + Merge luồng)

---

## Role (Vai trò)

**Xử lý luồng tính toán** (giá xe, trả góp, chi phí sạc), **form lead** và mock lưu trữ.

Trên thực tế repo, vai trò được cụ thể hóa thành:

1. **Luồng tính toán:** Đồng bộ công thức và contract dữ liệu giữa Python (`src/05_calculator_lead/`) và JS phục vụ UI (`Hackathon/Demo/src/lib/calculator.js`), khớp tool `calculate_cost` trong agent.
2. **Làm sạch & chuẩn hóa data:** Một catalog `data/vehicles.json` dùng chung cho gợi ý, calculator và tools — tránh nhiều bản JSON/mock lệch nhau.
3. **Merge code = gom toàn bộ code trong luồng sản phẩm thành một pipeline chạy được:** Không chỉ “dán” từng file riêng lẻ, mà nối **trọn end-to-end** từ đăng nhập / guest → Compass (chat + intake / AI) → Top 3 → chọn xe → Calculator → Lead → (tuỳ cấu hình) API Python + log đánh giá. Mọi phần do các thành viên khác làm (UI, auth, recommend, prompt) **cùng đi vào một app Demo và một nguồn dữ liệu**, không còn nhánh tách đôi “mock riêng / engine riêng”.

---

## Đóng góp (Contributions)

### 1. Calculator & chi phí sở hữu

- Python: `src/05_calculator_lead/calculator.py`, `adapters.py`, `demo_calculator.py`.
- Web: `Hackathon/Demo/src/lib/calculator.js` + `CompassCalculator.jsx` (trả trước %, kỳ hạn, lãi suất → trả góp, sạc, tổng quan theo spec).
- Đảm bảo `vehicle_id` / slug từ Top 3 khớp calculator khi user chọn xe.

### 2. Lead form & lưu trữ mock

- `Hackathon/Demo/src/lib/leadStore.js`, `CompassLeadForm.jsx`.
- Python: `lead.py`, `demo_lead.py` khi cần xuất mock / CLI.

### 3. Data cleaning & catalog thống nhất

- `data/vehicles.json` là single source of truth cho recommend + calculator + agent.
- Rà soát field để filter/score không rỗng hoặc sai lệch khi đổi batch dữ liệu.

### 4. Merge toàn bộ code trong luồng (integration map)

“Merge” ở đây là **toàn bộ các lớp code** sau cùng phục vụ **một user journey**, không chỉ module calculator:

| Lớp | Vai trò trong luồng | Vị trí chính trong repo |
|-----|---------------------|-------------------------|
| Shell app | Route, khung ứng dụng | `Hackathon/Demo/src/App.jsx`, `main.jsx` |
| Auth / guest | Vào được Compass | `AuthScreen.jsx`, `authSession.js` |
| Compass UI | Chat, Top 3, calculator, lead | `CompassScreen.jsx`, `CompassChatbot.jsx`, `CompassRecommendations.jsx`, … |
| Intake cố định | 5 bước khi không dùng AI | `intakeFlow.js`, `intakeNatural.js` |
| Gợi ý & scoring | Top 3 từ profile | `recommendEngine.js` ↔ `src/03_recommendation_engine/recommend.py` |
| Agent OpenAI + tools | Chat thông minh, gọi recommend/calculate/compare | `openaiAgent.js`, `agentTools.js`, `extractAgentProfile.js`, `system_prompt.txt`, `flow.py` |
| Dữ liệu xe | Mọi bước đọc cùng catalog | `data/vehicles.json`, alias `@data` trong `vite.config.js` |
| API tùy chọn | Browser → FastAPI → Python engine | `server/agent_api.py`, env `VITE_AGENT_API_URL` |
| Log / đánh giá model (nếu bật) | Lưu phiên chat | `chatLogger.js`, thư mục log theo cấu hình server |

Công việc merge cụ thể: chỉnh **import/alias**, **contract JSON** (top3, vehicle_id), **env** (OpenAI + base URL API), **CORS**, và loại bỏ duplicate (mock JSON cũ, file Python/HTML lạc nhánh) để `npm run dev` + (optional) `uvicorn` chạy một luồng thống nhất.

---

## Reflection cá nhân (Personal reflection)

Calculator và lead tưởng nhỏ nhưng là **điểm user tin số**; lệch slug hoặc công thức là cả chuỗi “Top 3 → tính tiền → lead” gãy dù chat vẫn mượt.

**Data** quyết định chất lượng gợi ý nhiều hơn thuật toán: catalog sai ghế/phân khúc sẽ làm filter “gia đình” hoặc ngân sách lớn cho kết quả vô lý — phải sửa nguồn hoặc điều chỉnh engine cho phù hợp thực tế dữ liệu.

**Merge toàn bộ luồng** tốn công nhất ở chỗ: mọi người làm song song theo `document.md` (UI, auth, recommend, prompt, calculator) nhưng **chỉ khi gom vào một app + một `vehicles.json` + một chuỗi tool** thì mới demo được như một sản phẩm. Chi phí tích hợp nằm ở thống nhất env, slug, và không ai giữ bản data riêng.

Hướng cải thiện: test nhỏ cho calculator (Python + JS), validate schema/version cho `vehicles.json`, và log tool rõ ràng để đo lường model sau demo.

---
