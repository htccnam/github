# TỔNG HỢP TOÀN BỘ QUY TRÌNH BÀI TẬP - GIT & GITHUB

<a id="mục-lục"></a>
## Mục lục
1. [Sơ đồ luồng dữ liệu tổng quan](#1-sơ-đồ-luồng-dữ-liệu-tổng-quan)
2. [Giai đoạn 1: Thiết lập hạ tầng & Khởi tạo](#2-giai-đoạn-1-thiết-lập-hạ-tầng--khởi-tạo)
3. [Giai đoạn 2: Quy trình làm việc cốt lõi](#3-giai-đoạn-2-quy-trình-làm-việc-cốt-lõi)
4. [Giai đoạn 3: Làm việc nhóm & Xử lý sự cố](#4-giai-đoạn-3-làm-việc-nhóm--xử-lý-sự-cố)
5. [Sổ tay lỗi kinh điển cần tránh](#5-sổ-tay-lỗi-kinh-điển-cần-tránh)

---

### 1. Sơ đồ luồng dữ liệu tổng quan

Trong kiến trúc Git, mã nguồn không được đẩy thẳng từ máy lên máy chủ từ xa (GitHub/GitLab) mà phải luân chuyển qua 4 khu vực lưu trữ logic độc lập. Hiểu rõ 4 vùng này giúp bạn nắm được bản chất tại sao một file có thể được sửa đổi mà không làm hỏng commit trước đó.

```text
+---------------------+         git add .         +---------------------+
|                     | ────────────────────────> |                     |
|  Working Directory  |                           | Staging Area/Index  |
|  (Thư mục làm việc) | <──────────────────────── |  (Vùng chuẩn bị)    |
|                     |     git restore --staged  |                     |
+---------------------+                           +---------------------+
           │                                                 │
           │                                                 │ git commit -m "..."
           │                                                 v
           │         git checkout / git restore   +---------------------+
           │ <─────────────────────────────────── |                     |
           │                                      |  Local Repository   |
           │         git reset HEAD~1             |  (Kho cục bộ .git)  |
           │ <─────────────────────────────────── |                     |
           +───────────────────────────────────── +---------------------+
                                                             │
                                                             │ git push
                                                             v
+-----------------------------------------------------------------------+
|                                                                       |
|                       Remote Repository/GitHub                        |
|                     (Kho lưu trữ từ xa trên Cloud)                    |
|                                                                       |
+-----------------------------------------------------------------------+
                                  │
                                  │ git pull (fetch + merge)
                                  v
                        [Working Directory & Local Repo]
```

#### Bản chất kỹ thuật của 4 khu vực:
1. **Working Directory (Thư mục làm việc thực tế):** Không gian thư mục trên ổ cứng máy tính nơi bạn trực tiếp tạo file `index.html`, `style.css`, `app.js` và viết code. Trạng thái file ở đây có thể là *Untracked* (chưa từng được Git theo dõi) hoặc *Modified* (đã chỉnh sửa so với commit trước).
2. **Staging Area / Index (Vùng dàn dựng / Vùng trạm cân):** Là một file nhị phân trung gian nằm ẩn bên trong `.git/index`. Đây là nơi lập trình viên chọn lọc chính xác những thay đổi nào sẽ được đóng gói vào snapshot tiếp theo.
3. **Local Repository (Kho lưu trữ cục bộ):** Nằm gọn trong thư mục ẩn `.git/`. Khi bạn thực thi lệnh commit, Git ghi nhận một ảnh chụp (snapshot) vĩnh viễn kèm mã hash SHA-1/SHA-256 vào database nội bộ của máy bạn. Tại đây, mọi lịch sử commit được bảo toàn hoàn toàn độc lập dù không có kết nối Internet.
4. **Remote Repository (Kho lưu trữ từ xa):** Máy chủ đám mây (GitHub, GitLab, Bitbucket) đóng vai trò làm trung tâm chia sẻ code cho toàn bộ thành viên trong đội ngũ phát triển.

---

### 2. Giai đoạn 1: Thiết lập hạ tầng & Khởi tạo

Ở giai đoạn này, chúng ta sẽ thiết lập danh tính cho Git, chuẩn hóa cấu hình môi trường, khởi tạo một dự án web tĩnh đơn giản (HTML/CSS/JS) và liên kết với kho lưu trữ từ xa trên GitHub.

#### Bước 1.1: Cấu hình danh tính lập trình viên toàn cục (Global Identity)
- **Tư duy logic của Senior:** Git là hệ thống lưu vết lịch sử. Mọi commit đều yêu cầu phải có chữ ký tác giả (Author name & Email). Nếu không khai báo, Git sẽ chặn việc commit hoặc sử dụng thông tin mặc định của hệ điều hành, làm hỏng tính minh bạch khi làm việc nhóm (không biết ai là người viết dòng code này).

Chạy các lệnh sau trên Terminal:
```bash
git config --global user.name "Nguyen Van A"
git config --global user.email "nguyenvana@example.com"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ dòng lệnh thực thi hệ thống quản lý mã nguồn Git.
- `config`: Lệnh con (subcommand) chịu trách nhiệm đọc/ghi các file cấu hình của Git.
- `--global`: Cờ tùy chọn (flag/option). Quy định áp dụng thiết lập này cho toàn bộ user đang đăng nhập trên hệ điều hành (ghi vào file `~/.gitconfig` hoặc `C:\Users\<User>\.gitconfig`), thay vì chỉ có hiệu lực trong một repo cục bộ (`--local`).
- `user.name`: Khóa định danh tên tác giả commit.
- `"Nguyen Van A"`: Giá trị chuỗi gán cho tên tác giả.
- `user.email`: Khóa định danh địa chỉ email đi kèm commit (nên trùng với email đăng ký tài khoản GitHub).
- `"nguyenvana@example.com"`: Giá trị email của lập trình viên.

```bash
git config --global init.defaultBranch main
git config --global core.autocrlf true
```

*Giải phẫu chi tiết lệnh Terminal:*
- `init.defaultBranch main`: Đặt tên nhánh mặc định khi khởi tạo repo mới là `main` (thay thế cho chuẩn cũ `master` để đồng bộ chuẩn quốc tế trên GitHub).
- `core.autocrlf true`: Tự động chuyển đổi ký tự kết thúc dòng (Line Ending). Trên Windows dùng `CRLF` (`\r\n`), trên Linux/macOS dùng `LF` (`\n`). Thiết lập `true` giúp tự động chuyển `CRLF` thành `LF` khi commit vào Git, và phục hồi lại `CRLF` khi checkout code ra Windows, loại trừ hoàn toàn lỗi lệch định dạng dòng khi phối hợp nhóm đa nền tảng.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
C:\Users\<User>\ (Thư mục người dùng hệ điều hành)
└── 📄 .gitconfig   <-- [MỚI TẠO HOẶC CẬP NHẬT từ lệnh git config]
                        Lưu trữ:
                        [user]
                            name = Nguyen Van A
                            email = nguyenvana@example.com
                        [init]
                            defaultBranch = main
                        [core]
                            autocrlf = true
```

---

#### Bước 1.2: Chuẩn bị thư mục dự án Web tĩnh (HTML/CSS/JS)
- **Tư duy logic của Senior:** Trước khi gắn động cơ Git, ta cần có bộ khung mã nguồn của một ứng dụng web tĩnh thực thụ để quan sát sự tương tác giữa file vật lý và hệ thống theo dõi của Git.

Tạo cấu trúc thư mục và file cơ bản:
```bash
mkdir my-static-web
cd my-static-web
```

*Giải phẫu chi tiết lệnh Terminal:*
- `mkdir`: Lệnh hệ điều hành (Make Directory) dùng để tạo mới một thư mục.
- `my-static-web`: Tên thư mục dự án được tạo.
- `cd`: Lệnh hệ điều hành (Change Directory) dùng để chuyển con trỏ dòng lệnh hiện hành vào thư mục được chỉ định.

Tạo các file mã nguồn cơ bản:
```bash
New-Item -ItemType File index.html, style.css, app.js, README.md
```
*(Nếu dùng Bash/Linux: `touch index.html style.css app.js README.md`)*

*Giải phẫu chi tiết lệnh Terminal:*
- `New-Item`: Lệnh PowerShell khởi tạo đối tượng mới trên hệ thống tệp.
- `-ItemType File`: Tham số chỉ định kiểu đối tượng cần tạo là tập tin (File).
- `index.html, style.css, app.js, README.md`: Danh sách tên các tập tin cần tạo đồng thời trong thư mục.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 index.html   <-- [MỚI TẠO từ lệnh tạo file] Thư mục bình thường, Git CHƯA biết đến sự tồn tại
├── 📄 style.css    <-- [MỚI TẠO từ lệnh tạo file] Thư mục bình thường, Git CHƯA biết đến sự tồn tại
├── 📄 app.js       <-- [MỚI TẠO từ lệnh tạo file] Thư mục bình thường, Git CHƯA biết đến sự tồn tại
└── 📄 README.md    <-- [MỚI TẠO từ lệnh tạo file] Thư mục bình thường, Git CHƯA biết đến sự tồn tại
```

---

#### Bước 1.3: Khởi tạo Local Repository bằng `git init`
- **Tư duy logic của Senior:** Thư mục máy tính thông thường không có khả năng tự nhớ lịch sử chỉnh sửa file. Lệnh `git init` sẽ "thổi hồn" Git vào thư mục bằng cách sinh ra cơ sở dữ liệu ngầm `.git/`. Từ giây phút này, mọi biến động của từng byte dữ liệu trong thư mục đều nằm trong tầm kiểm soát của Git.

Chạy lệnh:
```bash
git init
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Trình quản lý phiên bản phân tán Git.
- `init`: Lệnh con khởi tạo (Initialize), tạo lập toàn bộ cấu trúc dữ liệu của một kho lưu trữ Git trắng (bare repository structure) ngay bên trong thư mục hiện hành.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📁 .git/        <-- [MỚI TẠO chính xác từ lệnh git init] Chứa toàn bộ database ngầm, con trỏ HEAD, object database
├── 📄 index.html   <-- [CHUYỂN TRẠNG THÁI] Được Git phát hiện dưới dạng: Untracked (Chưa theo dõi)
├── 📄 style.css    <-- [CHUYỂN TRẠNG THÁI] Được Git phát hiện dưới dạng: Untracked (Chưa theo dõi)
├── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Được Git phát hiện dưới dạng: Untracked (Chưa theo dõi)
└── 📄 README.md    <-- [CHUYỂN TRẠNG THÁI] Được Git phát hiện dưới dạng: Untracked (Chưa theo dõi)
```

---

#### Bước 1.4: Kết nối kho lưu trữ từ xa Remote GitHub
- **Tư duy logic của Senior:** Code nằm trên máy cá nhân có nguy cơ mất trắng nếu hỏng ổ cứng hoặc mất máy. Ta cần liên kết Local Repo với một Remote Repo trên GitHub để làm nơi lưu trữ tập trung và phục vụ phối hợp nhóm.

Chạy lệnh:
```bash
git remote add origin https://github.com/<tai-khoan-cua-ban>/my-static-web.git
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `remote`: Nhóm lệnh con quản lý các đường dẫn kết nối đến máy chủ từ xa (Remote Repositories).
- `add`: Hành động thêm mới một liên kết remote vào cấu hình repo.
- `origin`: Tên bí danh quy ước chuẩn (alias/shortname) đại diện cho URL remote chính. Thay vì mỗi lần gõ cả đường dẫn dài ngoằng, ta chỉ cần gọi tên `origin`.
- `https://github.com/...`: Đường dẫn định tuyến thực tế trên Internet trỏ đến repo GitHub.

Kiểm tra lại kết nối:
```bash
git remote -v
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git remote`: Lệnh hiển thị các remote alias.
- `-v`: Cờ viết tắt của `--verbose` (chi tiết), yêu cầu Git in rõ cả URL dùng để Fetch (lấy về) và Push (đẩy lên).

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📁 .git/
    └── 📄 config   <-- [CẬP NHẬT chính xác từ lệnh git remote add]
                        Ghi thêm cấu hình:
                        [remote "origin"]
                            url = https://github.com/<tai-khoan-cua-ban>/my-static-web.git
                            fetch = +refs/heads/*:refs/remotes/origin/*
```

[⬆️ Quay lại Mục lục](#mục-lục)

---

### 3. Giai đoạn 2: Quy trình làm việc cốt lõi

Đây là giai đoạn bạn sẽ sống cùng nó mỗi ngày khi đi làm. Chúng ta sẽ làm chủ vòng đời file, kỹ thuật kiểm tra trạng thái, đưa file vào vùng dàn dựng, thực hành commit theo chuẩn quốc tế **Conventional Commits**, đọc cây lịch sử commit trực quan và quản lý nhánh (branching).

#### Vòng đời file trong Git (File Lifecycle):
```text
 +-----------------------------------------------------------------------+
 |                                                                       |
 |   [Untracked] ──(git add)──> [Staged] ──(git commit)──> [Unmodified]  |
 |        ^                        ^                            │        |
 |        │                        │                            │        |
 |    (tạo file)              (git add)                     (sửa code)   |
 |        │                        │                            │        |
 |        │                        │                            v        |
 |        +────────────────────────┴──────────────────── [Modified]      |
 |                                                                       |
 +-----------------------------------------------------------------------+
```

---

#### Bước 2.1: Kiểm tra trạng thái bằng `git status`
- **Tư duy logic của Senior:** Trước khi đưa ra bất kỳ quyết định đóng gói code nào, lập trình viên chuyên nghiệp luôn phải "soi gương" dự án để biết: Có file nào mới tạo chưa? Có file nào đang bị sửa đổi dở dang không? Tránh việc vô tình đẩy nhầm code rác lên repo.

Chạy lệnh:
```bash
git status
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `status`: Lệnh con yêu cầu Git so sánh toàn diện giữa 3 vùng: Working Directory, Staging Area và commit gần nhất của Local Repository, sau đó hiển thị báo cáo chi tiết về tình trạng hiện tại.

Git sẽ phản hồi:
```text
On branch main
No commits yet
Untracked files:
  (use "git add <file>..." to include in what will be committed)
	README.md
	app.js
	index.html
	style.css
nothing added to commit but untracked files present (use "git add" to track)
```

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 index.html   <-- [TRẠNG THÁI XÁC NHẬN] Untracked (Nằm ở Working Directory, CHƯA vào Staging)
├── 📄 style.css    <-- [TRẠNG THÁI XÁC NHẬN] Untracked (Nằm ở Working Directory, CHƯA vào Staging)
├── 📄 app.js       <-- [TRẠNG THÁI XÁC NHẬN] Untracked (Nằm ở Working Directory, CHƯA vào Staging)
└── 📄 README.md    <-- [TRẠNG THÁI XÁC NHẬN] Untracked (Nằm ở Working Directory, CHƯA vào Staging)
```

---

#### Bước 2.2: Đưa thay đổi vào Staging Area bằng `git add`
- **Tư duy logic của Senior:** Git không tự động lưu tất cả mọi file bạn chỉnh sửa vào commit. Staging Area đóng vai trò là "chiếc giỏ hàng". Bạn mua món hàng nào thì nhặt món đó vào giỏ. Việc này cho phép bạn chia nhỏ các thay đổi lớn thành nhiều commit logic, sạch sẽ.

Giả sử ta viết mã HTML cơ bản vào `index.html` và viết CSS vào `style.css`:
```bash
git add .
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `add`: Lệnh con di chuyển dữ liệu thay đổi từ Working Directory vào Staging Area (Index).
- `.`: Ký tự dấu chấm đại diện cho thư mục hiện hành và toàn bộ thư mục con đệ quy bên trong nó. Thay vì gõ từng file `git add index.html style.css app.js README.md`, dấu `.` gom tất cả file có thay đổi vào Staging Area.

*Lưu ý từ Tech Lead:* Nếu chỉ muốn đưa đích danh một file vào staging, hãy dùng `git add <tên_file>`.

Kiểm tra lại bằng `git status`:
```bash
git status
```

Git sẽ hiển thị màu xanh lá cây:
```text
Changes to be committed:
  (use "git rm --cached <file>..." to unstage)
	new file:   README.md
	new file:   app.js
	new file:   index.html
	new file:   style.css
```

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📁 .git/
│   └── 📄 index    <-- [MỚI GHI NHẬN] File nhị phân lưu danh sách 4 file chuẩn bị commit
├── 📄 index.html   <-- [CHUYỂN TRẠNG THÁI] Từ Untracked ──> Staged (Sẵn sàng commit)
├── 📄 style.css    <-- [CHUYỂN TRẠNG THÁI] Từ Untracked ──> Staged (Sẵn sàng commit)
├── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Từ Untracked ──> Staged (Sẵn sàng commit)
└── 📄 README.md    <-- [CHUYỂN TRẠNG THÁI] Từ Untracked ──> Staged (Sẵn sàng commit)
```

---

#### Bước 2.3: Đóng gói phiên bản với Chuẩn Quy ước Conventional Commits

##### 📖 BẢNG QUY TẮC CONVENTIONAL COMMITS (BẮT BUỘC THUỘC LÒNG)
Thay vì viết những câu commit vô nghĩa như: *"update"*, *"sửa code"*, *"fix bug"*, *"xong việc"* – những thông điệp khiến đồng đội và chính bạn sau 3 tháng không thể biết file đã đổi cái gì, các dự án công nghệ chuyên nghiệp bắt buộc áp dụng chuẩn **Conventional Commits**:

Cú pháp:
$$\text{<type>(<scope>): <mô tả ngắn bằng động từ thể hiện hành động>}$$

Trong đó:
- **`type` (Bắt buộc):** Thể hiện mục đích cốt lõi của lần thay đổi.
  - `feat`: Thêm tính năng mới (Feature) cho người dùng (VD: Thêm form đăng nhập, thêm hiệu ứng modal).
  - `fix`: Sửa chữa lỗi phát sinh trong mã nguồn (Bug fix) (VD: Vỡ giao diện trên iPhone, tính sai tiền giỏ hàng).
  - `docs`: Chỉ thay đổi tài liệu hướng dẫn, Markdown, tài liệu kỹ thuật (VD: Viết hướng dẫn cài đặt trong README).
  - `refactor`: Tái cấu trúc, tối ưu mã nguồn mà **hoàn toàn không** làm thay đổi logic hoạt động hay tính năng (VD: Tách 1 hàm 200 dòng thành 3 hàm con, đổi tên biến cho rõ nghĩa).
  - `chore`: Những công việc phụ trợ, cập nhật cấu hình build, công cụ, quản lý thư viện phụ thuộc (dependencies) (VD: Cập nhật file `.gitignore`, cấu hình webpack, thêm thư viện axios).
- **`scope` (Tùy chọn):** Phạm vi/Module bị tác động nằm trong dấu ngoặc đơn (VD: `(ui)`, `(auth)`, `(cart)`, `(api)`).
- **`mô tả ngắn`:** Viết ngắn gọn, rõ ràng, không viết hoa chữ cái đầu sau dấu hai chấm (theo chuẩn conventional), không để dấu chấm câu ở cuối.

---

##### THỰC HÀNH CÁC KỊCH BẢN COMMIT THỰC TẾ TRONG DỰ ÁN WEB TĨNH:

##### Kịch bản 1: Commit khởi tạo giao diện cơ bản (`feat`)
- **Tư duy logic:** Đóng gói phiên bản đầu tiên của giao diện web tĩnh gồm HTML và CSS.

Chạy lệnh:
```bash
git commit -m "feat(ui): khoi tao giao dien web tinh voi html va css"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `commit`: Lệnh con thực hiện chụp ảnh (snapshot) toàn bộ các file đang ở Staging Area và lưu vĩnh viễn vào cơ sở dữ liệu của Local Repository.
- `-m`: Cờ tùy chọn viết tắt của `--message`. Báo hiệu chuỗi văn bản đứng ngay sau đây trong dấu ngoặc kép sẽ là nội dung thông điệp của commit, giúp không phải bật trình soạn thảo mặc định (Vim/Nano).
- `"feat(ui): khoi tao giao dien web tinh voi html va css"`: Nội dung thông điệp tuân thủ 100% chuẩn Conventional Commits với `type` là `feat`, `scope` là `ui`.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📁 .git/
│   └── 📁 objects/ <-- [MỚI TẠO] Sinh ra commit object (mã hash SHA ví dụ: a1b2c3d) lưu trữ snapshot
├── 📄 index.html   <-- [CHUYỂN TRẠNG THÁI] Từ Staged ──> Unmodified (Đã an toàn nằm trong commit a1b2c3d)
├── 📄 style.css    <-- [CHUYỂN TRẠNG THÁI] Từ Staged ──> Unmodified (Đã an toàn nằm trong commit a1b2c3d)
├── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Từ Staged ──> Unmodified (Đã an toàn nằm trong commit a1b2c3d)
└── 📄 README.md    <-- [CHUYỂN TRẠNG THÁI] Từ Staged ──> Unmodified (Đã an toàn nằm trong commit a1b2c3d)
```

---

##### Kịch bản 2: Bổ sung tính năng tương tác người dùng (`feat`)
- **Tư duy logic:** Viết thêm code JavaScript vào `app.js` để bắt sự kiện click nút bấm tăng lượt đếm. Đây là tính năng mới.

Chỉnh sửa nội dung file `app.js`, sau đó chạy:
```bash
git add app.js
git commit -m "feat(counter): them logic dem so luot click nut bam"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add app.js`: Chỉ nhặt riêng file `app.js` vừa sửa vào Staging Area, không đụng tới các file khác.
- `git commit`: Tạo snapshot mới.
- `-m "feat(counter): them logic dem so luot click nut bam"`: Gắn cờ mô tả tính năng mới cho bộ đếm counter.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Modified ──(git add)──> Staged ──(git commit)──> Unmodified
├── 📄 index.html   <-- [GIỮ NGUYÊN] Unmodified (Không bị ảnh hưởng)
└── 📁 .git/        <-- [CẬP NHẬT] Con trỏ HEAD tiến lên 1 bước, trỏ vào commit mới (ví dụ: e4f5g6h)
```

---

##### Kịch bản 3: Sửa lỗi vỡ giao diện trên màn hình di động (`fix`)
- **Tư duy logic:** Khi mở trên điện thoại, khung chứa (container) trong `style.css` bị tràn chiều ngang (overflow-x). Ta sửa CSS và ghi nhận đây là hành động sửa lỗi.

Sửa file `style.css`, sau đó chạy:
```bash
git add style.css
git commit -m "fix(responsive): khac phuc loi vo layout tren man hinh di dong"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add style.css`: Đưa file CSS đã fix lỗi vào vùng Staging Area.
- `git commit`: Lưu ảnh chụp sửa lỗi.
- `-m "fix(responsive): ..."`: Type `fix` thông báo rõ cho cả team biết đây là bản vá lỗi giao diện phản hồi.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 style.css    <-- [CHUYỂN TRẠNG THÁI] Modified ──(git add)──> Staged ──(git commit)──> Unmodified
└── 📁 .git/        <-- [CẬP NHẬT] Tạo node commit mới trên cây lịch sử nhánh main
```

---

##### Kịch bản 4: Tối ưu code JavaScript mà không đổi tính năng (`refactor`)
- **Tư duy logic:** Hàm xử lý sự kiện trong `app.js` đang lặp code hoặc viết chưa tối ưu. Ta gom logic vào hàm riêng ngắn gọn hơn. Giao diện và kết quả chạy của người dùng không đổi.

Sửa file `app.js`, sau đó chạy:
```bash
git add app.js
git commit -m "refactor(counter): tach ham cap nhat dom ra module rieng"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add app.js`: Đưa file đã tái cấu trúc vào Staging.
- `git commit`: Đóng gói commit.
- `-m "refactor(counter): ..."`: Type `refactor` khẳng định việc thay đổi code này không làm thay đổi hành vi nghiệp vụ, tester không cần lo lắng về tính năng mới.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Modified ──(git add)──> Staged ──(git commit)──> Unmodified
└── 📁 .git/        <-- [CẬP NHẬT] Ghi nhận thêm 1 node snapshot refactor
```

---

##### Kịch bản 5: Cập nhật tài liệu hướng dẫn dự án (`docs`)
- **Tư duy logic:** Viết thêm vào `README.md` mô tả các bước mở file web bằng trình duyệt. Chỉ thuần túy là tài liệu, không có dòng code nào chạy trên trình duyệt bị ảnh hưởng.

Sửa file `README.md`, sau đó chạy:
```bash
git add README.md
git commit -m "docs(readme): cap nhat huong dan khoi chay du an web tinh"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add README.md`: Đưa file tài liệu vào Staging.
- `git commit`: Lưu commit.
- `-m "docs(readme): ..."`: Type `docs` giúp phân loại rõ đây là cập nhật tài liệu.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📄 README.md    <-- [CHUYỂN TRẠNG THÁI] Modified ──(git add)──> Staged ──(git commit)──> Unmodified
```

---

##### Kịch bản 6: Thêm cấu hình loại bỏ file rác (`chore`)
- **Tư duy logic:** Các file của hệ điều hành như `.DS_Store` (macOS), `Thumbs.db` (Windows) hay thư mục editor `.vscode/` không thuộc về sản phẩm. Ta tạo file `.gitignore` để Git tự động lờ chúng đi.

Tạo và chỉnh sửa file `.gitignore`, sau đó chạy:
```bash
git add .gitignore
git commit -m "chore(git): them file .gitignore loai bo file rac he dieu hanh"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add .gitignore`: Đưa file cấu hình vào Staging.
- `git commit`: Lưu commit.
- `-m "chore(git): ..."`: Type `chore` thể hiện công việc dọn dẹp cấu hình, bảo trì môi trường phát triển.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 .gitignore   <-- [MỚI TẠO & CHUYỂN TRẠNG THÁI] Untracked ──> Staged ──> Unmodified
└── 📁 .git/        <-- [CẬP NHẬT] Ghi nhận commit cấu hình hoàn tất
```

---

#### Bước 2.4: Xem lịch sử commit trực quan bằng `git log --oneline --graph`
- **Tư duy logic của Senior:** Lệnh `git log` mặc định in ra quá nhiều thông tin dài dòng (tên tác giả, ngày giờ, mô tả đầy đủ), làm ngập màn hình khi dự án có hàng nghìn commit. Sử dụng cờ `--oneline` kết hợp `--graph` giúp bạn có ngay góc nhìn chim ưng (bird's-eye view) về sơ đồ phân nhánh và lịch sử phát triển cực kỳ gọn gàng.

Chạy lệnh:
```bash
git log --oneline --graph
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `log`: Lệnh con truy vấn lịch sử commit theo thứ tự thời gian đảo ngược (mới nhất hiển thị trước).
- `--oneline`: Cờ rút gọn, yêu cầu Git nén mỗi commit thành đúng 1 dòng duy nhất: chỉ gồm 7 ký tự đầu của mã băm SHA (short hash) và tiêu đề commit message.
- `--graph`: Cờ vẽ cây phả hệ bằng ký tự ASCII dạng đồ thị nhánh (`*`, `|`, `/`, `\`), hiển thị rõ ràng điểm rẽ nhánh và điểm nhập nhánh (merge).

Kết quả hiển thị trực quan trên Terminal:
```text
* f3a1b2c (HEAD -> main) chore(git): them file .gitignore loai bo file rac he dieu hanh
* d9e8f7a docs(readme): cap nhat huong dan khoi chay du an web tinh
* c8b7a6d refactor(counter): tach ham cap nhat dom ra module rieng
* b7a6f5e fix(responsive): khac phuc loi vo layout tren man hinh di dong
* a6e5d4c feat(counter): them logic dem so luot click nut bam
* 9f8e7d6 feat(ui): khoi tao giao dien web tinh voi html va css
```

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
Cây lịch sử commit trong .git/:
(HEAD -> main) [f3a1b2c] chore(git): them file .gitignore...
      │
      v
   [d9e8f7a] docs(readme): cap nhat huong dan...
      │
      v
   [c8b7a6d] refactor(counter): tach ham...
      │
      v
   [b7a6f5e] fix(responsive): khac phuc loi...
      │
      v
   [a6e5d4c] feat(counter): them logic...
      │
      v
   [9f8e7d6] feat(ui): khoi tao giao dien... (Root commit)
```

---

#### Bước 2.5: Quản lý nhánh (Branching) chuyên nghiệp
- **Tư duy logic của Senior:** **QUY TẮC SỐNG CÒN:** Không bao giờ được phép code tính năng mới trực tiếp trên nhánh `main`!
  - Nhánh `main` đại diện cho sản phẩm đang chạy ngoài thực tế (Production).
  - Khi bắt đầu làm bất cứ tính năng hay sửa lỗi nào, phải rẽ ra một nhánh mới (Feature Branch). Nhánh này hoàn toàn độc lập, nếu bạn làm hỏng code thì nhánh `main` vẫn an toàn và người dùng không bị ảnh hưởng. Sau khi test hoàn chỉnh mới gộp (merge) lại.

Tạo và chuyển sang nhánh tính năng mới:
```bash
git checkout -b feature/login-modal
```
*(Hoặc dùng lệnh hiện đại hơn từ Git 2.23+: `git switch -c feature/login-modal`)*

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `checkout`: Lệnh con chuyển đổi môi trường làm việc giữa các nhánh hoặc khôi phục file.
- `-b`: Cờ viết tắt của `--branch`, ra lệnh cho Git: *Nếu nhánh này chưa tồn tại, hãy tạo mới nó ngay lập tức trước khi chuyển sang*.
- `feature/login-modal`: Tên nhánh mới tuân theo quy ước tiền tố chuẩn: `feature/` (tính năng), `bugfix/` (sửa lỗi), `hotfix/` (sửa khẩn cấp).

Kiểm tra danh sách nhánh hiện có:
```bash
git branch
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git branch`: Liệt kê tất cả các nhánh cục bộ trong máy. Nhánh nào có dấu `*` và màu xanh lá cây là nhánh con trỏ `HEAD` đang trỏ vào.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📁 .git/
    ├── 📄 HEAD                 <-- [CHUYỂN HƯỚNG] Trỏ từ "refs/heads/main" ──> "refs/heads/feature/login-modal"
    └── 📁 refs/
        └── 📁 heads/
            ├── 📄 main                 <-- [GIỮ NGUYÊN vị trí commit f3a1b2c]
            └── 📄 feature/login-modal  <-- [MỚI TẠO] Khởi sinh cùng vị trí commit f3a1b2c với main
```

[⬆️ Quay lại Mục lục](#mục-lục)

---

### 4. Giai đoạn 3: Làm việc nhóm & Xử lý sự cố

Khi bước vào môi trường doanh nghiệp, bạn sẽ phải làm việc cùng 5, 10 hoặc hàng trăm kỹ sư trên cùng một kho mã nguồn. Bạn bắt buộc phải nắm vững quy trình đồng bộ, tạo Pull Request, bắt và giải quyết xung đột mã nguồn (Merge Conflict), và kỹ năng cứu hộ dữ liệu (Reset/Revert).

#### Bước 4.1: Đồng bộ mã nguồn Push và Pull
- **Tư duy logic của Senior:**
  - **Luật bất thành văn:** Luôn luôn chạy `git pull` trước khi bắt đầu ngày làm việc và trước khi thực hiện `git push`. Vì sao? Vì trong lúc bạn đang ngủ hoặc đang code, đồng đội đã đẩy code mới lên GitHub. Việc kéo code mới nhất về máy giúp bạn phát hiện xung đột sớm ngay tại máy mình, tránh làm vỡ kho code chung.

Đẩy nhánh `main` đầu tiên lên GitHub:
```bash
git push -u origin main
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `push`: Lệnh con đẩy các commit từ Local Repository lên Remote Repository.
- `-u`: Viết tắt của `--set-upstream`. Thiết lập mối liên kết theo dõi mặc định (tracking reference) giữa nhánh cục bộ `main` và nhánh từ xa `origin/main`. Sau lần đầu tiên chạy cờ này, ở các lần sau bạn chỉ cần gõ ngắn gọn `git push` hoặc `git pull` mà không cần chỉ định rõ `origin main`.
- `origin`: Tên remote đích cần đẩy lên.
- `main`: Nhánh cục bộ cần đẩy lên máy chủ.

Kéo cập nhật mới nhất từ GitHub về máy:
```bash
git pull origin main
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `pull`: Lệnh con tải về toàn bộ thay đổi mới nhất từ remote và tự động gộp (merge) thẳng vào nhánh làm việc hiện tại. Thực chất `git pull` = `git fetch` (tải metadata về) + `git merge` (hợp nhất code vào Working Directory).
- `origin`: Tên remote nguồn cần kéo.
- `main`: Nhánh từ xa cần lấy code về.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
Trạng thái đồng bộ:
Local Repository [main]  <====================>  GitHub Remote [origin/main]
(Cả 2 bên đều khớp nhau chính xác từng commit SHA)
```

---

#### Bước 4.2: Quy trình tạo Pull Request (PR) chuẩn Tech Lead
- **Tư duy logic của Senior:** Trong môi trường làm việc chuyên nghiệp, không ai được phép tự ý gộp code từ nhánh cá nhân thẳng vào nhánh `main`. Bạn phải đẩy nhánh tính năng lên GitHub và mở một **Pull Request (PR)**. PR là một yêu cầu gửi tới Tech Lead và các đồng đội: *"Tôi đã làm xong tính năng này, kính mời các bạn vào đọc code, kiểm tra lỗ hổng bảo mật và cho ý kiến trước khi gộp vào sản phẩm"*.

Quy trình:
1. Đẩy nhánh tính năng lên GitHub:
```bash
git push -u origin feature/login-modal
```
*Giải phẫu:* Đẩy nhánh `feature/login-modal` lên GitHub và thiết lập upstream tương ứng.

2. Truy cập giao diện GitHub, nhấn **Compare & pull request**.
3. Điền mô tả PR theo biểu mẫu chuẩn:
```markdown
## 📌 Loại thay đổi (Type)
- [x] feat: Tính năng mới

## 📝 Tóm tắt thay đổi
- Bổ sung modal đăng nhập gồm form HTML và style CSS phản hồi nhanh.
- Xử lý mở/đóng modal bằng JavaScript.

## 🧪 Kiểm thử (Testing)
- Đã test mở/đóng mượt mà trên Chrome, Firefox và Safari Mobile.
- Không phát sinh lỗi console.
```

---

#### Bước 4.3: Bắt và xử lý xung đột mã nguồn (Merge Conflict)
- **Tư duy logic của Senior:** Rất nhiều bạn mới đi làm bị hoảng loạn khi thấy chữ **CONFLICT**. Bản chất của xung đột rất đơn giản: Khi 2 lập trình viên cùng sửa đổi **trên cùng một dòng** (hoặc cùng một vùng code) của một file, Git không thể tự ý quyết định lấy code của ai vì làm vậy có nguy cơ xóa mất tính năng của người kia. Do đó, Git dừng tiến trình merge lại và yêu cầu con người đưa ra phán quyết.

##### Mô phỏng kịch bản xung đột thực tế:
- **Đồng đội (Dev B):** Sửa dòng 10 trong `style.css` thành `background-color: blue;` và đã push lên `main`.
- **Bạn (Dev A):** Trên nhánh của mình cũng sửa đúng dòng 10 trong `style.css` thành `background-color: red;`.
- Khi thực hiện gộp nhánh:
```bash
git merge main
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `merge`: Lệnh con hợp nhất lịch sử của nhánh được chỉ định (`main`) vào nhánh hiện hành mà con trỏ `HEAD` đang đứng.

Git sẽ lập tức báo động:
```text
Auto-merging style.css
CONFLICT (content): Merge conflict in style.css
Automatic merge failed; fix conflicts and then commit the result.
```

##### Nhận diện dấu vết xung đột do Git tự động chèn vào file:
Mở file `style.css`, bạn sẽ thấy Git đánh dấu rõ ràng:
```css
<<<<<<< HEAD
/* Code hiện tại trên nhánh của bạn (Dev A) */
.btn-primary {
    background-color: red;
}
=======
/* Code mới nhất kéo từ nhánh main về (Dev B) */
.btn-primary {
    background-color: blue;
}
>>>>>>> main
```

*Bóc tách các ký hiệu xung đột:*
- `<<<<<<< HEAD`: Đánh dấu điểm bắt đầu vùng code của nhánh hiện tại bạn đang đứng.
- `=======`: Vạch ranh giới ngăn cách giữa 2 phiên bản code xung đột.
- `>>>>>>> main`: Đánh dấu điểm kết thúc của vùng code đến từ nhánh đang gộp vào (`main`).

##### 3 bước giải quyết xung đột chuẩn kỹ sư:
1. **Thảo luận & Lựa chọn:** Trao đổi với Dev B xem nên chọn màu đỏ, màu xanh hay kết hợp cả hai. Giả sử thống nhất chọn màu tím (`purple`).
2. **Xóa sạch ký hiệu rác:** Xóa toàn bộ các dòng `<<<<<<<`, `=======`, `>>>>>>>` và chỉ để lại code hoàn chỉnh đã thống nhất:
```css
.btn-primary {
    background-color: purple;
}
```
3. **Đóng dấu giải quyết xong và commit:**
```bash
git add style.css
git commit -m "fix(conflict): giai quyet xung dot mau sac nut bam giua main va feature"
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git add style.css`: Đưa file đã giải quyết xong xung đột vào Staging Area để báo cho Git biết: *"Tôi đã xử lý xong file này"*.
- `git commit -m "..."`: Hoàn tất tiến trình merge bằng một commit giải quyết xung đột (Merge commit).

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📄 style.css    <-- [CHUYỂN TRẠNG THÁI] Conflicted ──(xử lý tay & git add)──> Staged ──(git commit)──> Unmodified
└── 📁 .git/        <-- [CẬP NHẬT] Tiến trình merge hoàn tất, tạo merge commit nối 2 nhánh lại với nhau
```

---

#### Bước 4.4: Hoàn tác code an toàn với Reset, Revert và Restore
- **Tư duy logic của Senior:** Đi làm sẽ có lúc bạn gõ nhầm lệnh, sửa hỏng file hoặc commit nhầm tính năng chưa hoàn thiện. Bạn phải nắm rõ các công cụ "quay ngược thời gian" và biết rõ công cụ nào an toàn cho môi trường nhóm, công cụ nào chỉ được dùng nội bộ máy cá nhân.

##### Bảng phân loại công cụ cứu hộ:

| Lệnh | Vùng tác động | Khi nào nên dùng? | Mức độ an toàn |
| :--- | :--- | :--- | :--- |
| `git restore <file>` | Working Directory | Hủy bỏ thay đổi của một file chưa `git add` | An toàn cục bộ |
| `git restore --staged <file>` | Staging Area | Rút file từ Staging trở về Working Directory | Rất an toàn |
| `git reset --soft HEAD~1` | Local Repo -> Staging | Hủy commit gần nhất, giữ nguyên toàn bộ code trong Staging | Rất an toàn |
| `git reset --mixed HEAD~1` | Local Repo -> Working Dir | Hủy commit gần nhất, đưa code về trạng thái chưa add (Mặc định) | An toàn |
| `git reset --hard HEAD~1` | Xóa sạch mọi nơi | Xóa bỏ hoàn toàn commit và vứt bỏ toàn bộ code sửa đổi | **CỰC KỲ NGUY HIỂM** |
| `git revert <commit-hash>` | Local Repo (Tạo commit mới) | Đảo ngược một commit **đã đẩy lên GitHub** | **Chuẩn an toàn cho Team** |

---

##### Kịch bản Hoàn tác 1: Hủy bỏ sửa đổi chưa `git add`
Giả sử bạn lỡ tay xóa một đoạn mã trong `app.js` và muốn file trở về trạng thái y hệt như lần commit gần nhất:
```bash
git restore app.js
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `restore`: Lệnh con khôi phục lại nội dung tập tin trong Working Directory về trạng thái của commit gần nhất.
- `app.js`: Tên file cụ thể cần khôi phục.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📄 app.js       <-- [CHUYỂN TRẠNG THÁI] Từ Modified ──> Khôi phục lại y nguyên bản commit trước (Unmodified)
```

---

##### Kịch bản Hoàn tác 2: Rút file đã lỡ `git add` ra khỏi Staging Area
Giả sử bạn gõ `git add .` và vô tình đưa nhầm cả file `secret.txt` vào Staging Area, nhưng chưa commit:
```bash
git restore --staged secret.txt
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `restore`: Lệnh khôi phục.
- `--staged`: Cờ yêu cầu khôi phục trạng thái ở Staging Area (bỏ theo dõi đợt này), giữ nguyên nội dung file trong Working Directory.
- `secret.txt`: Tên file cần rút ra.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📄 secret.txt   <-- [CHUYỂN TRẠNG THÁI] Từ Staged (sẵn sàng commit) ──> Untracked (Nằm lại thư mục làm việc)
```

---

##### Kịch bản Hoàn tác 3: Đã lỡ commit trên máy cá nhân, muốn rút lại để sửa tiếp (`git reset`)
Giả sử bạn vừa commit xong nhưng phát hiện viết sai thông điệp hoặc thiếu 1 file:
```bash
git reset --soft HEAD~1
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `reset`: Lệnh con dịch chuyển con trỏ nhánh hiện tại về một commit trong quá khứ.
- `--soft`: Cờ an toàn nhất của reset. Nó chỉ hủy bỏ bản ghi commit trong `.git/`, toàn bộ code bạn đã viết vẫn được giữ nguyên vẹn 100% trong Staging Area.
- `HEAD~1`: Vị trí cần lùi về, nghĩa là commit nằm ngay trước commit hiện hành 1 bước (`~1`).

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
├── 📁 .git/        <-- [CẬP NHẬT] Con trỏ HEAD lùi lại 1 node commit
└── 📄 app.js       <-- [TRẠNG THÁI] Vẫn nằm nguyên vẹn trong Staging Area (Chờ bạn sửa và commit lại)
```

---

##### Kịch bản Hoàn tác 4: Đã push commit lỗi lên GitHub, cần hoàn tác an toàn (`git revert`)
- **Tư duy logic của Senior:** **TUYỆT ĐỐI KHÔNG DÙNG `git reset`** cho các commit đã đẩy lên GitHub! Vì reset sẽ xóa lịch sử, khiến lịch sử máy bạn lệch với đồng đội, gây hỏng repo của cả team khi họ pull về. Thay vào đó, ta dùng `git revert`: Git sẽ tạo ra một **commit mới** có nội dung đối nghịch hoàn toàn (nghĩa là dòng nào commit cũ thêm vào thì revert sẽ xóa đi, dòng nào bị xóa thì revert sẽ thêm lại).

Giả sử commit có mã hash `a6e5d4c` bị phát hiện gây lỗi trên server:
```bash
git revert a6e5d4c --no-edit
```

*Giải phẫu chi tiết lệnh Terminal:*
- `git`: Công cụ Git.
- `revert`: Lệnh con tạo ra một commit mới để đảo ngược chính xác những gì commit chỉ định đã làm.
- `a6e5d4c`: Mã hash SHA của commit lỗi cần đảo ngược.
- `--no-edit`: Chấp nhận thông điệp commit mặc định do Git tự sinh ra (ví dụ: *"Revert "feat(counter): them logic...""*) mà không cần mở trình soạn thảo văn bản.

#### 📂 MINH HỌA SỰ TIẾN HÓA CỦA TRẠNG THÁI/THƯ MỤC
```text
my-static-web/
└── 📁 .git/
    └── Cây commit:
        [a6e5d4c] feat(counter): them logic... (Commit lỗi)
           │
           v
        [b7a6f5e] fix(responsive)...
           │
           v
        [7c8d9e0] <-- [MỚI TẠO từ git revert] Commit đảo ngược code của a6e5d4c, bảo vệ lịch sử minh bạch
```

[⬆️ Quay lại Mục lục](#mục-lục)

---

### 5. Sổ tay lỗi kinh điển cần tránh

Dưới đây là 8 sai lầm thực tế (Lessons Learned) mà các kỹ sư Junior và Fresher hay mắc phải nhất tại các doanh nghiệp phần mềm, đi kèm giải pháp của Tech Lead:

#### 1. Code và commit trực tiếp lên nhánh `main`/`production`
- **Hậu quả:** Nhánh `main` là bộ mặt của sản phẩm. Khi bạn commit thẳng lên `main`, code chưa qua kiểm thử tự động (CI/CD) hay review có thể làm sập hệ thống thực tế của khách hàng.
- **Bài học khắc phục:** Thiết lập tính năng **Branch Protection Rules** trên GitHub settings. Bắt buộc mọi thay đổi muốn vào `main` phải thông qua Pull Request và có tối thiểu 1 Senior phê duyệt (Review approval).

#### 2. Đưa file nhạy cảm (`.env`, mật khẩu, Private Key) hoặc file rác (`node_modules`) lên Git
- **Hậu quả:** Đã có hàng ngàn lập trình viên bị lộ Access Key AWS/OpenAI và bị kẻ xấu lợi dụng chạy đào tiền ảo với hóa đơn hàng chục nghìn USD chỉ sau một đêm. Hơn nữa, thư mục `node_modules` nặng hàng trăm megabyte làm nghẽn băng thông và phình to repo vô ích.
- **Bài học khắc phục:** Luôn tạo file `.gitignore` ngay ở giây đầu tiên khởi tạo dự án. Khai báo ngay các mẫu:
  ```gitignore
  node_modules/
  .env
  .env.*
  *.pem
  *.log
  .DS_Store
  Thumbs.db
  ```
  Nếu lỡ commit `.env`, phải lập tức thu hồi và đổi mới toàn bộ secret key trên hệ thống cung cấp dịch vụ, sau đó dùng công cụ `git-filter-repo` hoặc BFG Repo-Cleaner để làm sạch lịch sử Git.

#### 3. Đặt thông điệp commit cẩu thả ("update", "fix bug", "sửa lỗi", "asdfg")
- **Hậu quả:** Khi dự án phát sinh lỗi ngầm sau 3 tháng, không một ai (kể cả tác giả) có thể biết được lỗi bắt đầu từ đoạn commit nào nếu đọc những thông điệp vô nghĩa. Việc tìm kiếm bằng `git bisect` trở thành cơn ác mộng.
- **Bài học khắc phục:** Bắt buộc áp dụng **Conventional Commits** (`feat:`, `fix:`, `docs:`, `refactor:`, `chore:`). Cài đặt công cụ `commitlint` kết hợp với Git Hook (`husky`) để tự động chặn các commit không đúng định dạng ngay tại máy lập trình viên.

#### 4. Sử dụng `git push --force` (`-f`) trên nhánh làm việc chung
- **Hậu quả:** Cờ `--force` sẽ ghi đè thô bạo toàn bộ lịch sử commit trên máy bạn lên máy chủ từ xa, xóa sạch toàn bộ các commit mà các đồng nghiệp khác đã đẩy lên trước đó. Đây là hành vi có thể phá hủy hàng tuần công sức của cả đội ngũ.
- **Bài học khắc phục:** Tuyệt đối cấm dùng `git push --force` trên các nhánh dùng chung (`main`, `develop`, `staging`). Nếu thực sự cần cập nhật nhánh cá nhân sau khi rebase, hãy sử dụng cờ an toàn: `git push --force-with-lease` (lệnh này sẽ tự động từ chối ghi đè nếu phát hiện có ai đó vừa đẩy commit mới lên remote).

#### 5. Chạy `git reset --hard` khi chưa sao lưu code
- **Hậu quả:** Khác với các lệnh khác của Git có cơ chế phục hồi, `git reset --hard` sẽ quét sạch toàn bộ thay đổi ở cả Working Directory lẫn Staging Area. Những đoạn code bạn vừa viết chưa kịp commit sẽ **mất vĩnh viễn không thể khôi phục**.
- **Bài học khắc phục:** Nếu chưa chắc chắn, hãy dùng `git stash` để cất tạm code vào két an toàn, hoặc dùng `git reset --soft` / `git reset --mixed` thay vì `--hard`.

#### 6. Tạo những commit "khổng lồ" (Mega Commits)
- **Hậu quả:** Một commit thay đổi tới 40 file và sửa đổi 2.000 dòng code cùng lúc. Không một Tech Lead hay đồng đội nào có đủ thời gian và sự kiên nhẫn để review một PR như vậy, và khả năng tiềm ẩn bug nghiêm trọng là cực kỳ cao.
- **Bài học khắc phục:** Áp dụng nguyên tắc **Atomic Commits** (Commit nguyên tử): Mỗi commit chỉ giải quyết duy nhất một vấn đề nhỏ độc lập. Thêm 1 component -> commit. Sửa 1 lỗi responsive -> commit. Cập nhật README -> commit.

#### 7. Không chạy `git pull` trước khi bắt đầu code
- **Hậu quả:** Bạn dành cả ngày ngồi code dựa trên phiên bản cũ từ hôm qua. Đến cuối ngày khi chuẩn bị về nhà, bạn gõ `git push` và bị từ chối, sau đó `git pull` về và đối mặt với hàng chục điểm xung đột (merge conflicts) phức tạp.
- **Bài học khắc phục:** Tạo phản xạ tự nhiên: Mỗi sáng bật máy tính lên, việc đầu tiên là mở Terminal và gõ:
  ```bash
  git checkout main
  git pull
  ```
  Sau đó mới rẽ nhánh hoặc tiếp tục công việc.

#### 8. Nhầm lẫn giữa Merge và Rebase khi làm việc nhóm
- **Hậu quả:** Tự ý chạy `git rebase` trên nhánh chung làm thay đổi toàn bộ mã băm commit (rewrite history), khiến toàn bộ thành viên khác trong nhóm không thể pull code về bình thường và lịch sử bị phân mảnh.
- **Bài học khắc phục:** Tuân thủ quy tắc vàng:
  - Chỉ dùng `git rebase` trên nhánh tính năng cá nhân cục bộ để giữ lịch sử thẳng hàng đẹp mắt trước khi tạo PR.
  - Luôn dùng `git merge` cho các nhánh dùng chung để bảo toàn tính trung thực của dòng thời gian lịch sử dự án.

---

[⬆️ Quay lại Mục lục](#mục-lục)
