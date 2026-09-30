# Figma MCP Workspace

Repository này cung cấp cấu hình và bộ công cụ Figma Model Context Protocol (MCP) Server, kết nối trực tiếp với Figma Desktop thông qua Local Plugin Bridge (không cần Figma API token, không lo giới hạn Rate Limit).

Trong workspace hiện tại đã tích hợp sẵn **3 giải pháp** tùy theo nhu cầu sử dụng:

---

## 📂 Cấu trúc Repository

```text
mcp_figma/
├── figma-mcp-android/      # [DÀNH RIÊNG CHO ANDROID] MCP Server tối ưu đọc thiết kế & convert SVG -> Android VectorDrawable
├── figma-mcp-rust/         # [ĐA DỤNG] MCP Server hỗ trợ cả READ & WRITE (73 tools: tạo node, vẽ layout, đổi màu, styles...)
├── figma-plugin/           # Plugin bridge cho figma-mcp-rust
├── figma-mcp-bridge/       # Plugin của gethopp/figma-mcp-bridge v0.0.22 (đã chỉnh sang port 1995)
└── docs/
    ├── figma-mcp.md        # Hướng dẫn chi tiết cho figma-mcp-rust
    └── figma-mcp-android.md # Hướng dẫn chi tiết cho figma-mcp-android (Android Dev)
```

---

## ⚡ So sánh & Lựa chọn giải pháp

| Tiêu chí | `figma-mcp-android` (Mới thêm) | `figma-mcp-rust` |
| :--- | :--- | :--- |
| **Đối tượng** | **Android Developer** (Jetpack Compose / XML) | **Mọi nền tảng** (Web, Mobile, Design System) |
| **Quyền hạn** | **Read-Only**: Đọc thiết kế, xuất specs & asset | **Read & Write**: Đọc và tự vẽ/sửa trực tiếp trên Figma |
| **Số lượng Tools** | **21 tools** | **73 tools** |
| **Tính năng Android** | Có tool `convert_svg_to_android_drawable` tự động xuất XML VectorDrawable | Phải trích xuất SVG rồi convert thủ công |
| **Tối ưu Token** | Có `get_design_context` tinh gọn cho LLM | Cung cấp raw tree / minimal detail |
| **Chi tiết tài liệu** | 📄 [docs/figma-mcp-android.md](./docs/figma-mcp-android.md) | 📄 [docs/figma-mcp.md](./docs/figma-mcp.md) |

---

## 🚀 Hướng dẫn nhanh cho Android Dev (`figma-mcp-android`)

### 1. Cài đặt Plugin vào Figma Desktop
1. Mở **Figma Desktop**.
2. Menu: **Plugins** → **Development** → **Import plugin from manifest...**.
3. Chọn file manifest tại: `figma-mcp-android/plugin/manifest.json`.
4. Mở file thiết kế của bạn và bấm chạy plugin **Figma MCP Android**.

### 2. Cấu hình AI Client
Thêm cấu hình vào Client của bạn (Antigravity, Claude Code, Cursor, VS Code):

```json
{
  "mcpServers": {
    "figma-mcp-android": {
      "command": "npx",
      "args": ["-y", "@impeterwayne/figma-mcp-android@latest"]
    }
  }
}
```

*Hoặc dùng lệnh Claude Code:*
```bash
claude mcp add figma-mcp-android -- npx -y @impeterwayne/figma-mcp-android@latest
```

Xem hướng dẫn đầy đủ và các câu prompt mẫu tại: [docs/figma-mcp-android.md](./docs/figma-mcp-android.md).

---

## 🌉 Figma MCP Bridge (`figma-mcp-bridge`)

[gethopp/figma-mcp-bridge](https://github.com/gethopp/figma-mcp-bridge) — 40 tools đọc & ghi (`get_node`, `get_design_context`, `get_screenshot`, `create_frame`, `set_auto_layout`, animation...). Hỗ trợ nhiều file Figma cùng lúc (`list_files` + `fileKey`).

> ⚠️ Bridge mặc định dùng port `1994`, **trùng với `figma-mcp-rust`**. Plugin trong repo này đã được sửa sang `ws://localhost:1995`, nên server phải chạy với `FIGMA_BRIDGE_PORT=1995` để 2 server chạy song song.

### 1. Cài đặt Plugin vào Figma Desktop
1. **Plugins** → **Development** → **Import plugin from manifest...**
2. Chọn `figma-mcp-bridge/plugin/manifest.json`.
3. Mở file thiết kế, chạy plugin **Figma MCP Bridge** (giữ cửa sổ plugin mở khi dùng).

### 2. Cấu hình AI Client

```json
{
  "mcpServers": {
    "figma-bridge": {
      "command": "npx",
      "args": ["-y", "@gethopp/figma-mcp-bridge"],
      "env": { "FIGMA_BRIDGE_PORT": "1995" }
    }
  }
}
```

*Hoặc dùng lệnh Claude Code:*
```bash
claude mcp add -s user figma-bridge -e FIGMA_BRIDGE_PORT=1995 -- npx -y @gethopp/figma-mcp-bridge
```

### 3. Cập nhật plugin lên bản mới
Tải zip ở [Releases](https://github.com/gethopp/figma-mcp-bridge/releases), giải nén đè vào `figma-mcp-bridge/`, rồi đổi lại port:
```bash
sed -i 's#ws://localhost:1994#ws://localhost:1995#g' figma-mcp-bridge/plugin/manifest.json figma-mcp-bridge/plugin/dist/index.html
```
