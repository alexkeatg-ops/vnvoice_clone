# Voice Clone trên Colab Pro

Repo này để chạy clone giọng nói (zero-shot: chỉ cần 1 đoạn giọng mẫu, không cần train lại) trên Google Colab Pro có GPU.

> ⚠️ Chỉ clone giọng của chính bạn, hoặc giọng bạn có sự đồng ý bằng văn bản của chủ giọng. Không dùng để giả mạo người khác, lừa đảo, hay làm nội dung gây nhầm lẫn là giọng thật của người khác.

## Cách chạy nhanh nhất (5 bước)

1. Tạo repo mới trên GitHub (tên gợi ý: `voice-clone-colab`), push toàn bộ thư mục này lên.
2. Mở Colab Pro, chọn Runtime → Change runtime type → GPU (ưu tiên GPU có VRAM ≥ 16 GB, ví dụ L4 / A100 nếu còn quota; T4 16 GB là mức tối thiểu, có thể chậm hoặc tràn VRAM với model F5).
3. Mở notebook trong repo bằng Colab: `File → Open notebook → GitHub → chọn repo → notebooks/voice_clone_colab.ipynb`
   Hoặc mở trực tiếp bằng link dạng:
   `https://colab.research.google.com/github/<tên-GitHub-của-bạn>/voice-clone-colab/blob/main/notebooks/voice_clone_colab.ipynb`
4. Chạy từng cell theo thứ tự. Cell đầu kiểm tra GPU, cell sau cài model.
5. Upload file giọng mẫu vào `voices/`, gõ câu cần đọc, chạy cell sinh giọng, tải file `.wav` ở `outputs/` về.

## Chọn engine nào?

Đây là điểm quan trọng nhất, vì không có 1 model nào ngon cho mọi ngôn ngữ:

| Nhu cầu | Engine mặc định trong repo | Vì sao |
|---|---|---|
| **Tiếng Việt** | `zalopay/vietnamese-tts` (nền F5-TTS) | Model ghi rõ trên Hugging Face: tiếng Việt, train 200+ giờ giọng Việt công khai, license CC-BY-4.0. Cần kèm **bản transcript chính xác** của file giọng mẫu (F5-TTS yêu cầu). |
| **Tiếng Việt, kiểu XTTS** | `capleaf/viXTTS` (XTTS v2 fine-tune cho tiếng Việt) | Lựa chọn dự phòng nếu bản F5 cho chất lượng chưa ưng. XTTS gốc không có tiếng Việt trong 17 ngôn ngữ chính thức, nên phải dùng bản fine-tune này mới nói tiếng Việt tốt. |
| **Tiếng Anh / đa ngữ (không phải tiếng Việt)** | `tts_models/multilingual/multi-dataset/xtts_v2` (Coqui XTTS v2) | Chỉ cần ~6 giây giọng mẫu, chạy nhẹ (~4 GB VRAM theo các bài test 2026), license weights là Coqui Public Model License (phi thương mại). |

Notebook mặc định chạy **tiếng Việt bằng zalopay/vietnamese-tts**. Muốn đổi engine, chỉ đổi 1 cell trong notebook, không cần tạo repo khác.

## Chuẩn bị giọng mẫu (quyết định 80% chất lượng)

- 1 người nói duy nhất, không nhạc nền, không tiếng ồn, không giọng khác chen vào.
- Dài 10–30 giây, đọc liền mạch, tốc độ tự nhiên. Đoạn quá ngắn (< 6 giây) clone sẽ kém giống; quá dài cũng không giúp thêm nhiều.
- File `.wav`, mono, sample rate ≥ 16 kHz. Đặt tên rõ ràng, ví dụ `voices/son-tinh.wav`, `voices/mi-nuong.wav`.
- Với engine F5 (mặc định): ghi thêm transcript chính xác từng chữ của đoạn mẫu vào `voices/<tên>.txt` (cùng tên file, khác đuôi). Sai transcript là lỗi hay gặp nhất làm giọng ra sai.
- Không upload giọng mẫu thật lên GitHub public. Thư mục `voices/` đã bị `.gitignore` chặn. Chỉ giữ file mẫu ở Colab / Drive riêng.

## Cấu trúc repo

```
voice-clone-colab/
├── notebooks/
│   └── voice_clone_colab.ipynb   # Notebook chạy trên Colab (bản chính)
├── src/
│   └── clone_vi.py               # Script chạy hàng loạt từ file CSV (lồng tiếng nhiều câu)
├── voices/
│   └── README.md                 # Hướng dẫn chuẩn bị giọng mẫu, KHÔNG commit file wav thật
├── examples/
│   └── dialogue_example.csv      # Mẫu: id,voice,text — 1 dòng = 1 câu thoại
├── outputs/
│   └── .gitkeep                  # File sinh ra sẽ nằm đây, KHÔNG commit lên GitHub
├── requirements.txt
└── .gitignore
```

## Chạy hàng loạt (ví dụ lồng tiếng nhiều câu)

Sau khi test 1 câu thành công trong notebook, dùng script:

```bash
python src/clone_vi.py \
  --ref-audio voices/son-tinh.wav \
  --ref-text "Bản transcript chính xác của file son-tinh.wav" \
  --csv examples/dialogue_example.csv \
  --out-dir outputs
```

File CSV có 3 cột: `id,voice,text`. Script sẽ tách câu dài thành từng câu ngắn rồi nối lại, vì model có giới hạn độ dài mỗi lần sinh.

## Dùng cho phim "Món nợ của nước" — nhiều nhân vật

Quy trình này khác test 1 giọng ở chỗ: mỗi nhân vật phải có giọng mẫu riêng.

1. **Thu giọng mẫu theo nhân vật, không dùng chung.** Sơn Tinh, Thủy Tinh, Mị Nương, Vua Hùng, Lạch, người kể chuyện — mỗi vai 1 file `voices/<tên-vai>.wav` + transcript chính xác. Dùng chung 1 file mẫu cho 2 vai thì 2 vai sẽ nghe giống nhau.
2. **Khai báo trong `voices.json`.** Copy `voices.json.example` thành `voices.json` (file này không commit giọng, chỉ commit đường dẫn + transcript), điền đúng audio + transcript cho từng vai.
3. **Tách thoại từ storyboard thành CSV.** File mẫu là `examples/dialogue_film_example.csv`, cột `shot_id,character,text,speed`. Lấy đúng thoại trong storyboard, ví dụ Thủy Tinh: "Nếu cống tiếp tục đóng, dân ta chắc chắn chết." / Sơn Tinh: "Ta đang hỏi anh có biết mình sẽ thả thứ gì xuống đồng bằng không."
4. **Chạy batch theo nhân vật:**
```bash
cp voices.json.example voices.json
# điền voices.json xong rồi chạy:
python src/clone_vi.py --voices-map voices.json \
  --csv examples/dialogue_film_example.csv --out-dir outputs
```
5. **Ghép theo mã shot.** Output sẽ là `outputs/C2-S01.wav`, `outputs/C2-S02.wav`… giữ nguyên mã shot của storyboard để ghép đúng timeline, đừng đổi tên.
6. **Kiểm tra độ dài thoại trước khi ghép.** Đo thời lượng file wav sinh ra, so với thời lượng shot trong storyboard (Cảnh 1 các shot khoảng 12–18 giây). Nếu thoại dài hơn shot thì rút thoại hoặc chia làm 2 nhịp thoại, đừng cố kéo dài video để khớp giọng — video previz hiện tại đã chốt nhịp tách đôi riêng.
7. **Vai trò của giọng clone trong phim này:** đây là bản lồng tiếng nháp / temp dub để kiểm tra nhịp thoại, cảm xúc và độ dài trước khi chốt bản thu thật. Nếu sau này dùng giọng diễn viên thật cho bản chính thức, bản clone chỉ nên giữ ở giai đoạn previz, trừ khi diễn viên đó đã đồng ý dùng giọng clone cho bản phát hành.

Trong notebook, cell 5 đã đổi thành lệnh chạy theo `voices-map` cho phim, vẫn giữ được cách chạy 1 giọng cũ nếu bạn bỏ `--voices-map` và dùng `--ref-audio`.

## Lưu ý Colab Pro

- Colab sẽ xóa máy sau khi hết phiên. **File giọng mẫu và file output phải tải về hoặc lưu lên Google Drive ngay**, đừng để mỗi file ở `/content`.
- Model tải lần đầu nặng (~1–2 GB), lần sau Colab cache lại nếu cùng phiên, nhưng qua phiên mới có thể phải tải lại.
- Nếu gặp lỗi tràn VRAM (CUDA out of memory): đổi sang GPU lớn hơn, hoặc rút ngắn câu, hoặc giảm `nfe_step` trong script.
- Nếu gặp lỗi import do phiên bản thư viện F5-TTS thay đổi: đối chiếu lại đúng trang Hugging Face của model đang dùng, vì code load model trong repo này bám theo trang đó tại thời điểm viết.

## Trạng thái của repo này

Đây là **skeleton đã kiểm tra theo tài liệu công khai, chưa chạy thử trên Colab Pro của bạn**. Bước kiểm chứng bắt buộc là: chạy notebook với 1 câu ngắn, 1 giọng mẫu của chính bạn, nghe thử rồi mới chạy hàng loạt. Nếu cell nào lỗi, copy nguyên thông báo lỗi gửi lại, đừng tự sửa lung tung nhiều chỗ cùng lúc.
