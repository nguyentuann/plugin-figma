# Hướng dẫn sử dụng Figma MCP Android

Dành riêng cho **Android Developer**: Hướng dẫn cài đặt và sử dụng MCP Server chuyên biệt để đọc thiết kế từ Figma, xuất Design Context gọn nhẹ và tự động chuyển đổi asset SVG thành Android XML `VectorDrawable`.

---

## 1. Điểm nổi bật cho Android Dev

- **Không cần Figma API Token**: Kết nối trực tiếp với Figma Desktop qua plugin bridge local (WebSocket nội bộ).
- **Không giới hạn Rate Limit**: Không lo hết quota API.
- **Tối ưu Token AI (`get_design_context`)**: Cung cấp cây cấu trúc phân cấp tinh gọn, giúp AI sinh code Jetpack Compose hoặc XML Layout chính xác mà không tràn context window.
- **Convert SVG -> VectorDrawable (`convert_svg_to_android_drawable`)**: Tự động chuyển đổi các asset đồ họa SVG từ Figma thành file `<vector>` XML chuẩn của Android, đưa trực tiếp vào thư mục `res/drawable/`.
- **Xuất Tokens & Styles**: Hỗ trợ xuất colors, typography, variables để map vào `Theme.kt` hoặc `colors.xml`.

---

## 2. Cài đặt Plugin vào Figma Desktop

Plugin bridge đã được build sẵn file chạy (`dist/code.js` & `dist/index.html`):

1. Mở ứng dụng **Figma Desktop**.
2. Trên thanh menu, chọn: **Plugins** → **Development** → **Import plugin from manifest...**.
3. Chọn file `manifest.json` tại đường dẫn trong thư mục này:
   ```text
   d:\Braly_Repo\mcp_figma\figma-mcp-android\plugin\manifest.json
   ```
4. Plugin sẽ xuất hiện với tên **Figma MCP Android**.
5. Mở file thiết kế của bạn và bấm chạy plugin (nó sẽ lắng nghe kết nối tại `ws://127.0.0.1:1994`).

---

## 3. Cấu hình AI Client

Server được phân phối qua package npm `@impeterwayne/figma-mcp-android`, không cần cài đặt Go hoặc build binary thủ công.

### A. Cấu hình cho Antigravity
Vào cài đặt MCP (MCP Store -> **View raw config**), hoặc thêm vào file config workspace `.agents/mcp_config.json` (hoặc `~/.gemini/config/mcp_config.json`):

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

### B. Cấu hình cho Claude Code
Chạy lệnh trong terminal:
```bash
claude mcp add figma-mcp-android -- npx -y @impeterwayne/figma-mcp-android@latest
```
Hoặc cấu hình qua file `.mcp.json` ở thư mục dự án:
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

### C. Cấu hình cho Cursor / VS Code
Trong file `.cursor/mcp.json` hoặc `.vscode/mcp.json`:
```json
{
  "servers": {
    "figma-mcp-android": {
      "type": "stdio",
      "command": "npx",
      "args": ["-y", "@impeterwayne/figma-mcp-android@latest"]
    }
  }
}
```

---

## 4. Quy trình làm việc (Workflow cho Android Dev)

### Bước 1: Khởi động
1. Bật Figma Desktop, mở file thiết kế.
2. Bật plugin: **Plugins** → **Development** → **Figma MCP Android**.
3. Mở AI Client (Antigravity / Claude Code / Cursor).

### Bước 2: Đọc cấu trúc màn hình & Sinh code UI
Yêu cầu AI phân tích màn hình đang chọn:
```text
Hãy kết nối figma-mcp-android, gọi get_design_context trên node đang được chọn (hoặc màn hình Login). 
Sau đó viết code Jetpack Compose tương ứng, chia nhỏ component rõ ràng, hỗ trợ Material 3.
```

### Bước 3: Xuất Icon & Chuyển đổi sang Android VectorDrawable
Khi cần lấy icon:
```text
Tìm các icon vector trong màn hình này, lưu thành file SVG và sử dụng tool 
convert_svg_to_android_drawable để chuyển thành Android XML VectorDrawable 
đặt vào thư mục app/src/main/res/drawable/.
```

### Bước 4: Trích xuất màu sắc & Typography
```text
Sử dụng get_styles và get_variable_defs để lấy toàn bộ bảng màu và kiểu chữ, 
sau đó generate file Color.kt và Type.kt cho Jetpack Compose Theme.
```

---

## 5. Danh mục 21 Tools

### Nhóm đọc Document & Node
- `get_design_context`: Cây node rút gọn theo độ sâu, tối ưu token. **Nên dùng đầu tiên**.
- `get_document`: Toàn bộ cây node của page hiện tại.
- `get_pages`: Danh sách các page (ID + tên).
- `get_metadata`: Tên file, danh sách page, page đang mở.
- `get_selection`: Các node đang được chọn trong Figma.
- `get_node`: Lấy chi tiết một node theo ID.
- `get_nodes_info`: Lấy thông tin nhiều node cùng lúc.
- `search_nodes`: Tìm node theo tên hoặc type.
- `scan_nodes_by_types`: Lấy tất cả node thuộc các loại chỉ định.
- `scan_text_nodes`: Lấy toàn bộ text và nội dung chữ.
- `get_reactions`: Đọc interaction/prototype trigger và action.
- `get_viewport`: Vị trí khung nhìn, mức zoom.
- `get_fonts`: Danh sách font chữ đang sử dụng.

### Nhóm Styles & Tokens
- `get_styles`: Styles (paint, text, effect, grid).
- `get_variable_defs`: Variables (Color, Float, String, Boolean tokens).
- `get_local_components`: Danh sách component nội bộ.
- `get_annotations`: Annotation dev-mode.
- `export_tokens`: Xuất design tokens ra JSON hoặc CSS.

### Nhóm Export & Android Helpers
- `convert_svg_to_android_drawable`: Convert SVG thành file Android XML `VectorDrawable`.
- `get_screenshot`: Chụp ảnh node trả về Base64.
- `save_screenshots`: Chụp ảnh node và lưu trực tiếp ra file trên đĩa cứng.
