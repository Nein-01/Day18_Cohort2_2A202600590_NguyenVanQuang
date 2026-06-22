# Ngày 18 — Bài lab Lakehouse (Track 2)

Đây là bài lab cho **AICB-P2T2 · Ngày 18 · Data Lakehouse Architecture**.
Trong bài này, mình sẽ tập xây một pipeline đơn giản theo mô hình
**Bronze → Silver → Gold** bằng Delta Lake.

Nếu bạn mới học Lakehouse, hãy hiểu nhanh như sau:

- **Bronze**: nơi lưu dữ liệu thô, gần như giữ nguyên lúc nhận vào.
- **Silver**: nơi dữ liệu đã được làm sạch, ép kiểu, loại trùng.
- **Gold**: nơi dữ liệu đã được tổng hợp để làm báo cáo hoặc dashboard.

## Chọn cách chạy bài lab

Repo này có 2 cách chạy. Nếu chưa quen Spark hoặc máy không quá mạnh, nên chọn
**Lightweight**.

| Cách chạy | Công cụ dùng | Cài đặt | RAM cần | Khi nào nên dùng |
|---|---|---|---|---|
| **Lightweight (mặc định)** | `deltalake` + DuckDB + Polars | `make setup` (~10 giây) | ~500 MB | Phù hợp với hầu hết học viên, nhất là khi muốn tập trung hiểu concept |
| **Spark (Docker)** | PySpark + delta-spark + MinIO | `make spark-up` (~3 phút) | ~4 GB | Dành cho bạn muốn thử API Spark gần giống môi trường production |

> Cả hai cách đều ghi ra **cùng một định dạng Delta Lake trên ổ đĩa**.
> Vì vậy, dữ liệu tạo bằng cách này vẫn có thể đọc bằng cách kia.

---

## Bắt đầu nhanh — Lightweight (khuyến nghị)

```bash
git clone https://github.com/VinUni-AI20k/Day18-Track2-Lakehouse-Lab.git
cd Day18-Track2-Lakehouse-Lab
make setup    # cài môi trường, khoảng 10 giây với pip hoặc 2 giây với uv
make smoke    # kiểm tra nhanh xem môi trường chạy được chưa
make lab      # mở Jupyter Lab tại http://localhost:8888
```

Yêu cầu: **Python 3.10–3.13**.

Lưu ý nhỏ cho người mới:

- Python 3.14 hiện chưa phù hợp vì `pyarrow` chưa có wheel ổn định cho bản này.
- Nếu `make setup` báo lỗi Python, có thể cài `uv` để tự lấy Python 3.12.
- Cách Lightweight không cần Docker, không cần Java, không cần MinIO.

Khi `make smoke` in ra `All checks passed`, mở notebook đầu tiên tại:

**http://localhost:8888/lab/tree/01_delta_basics.ipynb**

Tạo dữ liệu mẫu cho notebook 4:

```bash
make data    # tạo 200K dòng vào _lakehouse/bronze/llm_calls_raw/
```

## Các lệnh `make` cần biết

```text
make setup     Lightweight: tạo virtual environment và cài thư viện (~80 MB)
make smoke     Lightweight: kiểm tra nhanh toàn bộ môi trường
make lab       Lightweight: mở Jupyter Lab
make data      Lightweight: tạo dữ liệu Bronze mẫu
make clean     Lightweight: xóa venv và thư mục _lakehouse/

make spark-up      Spark/Docker: khởi động toàn bộ stack
make spark-smoke   Spark/Docker: kiểm tra nhanh trong container Spark
make spark-data    Spark/Docker: tạo dữ liệu Bronze 1 triệu dòng
make spark-down    Spark/Docker: dừng stack, dữ liệu vẫn giữ lại
make spark-clean   Spark/Docker: reset sạch stack Spark
```

---

## Bắt đầu nhanh — Spark/Docker (tùy chọn)

```bash
make spark-up && make spark-smoke
```

Yêu cầu: Docker Desktop ≥ 4.x và còn trống ít nhất 8 GB RAM.

Nếu chọn cách này, xem thêm hướng dẫn và lỗi thường gặp ở
[`notebooks-spark/README.md`](notebooks-spark/). Các notebook trong thư mục đó
dùng PySpark API.

---

## Nội dung từng notebook

| Notebook | Kỹ năng luyện tập | Yêu cầu nộp tương ứng | Đạt khi nào |
|---|---|---|---|
| `01_delta_basics` | Ghi/đọc Delta, schema enforcement, transaction log | NB1 — thấy file JSON trong `_delta_log/` và dùng `schema_mode="merge"` | Ghi sai schema bị chặn, sau đó thêm được cột `tier` |
| `02_optimize_zorder` | Vấn đề nhiều file nhỏ, OPTIMIZE và Z-order | NB2 — speedup ≥ 3× **hoặc** files-pruned ≥ 10× | Notebook in cả hai chỉ số, chỉ cần một chỉ số đạt |
| `03_time_travel` | `versionAsOf`, RESTORE, MERGE, `history()` | NB3 — MERGE 100K, RESTORE, `history()` có ≥ 5 version | Lịch sử cuối cùng có v0 đến v4, gồm cả RESTORE |
| `04_medallion` | Pipeline Bronze → Silver → Gold cho LLM observability | NB4 — thấy dedup và Gold có p50/p95/cost qua ≥ 7 ngày | Silver ít dòng hơn Bronze, Gold có ≥ 7 ngày × 3 model |

### Về định dạng notebook

Notebook gốc được lưu dưới dạng Jupytext `.py`. Cách này giúp file nhỏ và dễ
review hơn. Khi chạy `make setup` hoặc `make lab`, repo sẽ tự chuyển các file
`.py` thành `.ipynb`.

Nếu bạn chỉnh `.ipynb` trong Jupyter, Jupytext sẽ giúp đồng bộ lại với file
`.py`.

### So sánh với Spark

Trong mỗi notebook Lightweight có chú thích API PySpark tương đương. Bạn có thể
đọc các chú thích đó để hiểu cùng một ý tưởng sẽ viết như thế nào trong Spark.

---

## Cần nộp gì?

Bạn cần nộp **4 notebook đã chạy xong và còn output**, kèm bằng chứng ảnh chụp.
Các yêu cầu khớp với slide deliverable:

1. **NB1** — Tạo được Delta table; thấy file
   `_delta_log/00000000000000000000.json`; ghi sai schema bị chặn;
   `schema_mode="merge"` thêm được cột `tier`.
2. **NB2** — `OPTIMIZE + Z-ORDER` tạo được **speedup ≥ 3× hoặc
   files-pruned ratio ≥ 10×**. Notebook sẽ in cả hai chỉ số, chụp chỉ số nào đạt.
3. **NB3** — `history()` có ≥ 5 version, **bao gồm dòng RESTORE**. Notebook in
   history sau khi chạy `restore()`, đây là phần nên chụp. MERGE 100K chạy
   thành công; RESTORE < 30 giây và xóa hết dòng có `score < 0`.
4. **NB4** — Bronze, Silver, Gold đều có trên ổ đĩa; **Silver < Bronze** để
   chứng minh dedup đã chạy; Gold có **≥ 7 ngày × 3 model** với các cột
   p50/p95 latency, `cost_usd`, và `error_rate`.

Xem thang điểm chi tiết trong [`rubric.md`](rubric.md). Tổng điểm là 100, thuộc
Track-2 Daily Lab (30%).

---

## Bonus Challenge — Thiết kế Lakehouse riêng (tùy chọn, không tính điểm)

Phần này là bài viết mở rộng, không bắt buộc. Bạn chọn một bài toán dữ liệu thực
tế khó hơn, ví dụ:

- quan sát hệ thống LLM ở quy mô 1 tỷ request/ngày,
- CDC pipeline tuân thủ Decree 13,
- kho dữ liệu huấn luyện nghìn tỷ token,
- multimodal RAG,
- tối ưu chi phí FinOps,
- migration catalog,
- lineage cho feature store.

Deliverable của phần bonus là **một tài liệu thiết kế**. Code là tùy chọn.
Giảng viên sẽ review phần này theo góc nhìn tư duy thiết kế: bạn có nêu rõ các
lựa chọn bị loại không, số liệu có thực tế không, và có áp dụng các ý chính của
Ngày 18 như medallion, ACID, time travel, catalog, lineage, security, FinOps
không.

Phần bonus **không ảnh hưởng điểm core**. Nó chỉ dành cho bạn nào muốn luyện
thêm và có thêm một bài portfolio.

Xem đề đầy đủ tại:

- [`BONUS-CHALLENGE.md`](BONUS-CHALLENGE.md) (tiếng Việt)
- [`BONUS-CHALLENGE-EN.md`](BONUS-CHALLENGE-EN.md) (tiếng Anh)

---

## Cấu trúc repo

```text
.
├── Makefile              # chứa lệnh cho cả Lightweight và Spark
├── README.md             # file hướng dẫn bạn đang đọc
├── BONUS-CHALLENGE.md    # đề bonus tiếng Việt, không bắt buộc
├── BONUS-CHALLENGE-EN.md # đề bonus tiếng Anh, không bắt buộc
├── requirements.txt      # thư viện cho Lightweight
├── requirements-spark.txt# thư viện cho Spark
├── rubric.md             # thang điểm
├── notebooks/            # notebook Lightweight, dùng mặc định
│   ├── 01_delta_basics.py
│   ├── 02_optimize_zorder.py
│   ├── 03_time_travel.py
│   └── 04_medallion.py
├── notebooks-spark/      # notebook Spark/Docker, cùng bài học nhưng dùng PySpark
├── scripts/
│   ├── lakehouse.py            # helper tạo đường dẫn cho Lightweight
│   ├── generate_data_lite.py   # tạo dữ liệu Bronze cho Lightweight
│   ├── verify_lite.py          # smoke test cho Lightweight
│   ├── spark_session.py        # tạo Spark session
│   ├── generate_data.py        # tạo dữ liệu Bronze cho Spark
│   └── verify.py               # smoke test cho Spark
└── docker/
    └── docker-compose.yml      # stack Spark/MinIO/Jupyter
```

---

## Lỗi thường gặp khi chạy Lightweight

| Triệu chứng | Cách xử lý |
|---|---|
| `make setup` báo `python3: command not found` | Cài Python 3.10–3.13 tại https://www.python.org/downloads/ hoặc cài `uv` |
| `make setup` lỗi build `pyarrow`/`cmake` | Có thể bạn đang dùng Python 3.14. Hãy dùng `uv` để lấy Python 3.12 hoặc chạy `python3.12 -m venv .venv` |
| `make lab` báo "port 8888 in use" | Đổi port trong Makefile, ví dụ `$(JUPYTER) lab --port 8889` |
| NB2 speedup < 3× | Có thể do RAM thấp hoặc DuckDB cache làm kết quả trước/sau gần nhau. Thử reset bằng `make clean && make setup` |
| NB4 báo "Path does not exist" | Bạn có thể đã quên chạy `make data` |

---

## Nộp bài

Fork repo, sau đó push các phần sau:

- 4 notebook đã chạy và còn output.
- `submission/REFLECTION.md`, tối đa 200 từ. Nội dung: anti-pattern nào trong
  slide “Top 5 Lakehouse Anti-Patterns” mà team bạn dễ gặp nhất, và vì sao.
- Ảnh chụp hoặc bằng chứng trong `submission/screenshots/` cho layout
  `_lakehouse/` và file `_delta_log/*.json` nếu dùng Lightweight.

Sau đó mở PR về upstream với title:

```text
[2A202600590] Lab18 — Nguyễn Văn Quang
```

---

© VinUniversity AICB program. Nội dung dựa trên slide Track 2 Ngày 18.
