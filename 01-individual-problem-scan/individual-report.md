# Case mới: Dự báo số ngày chất lượng còn lại của thanh long trong container lạnh

**Nhân vật ví dụ:** Hà, điều phối viên chất lượng chuỗi lạnh tại một doanh nghiệp xuất khẩu thanh long. Với mỗi chuyến hàng, Hà phải đánh giá liệu các pallet còn đủ chất lượng khi đến nơi, pallet nào cần kiểm tra hoặc phân phối trước, và công đoạn nào có thể đã gây suy giảm nếu khách hàng khiếu nại. Hiện nay, ảnh kiểm phẩm, phiếu QC, dữ liệu tiền lạnh, sơ đồ xếp pallet, cảm biến container, ETA và kết quả kiểm hàng tại điểm đến nằm ở nhiều hệ thống khác nhau. Hà thường chỉ thấy cảnh báo vượt ngưỡng nhiệt độ, chứ chưa biết lô hàng đã mất bao nhiêu ngày chất lượng.

Trong case này, **số ngày chất lượng còn lại (Remaining Shelf Life — RSL)** là số ngày dự kiến trước khi pallet vượt ngưỡng chất lượng thương mại đã thống nhất, chẳng hạn độ cứng, hao hụt khối lượng, màu vỏ/tai quả, tỷ lệ hư hỏng hoặc tiêu chuẩn nhận hàng. RSL **không phải** kết luận về an toàn thực phẩm.

## Vì sao đây là ví dụ tốt?

- Có actor cụ thể và workflow lặp lại theo từng chuyến xuất khẩu.
- Có bottleneck rõ ràng: dữ liệu phân mảnh và quyết định dựa nhiều vào kinh nghiệm.
- Có metric kỹ thuật, vận hành và kinh doanh để kiểm chứng.
- AI phải kết hợp ảnh, dữ liệu QC, chuỗi thời gian cảm biến, vị trí pallet và ETA.
- Có thể so sánh Rule / Workflow / Agent và vẽ before/after.
- Market scan cho thấy đã có công cụ giám sát reefer và dự báo shelf-life, nhưng chưa thấy bằng chứng công khai về một giải pháp kết hợp đầy đủ mô hình riêng cho thanh long Việt Nam, chênh lệch giữa pallet và truy nguyên công đoạn có lưu vết.
- Con người vẫn phê duyệt mọi quyết định kiểm hàng, phân phối và khiếu nại.

## 01 — Individual Problem Scan

### Scan rộng

Hà scan các vấn đề trong quy trình. Các con số dưới đây là giả định thiết kế và mục tiêu cần kiểm chứng trong pilot, không phải số liệu vận hành đã được xác nhận.

| # | Lăng kính | Problem quan sát được | Ai chịu ảnh hưởng? | Dấu hiệu thật |
|---:|---|---|---|---|
| 1 | Lặp lại | Ghi nhận độ chín, ngoại quan và lỗi đầu vào cho từng lô | Nhân viên QC | Lặp lại ở mỗi ca đóng hàng |
| 2 | Tốn thời gian | Ghép phiếu QC, sơ đồ pallet, cảm biến và ETA bằng tay | Hà, nhân viên QC | 30–60 phút/chuyến |
| 3 | AI có thể tốt hơn | Cảnh báo nhiệt độ không cho biết đã mất bao nhiêu ngày chất lượng | Hà, bộ phận bán hàng | Có cảnh báo nhưng chưa có RSL |
| 4 | AI có thể tốt hơn | Một đánh giá chung che khuất khác biệt giữa các pallet | QC điểm đến | Chỉ có vài điểm đo trong container |
| 5 | Pain từ người khác | Người mua không biết pallet nào cần nhận hoặc bán trước | Nhà nhập khẩu | Chỉ quyết định sau khi mở hàng |
| 6 | Lặp lại | Nhân viên xem lại biểu đồ cảm biến cho mọi chuyến | Hà | Cùng thao tác, khác định dạng dữ liệu |
| 7 | Tốn thời gian | Kiểm tra ngẫu nhiên nhiều pallet khi có sự cố | QC điểm đến | Mở rộng kiểm sau khi phát hiện pallet lỗi |
| 8 | AI có thể tốt hơn | Khó tách ảnh hưởng của chất lượng đầu vào, tiền lạnh, xếp hàng và vận chuyển | Hà, quản lý QA | Nhiều nguyên nhân cho cùng biểu hiện hỏng |
| 9 | Pain từ người khác | Bộ phận bán hàng điều chỉnh đơn khi hàng đã đến | Bán hàng, khách hàng | Phản ứng muộn, giảm giá hoặc đổi khách |
| 10 | Pain từ người khác | Hồ sơ khiếu nại phải ghép từ nhiều bên | QA, logistics, pháp chế | Có thể mất 1–3 ngày/ca |

### Top 3

| Rank | Problem | Vì sao chọn | Điều còn chưa chắc |
|---:|---|---|---|
| 1 | Không biết mỗi pallet còn bao nhiêu ngày chất lượng | Tác động trực tiếp tới xuất hàng và bán hàng | Cần nhãn QC sau hành trình đủ tốt |
| 2 | Không biết pallet nào cần kiểm tra hoặc phân phối trước | Biến dự báo thành hành động vận hành | Cần ánh xạ pallet, vị trí và cảm biến |
| 3 | Khó truy nguyên công đoạn gây suy giảm | Giá trị lớn khi cải tiến quy trình hoặc khiếu nại | Nhân quả có thể không xác định được hoàn toàn |

## Problem Card #1 — Dự báo số ngày chất lượng còn lại của từng pallet

**Problem 1 câu:**  
Hà chưa thể dự báo nhất quán mỗi pallet thanh long còn bao nhiêu ngày đạt chuẩn thương mại tại thời điểm đến, nên quyết định chủ yếu dựa vào mẫu kiểm, tuổi quả và kinh nghiệm.

**Actor:**  
Điều phối viên chất lượng chuỗi lạnh và trưởng QC của doanh nghiệp xuất khẩu.

**Thời điểm / bối cảnh:**  
Từ lúc kiểm tra đầu vào, đóng pallet đến trong suốt hành trình. Mỗi pallet hoặc nhóm pallet phải có ID liên kết với lô nguyên liệu, kết quả QC và vị trí xếp. Nếu dữ liệu chỉ có ở cấp lô hoặc container, hệ thống phải dự báo ở đúng cấp đó thay vì tạo độ chính xác giả.

Để tạo ground truth cho RSL, pilot phải khóa điều kiện bảo quản sau khi đến, số thùng/quả đại diện cho mỗi pallet và lịch kiểm tra định kỳ cho tới khi mẫu vượt ngưỡng chất lượng đã thống nhất. Trường hợp chưa vượt ngưỡng khi kết thúc theo dõi được ghi nhận là dữ liệu kiểm duyệt phải (right-censored), không gán một ngày hỏng giả.

**Current workflow:**

1. QC lấy mẫu, phân hạng và ghi chất lượng đầu vào.
2. Nhân viên tiền lạnh, đóng thùng, tạo pallet và lập sơ đồ xếp.
3. Một số logger được đặt trong container.
4. Trong hành trình, hệ thống báo khi nhiệt độ vượt ngưỡng.
5. Hà kết hợp tuổi quả, cảnh báo và ETA để đánh giá chung cho container.
6. Khi hàng đến, nhà nhập khẩu mở mẫu và mới quyết định nhận, bán nhanh, giữ hoặc khiếu nại.

**Bottleneck:**  
Bước 5 — cảnh báo ngưỡng không chuyển được tác động tích lũy của chất lượng đầu vào, chậm tiền lạnh, dao động nhiệt và ETA thành số ngày chất lượng còn lại theo pallet.

**Impact:**

- Pallet yếu có thể được giao cho khách cần thời gian bán dài.
- Pallet tốt có thể bị giảm giá hoặc kiểm tra phá hủy không cần thiết.
- Bộ phận bán hàng khó điều chỉnh kênh bán trước khi hàng đến.
- Dữ liệu các chuyến trước chưa trở thành tri thức dự báo cho chuyến sau.

**Success metric:**

- Các KPI cấp pallet chỉ áp dụng cho tập pilot có ID, vị trí, dữ liệu đầu vào và nhãn theo pallet; chuyến không đủ điều kiện được đánh giá ở cấp lô/vùng hoặc nằm ngoài pilot này.
- Mốc đánh giá chính là snapshot tại ETA−48 giờ và chỉ dùng dữ liệu tồn tại trước mốc đó; tập train/test được tách theo chuyến hoặc theo thời gian, không chia ngẫu nhiên các pallet cùng container.
- Với mẫu đã quan sát được ngày vượt ngưỡng, sai số tuyệt đối trung bình của RSL không quá 2 ngày. Trên toàn bộ dữ liệu kể cả trường hợp right-censored, Integrated Brier Score trong 14 ngày sau khi đến (càng thấp càng tốt) phải thấp hơn ít nhất 20% so với bảng quy tắc trên cùng tập kiểm thử — mục tiêu pilot.
- Khoảng dự báo chứa ngày suy giảm thực tế ở ít nhất 80% pallet, với độ rộng trung bình không quá 4 ngày — mục tiêu pilot.
- Với pallet thực tế không đạt trong vòng ba ngày sau khi đến, recall đạt ít nhất 85% và precision đạt ít nhất 70% tại cùng ngưỡng cảnh báo — mục tiêu pilot.
- Cờ rủi ro được cập nhật trong một chu kỳ dữ liệu sau khi xuất hiện tín hiệu mới; với rủi ro đã quan sát trước mốc ETA−48 giờ, cảnh báo phải có trước mốc đó — mục tiêu pilot.
- Thời gian Hà xem và duyệt dự báo không quá 10 phút/container — mục tiêu pilot.
- Trong backtest, chi phí quyết định giả lập từ pallet lỗi bị bỏ sót và pallet tốt bị xử lý quá mức thấp hơn ít nhất 10% so với baseline — mục tiêu pilot.

**Non-AI alternative:**  
Dùng bảng tuổi quả, số giờ vượt ngưỡng nhiệt độ và hệ số an toàn cố định theo tuyến vận chuyển.

**Hạn chế:**

- Không phản ánh khác biệt chất lượng đầu vào giữa các lô.
- Không mô hình hóa tác động phi tuyến và tích lũy của nhiệt độ.
- Không cập nhật tốt khi ETA, vị trí pallet hoặc điều kiện bảo quản thay đổi.

**AI hypothesis:**  
Một mô hình đa phương thức dạng time-to-event có thể:

- sử dụng ngày thu hoạch, vùng trồng, giống, ảnh kiểm phẩm và kết quả QC đầu vào;
- kết hợp thời gian chờ tiền lạnh, sơ đồ xếp và vị trí pallet;
- cập nhật bằng chuỗi nhiệt độ, độ ẩm, trạng thái mất điện/mở cửa và ETA;
- sử dụng O₂/CO₂ hoặc dữ liệu khí chỉ khi chuyến hàng thực sự có cảm biến;
- trả về RSL kèm khoảng bất định, yếu tố ảnh hưởng chính và nguồn dữ liệu.

**Quick gut:**  
Workflow có mô hình dự báo và human-in-the-loop. Card #1 chỉ tạo RSL, khoảng bất định và cờ cần xem xét; Card #2 mới biến kết quả đó thành thứ tự kiểm tra hoặc phân phối.

### Draft current workflow — Card #1

**CURRENT STATE — 35–60 phút/container, chưa có RSL định lượng**

```text
[1 Lấy mẫu + ghi QC đầu vào]
        ↓
[2 Tiền lạnh + đóng pallet]
        ↓
[3 Theo dõi vài logger và cảnh báo ngưỡng]
        ↓
[4 Hà tự ghép tuổi quả + cảnh báo + ETA: 15–30'] ← bottleneck
        ↓
[5 Chờ QC tại điểm đến mới biết chất lượng]
```

### Draft future workflow — Card #1

**FUTURE STATE — tối đa 10 phút duyệt/container**

```text
[1 Đồng bộ ID pallet + QC đầu vào]
        ↓
[2 AI cập nhật RSL từ hành trình: 1–3']
        ↓
[3 AI hiển thị khoảng dự báo + nguyên nhân]
        ↓
[4 Hà kiểm tra dữ liệu và phê duyệt]
        ↓
[5 Chuyển snapshot RSL sang Card #2]
```

**Fallback:**  
Nếu thiếu dữ liệu, cảm biến lỗi hoặc lô nằm ngoài phạm vi đã kiểm định, AI không đưa ra một con số chắc chắn. Hệ thống trả về khoảng rủi ro bảo thủ hoặc trạng thái “không đủ dữ liệu”, rồi yêu cầu lấy mẫu và áp dụng SOP hiện hành. AI không kết luận an toàn thực phẩm và không tự thay đổi cài đặt container.

## Problem Card #2 — Xếp hạng pallet rủi ro để ưu tiên kiểm tra và phân phối

**Problem 1 câu:**  
Khi container sắp đến, QC và điều phối kho chưa biết pallet nào cần mở kiểm tra, giao trước hoặc chuyển sang kênh bán nhanh nên vẫn kiểm ngẫu nhiên hoặc xử lý cả container như nhau.

**Actor:**  
Hà, trưởng QC tại điểm nhận và điều phối viên kho nhập khẩu.

**Thời điểm / bối cảnh:**  
Từ 12–48 giờ trước ETA đến khi dỡ hàng. Card này dùng snapshot RSL từ Card #1, vị trí pallet, mức bất định và yêu cầu của đơn hàng để tạo thứ tự hành động. Mốc đánh giá chính là ETA−12 giờ và chỉ dùng dữ liệu có trước mốc đó. Nếu Card #1 chỉ đủ dữ liệu ở cấp lô/vùng, Card #2 cũng phải xếp hạng ở cùng cấp. Nó không tự quyết định loại bỏ hoặc chuyển khách hàng.

**Current workflow:**

1. Kho nhận ETA và hồ sơ chuyến hàng.
2. Nhân viên xem biểu đồ cảm biến ở cấp container.
3. QC chọn ngẫu nhiên hoặc chọn pallet dễ tiếp cận để kiểm tra.
4. Nếu phát hiện pallet lỗi, đội nhận hàng mới mở rộng phạm vi kiểm.
5. Bán hàng điều chỉnh đơn và thứ tự phân phối sau khi hàng đã ở kho.

**Bottleneck:**  
Bước 3–4 — vài điểm đo và một đánh giá chung không chỉ ra được vùng rủi ro trong container; kiểm ngẫu nhiên có thể bỏ sót pallet yếu hoặc mở quá nhiều pallet tốt.

**Impact:**

- Pallet rủi ro có thể không được kiểm tra trước.
- Thời gian dừng container và khối lượng kiểm hàng tăng.
- Pallet có vòng đời ngắn không được ưu tiên bán sớm.
- Điều chỉnh đơn hàng xảy ra muộn và gây xáo trộn vận hành.

**Success metric:**

- Các KPI theo pallet chỉ tính trên chuyến có ánh xạ pallet–vị trí và ground truth ở cùng cấp.
- Baseline là lấy mẫu phân tầng theo lô, vị trí và ngày thu hoạch với cùng ngân sách kiểm tra.
- Ở ngân sách kiểm tra cố định 20% số pallet, recall@budget và precision phải được báo cáo. Mức cải thiện recall chuẩn hóa `(Recall_AI − Recall_baseline) / (1 − Recall_baseline)` đạt ít nhất 30% — mục tiêu pilot.
- Tại cùng tỷ lệ bỏ sót với baseline, giảm ít nhất 30% số pallet phải mở kiểm tra — mục tiêu pilot.
- Danh sách ưu tiên sẵn sàng ít nhất 12 giờ trước ETA — mục tiêu pilot.
- 100% pallet rủi ro cao có lý do, nguồn dữ liệu và mức tin cậy.
- Giảm ít nhất 20% thời gian từ lúc mở container đến khi có kế hoạch phân bổ — mục tiêu pilot.
- Chi phí quyết định giả lập ở cùng ngân sách kiểm tra thấp hơn ít nhất 10% so với baseline — mục tiêu pilot.

**Non-AI alternative:**  
Dùng ma trận điểm cố định theo ngày thu hoạch, số giờ vượt ngưỡng, vị trí pallet và mức độ chậm hành trình; đồng thời lấy mẫu phân tầng thay vì chọn ngẫu nhiên.

**Hạn chế:**

- Quy tắc cố định khó kết hợp nhiều tín hiệu yếu và tương tác với nhau.
- Thêm logger cho mọi pallet làm tăng chi phí nhưng vẫn chỉ đo tại một điểm.
- Không tận dụng được ảnh QC, kết quả chuyến trước và mức bất định của RSL.

**AI hypothesis:**  
Một mô hình learning-to-rank có thể kết hợp:

- RSL và mức bất định từ Card #1;
- lịch sử nhiệt, sự kiện bất thường và vị trí pallet;
- vùng luồng khí ước tính trong container;
- lô nguyên liệu, ảnh QC và kết quả kiểm của các chuyến tương tự;
- ETA, thời gian giao tiếp theo và yêu cầu chất lượng của đơn hàng.

AI tạo danh sách kiểm trước / xuất trước / giữ để kiểm sâu, kèm tối đa ba yếu tố ảnh hưởng chính. Quy tắc hợp đồng, FEFO và người có thẩm quyền được áp dụng sau điểm số AI.

**Quick gut:**  
Workflow xếp hạng có ràng buộc. AI sắp ưu tiên; QC và bộ phận thương mại phê duyệt hành động.

### Draft current workflow — Card #2

**CURRENT STATE — 30–90 phút để có kế hoạch sau khi mở hàng**

```text
[1 Nhận ETA + hồ sơ container]
        ↓
[2 Xem biểu đồ cảm biến chung]
        ↓
[3 Chọn pallet ngẫu nhiên / dễ lấy] ← bottleneck
        ↓
[4 Phát hiện lỗi rồi mở rộng kiểm]
        ↓
[5 Điều chỉnh phân bổ khi hàng đã ở kho]
```

### Draft future workflow — Card #2

**FUTURE STATE — có shortlist trước ETA, 10–20 phút duyệt**

```text
[1 Nhận snapshot RSL + sơ đồ pallet]
        ↓
[2 AI xếp hạng pallet rủi ro: 1–3']
        ↓
[3 AI nêu lý do + mức bất định]
        ↓
[4 QC duyệt kế hoạch kiểm phân tầng]
        ↓
[5 Kho chuẩn bị dỡ hàng và phân bổ]
```

**Fallback:**  
Nếu thiếu ánh xạ pallet–vị trí hoặc độ phủ cảm biến không đủ, hệ thống chỉ xếp hạng ở cấp lô hoặc vùng container. Khi các điểm số gần nhau, QC quay về lấy mẫu phân tầng và giữ một nhóm kiểm tra ngẫu nhiên. AI không tự loại hàng, đổi khách hoặc xác nhận pallet đạt chuẩn.

## Problem Card #3 — Truy nguyên công đoạn gây suy giảm và tạo hồ sơ bằng chứng

**Problem 1 câu:**  
Khi có khiếu nại, Hà và nhóm QA phải mất nhiều thời gian ghép dữ liệu để xác định chất lượng đã suy giảm ở khâu đầu vào, tiền lạnh, xếp container, vận chuyển hay bàn giao.

**Actor:**  
Hà, quản lý QA và chuyên viên xử lý khiếu nại của doanh nghiệp xuất khẩu.

**Thời điểm / bối cảnh:**  
Khi nhà nhập khẩu báo thanh long mềm, mất nước, thối hỏng hoặc không đạt ngoại quan; đồng thời dùng cho post-mortem sau mỗi chuyến có sự cố. Đầu vào phải gồm snapshot RSL của Card #1, thứ hạng/khuyến nghị và quyết định đã duyệt của Card #2, cùng kết quả QC thực tế tại điểm đến. AI chỉ lập timeline và xếp hạng giả thuyết, không tự quy trách nhiệm pháp lý.

**Current workflow:**

1. Nhóm QA yêu cầu dữ liệu từ nhà đóng gói, hãng tàu và nhà nhập khẩu.
2. Nhân viên đổi định dạng, đối chiếu ID và chuẩn hóa múi giờ.
3. Hà lập timeline từ thu hoạch đến giao nhận.
4. Chuyên gia so sự kiện với SOP, hợp đồng và biểu hiện hư hỏng.
5. Nhóm khiếu nại viết báo cáo, chèn ảnh và dẫn tài liệu.
6. Khi thiếu bằng chứng, nhóm tiếp tục liên hệ các bên và sửa hồ sơ.

**Bottleneck:**  
Bước 1–4 — dữ liệu phân tán, ID và đồng hồ không thống nhất; một cảnh báo nhiệt chỉ cho thấy tương quan chứ chưa chứng minh nguyên nhân. Một ca có thể mất 1–3 ngày theo giả định ban đầu.

**Impact:**

- Hồ sơ khiếu nại chậm hoặc yếu về khả năng truy vết.
- Doanh nghiệp khó phân biệt lỗi nội bộ với lỗi vận tải.
- Thời hạn thông báo có thể trôi qua khi đang thu thập tài liệu.
- Nguyên nhân cũ có thể lặp lại vì bài học không được chuẩn hóa.

**Success metric:**

- Sau lần nạp đầu tiên, phát hiện và yêu cầu tài liệu còn thiếu trong không quá 30 phút — mục tiêu pilot.
- Thời gian xử lý để tạo timeline không quá 2 giờ sau khi các nguồn hợp lệ được nạp — mục tiêu pilot.
- Thời gian end-to-end từ lúc mở case đến bản nháp đầu tiên không quá 1 ngày làm việc với nguồn dữ liệu thuộc quyền kiểm soát nội bộ — mục tiêu pilot.
- Giảm ít nhất 50% thời gian con người chuẩn bị hồ sơ — mục tiêu pilot.
- Với thông tin thực sự tồn tại trong nguồn, precision và recall trích xuất các trường bắt buộc đạt ít nhất 95% — mục tiêu pilot.
- Recall phát hiện đúng tài liệu còn thiếu theo checklist đạt ít nhất 90% — mục tiêu pilot.
- 100% nhận định quan trọng trong bản nháp liên kết tới nguồn và vượt kiểm tra claim–evidence.
- Trong các ca được hội đồng độc lập xác nhận là đủ bằng chứng để xếp hạng, Top 2 giả thuyết trùng với ưu tiên của hội đồng trong ít nhất 75% ca — mục tiêu pilot. Đây là agreement về ưu tiên điều tra, không phải bằng chứng nhân quả.
- Trong các ca hội đồng xác nhận chưa đủ bằng chứng, hệ thống chọn “chưa đủ căn cứ” ở ít nhất 90% ca — mục tiêu pilot.
- 0 nhận định nhân quả quan trọng thiếu bằng chứng hỗ trợ.
- 0 hồ sơ tự động gửi và 0 kết luận trách nhiệm khi chưa được phê duyệt.

**Non-AI alternative:**  
Chuẩn hóa tên file, ID, múi giờ và thư mục; dùng checklist claim pack cùng bảng tính timeline; chạy script để tìm sự kiện vượt ngưỡng.

**Hạn chế:**

- Dữ liệu phi cấu trúc như email, ảnh và biên bản vẫn phải đọc thủ công.
- Khó nối cùng một pallet qua các hệ thống dùng ID khác nhau.
- Script ngưỡng không phân biệt tương quan với nguyên nhân và không chỉ ra bằng chứng phản bác.

**AI hypothesis:**  
Một trợ lý điều tra đa phương thức có thể:

- trích xuất dữ liệu từ phiếu QC, email, biên bản, ảnh và file logger;
- chuẩn hóa ID, múi giờ và tạo event graph cho hành trình;
- phát hiện bất thường về tiền lạnh, chờ đóng hàng, luồng khí, mất điện và bàn giao;
- đối chiếu RSL đã dự báo, thứ hạng pallet, quyết định đã duyệt và kết quả QC thực tế để xác định dự báo hoặc hành động nào cần xem lại;
- xếp hạng các giả thuyết nguyên nhân đã được định nghĩa trước;
- hiển thị bằng chứng ủng hộ, bằng chứng phản bác, dữ liệu còn thiếu và liên kết nguồn;
- giữ file gốc ở chế độ chỉ đọc và đúng phân quyền, lưu checksum, phiên bản bản nháp và audit log cho mọi lần đổi ID hoặc múi giờ;
- tạo bản nháp hồ sơ để QA và người phụ trách khiếu nại duyệt.

**Quick gut:**  
Agent hỗ trợ điều tra có phạm vi giới hạn. Agent thu thập và nối bằng chứng; con người đánh giá nhân quả, trách nhiệm và nội dung gửi ra ngoài.

### Draft current workflow — Card #3

**CURRENT STATE — 1–3 ngày/ca (giả định ban đầu)**

```text
[1 Yêu cầu dữ liệu từ nhiều bên]
        ↓
[2 Đổi file + đối chiếu ID + múi giờ]
        ↓
[3 Ghép timeline thủ công] ← bottleneck
        ↓
[4 Chuyên gia đánh giá nguyên nhân]
        ↓
[5 Viết và rà hồ sơ khiếu nại]
```

### Draft future workflow — Card #3

**FUTURE STATE — xử lý timeline ≤2 giờ; bản nháp nội bộ ≤1 ngày**

```text
[1 Nạp nguồn dữ liệu + đầu ra Card #1/#2 + QC thực tế]
        ↓
[2 Agent chuẩn hóa ID + thời gian]
        ↓
[3 Agent tạo timeline + xếp giả thuyết]
        ↓
[4 Hiển thị bằng chứng thuận / nghịch / còn thiếu]
        ↓
[5 QA đánh giá và phê duyệt bản nháp]
```

**Fallback:**  
Nếu nguồn dữ liệu thiếu, mâu thuẫn hoặc không đủ để đánh giá nhân quả, hệ thống chỉ tạo timeline và checklist phần còn thiếu, đồng thời ghi “chưa đủ căn cứ kết luận”. Mọi đoạn không có dẫn chiếu nguồn bị loại khỏi bản nháp. Không hồ sơ nào được gửi ra ngoài trước khi QA và người phụ trách khiếu nại phê duyệt.
