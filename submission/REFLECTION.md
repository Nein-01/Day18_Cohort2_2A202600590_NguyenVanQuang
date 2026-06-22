Anti-pattern mà nhóm chúng tôi có nguy cơ gặp nhất là xem Bronze như một nơi
đổ dữ liệu không có hợp đồng rõ ràng. Dữ liệu quan sát LLM thay đổi rất nhanh:
nhà cung cấp có thể thêm trường mới, các lần retry tạo ra request ID trùng lặp,
và JSON sai định dạng có thể xuất hiện trong lúc có sự cố. Nếu cho dashboard
phía downstream truy vấn trực tiếp dữ liệu raw ở Bronze, các chỉ số về chi phí,
độ trễ và tỉ lệ lỗi sẽ bị lệch vì mỗi consumer sẽ tự cài đặt lại logic parse và
dedup theo cách khác nhau.

Pattern an toàn hơn là giữ Bronze bất biến, sau đó áp dụng schema, validation
và dedup theo request_id ở Silver trước khi publish bất kỳ business metric nào.
Gold chỉ nên expose các chỉ số model hằng ngày đã được curate, với định nghĩa
đã được test cho p50/p95 latency, cost_usd và error_rate. Cách này vẫn giữ dữ
liệu raw để audit và time travel, đồng thời ngăn việc các phân tích không nhất
quán lan rộng.
