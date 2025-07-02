# 🚀 Zabbix Monitoring for Galera Cluster

![Zabbix](https://img.shields.io/badge/Zabbix-6.0+-green)
![MySQL](https://img.shields.io/badge/MySQL-5.7%2B-blue)
![Galera](https://img.shields.io/badge/Galera-4%2B-orange)

Tự động triển khai giám sát Galera Cluster trên Zabbix với script all-in-one.

## 📦 Cài đặt nhanh

### Yêu cầu
- Hệ điều hành: Ubuntu/Debian/CentOS/RHEL/AlmaLinux
- Quyền root hoặc sudo

### 1. Tải script
```bash
wget https://raw.githubusercontent.com/your_repo/galera-zabbix-monitoring/main/deploy_galera_monitoring.sh
chmod +x deploy_galera_monitoring.sh
```

### 2. Chạy triển khai
```bash
sudo ./deploy_galera_monitoring.sh
```

### 3. Import template vào Zabbix
```bash
# Xuất template tự động
sudo ./deploy_galera_monitoring.sh --export-template

# Sau đó import file XML qua giao diện Zabbix
```

## 📝 Các lệnh thủ công

### Kiểm tra kết nối
```bash
# Từ Zabbix Server
zabbix_get -s <node_ip> -k galera.cluster_size
```

### Xem log
```bash
tail -f /var/log/zabbix/zabbix_agentd.log
```

### Gỡ cài đặt
```bash
sudo ./deploy_galera_monitoring.sh --uninstall
```

## 📊 Các metrics chính
| Metric | Key | Mô tả |
|--------|-----|-------|
| Cluster Size | `galera.cluster_size` | Số node trong cluster |
| Node Status | `galera.ready` | Trạng thái sẵn sàng (0/1) |
| Replication Lag | `galera.flow_control_paused` | Độ trễ replication |

## 🛠 Tùy chỉnh
Sửa file cấu hình tại:
```bash
/etc/zabbix/zabbix_agentd.conf.d/20_galera.conf
```

## 📜 License
MIT © [Your Name]

---

💡 **Tip**: Dùng Grafana + [Zabbix plugin](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/) để visualize dữ liệu đẹp hơn!
