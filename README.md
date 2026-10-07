# 1. Các bước thực hiện

## Bước 1: Tạo nhóm `devops-admin`

```bash
sudo groupadd devops-admin
```

Kiểm tra nhóm đã được tạo:

```bash
grep devops-admin /etc/group
```

**Kết quả mong đợi:**

```
devops-admin:x:1001:
```

---

## Bước 2: Tạo tài khoản `deployer` và thêm vào nhóm `devops-admin`

```bash
sudo adduser deployer
sudo usermod -aG devops-admin deployer
```

Kiểm tra user đã thuộc nhóm:

```bash
id deployer
```

**Kết quả mong đợi:**

```
uid=1001(deployer) gid=1001(deployer) groups=1001(deployer),1002(devops-admin)
```

---

## Bước 3: Cấu hình sudoers bằng `visudo`

Mở file cấu hình sudoers an toàn:

```bash
sudo visudo
```

Thêm dòng sau vào cuối file `/etc/sudoers`:

```sudoers
## Allow devops-admin group to manage systemd services without password
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

> **Giải thích cấu hình:**
>
> | Thành phần                   | Ý nghĩa                                                                            |
> | ---------------------------- | ---------------------------------------------------------------------------------- |
> | `%devops-admin`              | Áp dụng cho tất cả user thuộc nhóm `devops-admin` (dấu `%` biểu thị nhóm)          |
> | `ALL=`                       | Cho phép trên tất cả các host                                                      |
> | `(ALL)`                      | Có thể chạy lệnh dưới danh nghĩa bất kỳ user nào                                   |
> | `NOPASSWD:`                  | Không yêu cầu nhập mật khẩu khi thực thi                                           |
> | `/usr/bin/systemctl start *` | Chỉ cho phép lệnh `systemctl` với các tham số `start`, `stop`, `restart`, `status` |

---

# 2. Kiểm tra kết quả

## 3.1. Chuyển sang tài khoản `deployer`

```bash
su - deployer
```

## 3.2. Kiểm tra quyền sudo được phép

```bash
sudo -l
```

**Kết quả mong đợi:**

```
Matching Defaults entries for deployer on hostname:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin

User deployer may run the following commands on hostname:
    (ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

## 3.3. Chạy thử lệnh restart dịch vụ `cron`

```bash
sudo systemctl restart cron
```

**Kết quả:** Lệnh thực thi thành công, **không yêu cầu nhập mật khẩu**.

Kiểm tra trạng thái dịch vụ sau restart:

```bash
sudo systemctl status cron
```

**Kết quả mong đợi:**

```
● cron.service - Regular background program processing daemon
     Loaded: loaded (/lib/systemd/system/cron.service; enabled; vendor preset: enabled)
     Active: active (running) since ...
   Main PID: 12345 (cron)
      Tasks: 1 (limit: 4915)
     Memory: 1.2M
        CPU: 10ms
     CGroup: /system.slice/cron.service
             └─12345 /usr/sbin/cron -f
```

## 3.4. Kiểm tra lệnh KHÔNG được phép (xác nhận giới hạn quyền)

```bash
sudo apt update
```

**Kết quả mong đợi:** Bị từ chối vì `apt` không nằm trong danh sách lệnh được cho phép:

```
Sorry, user deployer is not allowed to execute '/usr/bin/apt update' as root on hostname.
```

---

# 3. Dòng cấu hình sudoers đã thêm

```
%devops-admin ALL=(ALL) NOPASSWD: /usr/bin/systemctl start *, /usr/bin/systemctl stop *, /usr/bin/systemctl restart *, /usr/bin/systemctl status *
```

---
