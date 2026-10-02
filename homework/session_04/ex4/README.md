# Bài 4: Quản lý tệp tin bỏ qua (.gitignore) và sửa lịch sử (Amend)

Môn DevOps, Session 04. Toàn bộ lệnh trong báo cáo này được chạy thật bằng Git 2.49.0.windows.1 trên Windows 11 (Git Bash). Output được chép từ `terminal_log.txt` (cùng thư mục), không chỉnh sửa hash hay ngày giờ; một số đoạn được rút gọn phần đầu/cuối, bản đầy đủ nằm trong `terminal_log.txt`.

## 1. Mục tiêu

- Cấu hình `.gitignore` để Git tự động bỏ qua file nhạy cảm và file rác hệ thống.
- Gỡ file đã lỡ commit ra khỏi sự theo dõi của Git (khỏi index) bằng `git rm --cached` mà không làm mất file vật lý trên đĩa.
- Sửa thông điệp (và nội dung) của commit gần nhất bằng `git commit --amend`.

Bối cảnh: học viên vô tình commit file `credentials.txt` chứa thông tin bảo mật. Cần (1) gỡ file khỏi sự theo dõi nhưng giữ file ở máy, (2) cấu hình để Git bỏ qua file này về sau, (3) sửa lại thông điệp commit gần nhất cho sạch sẽ.

Ràng buộc: không dùng lệnh xóa vật lý file của hệ điều hành (`rm`, `del`, ...), phải giữ lại tệp ở thư mục làm việc.

Lưu ý về dữ liệu: toàn bộ nội dung trong `credentials.txt` ở bài này là giá trị GIẢ (`DB_USER=demo_user`, `DB_PASSWORD=example-not-real`, `API_KEY=dummy-key-for-exercise`). File `credentials.txt` thật KHÔNG được đưa vào thư mục nộp bài.

Thư mục nộp bài `homework/session_04/ex4/` gồm: `.gitignore` (bản cuối trong repo demo), `README.md` (báo cáo này), `terminal_log.txt` (toàn bộ lệnh và output thật).

## 2. Môi trường và cách làm

- Repo demo: `/d/Rikkei/DevOps/Session4/_demo/bai4/gitignore-amend-demo` (tạo riêng, không dính repo nào khác).
- Cấu hình local (không dùng `--global`): `user.name = Lê Toàn`, `user.email = trinhhanh261293@gmail.com`, `core.autocrlf = false`.
- Dùng `-m` cho mọi lệnh commit và amend để không mở trình soạn thảo.

```
$ git config --local --list | grep -E '^(user|core.autocrlf)'
core.autocrlf=false
user.name=Lê Toàn
user.email=trinhhanh261293@gmail.com

$ git commit -m 'feat: khoi tao du an voi README va app.py'
[main (root-commit) 8e300d2] feat: khoi tao du an voi README va app.py
 2 files changed, 9 insertions(+)
 create mode 100644 README.md
 create mode 100644 app.py
```

## 3. Tái hiện sự cố

Tạo `credentials.txt` (giá trị giả) và một file rác `debug.log`, sau đó `git add .` rồi commit với thông điệp cẩu thả `update`:

```
$ git status --short
?? credentials.txt
?? debug.log

$ git add .

$ git commit -m 'update'
[main 9585a29] update
 2 files changed, 5 insertions(+)
 create mode 100644 credentials.txt
 create mode 100644 debug.log

$ git log --oneline
9585a29 update
8e300d2 feat: khoi tao du an voi README va app.py

$ git ls-files
README.md
app.py
credentials.txt
debug.log
```

Sự cố đã xảy ra: `credentials.txt` và `debug.log` đang nằm trong danh sách file được Git theo dõi (`git ls-files`), và đã nằm trong commit `9585a29`. Thông điệp `update` cũng không mô tả gì về thay đổi.

## 4. Gỡ file khỏi theo dõi nhưng giữ lại trên đĩa: git rm --cached

```
$ git rm --cached credentials.txt
rm 'credentials.txt'

$ git rm --cached debug.log
rm 'debug.log'

$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    credentials.txt
	deleted:    debug.log

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	credentials.txt
	debug.log
```

Giải thích output của `git status`:

- Mục `Changes to be committed ... deleted:` nghĩa là trong index (vùng staged), hai file đã bị gỡ. Khi commit, snapshot mới sẽ không còn chứa chúng.
- Đồng thời hai file xuất hiện ở `Untracked files`, vì chúng vẫn còn trong thư mục làm việc nhưng không còn trong index. Đây chính là dấu hiệu `--cached` đã hoạt động đúng: Git quên file, nhưng đĩa vẫn giữ file.

Chứng minh file vẫn còn trên đĩa và nội dung nguyên vẹn:

```
$ ls -la
total 16
drwxr-xr-x 1 gigabyte 197610  0 Oct  2 12:47 .
drwxr-xr-x 1 gigabyte 197610  0 Oct  2 12:47 ..
drwxr-xr-x 1 gigabyte 197610  0 Oct  2 12:47 .git
-rw-r--r-- 1 gigabyte 197610 50 Oct  2 12:47 README.md
-rw-r--r-- 1 gigabyte 197610 81 Oct  2 12:47 app.py
-rw-r--r-- 1 gigabyte 197610 78 Oct  2 12:47 credentials.txt
-rw-r--r-- 1 gigabyte 197610 80 Oct  2 12:47 debug.log

$ cat credentials.txt | head -2
DB_USER=demo_user
DB_PASSWORD=example-not-real
```

### 4.1. So sánh git rm --cached, git rm và rm

| Lệnh | Xóa khỏi index (Git thôi theo dõi) | Xóa khỏi thư mục làm việc (đĩa) | Ghi nhận vào lịch sử khi commit | Dùng khi nào |
|---|---|---|---|---|
| `git rm --cached <file>` | Có | Không, file vẫn còn | Có, commit tiếp theo ghi "file bị xóa khỏi repo" | Lỡ commit file nhạy cảm/rác, muốn giữ file ở máy (đúng bài này) |
| `git rm <file>` | Có | Có, file bị xóa | Có | Muốn xóa hẳn file khỏi dự án và khỏi đĩa |
| `rm <file>` (lệnh hệ điều hành) | Không, index chưa đổi | Có | Chưa: `git status` báo `deleted` ở dạng chưa staged, phải `git add`/`git rm` tiếp | Xóa file ngoài Git; không phù hợp với ràng buộc của bài |

Vì đề bài cấm xóa vật lý, chỉ `git rm --cached` thỏa ràng buộc: nó tác động lên index chứ không đụng tới working tree.

## 5. Tạo .gitignore

### 5.1. Nội dung file

File `.gitignore` đầy đủ nằm cùng thư mục với báo cáo này, chia 5 nhóm có comment tiếng Việt:

| Nhóm | Pattern | Mục đích |
|---|---|---|
| 1. File bí mật | `credentials.txt`, `.env`, `.env.*`, `*.pem`, `*.key`, `id_rsa*`, `id_ed25519*`, ngoại lệ `!*.pub` | Không bao giờ commit thông tin nhạy cảm; vẫn cho phép khóa công khai `.pub` |
| 2. Log và file tạm | `*.log`, `*.tmp`, `tmp/` | Bỏ file sinh ra khi chạy chương trình |
| 3. Phụ thuộc và build | `node_modules/`, `__pycache__/`, `*.pyc`, `dist/`, `build/`, `venv/` | Bỏ thư mục sinh tự động |
| 4. Rác hệ điều hành và IDE | `.DS_Store`, `Thumbs.db`, `desktop.ini`, `.vscode/`, `.idea/` | Bỏ file riêng của máy/IDE |
| 5. Sao lưu của trình soạn thảo | `*.bak`, `*.swp` | Thêm ở bước trình diễn amend nội dung (mục 7) |

### 5.2. Kiểm tra hiệu lực

```
$ git status
On branch main
Changes to be committed:
  (use "git restore --staged <file>..." to unstage)
	deleted:    credentials.txt
	deleted:    debug.log

Untracked files:
  (use "git add <file>..." to include in what will be committed)
	.gitignore

$ git check-ignore -v credentials.txt debug.log
.gitignore:6:credentials.txt	credentials.txt
.gitignore:17:*.log	debug.log

$ git status --ignored --short
D  credentials.txt
D  debug.log
?? .gitignore
!! credentials.txt
!! debug.log
```

Phân tích:

- Trong `git status`, `credentials.txt` và `debug.log` đã biến mất khỏi mục `Untracked files`; chỉ còn `.gitignore` là file chưa theo dõi.
- `git check-ignore -v` cho biết chính xác quy tắc nào chặn: định dạng `nguồn:số dòng:pattern<TAB>đường dẫn`. Ví dụ `credentials.txt` bị chặn bởi dòng 6, `debug.log` bị chặn bởi dòng 17 (pattern `*.log`).
- `git status --ignored --short`: ký hiệu `!!` nghĩa là file bị bỏ qua (ignored); `D ` (cột đầu) là deleted đã staged; `??` là untracked.

### 5.3. Cú pháp và thứ tự ưu tiên của .gitignore

| Cú pháp | Ý nghĩa | Ví dụ trong bài |
|---|---|---|
| `# ...` | Dòng comment | `# --- Nhóm 2: Log và file tạm ---` |
| `tên` hoặc `*.ext` | Khớp ở mọi cấp thư mục bên dưới vị trí file `.gitignore` | `credentials.txt`, `*.log` |
| `*` | Khớp mọi ký tự trừ dấu `/` | `id_rsa*` khớp `id_rsa`, `id_rsa.pub` |
| `thư_mục/` | Dấu `/` ở cuối: chỉ khớp THƯ MỤC (và toàn bộ nội dung bên trong) | `tmp/`, `node_modules/` |
| `/tên` | Dấu `/` ở đầu: neo vào thư mục chứa `.gitignore`, không khớp ở cấp sâu hơn | `/build` chỉ chặn `build` ở gốc, `build/` chặn mọi nơi |
| `**/tên` | `**` ở đầu: khớp ở mọi cấp thư mục | `**/logs` |
| `tên/**` | `**` ở cuối: mọi thứ bên trong thư mục | `logs/**` |
| `a/**/b` | `**` ở giữa: khớp 0 hoặc nhiều cấp thư mục trung gian | `a/b`, `a/x/b`, `a/x/y/b` |
| `!pattern` | Phủ định: bỏ chặn (đưa lại vào theo dõi) file mà pattern trước đó đã chặn | `!*.pub` |

Thứ tự ưu tiên (quan trọng):

1. Trong cùng một file `.gitignore`, pattern ở dòng SAU thắng pattern ở dòng TRƯỚC. Vì vậy `!*.pub` phải nằm sau `id_rsa*`: `id_rsa.pub` bị `id_rsa*` chặn, sau đó `!*.pub` bỏ chặn lại. Nếu đảo thứ tự, ngoại lệ không có tác dụng.
2. Nhiều file `.gitignore` (ở thư mục con) ưu tiên cao hơn file ở thư mục cha; file ở thư mục con chỉ ảnh hưởng cây thư mục của nó.
3. Thứ tự các nguồn quy tắc, từ ưu tiên cao xuống thấp: tham số dòng lệnh, `.gitignore` trong thư mục (gần file nhất thắng), `.git/info/exclude` (riêng cho một repo, không commit), và `core.excludesFile` (file ignore toàn cục của người dùng).
4. Giới hạn của phủ định: nếu một thư mục cha đã bị chặn (ví dụ `tmp/`), Git không đi vào thư mục đó nên không thể dùng `!tmp/giu_lai.txt` để cứu một file bên trong. Phải chặn nội dung (`tmp/*`) thay vì chặn cả thư mục thì mới phủ định được.

### 5.4. Vì sao .gitignore không có tác dụng với file đã được theo dõi

`.gitignore` chỉ áp dụng cho file UNTRACKED (chưa nằm trong index). Khi file đã nằm trong index, Git tiếp tục theo dõi mọi thay đổi của nó, bất kể có pattern chặn hay không. Đó là lý do chỉ thêm `credentials.txt` vào `.gitignore` là CHƯA đủ khi file đã được commit: phải gỡ khỏi index trước (`git rm --cached`), `.gitignore` mới phát huy tác dụng để file không bị `git add .` kéo vào lần nữa.

Thứ tự đúng của bài này: (1) `git rm --cached` để gỡ khỏi index, (2) ghi pattern vào `.gitignore` để ngăn file bị add lại, (3) commit cả hai thay đổi.

## 6. Commit và sửa thông điệp bằng git commit --amend

### 6.1. Commit với thông điệp chưa tốt

```
$ git add .gitignore

$ git status --short
A  .gitignore
D  credentials.txt
D  debug.log

$ git commit -m 'fix'
[main 8c870df] fix
 3 files changed, 34 insertions(+), 5 deletions(-)
 create mode 100644 .gitignore
 delete mode 100644 credentials.txt
 delete mode 100644 debug.log

$ git log -n 1
commit 8c870dffb84843061abc3d849cfa3fb1ab9204b9
Author: Lê Toàn <trinhhanh261293@gmail.com>
Date:   Fri Oct 2 12:47:46 2026 +0700

    fix
```

Commit này gồm cả việc thêm `.gitignore` lẫn hai thay đổi `delete` (do `git rm --cached` đã staged từ trước). Hash lúc này: `8c870df`. Thông điệp `fix` quá mơ hồ.

### 6.2. Sửa bằng --amend

```
$ git commit --amend -m 'chore: go credentials.txt khoi Git va them .gitignore'
[main 44bd832] chore: go credentials.txt khoi Git va them .gitignore
 Date: Fri Oct 2 12:47:46 2026 +0700
 3 files changed, 34 insertions(+), 5 deletions(-)
 create mode 100644 .gitignore
 delete mode 100644 credentials.txt
 delete mode 100644 debug.log

$ git log -n 1
commit 44bd832caed82fd4c143848663006d2c5a7cd635
Author: Lê Toàn <trinhhanh261293@gmail.com>
Date:   Fri Oct 2 12:47:46 2026 +0700

    chore: go credentials.txt khoi Git va them .gitignore

$ git log --oneline
44bd832 chore: go credentials.txt khoi Git va them .gitignore
9585a29 update
8e300d2 feat: khoi tao du an voi README va app.py

$ git show --stat HEAD   (rút gọn phần đầu, bản đầy đủ trong terminal_log.txt)
 .gitignore      | 34 ++++++++++++++++++++++++++++++++++
 credentials.txt |  3 ---
 debug.log       |  2 --
 3 files changed, 34 insertions(+), 5 deletions(-)
```

So sánh trước và sau:

| | Trước amend | Sau amend |
|---|---|---|
| Hash | `8c870dffb84843061abc3d849cfa3fb1ab9204b9` (`8c870df`) | `44bd832caed82fd4c143848663006d2c5a7cd635` (`44bd832`) |
| Thông điệp | `fix` | `chore: go credentials.txt khoi Git va them .gitignore` |
| Nội dung (snapshot) | 3 file đổi | Giống hệt: 3 file đổi, 34 dòng thêm, 5 dòng xóa |
| Số commit trong `git log --oneline` | 3 | 3 (không tăng, commit `fix` đã bị thay thế) |

### 6.3. --amend thực chất làm gì

`git commit --amend` KHÔNG sửa commit cũ tại chỗ. Nó tạo ra một commit MỚI có cùng commit cha với commit cũ, rồi dời con trỏ nhánh `main` sang commit mới. Hash đổi vì nó được tính từ toàn bộ nội dung commit (thông điệp, thời gian, cây thư mục, commit cha). Commit cũ không bị xóa ngay mà trở thành commit "mồ côi" (không còn nhánh nào trỏ tới), vẫn còn trong `reflog` cho đến khi bị `git gc` dọn.

```
Truoc amend:   8e300d2 <- 9585a29 <- 8c870df [main]   (fix)

Sau amend:     8e300d2 <- 9585a29 <- 44bd832 [main]   (chore: ...)
                                \
                                 8c870df  (mo coi, chi con trong reflog)
```

Các cách dùng chính:

- `git commit --amend -m "thông điệp mới"`: chỉ đổi thông điệp (cũng đổi cả nội dung nếu index đang có thay đổi staged).
- `git add <file>` rồi `git commit --amend --no-edit`: gộp thêm thay đổi vào commit gần nhất, giữ nguyên thông điệp.

### 6.4. Khi nào KHÔNG được amend

- Commit đã được push lên nhánh chung mà người khác có thể đã pull về. Amend làm lịch sử cục bộ rẽ nhánh khỏi lịch sử trên remote (hai commit khác hash cho cùng một thay đổi). Muốn đẩy lên phải ép ghi đè (`git push --force`), gây rắc rối cho đồng đội.
- Nếu bắt buộc phải sửa commit đã push trên nhánh cá nhân (chưa ai dùng), hãy dùng `git push --force-with-lease`, an toàn hơn `--force` vì nó từ chối ghi đè nếu remote đã có commit mới mà bạn chưa thấy.
- Chỉ amend được commit GẦN NHẤT (HEAD). Sửa commit cũ hơn cần `git rebase -i` (có cùng các hạn chế về lịch sử chung).
- Quy tắc ngón tay cái: chưa push thì amend thoải mái; đã push lên nhánh chung thì tạo commit mới (`git commit`, hoặc `git revert`) thay vì viết lại lịch sử.

## 7. Trình diễn thêm: amend nội dung (quên một dòng trong .gitignore)

Giả sử sau khi commit mới nhớ ra cần chặn thêm file sao lưu `*.bak` và `*.swp`. Sửa file rồi gộp vào commit gần nhất bằng `--amend --no-edit`:

```
$ printf '\n# --- Nhóm 5: File sao lưu và file tạm của trình soạn thảo ---\n*.bak\n*.swp\n' >> .gitignore

$ git status --short
 M .gitignore

$ git add .gitignore

$ git commit --amend --no-edit
[main 4fe6a10] chore: go credentials.txt khoi Git va them .gitignore
 Date: Fri Oct 2 12:47:46 2026 +0700
 3 files changed, 38 insertions(+), 5 deletions(-)
 create mode 100644 .gitignore
 delete mode 100644 credentials.txt
 delete mode 100644 debug.log

$ git log -n 1 --stat
commit 4fe6a10a8e2dcf8c865779bb9f51e0899a86560f
Author: Lê Toàn <trinhhanh261293@gmail.com>
Date:   Fri Oct 2 12:47:46 2026 +0700

    chore: go credentials.txt khoi Git va them .gitignore

 .gitignore      | 38 ++++++++++++++++++++++++++++++++++++++
 credentials.txt |  3 ---
 debug.log       |  2 --
 3 files changed, 38 insertions(+), 5 deletions(-)
```

Nhận xét: thông điệp giữ nguyên, nhưng số dòng của `.gitignore` tăng từ 34 lên 38 và hash đổi từ `44bd832` sang `4fe6a10`. `git reflog` ghi lại toàn bộ các bước:

```
$ git reflog -n 5
4fe6a10 HEAD@{0}: commit (amend): chore: go credentials.txt khoi Git va them .gitignore
44bd832 HEAD@{1}: commit (amend): chore: go credentials.txt khoi Git va them .gitignore
8c870df HEAD@{2}: commit: fix
9585a29 HEAD@{3}: commit: update
8e300d2 HEAD@{4}: commit (initial): feat: khoi tao du an voi README va app.py
```

Dòng `commit (amend)` xuất hiện hai lần, tương ứng hai lần amend. Reflog là "lưới an toàn": nếu amend nhầm, có thể quay về bằng `git reset --hard HEAD@{n}` (với `n` là vị trí cần về).

## 8. CẢNH BÁO: file vẫn nằm trong lịch sử, phải coi thông tin đã bị lộ

`git rm --cached` và `--amend` chỉ xử lý commit MỚI nhất. Commit cũ `9585a29 update` vẫn chứa `credentials.txt`, và nội dung vẫn đọc được:

```
$ git log --oneline --all -- credentials.txt
4fe6a10 chore: go credentials.txt khoi Git va them .gitignore
9585a29 update

$ git show 9585a29:credentials.txt
DB_USER=demo_user
DB_PASSWORD=example-not-real
API_KEY=dummy-key-for-exercise

$ git cat-file -t 8c870df
commit
```

Lệnh `git log --all -- credentials.txt` cho thấy lịch sử vẫn ghi nhận file ở commit `9585a29` (và ghi nhận việc xóa ở `4fe6a10`), còn `git show 9585a29:credentials.txt` đọc ra nguyên văn nội dung. Lệnh `git cat-file -t 8c870df` cho thấy ngay cả commit `fix` bị amend thay thế vẫn tồn tại trong kho đối tượng (chỉ còn trong reflog, sẽ bị `git gc` dọn sau một thời gian).

Hệ quả và cách xử lý nếu đây là thông tin THẬT:

1. Coi như thông tin ĐÃ BỊ LỘ. Việc xóa file khỏi commit mới nhất không làm mật khẩu hay khóa trở nên an toàn lại, đặc biệt nếu đã từng push, fork, hoặc có bản sao nào khác.
2. Việc đầu tiên và quan trọng nhất: ĐỔI (rotate) mật khẩu, thu hồi và cấp lại API key, token, khóa SSH/PEM. Đây là cách duy nhất đảm bảo giá trị cũ vô dụng.
3. Nếu repo CHƯA push: có thể viết lại lịch sử cục bộ (ví dụ `git rebase -i`, hoặc tạo lại repo) để loại file khỏi commit `9585a29`, rồi dọn reflog và chạy `git gc`.
4. Nếu repo ĐÃ push lên remote (GitHub, GitLab...): cần xóa file khỏi TOÀN BỘ lịch sử bằng công cụ chuyên dụng như `git filter-repo` (khuyến nghị chính thức thay cho `filter-branch`) hoặc BFG Repo-Cleaner, sau đó `git push --force-with-lease` (hoặc `--force` với toàn bộ nhánh và tag), và yêu cầu mọi người clone lại. Ví dụ hướng đi với `git filter-repo`: `git filter-repo --path credentials.txt --invert-paths`. Ngoài ra nên liên hệ nhà cung cấp (ví dụ GitHub Support) để xóa các bản cache/PR cũ còn giữ commit.
5. Phòng ngừa: tạo `.gitignore` NGAY từ commit đầu tiên, dùng `git status` trước khi `git add`, hạn chế `git add .` bừa bãi, cân nhắc công cụ quét bí mật (pre-commit hook, secret scanning) và lưu bí mật bằng biến môi trường hay secret manager thay vì file trong repo.

## 9. Kiểm tra cuối đúng theo đề bài

```
$ git status
On branch main
nothing to commit, working tree clean

$ git log -n 1
commit 4fe6a10a8e2dcf8c865779bb9f51e0899a86560f
Author: Lê Toàn <trinhhanh261293@gmail.com>
Date:   Fri Oct 2 12:47:46 2026 +0700

    chore: go credentials.txt khoi Git va them .gitignore

$ git ls-files
.gitignore
README.md
app.py

$ ls -la
total 20
drwxr-xr-x 1 gigabyte 197610   0 Oct  2 12:47 .
drwxr-xr-x 1 gigabyte 197610   0 Oct  2 12:47 ..
drwxr-xr-x 1 gigabyte 197610   0 Oct  2 12:47 .git
-rw-r--r-- 1 gigabyte 197610 749 Oct  2 12:47 .gitignore
-rw-r--r-- 1 gigabyte 197610  50 Oct  2 12:47 README.md
-rw-r--r-- 1 gigabyte 197610  81 Oct  2 12:47 app.py
-rw-r--r-- 1 gigabyte 197610  78 Oct  2 12:47 credentials.txt
-rw-r--r-- 1 gigabyte 197610  80 Oct  2 12:47 debug.log
```

Kết quả: working tree clean, `credentials.txt` không có trong `git ls-files` (không Staged, Modified hay Untracked), file vẫn còn trên đĩa (78 byte), `git log -n 1` hiển thị thông điệp đã sửa. Pattern `*.log` không khớp `terminal_log.txt` (đuôi `.txt`), nên `.gitignore` không chặn file nộp.

## 10. Bảng đối chiếu yêu cầu đề bài

| Yêu cầu của đề | Cách thực hiện | Bằng chứng | Kết quả |
|---|---|---|---|
| Cấu hình `.gitignore` bỏ qua file nhạy cảm và file rác | Tạo `.gitignore` 5 nhóm có comment tiếng Việt | Mục 5.1, `git check-ignore -v` dòng 6 và 17 | Đạt |
| Gỡ file đã commit khỏi cache theo dõi an toàn | `git rm --cached credentials.txt` (và `debug.log`) | Mục 4, output `rm 'credentials.txt'` | Đạt |
| Giữ lại file vật lý, không dùng lệnh xóa của hệ điều hành | Chỉ dùng `git rm --cached`, không dùng `rm`/`del`/`git rm` | `ls -la` (mục 4 và mục 9) vẫn còn `credentials.txt` | Đạt |
| Sửa thông điệp commit gần nhất bằng amend | `git commit --amend -m "chore: go credentials.txt khoi Git va them .gitignore"` | `git log -n 1`: hash `8c870df` -> `44bd832`, thông điệp `fix` -> thông điệp mới | Đạt |
| `credentials.txt` không còn trong theo dõi (không Staged/Modified) | Index đã gỡ, `.gitignore` chặn | `git status` clean, `git ls-files` không có file | Đạt |
| `git log -n 1` hiển thị thông điệp đã sửa | Xem mục 9 | Hash `4fe6a10`, thông điệp `chore: go credentials.txt khoi Git va them .gitignore` | Đạt |
| Nộp `.gitignore` | `homework/session_04/ex4/.gitignore` | Giống hệt file trong repo demo | Đạt |
| README giải thích `git rm --cached` và kết quả `git log -n 1` chứng minh `--amend` | Mục 4 và mục 6 của báo cáo này | Output thật | Đạt |

Lưu ý minh bạch: hash `4fe6a10` ở bước kiểm tra cuối khác hash `44bd832` của lần amend thông điệp, vì ở mục 7 có thêm một lần amend nội dung (`--no-edit`) để trình diễn. Chứng minh việc dùng `--amend` sửa thông điệp nằm ở mục 6.2 (hash `8c870df` đổi thành `44bd832`).

## 11. Kết luận

- `git rm --cached` là lệnh đúng để gỡ file đã commit khỏi sự theo dõi mà vẫn giữ file trên đĩa; khác với `git rm` (xóa cả file) và `rm` (không động tới index).
- `.gitignore` chỉ có tác dụng với file chưa được theo dõi. Vì thế quy trình đúng là gỡ khỏi index rồi mới thêm pattern vào `.gitignore`, và nên có `.gitignore` ngay từ đầu dự án.
- `git commit --amend` tạo commit mới thay thế commit gần nhất (hash đổi), dùng để sửa thông điệp hoặc gộp thêm thay đổi. Chỉ dùng khi commit chưa được chia sẻ; nếu bắt buộc phải ghi đè nhánh đã push thì dùng `--force-with-lease`.
- Điều quan trọng nhất: gỡ file khỏi commit mới không xóa nó khỏi lịch sử. Với bí mật thật, phải đổi mật khẩu/khóa ngay, và nếu đã push thì làm sạch lịch sử bằng `git filter-repo` hoặc BFG.
