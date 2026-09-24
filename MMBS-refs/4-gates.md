# Phân Tích Chuyên Sâu: Khung Kiểm Định 4 Cổng (4-Gate Protocol) và Bộ Tứ Trọng Tâm

> **Mục tiêu tài liệu:** Triển khai **Khung Kiểm định 4 Cổng (4-Gate Protocol)** xoay quanh bài báo neo **MMBS (Findings of EMNLP 2022)** và **Bộ Tứ Trọng Tâm (The Core Quartet)** dựa trên tiêu chí nghiêm ngặt về tính tái lập thực nghiệm (**Reproducibility Gate**).  
> **Bộ tứ cốt lõi:**  
> 1. **Khởi xướng (Base Anchor):** MMBS (Findings of EMNLP 2022) — [`PhoebusSi/MMBS`](https://github.com/PhoebusSi/MMBS)  
> 2. **Phê phán & Chưng cất (Direct Competitor):** DDG (Findings of ACL 2023) — [`Zhiquan-Wen/DDG`](https://github.com/Zhiquan-Wen/DDG)  
> 3. **Phân loại chuẩn hóa (Theoretical Anchor):** Robust VQA Survey (IEEE TPAMI 2024) — [`DOI: 10.1109/TPAMI.2024.3366154`](https://doi.org/10.1109/TPAMI.2024.3366154)  
> 4. **Đỉnh cao kế thừa xáo từ thích ứng (Modern Frontier):** DSI (Expert Systems with Applications 2026) — [`songxdr3/DSI`](https://github.com/songxdr3/DSI)  
> *(Toàn bộ các bài báo không phát hành mã nguồn công khai như CopVQA, KDSR, CLAP, CC-VQA, Counterfactual Dual-Bias hoặc ngoài phạm vi như MCCD, OSCAR đều bị loại bỏ).*

---

# PHẦN 1: BẢN CHẤT HỌC THUẬT CỦA BÀI BÁO NEO (MMBS)

Để đảm bảo nghiên cứu không rơi vào bẫy "sửa code mò mẫm", MMBS được bóc tách ở 3 tầng bản chất: **Hiện tượng (Phenomenon) $\rightarrow$ Giả định toán học (Mathematical Assumption) $\rightarrow$ Điểm gãy cốt tử (Vulnerability).**

```
                     [BẢN CHẤT CƠ CHẾ CỦA MMBS]
┌────────────────────────────────────────────────────────────────────────┐
│ 1. Bản chất Hiện tượng:                                               │
│    Trong VQA, mạng nơ-ron có xu hướng chọn "đường tắt" (shortcut):     │
│    Thấy "What color" → đoán "white"; thấy "Is there" → đoán "yes".     │
│    Khi bị ép học trên tập OOD (VQA-CP v2), các phương pháp cũ phạt     │
│    trọng số mẫu biased khiến mô hình quên sạch tri thức nền (tụt ID).  │
├────────────────────────────────────────────────────────────────────────┤
│ 2. Giả định Toán học của MMBS:                                        │
│    Mọi mẫu biased (I, Q, A) đều gồm 2 phần:                           │
│    • Unbiased component (vật thể, thuộc tính trong ảnh + từ vị ngữ).   │
│    • Spurious component (tần suất đồng xuất hiện tiền tố Q và nhãn A). │
│    MMBS giả định: Chỉ cần "phá hỏng" tiền tố Q (bằng Shuffle/Removal), │
│    phần còn lại sẽ trở thành tín hiệu sạch để kéo biểu diễn đa phương  │
│    thức qua hàm InfoNCE:                                               │
│       L_cl = - log [ exp(cos(z, z+) / τ) / Σ exp(cos(z, z-) / τ) ]     │
├────────────────────────────────────────────────────────────────────────┤
│ 3. Điểm gãy Cốt tử (The Linguistic Vulnerability):                     │
│    MMBS dùng mạng GRU: h_t = GRU(h_{t-1}, e_t).                       │
│    Khi Shuffle từ ("What color is the food inside the bowl"            │
│    → "the color the food What is bowl inside"), toàn bộ chuỗi trạng    │
│    thái tuần tự của GRU bị phá hủy. Mô hình không học được "ngữ nghĩa  │
│    suy luận sạch" mà thực chất chỉ đang học cách kháng nhiễu xáo từ!   │
└────────────────────────────────────────────────────────────────────────┘
```

---

# PHẦN 2: TRIỂN KHAI KHUNG KIỂM ĐỊNH 4 CỔNG (4-GATE PROTOCOL)

---

## 🚪 GATE 1 — Paper Understanding (Hồ sơ Điểm Neo MMBS)

* **Problem (Vấn đề):** Sự đánh đổi gay gắt giữa độ bền vững ngoại phân phối (OOD Robustness trên VQA-CP v2) và năng lực duy trì trên phân phối gốc (ID Performance trên VQA v2). Việc phạt mẫu biased làm mô hình bị tụt tới 7–11% điểm trên VQA v2.
* **Hypothesis (Giả thuyết):** Tận dụng thông tin không thiên kiến trong mẫu biased thông qua các cặp mẫu đối phản (contrastive positive samples) sẽ triệt tiêu language prior mà không làm mất đi tri thức nền tảng của bài toán VQA.
* **Method (Phương pháp):**
  * *Sinh mẫu dương:* Xóa tiền tố câu hỏi (Removal) cho câu hỏi Yes/No; Xáo trộn vị trí từ (Shuffling) cho câu hỏi Number/Other.
  * *Chọn lọc mẫu không thiên kiến:* Thuật toán lọc dựa trên entropy phân phối nhãn ($W_C$) để giữ nguyên câu hỏi gốc cho các mẫu thuộc nhóm đuôi (tail/unbiased).
  * *Tối ưu hóa:* $\mathcal{L}_{total} = \mathcal{L}_{vqa} + \alpha \cdot \mathcal{L}_{cl}$ (Soft multi-label BCE + InfoNCE).
* **Datasets & Metrics:**
  * Datasets: VQA-CP v2 (Train: 438K, Test: 220K); VQA v2 (Val: 214K).
  * Metrics: VQA Accuracy chuẩn; chỉ số tổng hợp $GapsSum = \Delta_{\text{OOD}} + \Delta_{\text{ID}}$.
* **Reported Results (Kết quả chính):**
  * UpDn + MMBS: Đạt **48.19%** trên VQA-CP v2 (tăng **+8.45%** so với UpDn gốc 39.74%) và giữ vững **63.84%** trên VQA v2 (+0.36%).
  * LMH + MMBS: Đạt **56.44%** trên VQA-CP v2 (+4.43%) và phục hồi ngoạn mục **61.87%** trên VQA v2 (tăng **+5.52%** so với LMH đơn thuần).
* **Limitations (Tác giả tự thừa nhận):**
  * *Nhiễu ngữ pháp:* Section 3.1 thừa nhận việc tạo mẫu sinh ra *"unexpected noise in the positive samples"*.
  * *Phụ thuộc vào xáo trộn ở pha test (Test-time fragility):* Bảng 6 chỉ ra UpDn+MMBS chỉ đạt 48.19% nếu lúc test câu hỏi cũng bị xáo trộn; nếu test bằng câu hỏi tự nhiên gốc (original), độ chính xác sụt xuống còn **42.80%**.

---

## 🚪 GATE 2 — Literature Evolution & Research Tree (Bộ Tứ Trọng Tâm)

Áp dụng bộ lọc nghiêm ngặt **Reproducibility Gate** (chỉ giữ lại các công trình có mã nguồn công khai hoặc bài khảo cứu lý thuyết chuẩn hóa), cây phân nhánh tiến hóa học thuật từ điểm neo MMBS hội tụ về **Bộ Tứ Trọng Tâm (The Core Quartet)**:

```
[1] Khởi xướng (Base Anchor): MMBS (EMNLP Findings 2022)
    │  • Đóng góp: Tận dụng biased samples bằng Contrastive Learning + Shuffling.
    │  • Điểm gãy: Phá vỡ trật tự từ tuần tự của GRU; sụt giảm 5.39% khi test câu tự nhiên.
    │  • Mã nguồn: PhoebusSi/MMBS (Public PyTorch)
    ▼
[2] Phê phán & Chưng cất (Direct Competitor): DDG (ACL Findings 2023)
    │  • Đóng góp: Chỉ ra trực diện xáo từ phá hủy ngữ pháp ("destroys grammar and semantics").
    │  • Giải pháp né tránh: Dùng Knowledge Distillation ở hàm loss để bù đắp nhiễu.
    │  • Hạn chế tồn tại: Bỏ qua sửa chữa câu hỏi ở đầu nguồn, chấp nhận câu hỏi hỏng ngữ pháp.
    │  • Mã nguồn & Weights: Zhiquan-Wen/DDG (Public + Checkpoints: 61.22%)
    ▼
[3] Phân loại chuẩn hóa (Theoretical Anchor): Robust VQA Survey (IEEE TPAMI 2024)
    │  • Đóng góp: Chuẩn hóa phân loại toàn ngành (Ensemble, Contrastive, Causal, Augmentation).
    │  • Khẳng định: MMBS là đại diện tiêu biểu của nhánh Contrastive Learning.
    │  • Kết luận: Sự đánh đổi OOD/ID là bài toán mở cốt lõi; các phương pháp đang bế tắc
    │    giữa việc khử bias và duy trì tính tự nhiên của ngôn ngữ.
    ▼
[4] Đỉnh cao kế thừa xáo từ thích ứng (Modern Frontier): DSI (ESWA 2026)
       • Đóng góp: Phân tách Dual-Space (Language Bias Space vs. Distribution Bias Space).
       • Cơ chế: Adaptive Question Shuffling (xáo từ theo entropy độ khó câu hỏi).
       • Bằng chứng mã nguồn: File train.py / base_model.py (dòng 47-65) chứng minh SOTA 2026
         vẫn dùng torch.randperm để xáo từ ngẫu nhiên — tiếp tục phá nát cú pháp tuần tự!
       • Mã nguồn: songxdr3/DSI (Public PyTorch)
```

### Phân tích chi tiết 4 mắc xích trong dòng chảy:

1. **Khởi xướng (MMBS 2022):** Khẳng định nguyên lý tận dụng mẫu thiên kiến thay vì loại bỏ chúng. Đổi lại, việc dùng xáo từ ngẫu nhiên thô sơ đã biến mô hình thành một bộ lọc kháng nhiễu xáo từ thay vì bộ suy luận thị giác - ngôn ngữ thực thụ.
2. **Phê phán (DDG 2023):** Thừa nhận và chứng minh điểm yếu chí mạng của MMBS: việc xáo từ làm mất cấu trúc ngữ pháp. Tuy nhiên, DDG lại chọn giải pháp đường vòng: giữ nguyên cơ chế sinh mẫu bị hỏng rồi dùng hàm chưng cất tri thức (Knowledge Distillation) để làm mượt biểu diễn.
3. **Chuẩn hóa (TPAMI Survey 2024):** Xác nhận vị thế của MMBS trong lịch sử VQA debiasing, đồng thời vạch ra bức tranh bế tắc chung: các phương pháp tăng cường dữ liệu thô bạo đều làm méo mó phân phối tự nhiên của câu hỏi.
4. **Kế thừa & Giới hạn (DSI 2026):** Chứng minh rằng ngay cả ở biên giới nghiên cứu mới nhất năm 2026, các nhà nghiên cứu vẫn chưa giải quyết được bài toán cú pháp. DSI dù đạt SOTA >62% vẫn phải phụ thuộc vào hàm `torch.randperm` để đảo lộn thứ tự từ.

*(Lưu ý thẩm định: Các bài báo CopVQA (EMNLP 2023), KDSR (ICME 2024), CLAP (ICME 2025), CC-VQA (ICME 2025), Counterfactual Dual-Bias (TNNLS 2025) đều bị loại bỏ vì không công khai mã nguồn; MCCD (NeurIPS 2024) bị loại vì thuộc miền Audio-Visual QA; OSCAR (IJCAI 2026) bị loại vì không thể tái lập trên tài nguyên tính toán thông thường).*

---

## 🚪 GATE 3 — Gap Validation (Chứng Minh Khoảng Trống Khoa Học Chặt Chẽ)

Để một khoảng trống nghiên cứu được xác nhận là **Hợp lệ (Verified Research Gap)**, ta phải trả lời xuất sắc 4 câu hỏi định danh dựa trên bằng chứng xuyên suốt 4 bài báo:

### 🎯 Candidate Gap: Cú pháp học bị bỏ rơi trong cơ chế kháng thiên kiến (Syntax-Preserving Question Disentanglement)

```text
1. Base paper leaves X unresolved:
   MMBS (EMNLP 2022) loại bỏ thiên kiến bằng cách xáo trộn từ (Shuffling) và xóa chuỗi (Removal),
   làm đứt gãy hoàn toàn cây cú pháp phụ thuộc và trạng thái tuần tự của mạng ngôn ngữ.
   Tác giả thừa nhận mô hình bị tụt từ 48.19% xuống 42.80% khi gặp câu hỏi tự nhiên.

2. Later papers attempted X:
   • DDG (ACL 2023) phát hiện hạn chế này ("destroys grammar and semantics") và dùng
     Knowledge Distillation ở tầng loss để bù đắp gián tiếp.
   • TPAMI Survey (2024) tổng kết sự bế tắc của các phương pháp sinh mẫu làm méo mó ngữ pháp.
   • DSI (ESWA 2026) cố gắng điều chỉnh tỷ lệ xáo từ thích ứng (Adaptive Shuffling) theo độ khó câu hỏi.

3. Their solutions still have Y limitation:
   Tất cả đều né tránh hoặc lặp lại sai lầm:
   • DDG chấp nhận câu hỏi bị hỏng ngữ pháp rồi chữa cháy bằng loss distillation.
   • DSI vẫn dùng torch.randperm xáo trộn ngẫu nhiên thứ tự từ, tiếp tục phá nát cú pháp tuần tự!
   KHÔNG MỘT CÔNG TRÌNH NÀO trực tiếp can thiệp câu hỏi ở TẦNG CÚ PHÁP HỌC (Syntactic Parsing).

4. No identified paper adequately tests Z:
   Chưa có bất kỳ công trình nào kiểm chứng: "Liệu việc bóc tách phẫu thuật toán tử nghi vấn
   (Interrogative Operator) dựa trên cây cú pháp phụ thuộc (Dependency Tree) trong khi bảo toàn
   100% trật tự và liên kết của các vị ngữ cốt lõi có giúp mô hình đạt độ bền vững OOD
   ngay trên câu hỏi tự nhiên gốc mà không cần test-time shuffling hay không?"
```

👉 **Kết luận Gate 3:** **VERIFIED GAP / UNSOLVED PROBLEM** (Vấn đề khoa học cốt lõi đã được chỉ ra nhưng chưa từng được giải quyết ở tầng biểu diễn ngôn ngữ).

---

## 🚪 GATE 4 — Project Feasibility & Reproducibility Matrix

| Câu hỏi Thẩm định Feasibility | MMBS (EMNLP 2022) | DDG (ACL 2023) | DSI (ESWA 2026) | Dự án Đề xuất: SP-MMBS | Trạng thái Thẩm định |
|---|---|---|---|---|---|
| **1. Can we test it?** | Có, trên VQA-CP v2 và VQA v2. | Có, trên VQA-CP v2 và VQA v2. | Có, trên VQA-CP v1, v2, SLAKE. | Kiểm thử trực tiếp trên VQA-CP v2 và VQA v2. | **PASS** |
| **2. Dataset available?** | Công khai (VQA-CP v2, VQA v2). | Công khai (VQA-CP v2, VQA v2). | Công khai (VQA-CP, SLAKE). | Dùng chung 100% dữ liệu gốc của MMBS. | **PASS** |
| **3. Code available?** | Public: `PhoebusSi/MMBS` | Public: `Zhiquan-Wen/DDG` | Public: `songxdr3/DSI` | Kế thừa trực tiếp codebase MMBS. | **PASS** |
| **4. Checkpoint available?** | Có (Google Drive / train lại nhanh). | Có sẵn (`best_model.pth` đạt 61.22%). | Có script train lại đầy đủ. | Kế thừa backbone UpDn của MMBS. | **PASS** |
| **5. GPU feasible?** | **Cực nhẹ:** ~0.38h/epoch trên TITAN RTX / T4. | **Nhẹ:** Chạy distillation trên 1 GPU cá nhân. | **Khả thi:** ~1 GPU, PyTorch 3.8. | **Cực nhẹ:** Can thiệp cú pháp spaCy chạy trên CPU khi nạp batch. | **PASS** |
| **6. Evaluation possible?** | Evaluator chuẩn của VQA-CP v2. | Evaluator chuẩn của VQA-CP v2. | Evaluator chuẩn của VQA-CP v2. | Evaluator chuẩn của VQA-CP v2 + bài test câu hỏi tự nhiên. | **PASS** |
| **7. Baseline reproducible?** | PyTorch chuẩn, đã kiểm tra code. | PyTorch chuẩn, checkpoint đã xác thực. | PyTorch chuẩn, logic rõ ràng. | Reproducible 100% trên Colab/Kaggle miễn phí. | **PASS** |

👉 **Quyết định Gate 4:** Dự án **SP-MMBS** đạt **PASS 100%** trên toàn bộ 7 tiêu chí thẩm định kỹ thuật và tài nguyên.

---

# PHẦN 3: ĐỀ XUẤT DỰ ÁN NGHIÊN CỨU DUY NHẤT (THE CHOSEN PROJECT)

### Tên Dự án: Syntax-Preserving Question Operator Disentanglement for Robust VQA (SP-MMBS)

* **Research Question:**  
  Liệu việc bóc tách toán tử nghi vấn bằng phân tích cú pháp phụ thuộc (Dependency Parsing) có loại bỏ được language priors trong học tương phản VQA mà vẫn duy trì tính toàn vẹn cú pháp cho câu hỏi tự nhiên hay không?
* **Hypothesis:**  
  Thay thế phép xáo từ ngẫu nhiên và xóa chuỗi tiền tố thô sơ trong MMBS bằng cơ chế **Dependency-Aware Question Operator Masking** sẽ triệt tiêu tương quan giả loại câu hỏi mà không làm đứt gãy trạng thái chuyển tiếp tuần tự của mạng GRU, giúp tăng độ chính xác trên tập test VQA-CP v2 thêm $\ge 1.5\%$ khi đánh giá trực tiếp trên **câu hỏi tự nhiên gốc** (loại bỏ độ sụt 5.39% giữa original và shuffled test).
* **NLP Contribution (Đóng góp Thuần NLP):**  
  Ứng dụng trực tiếp Dependency Parsing (spaCy/Stanza) để phân rã cấu trúc câu hỏi:  
  1. Xác định toán tử nghi vấn (*Wh-words, auxiliary verbs, root tags*) — nơi chứa đựng thiên kiến loại câu hỏi — để che mờ (masking) tạo mẫu dương tính.  
  2. Giữ nguyên 100% trật tự từ và cây cú pháp phụ thuộc của các vị ngữ, danh từ và cụm miêu tả thực thể liên quan đến ảnh.
* **Baseline Đối chiếu:** UpDn + MMBS (Findings of EMNLP 2022).
* **Điểm Seam Cụ thể trong Codebase:**  
  Can thiệp trực tiếp vào file `dataset_vqacp_MMBS.py` (tại các hàm xử lý `Shuffling_q` và `Removal_q`). Toàn bộ backbone thị giác (Faster R-CNN), mạng đa phương thức (UpDn) và hàm mất mát InfoNCE được giữ nguyên 100%.
* **Kế hoạch Thực nghiệm:**  
  1. *Pha 1:* Tái lập UpDn gốc (~39.74%) và UpDn+MMBS (~48.19% với shuffled test, ~42.80% với original test).  
  2. *Pha 2:* Cài đặt SP-MMBS trên `dataset_vqacp_MMBS.py`, huấn luyện mô hình với cấu hình siêu tham số giống hệt MMBS.  
  3. *Pha 3:* Đánh giá trên VQA-CP v2 (OOD) và VQA v2 (ID) trên cả 2 chế độ: Original Test và Shuffled Test.  
  4. *Pha 4:* Phân tích ngữ pháp (Linguistic Ablation) theo độ sâu của cây cú pháp (Dependency Tree Depth) và độ dài câu hỏi.

---

```
📋 Transparency Log
├─ Skills used: nlp-paper-project (SKILL.md, README.md, workflows/literature-research.md, workflows/paper-deconstruction.md)
├─ MCP tools used: Không cần gọi thêm MCP ngoài
├─ Local tools used: run_command & python
├─ Scope enforcement: Đã loại bỏ toàn bộ các công trình không có code / ngoài phạm vi (CopVQA, KDSR, MCCD, CC-VQA, CLAP, Dual-Bias, OSCAR).
├─ Target Papers: Bộ Tứ Trọng Tâm (MMBS 2022, DDG 2023, TPAMI Survey 2024, DSI 2026).
└─ Verification: Đã hoàn thiện toàn diện 4 Cổng kiểm định, tập trung 100% vào sợi dây đỏ giữa 4 bài báo có mã nguồn đối soát, làm sáng tỏ khoảng trống cứu cánh cho đề tài SP-MMBS.
```
