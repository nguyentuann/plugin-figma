# Figma MCP Setup Guide

Tài liệu này hướng dẫn cài đặt và sử dụng Figma MCP server trong môi trường local.
**Lưu ý**: Do repository `figma-mcp-go` bản gốc đã bị gỡ xuống (DMCA takedown), chúng ta sử dụng bản thay thế tương đương bằng Rust là `figma-mcp-rust`.

## 1. Requirements
- **Node.js** & **npm** (Dùng npx để chạy trực tiếp server)
- **Figma Desktop App** (Bản chạy trên trình duyệt không được hỗ trợ cho plugin local network bridge).
- **Claude Code** (hoặc AI agent hỗ trợ giao thức MCP).

## 2. Installation
Không cần cài đặt global, chúng ta sẽ sử dụng trực tiếp qua `npx`.
Package NPM: `@alvinindra/figma-mcp-rust`

## 3. Build
Vì sử dụng package qua `npx`, quy trình build source code không cần thiết. Bạn luôn nhận được phiên bản mới nhất từ NPM registry.

## 4. Figma Plugin Installation
MCP server tương tác với Figma qua một Local Bridge Plugin.

1. Tải plugin bridge cho Figma: Bạn có thể sử dụng mã nguồn plugin đã được tải sẵn ở thư mục `d:\Braly_Repo\mcp_figma\figma-plugin\plugin`.
2. Mở Figma Desktop.
3. Chọn menu: **Plugins** → **Development** → **Import plugin from manifest**.
4. Chọn file `manifest.json` trong thư mục `d:\Braly_Repo\mcp_figma\figma-plugin\plugin\manifest.json`.

## 5. Claude Code Configuration
Chạy lệnh sau trên terminal để cấu hình MCP server cho Claude Code:
```bash
claude mcp add figma-mcp-go -- npx @alvinindra/figma-mcp-rust@latest
```
(Tên server được đặt là `figma-mcp-go` để tương thích với các dự án cũ).

Bạn có thể kiểm tra trạng thái kết nối bằng lệnh:
```bash
claude mcp list
```

## 6. Startup
Để chạy hệ thống:

**Bước 1**: Mở Figma Desktop và mở một file thiết kế bất kỳ.
**Bước 2**: Chạy plugin `Figma MCP` vừa cài đặt (Plugins → Development → Figma MCP).
**Bước 3**: Mở Claude Code bằng terminal (`claude`). Khi Claude Code khởi động, nó sẽ tự động chạy MCP server thông qua cấu hình `npx` đã setup.

## 7. Testing
**E2E Read Test:**
Trong Claude Code, gõ câu lệnh:
```text
Connect to the figma-mcp-go MCP server.
Inspect the currently opened Figma document.
Return:
1. Current page name
2. Top-level nodes
3. Their dimensions
4. Text content
5. Node hierarchy

Do not modify the document.
```
Claude Code sẽ tương tác trực tiếp với Figma và in ra thông tin chi tiết.

**E2E Write Test (Cảnh báo: Non-destructive write test):**
Mở một test file, yêu cầu Claude Code tạo một frame mới:
```text
Create a temporary test frame named "MCP_TEST".
Do not modify any existing design.
```

## 8. Troubleshooting
- **MCP server không xuất hiện trong Claude**: Chạy `claude mcp list` và xem lại lệnh có lỗi không. Bạn có thể chạy tay `npx @alvinindra/figma-mcp-rust@latest` để xem có lỗi Node.js hay không.
- **Figma plugin không connect / Lỗi WebSocket connection refused**: Đảm bảo rằng Claude Code đang chạy (vì server stdio chỉ chạy khi client gọi tới). 
- **MCP tool không trả về dữ liệu**: Cần kiểm tra Figma Desktop có đang mở document nào và plugin đã chạy trong document đó chưa.

## 9. Security
- Đây là setup chạy trên `stdio` local, server không tự mở HTTP/WebSocket port ra ngoài internet (chỉ bind localhost port ngầm để kết nối với plugin).
- Không cần Figma token REST API, do đó không lo lộ lọt private API token.
- Đừng commit file `.claude.json` hoặc lưu trữ file config plugin lên các public repository.

## 10. Rate-limit Limitations
> This setup removes dependency on the official Figma MCP service for the local plugin-bridge transport.
> It does NOT guarantee unlimited requests.
> Claude model limits, local resource limits, WebSocket limits, plugin limitations, and any limits implemented by Figma or the selected MCP implementation may still apply. Mặc dù không bị giới hạn 200 calls/day như Figma REST API, bạn vẫn phải lưu ý giới hạn request từ chính AI Model mà bạn sử dụng.
