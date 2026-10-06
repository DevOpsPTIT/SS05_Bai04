# Báo Cáo Thực Hành: Mô Phỏng Quy Trình Hotfix & Gitflow Thực Tế

- **Bài tập**: Bài 4 - Mô phỏng quy trình Hotfix & Gitflow thực tế (Session 05 / Git Intermediate & Advanced Recovery)
- **Đường dẫn nộp bài**: `SS05/Bai04/README.md`
- **GitHub Remote Repository**: `https://github.com/DevOpsPTIT/SS05_Bai04.git`

---

## 1. Bối cảnh & Mục tiêu

### Bối cảnh
Trong môi trường sản xuất thực tế (Production), khi ứng dụng đang chạy ổn định ở phiên bản `v1.0.0` trên nhánh `main` bất ngờ phát sinh lỗ hổng bảo mật nghiêm trọng (như rò rỉ dữ liệu người dùng), việc khắc phục phải được triển khai ngay lập tức. Trong khi đó, nhánh phát triển `develop` đang tích hợp các tính năng mới dở dang cho phiên bản `v2.0` và chưa thể đóng gói phát hành ngay.

Để giải quyết bài toán này, mô hình **Gitflow** quy định việc tách riêng một nhánh bản vá khẩn cấp (**Hotfix Branch**) trực tiếp từ `main`. Sau khi sửa lỗi và kiểm thử hoàn tất, bản vá nóng sẽ được gộp đồng thời vào cả nhánh `main` (để release phiên bản `v1.0.1`) và gộp ngược lại nhánh `develop` (Back-merge) nhằm đảm bảo mã nguồn trong tương lai không bị trôi lỗi (bug regression).

### Mục tiêu
- **Thực hành mô hình Gitflow chuẩn**: Quản lý vòng đời phần mềm thông qua các nhánh chuyên biệt (`main`, `develop`, `hotfix/*`).
- **Triển khai quy trình Hotfix khẩn cấp**: Tách nhánh `hotfix/v1.0.1` từ `main`, khắc phục lỗi bảo mật và kiểm thử độc lập.
- **Tạo Release Tag**: Đóng gói nhãn phiên bản `v1.0.1` đánh dấu cột mốc release bản vá trên `main`.
- **Đồng bộ hóa Back-merge**: Gộp bản vá về `develop` để bảo toàn mã nguồn sửa lỗi cho các phiên bản tương lai.
- **Trích xuất đồ thị lịch sử Git**: Kiểm tra cây phân nhánh `git log --graph --oneline --all` minh họa luồng gộp chuẩn xác.

---

## 2. Thông Số Cấu Hình & Ràng Buộc

| Thông số / Cấu hình | Giá trị lựa chọn / Thao tác | Ghi chú / Ràng buộc |
| :--- | :--- | :--- |
| **Thư mục bài tập** | `SS05/Bai04` | Repository quản lý cục bộ |
| **Tệp tin chương trình** | `app.js` | Tệp tin mã nguồn chính của ứng dụng |
| **Phiên bản ban đầu** | `v1.0.0` (Tag: `v1.0.0`) | Mã nguồn ổn định trên nhánh `main` |
| **Nhánh phát triển** | `develop` | Chứa code tính năng dở dang cho phiên bản v2.0 |
| **Nhánh sửa lỗi khẩn cấp**| `hotfix/v1.0.1` | Tách trực tiếp từ `main` |
| **Phiên bản bản vá** | `v1.0.1` (Tag: `v1.0.1`) | Gộp vào `main` và gắn tag `v1.0.1` |
| **Cơ chế gộp back-merge**| `git merge --no-ff` | Gộp `hotfix/v1.0.1` ngược lại về `develop` |
| **Remote Repository** | `https://github.com/DevOpsPTIT/SS05_Bai04.git` | Kho lưu trữ GitHub |

---

## 3. Sơ Đồ Quy Trình Gitflow Hotfix Workflow

```
[v1.0.0] 8cf203e (Tag: v1.0.0)
   |
   +---------------------------------------+
   | (git checkout -b develop)             | (git checkout -b hotfix/v1.0.1)
   v                                       v
[develop] ceec929 (v2.0 work)          [hotfix/v1.0.1] 7607b9a (Security patch)
   |                                       |
   |                                       +------------------------+
   |                                       | (git merge to main)    |
   |                                       v                        |
   |                                    [main] 60b512c (Tag: v1.0.1)|
   |                                                                |
   +<---------------------------------------------------------------+ (Back-merge)
   | (git merge to develop)
   v
[develop] 3647bfd (Tích hợp hotfix v1.0.1 vào develop)
```

---

## 4. Các Bước Thực Hiện Chi Tiết

### Bước 1: Khởi tạo repository & Release phiên bản stable `v1.0.0`

Khởi tạo repository, chuyển nhánh mặc định thành `main`, tạo tệp `app.js` và gán tag `v1.0.0`:

```bash
git init
git branch -M main

# Tạo mã nguồn phiên bản stable v1.0.0
echo '// Application v1.0.0 - Stable Release' > app.js
echo 'console.log("App v1.0.0 running on production");' >> app.js

git add app.js
git commit -m "feat: initial release v1.0.0"

# Gán nhãn phiên bản v1.0.0
git tag -a v1.0.0 -m "Release v1.0.0"
```

---

### Bước 2: Khởi tạo nhánh `develop` và phát triển tính năng tương lai

Tách nhánh `develop` từ `main` và thực hiện commit thêm tính năng dở dang cho v2.0:

```bash
git checkout -b develop

# Thêm code tính năng v2.0 đang phát triển dở dang
echo '// Developing v2.0 feature' >> app.js
echo 'console.log("Feature v2.0 in development");' >> app.js

git add app.js
git commit -m "feat: ongoing work for v2.0"
```

---

### Bước 3: Tách nhánh `hotfix/v1.0.1` từ `main` để vá lỗi khẩn cấp

Khi phát hiện sự cố rò rỉ dữ liệu trên production (v1.0.0), quay lại `main` và tạo ngay nhánh hotfix:

```bash
git checkout main
git checkout -b hotfix/v1.0.1

# Cập nhật bản vá bảo mật trong app.js
echo '// Application v1.0.1 - Stable Release (Hotfixed)' > app.js
echo 'console.log("App v1.0.1 running on production - Security Patch Applied");' >> app.js

git add app.js
git commit -m "fix: patch critical security data leak"
```

---

### Bước 4: Gộp Hotfix vào `main` & Đánh nhãn phiên bản `v1.0.1`

Chuyển về nhánh `main`, thực hiện gộp nhánh hotfix (sử dụng cờ `--no-ff` để giữ vết merge commit) và gắn tag `v1.0.1`:

```bash
git checkout main
git merge --no-ff hotfix/v1.0.1 -m "merge: hotfix v1.0.1 into main"

# Gán nhãn phiên bản Hotfix v1.0.1
git tag -a v1.0.1 -m "Release Hotfix 1.0.1"
```

---

### Bước 5: Gộp ngược lại (Back-merge) Hotfix vào nhánh `develop`

Chuyển sang nhánh `develop` và gộp nhánh `hotfix/v1.0.1` về để đồng bộ bản vá bảo mật:

```bash
git checkout develop
git merge --no-ff hotfix/v1.0.1 -m "merge: hotfix v1.0.1 into develop"
```

> **Xử lý xung đột (Conflict Resolution)**: 
> Do cả 2 nhánh đều chỉnh sửa tệp `app.js`, xung đột xảy ra tại dòng đầu của tệp. Tiến hành giữ lại cả bản vá bảo mật v1.0.1 lẫn đoạn mã tính năng v2.0 đang phát triển:
> ```javascript
> // Application v1.0.1 - Stable Release (Hotfixed)
> console.log("App v1.0.1 running on production - Security Patch Applied");
> // Developing v2.0 feature
> console.log("Feature v2.0 in development");
> ```
> Lưu tệp, đưa vào staging và hoàn tất merge:
> ```bash
> git add app.js
> git commit -m "merge: hotfix v1.0.1 into develop"
> ```

---

### Bước 6: Xóa nhánh Hotfix cục bộ sau khi hoàn tất

Sau khi đã gộp thành công vào cả `main` và `develop`, xóa nhánh hotfix để giữ repository sạch sẽ:

```bash
git branch -d hotfix/v1.0.1
```

---

## 5. Trích Xuất Log Terminal Chứng Minh (Verification Logs)

### 1. Kiểm tra danh sách các nhánh hiện có (`git branch -a`)

```console
$ git branch -a
* develop
  main
```
> **Đánh giá**: Nhánh `hotfix/v1.0.1` đã được xóa sạch sẽ sau khi hoàn tất công việc.

---

### 2. Kiểm tra danh sách nhãn phiên bản (`git tag`)

```console
$ git tag
v1.0.0
v1.0.1
```
> **Đánh giá**: Đã xuất hiện đầy đủ 2 Tag phiên bản `v1.0.0` và `v1.0.1`.

---

### 3. Sơ đồ lịch sử gộp nhánh Gitflow chuẩn (`git log --graph --oneline --all`)

```console
$ git log --graph --oneline --all
*   3647bfd (HEAD -> develop) merge: hotfix v1.0.1 into develop
|\  
* | ceec929 feat: ongoing work for v2.0
| | * 60b512c (tag: v1.0.1, main) merge: hotfix v1.0.1 into main
| |/| 
|/|/  
| * 7607b9a fix: patch critical security data leak
|/  
* 8cf203e (tag: v1.0.0) feat: initial release v1.0.0
```

> **Đánh giá sơ đồ đồ thị Git**:
> - Tag `v1.0.0` tại gốc `8cf203e`.
> - Bản vá `7607b9a` được tách từ `8cf203e`.
> - Commit `60b512c` gộp bản vá vào `main` và gắn tag `v1.0.1`.
> - Commit `3647bfd` gộp ngược bản vá vào `develop`, tích hợp song song với commit tính năng `ceec929`.

---

### 4. Kiểm tra nội dung mã nguồn tệp `app.js` trên `develop`

```console
$ Get-Content app.js
// Application v1.0.1 - Stable Release (Hotfixed)
console.log("App v1.0.1 running on production - Security Patch Applied");
// Developing v2.0 feature
console.log("Feature v2.0 in development");
```
> **Đánh giá**: Nhánh `develop` đã tiếp nhận bản vá bảo mật của v1.0.1 mà không làm mất mã nguồn v2.0 đang phát triển.

---

## 6. Các Quy Tắc Vàng Trong Quy Trình Hotfix Workflow

1. **Luôn tách Hotfix từ `main`**: Nhánh Hotfix **bắt buộc phải tách từ `main`** (nơi lưu trữ mã nguồn production hiện tại), tuyệt đối không tách từ `develop` vì `develop` chứa code chưa kiểm thử xong.
2. **Bắt buộc Back-merge về `develop`**: Sau khi gộp vào `main`, phải ngay lập tức gộp về `develop` (hoặc qua nhánh `release` nếu đang trong chu kỳ release) để tránh trôi lỗi khi ra mắt phiên bản tiếp theo.
3. **Đánh nhãn (Tagging) đúng chuẩn Semantic Versioning**: Tăng chỉ số patch version (ví dụ: `v1.0.0` -> `v1.0.1`).
4. **Sử dụng `--no-ff` khi Merge**: Giữ lại các nút Merge Commit giúp đồ thị lịch sử minh bạch và dễ truy vết nguyên nhân bản vá sau này.

---

## 7. Kết Luận

- Triển khai thành công quy trình **Hotfix Workflow** chuẩn theo mô hình Gitflow.
- Khắc phục sự cố rò rỉ dữ liệu khẩn cấp trên production thông qua nhánh `hotfix/v1.0.1`.
- Đóng gói thành công nhãn phiên bản `v1.0.1` trên nhánh `main`.
- Thực hiện Back-merge chuẩn xác về nhánh `develop`, đảm bảo mã nguồn v2.0 kế thừa đầy đủ bản vá bảo mật.
- Đã đẩy kho lưu trữ hoàn chỉnh kèm toàn bộ nhãn Tag lên GitHub remote `https://github.com/DevOpsPTIT/SS05_Bai04.git`.
