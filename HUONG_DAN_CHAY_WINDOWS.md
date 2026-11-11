# Hướng dẫn chạy trên Windows PowerShell

## 1. Mở đúng thư mục project

Giải nén project, mở PowerShell tại thư mục `K5-L3-Day4-ReactLoopAgent` (thư mục có `run.py`, `agent.py`, `providers.py`).

Kiểm tra Python:

```powershell
python --version
```

Yêu cầu Python 3.10 trở lên. Project dùng standard library, không cần `pip install`, `python-dotenv`, LangChain hoặc framework agent.

## 2. Cấu hình OpenAI API trong phiên PowerShell hiện tại

```powershell
$env:PROVIDER = "openai"
$env:MODEL = "gpt-4o-mini"
$env:OPENAI_API_KEY = Read-Host "Nhap OpenAI API Key"
```

Nhập API key của bạn tại dấu nhắc. Không gửi key trong chat, không ghi key vào `README.md`, không commit key lên GitHub. Biến môi trường chỉ áp dụng cho cửa sổ PowerShell hiện tại; mở cửa sổ mới thì cần cấu hình lại.

Nếu dùng provider khác, chỉ cấu hình MỘT provider:

- Gemini: `$env:PROVIDER = "gemini"`; `$env:MODEL` đặt model Gemini có function calling; key ở `$env:GEMINI_API_KEY`.
- Anthropic: `$env:PROVIDER = "anthropic"`; `$env:MODEL` đặt model Claude có tool use; key ở `$env:ANTHROPIC_API_KEY`.
- OpenRouter: `$env:PROVIDER = "openrouter"`; `$env:MODEL` đặt Model ID đầy đủ; key ở `$env:OPENROUTER_API_KEY`.

Tên model phải đúng model mà tài khoản/API key của bạn được phép sử dụng và có hỗ trợ tool calling. Nếu model không tồn tại hoặc tài khoản hết quota/billing, chương trình sẽ báo lỗi từ provider.

## 3. Kiểm tra code offline

```powershell
python -m unittest discover -s tests -v
```

Kết quả mong đợi hiện tại: `Ran 37 tests ... OK`.

Có thể kiểm tra snapshot không cần API key:

```powershell
python -c "import json; from status_tools import StatusTools; print(json.dumps(StatusTools().get_status('github', 'Actions'), ensure_ascii=False, indent=2))"
```

## 4. Chạy thử Agent với một câu hỏi

Sau khi cấu hình provider, model và API key:

```powershell
python run.py "GitHub Actions hiện có sự cố không? Hãy kiểm tra incident gần đây."
```

Chế độ mặc định là `snapshot`, đọc dữ liệu lưu sẵn. Muốn gọi status API trực tiếp:

```powershell
python run.py --mode live "Cloudflare có incident liên quan cache purge không?"
```

Live cần Internet; API công khai có thể timeout hoặc rate limit.

## 5. Chạy Eval thử một case trước

Eval gọi LLM hai vai trò: agent và judge, vì vậy có thể phát sinh chi phí token. Hãy chạy một case trước để xác nhận API hoạt động:

```powershell
python evals/run_evals.py --case E2 --output eval_test.json
```

Chỉ chạy toàn bộ 22 case khi lệnh trên chấm được case thành công:

```powershell
python evals/run_evals.py --output eval_before.json
```

Sau khi xem trace, chọn một lỗi thực tế, cải tiến `agent.py`, chạy lại Unit Tests rồi mới chạy:

```powershell
python evals/run_evals.py --output eval_after.json
```

Giữ nguyên provider, model, judge, max steps và dataset giữa hai lần chạy để so sánh hợp lệ. Mỗi file phải có đủ 22 case và `summary.score_0_100` khác `null`.

## 6. Lưu ý bảo mật và nộp bài

- Không tạo file `.env` trừ khi bạn tự quản lý và chắc chắn file đã bị Git ignore.
- Không commit API key hoặc trace chứa dữ liệu nhạy cảm.
- Không sửa `fixtures/`, `evals/cases.json`, `evals/scenarios.py`, `evals/grading.py` hoặc rubric để làm điểm cao hơn.
- `eval_before.json`, `eval_after.json` và `NOTES.md` phải dựa trên lần chạy thật của chính bạn.
