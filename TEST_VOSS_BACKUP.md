# Test Extreme VOSS Backup - Hướng dẫn từng bước

## Bước 1: Chuẩn bị Inventory
Thêm vào `inventory/host_file.ini`:
```ini
[extreme_voss]
voss-switch-01 ansible_host=10.1.1.10
voss-switch-02 ansible_host=10.1.1.11
```

## Bước 2: Cấu hình Group Variables
Tạo file `inventory/group_vars/extreme_voss.yml`:
```yaml
---
ansible_connection: network_cli
ansible_network_os: community.network.voss
ansible_user: admin
ansible_password: your_password
ansible_become: yes
ansible_become_method: enable
```

## Bước 3: Cài đặt Collection
```bash
ansible-galaxy collection install community.network
```

## Bước 4: Test kết nối cơ bản (QUAN TRỌNG!)
```bash
# Test ping
ansible extreme_voss -i inventory/host_file.ini -m ping

# Kết quả mong đợi:
# voss-switch-01 | SUCCESS => {
#     "changed": false,
#     "ping": "pong"
# }
```

## Bước 5: Chạy file test (test_extreme_voss.yml)
```bash
cd skylight
ansible-playbook test_extreme_voss.yml -i inventory/host_file.ini -v

# Option: Thêm -v, -vv, -vvv để xem chi tiết hơn
ansible-playbook test_extreme_voss.yml -i inventory/host_file.ini -vvv
```

### Kết quả mong đợi từ test:
- ✅ Task 1: Kết nối SSH thành công
- ✅ Task 2: Lấy được system info
- ✅ Task 3: Đọc được running-config
- ✅ Task 4: Hiển thị version
- ✅ Task 5: Kiểm tra thư mục backup

## Bước 6: Test Backup ở chế độ CHECK (không thay đổi gì)
```bash
# Dry run - chỉ kiểm tra, không thực thi
ansible-playbook backup_extreme_voss.yml -i inventory/host_file.ini --check -v

# Kết quả: Sẽ hiển thị những gì sẽ được thực hiện
```

## Bước 7: Chạy Backup thật
```bash
# Chạy backup thực tế
ansible-playbook backup_extreme_voss.yml -i inventory/host_file.ini -v

# Kiểm tra kết quả:
ls -lh /home/backups/voss-switch-01/

# Xem nội dung file backup:
cat /home/backups/voss-switch-01/voss-switch-01-24-12-2025
```

## Bước 8: Test từng task riêng lẻ
```bash
# Chỉ chạy task tạo thư mục:
ansible-playbook backup_extreme_voss.yml -i inventory/host_file.ini --tags never --start-at-task "Create backup directory"

# Chỉ chạy task backup:
ansible-playbook backup_extreme_voss.yml -i inventory/host_file.ini --start-at-task "Backup VOSS running config"
```

## Troubleshooting

### Lỗi: "Module not found"
```bash
ansible-galaxy collection install community.network --force
```

### Lỗi: "Authentication failed"
- Kiểm tra username/password trong group_vars
- Thử SSH thủ công: `ssh admin@10.1.1.10`

### Lỗi: "Permission denied" khi tạo backup
```bash
# Tạo thư mục thủ công:
mkdir -p /home/backups
chmod 755 /home/backups

# Hoặc đổi đường dẫn trong playbook thành:
# dir_path: "./backups/{{ inventory_hostname }}"
```

### Kiểm tra syntax trước khi chạy:
```bash
ansible-playbook backup_extreme_voss.yml --syntax-check
```

## Quick Test Commands (Copy-Paste)
```bash
# 1. Test kết nối
ansible extreme_voss -i inventory/host_file.ini -m ping

# 2. Chạy test playbook
ansible-playbook skylight/test_extreme_voss.yml -i inventory/host_file.ini -v

# 3. Dry-run backup
ansible-playbook skylight/backup_extreme_voss.yml -i inventory/host_file.ini --check

# 4. Chạy backup thật
ansible-playbook skylight/backup_extreme_voss.yml -i inventory/host_file.ini

# 5. Verify backup file
ls -lh /home/backups/*/
```

## Kết quả mong đợi
```
PLAY RECAP *******************************************************************
voss-switch-01 : ok=5    changed=2    unreachable=0    failed=0    skipped=0
voss-switch-02 : ok=5    changed=2    unreachable=0    failed=0    skipped=0

Backup files:
/home/backups/voss-switch-01/voss-switch-01-24-12-2025
/home/backups/voss-switch-02/voss-switch-02-24-12-2025
```
