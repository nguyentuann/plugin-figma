# Figma MCP Setup & Repository

Chào mừng bạn đến với repository cấu hình và cài đặt Figma Model Context Protocol (MCP) server. Repository này cung cấp mọi thứ bạn cần để thiết lập một MCP server local, kết nối trực tiếp với Figma thông qua một plugin bridge, cho phép các AI agents (như Claude Code) tương tác (đọc/ghi) với thiết kế Figma của bạn mà không gặp phải giới hạn về API rate limit.

Do repository `figma-mcp-go` bản gốc đã bị gỡ xuống, dự án này sử dụng bản thay thế tương đương bằng Rust là `figma-mcp-rust`.

## Cấu trúc Repository

- `docs/`: Chứa các tài liệu hướng dẫn chi tiết. Hãy xem [`docs/figma-mcp.md`](./docs/figma-mcp.md) để biết hướng dẫn cài đặt và cấu hình đầy đủ.
- `figma-mcp-rust/`: Mã nguồn của MCP server được viết bằng Rust (một port từ bản gốc bằng Go).
- `figma-plugin/`: Chứa mã nguồn của plugin bridge dùng để cài đặt vào Figma Desktop.

## Hướng dẫn nhanh (Quick Start)

### 1. Cài đặt Figma Plugin Bridge

Để MCP server có thể giao tiếp với thiết kế của bạn, bạn cần cài đặt plugin cục bộ vào Figma Desktop:

1. Mở ứng dụng **Figma Desktop**.
2. Trên thanh menu, chọn **Plugins** → **Development** → **Import plugin from manifest**.
3. Duyệt đến thư mục của repository này và chọn file `manifest.json` tại đường dẫn: `figma-plugin/plugin/manifest.json`.

### 2. Cấu hình AI Agent (ví dụ: Claude Code)

Không cần cài đặt ứng dụng server global, bạn có thể chạy trực tiếp qua `npx`. Mở terminal và chạy lệnh sau để thêm MCP server vào Claude Code:

```bash
claude mcp add figma-mcp-go -- npx @alvinindra/figma-mcp-rust@latest
```
*(Tên server được đặt là `figma-mcp-go` để giữ tính tương thích với các script hoặc cấu hình cũ).*

### 3. Khởi chạy hệ thống

1. Mở một file thiết kế bất kỳ trên Figma Desktop.
2. Chạy plugin vừa cài đặt (Từ menu: **Plugins** → **Development** → **Figma MCP**).
3. Mở Claude Code bằng lệnh `claude` trong terminal. Claude sẽ tự động khởi chạy MCP server và kết nối với Figma thông qua plugin bridge.

## Hướng dẫn chi tiết & Xử lý sự cố

Để xem đầy đủ các thông tin về yêu cầu hệ thống, các lệnh kiểm thử (E2E testing), và cách khắc phục các lỗi thường gặp, vui lòng tham khảo file [Hướng dẫn chi tiết](./docs/figma-mcp.md).

## Bảo mật và Giới hạn

- **Không cần Figma API Token**: Hệ thống sử dụng một local bridge trên cổng `stdio`, vì vậy không cần truy cập internet qua HTTP/WebSocket, hạn chế rủi ro bảo mật.
- **Không giới hạn Rate Limit**: Vượt qua giới hạn gọi API của Figma, tuy nhiên vẫn cần lưu ý giới hạn request từ chính AI model bạn đang sử dụng.
