# Bài 2: Quản lý nhánh và Giải quyết xung đột (Merge Conflict)

> File [`README.md`](README.md) trong thư mục này là **file sau khi đã giải quyết xung đột**. File này (`BAO-CAO.md`) là báo cáo các bước thực hiện.

## Mục tiêu
- Tạo và chuyển đổi giữa các nhánh cục bộ.
- Cố ý tạo xung đột trên cùng một dòng của `README.md`, rồi xử lý thủ công các ký hiệu `<<<<<<<`, `=======`, `>>>>>>>`.
- Hoàn thành merge commit và hiểu cơ chế **3-Way Merge**.

## Môi trường
- Windows 10, Git Bash (MINGW64)
- Thư mục thực hành: `/d/rikkei/IT209/Ss4/Ls2`

---

## Các bước thực hiện

### Bước 1 – Khởi tạo repo và commit gốc trên `main`

```bash
git init -b main
git config --local user.name "thang-260704"
git config --local user.email "phanhtran0501@gmail.com"
printf "# Project Demo\n\nVersion: 1.0\n" > README.md
git add README.md
git commit -m "Initial commit: add README with version 1.0"
```

### Bước 2 – Tạo nhánh `feature-update` và sửa dòng Version

```bash
git switch -c feature-update
printf "# Project Demo\n\nVersion: 2.0 (feature-update)\n" > README.md
git commit -am "Feature: bump version to 2.0"
```

### Bước 3 – Quay về `main` và sửa **cùng dòng đó**

```bash
git switch main
printf "# Project Demo\n\nVersion: 1.1 (hotfix on main)\n" > README.md
git commit -am "Hotfix: bump version to 1.1 on main"
```

Lúc này hai nhánh đã **rẽ nhánh** (diverged). Mỗi nhánh có một commit riêng, và cả hai đều sửa dòng 3 của `README.md`.

### Bước 4 – Merge và gặp xung đột

```bash
git merge feature-update
```

```text
Auto-merging README.md
CONFLICT (content): Merge conflict in README.md
Automatic merge failed; fix conflicts and then commit the result.
```

Nội dung `README.md` lúc xung đột:

```text
# Project Demo

<<<<<<< HEAD
Version: 1.1 (hotfix on main)
=======
Version: 2.0 (feature-update)
>>>>>>> feature-update
```

| Ký hiệu | Ý nghĩa |
|---|---|
| `<<<<<<< HEAD` | Bắt đầu phần nội dung của nhánh hiện tại (`main`) |
| `=======` | Ranh giới giữa hai phiên bản |
| `>>>>>>> feature-update` | Kết thúc phần nội dung của nhánh được gộp vào |

![Xung đột khi merge](images/01-conflict.png)

### Bước 5 – Giải quyết xung đột thủ công

Mở file bằng trình soạn thảo:

```bash
notepad README.md
```

Xoá cả **3 dòng ký hiệu** (`<<<<<<<`, `=======`, `>>>>>>>`), rồi gộp nội dung của cả hai nhánh thành một dòng thống nhất. Lý do: nhánh feature nâng lên phiên bản 2.0, còn nhánh main có bản vá (hotfix), nên cần giữ cả hai thay đổi.

```text
# Project Demo

Version: 2.0 (feature-update) + hotfix (main)
```

### Bước 6 – Đánh dấu đã xử lý và tạo merge commit

```bash
git add README.md
git commit -m "Merge branch 'feature-update' into main: resolve version conflict"
```

`git add` báo cho Git biết xung đột trong file đã được xử lý. Sau đó `git commit` tạo ra **merge commit** có **2 commit cha**.

### Bước 7 – Kiểm tra đồ thị nhánh

```bash
git log --graph --oneline --all
```

```text
*   c073ac9 (HEAD -> main) Merge branch 'feature-update' into main: resolve version conflict
|\
| * 1b6fadf (feature-update) Feature: bump version to 2.0
* | ad29805 Hotfix: bump version to 1.1 on main
|/
* 16e3290 Initial commit: add README with version 1.0
```

![git log --graph](images/02-git-log-graph.png)

---

## Cơ chế 3-Way Merge

Khi merge `feature-update` vào `main`, Git so sánh **3 phiên bản**:

| Vai trò | Commit | Dòng Version |
|---|---|---|
| **Base** (tổ tiên chung) | Initial commit | `1.0` |
| **Ours** (`main`, HEAD) | Hotfix | `1.1 (hotfix on main)` |
| **Theirs** (`feature-update`) | Feature | `2.0 (feature-update)` |

- Nếu chỉ **một** bên sửa so với Base, Git tự lấy thay đổi của bên đó.
- Ở đây **cả hai** bên cùng sửa **cùng một dòng** so với Base, nên Git không tự quyết định được và báo **CONFLICT**. Người dùng phải chọn hoặc gộp nội dung thủ công.
- Merge commit `c073ac9` có **2 commit cha**: `ad29805` (Hotfix, phía main) và `1b6fadf` (Feature, phía feature-update). Tổ tiên chung của hai nhánh là `16e3290`. Đồ thị `git log --graph` thể hiện điều này qua nhánh tách ra (`|\`) rồi nhập lại (`|/`).

## Đối chiếu yêu cầu

| Yêu cầu | Kết quả |
|---|---|
| Tạo nhánh `feature-update` tách ra từ `main` | ✅ |
| Cả hai nhánh sửa cùng một dòng trong `README.md`, gây xung đột | ✅ |
| Xử lý thủ công các ký hiệu `<<<<<<<`, `=======`, `>>>>>>>` | ✅ |
| Có merge commit với 2 commit cha | ✅ |
| Có ảnh `git log --graph --oneline` | ✅ |
