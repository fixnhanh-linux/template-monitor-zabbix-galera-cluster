
# 🚀 Zabbix Monitoring for Galera Cluster

![Zabbix](https://img.shields.io/badge/Zabbix-6.0+-green)
![MySQL](https://img.shields.io/badge/MySQL-5.7%2B-blue)
![Galera](https://img.shields.io/badge/Galera-4%2B-orange)

Tự động triển khai giám sát Galera Cluster trên Zabbix với script all-in-one. Hỗ trợ export template XML, thu thập metrics cluster và kiểm tra trạng thái node nhanh chóng.
## 📅 Kế hoạch triển khai

| Bước | Nội dung                                               | Ghi chú                            |
|------|--------------------------------------------------------|------------------------------------|
| 1    | Tạo user giám sát MySQL (`zabbix`)                     | Cấp quyền `PROCESS`, `REPLICATION CLIENT` |
| 2    | Cấu hình `UserParameter` cho Zabbix Agent              | Tạo file `.conf` riêng             |
| 3    | Khởi động lại dịch vụ `zabbix-agent`                   | Kiểm tra status                    |
| 4    | Kiểm tra metric từ `zabbix_get`                        | Xác minh dữ liệu trả về            |
| 5    | Import template XML vào Zabbix                         | Dùng file `template_galera_zabbix.xml` |
| 6    | Gán template cho host phù hợp                          | Gán đúng IP/node                   |
| 7    | Theo dõi dữ liệu trên dashboard                        | Có thể dùng thêm Grafana           |

---

## 📦 Hướng dẫn cài đặt nhanh

```bash
# Tải và cấp quyền script
wget https://raw.githubusercontent.com/your_username/galera-zabbix-monitoring/main/deploy_galera_monitoring.sh
chmod +x deploy_galera_monitoring.sh

# Chạy cài đặt
sudo ./deploy_galera_monitoring.sh
```

---

## 📥 Import Template vào Zabbix Server

```bash
# Tạo file XML từ script
sudo ./deploy_galera_monitoring.sh --export-template
```

> Sau đó đăng nhập Zabbix Web → Configuration → Templates → Import file XML

---

## 🛠 Kiểm tra thủ công

```bash
# Kiểm tra từ Zabbix Server
zabbix_get -s <IP_NODE> -k galera.cluster_size

# Xem log zabbix-agent
tail -f /var/log/zabbix/zabbix_agentd.log
```

---

## 🧹 Gỡ cài đặt

```bash
sudo ./deploy_galera_monitoring.sh --uninstall
```

---

## 📊 Các metrics giám sát chính

| Metric                        | Key                                | Mô tả                                           |
|------------------------------|-------------------------------------|------------------------------------------------|
| Cluster Size                 | `galera.cluster_size`              | Số node trong cluster                          |
| Cluster Status              | `galera.cluster_status`            | PRIMARY hoặc NON_PRIMARY                       |
| Node Ready                  | `galera.wsrep_ready`               | Node sẵn sàng query (ON/OFF)                   |
| Replication Lag             | `galera.wsrep_flow_control_paused` | Độ trễ replica do FC                           |
| Send Queue                  | `galera.wsrep_local_send_queue`    | Queue gửi hiện tại                             |
| Receive Queue               | `galera.wsrep_local_recv_queue`    | Queue nhận hiện tại                            |
| Bytes Replicated            | `galera.wsrep_replicated_bytes`    | Tổng số byte đã replicate                      |
| Bytes Received              | `galera.wsrep_received_bytes`      | Tổng số byte nhận từ các node khác             |
| Last Committed              | `galera.wsrep_last_committed`      | Giao dịch cuối cùng được commit                |

---

## 📜 License

MIT © 2025 [FixNhanh]

---

💡 **Tip**: Sử dụng Grafana với [Zabbix plugin](https://grafana.com/grafana/plugins/alexanderzobnin-zabbix-app/) để hiển thị dữ liệu trực quan hơn!

---

## 🔧 Hướng dẫn triển khai thủ công (FIX NHANH)

Dành cho trường hợp không dùng script tự động, bạn có thể làm theo các bước sau để cấu hình Zabbix giám sát Galera Cluster:

### Bước 1: Tạo user `zabbix` trong MySQL

```sql
CREATE USER 'zabbix'@'localhost' IDENTIFIED BY 'FixNhanhPass123!';
GRANT USAGE, REPLICATION CLIENT, PROCESS ON *.* TO 'zabbix'@'localhost';
FLUSH PRIVILEGES;
```

### Bước 2: Thêm các dòng `UserParameter` vào file cấu hình agent

```bash
nano /etc/zabbix/zabbix_agentd.conf.d/userparameter_galera.conf
```

Thêm vào toàn bộ nội dung sau:  
(Lưu ý nếu cần pass: thêm `-uzabbix -pFixNhanhPass123!` vào lệnh `mysql` trong từng dòng)

```bash
#Total number of cluster membership changes happened.
UserParameter=galera.cluster_conf_id,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_conf_id';" 2>/dev/null | awk '{print $2}' || echo 0

#Current number of members in the cluster.
UserParameter=galera.cluster_size,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_size';" 2>/dev/null | awk '{print $2}' || echo 0

#Status of this cluster component.
UserParameter=galera.cluster_status,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_cluster_status';" 2>/dev/null | awk '{print $2}' || echo 0

#If the value is OFF, the node has not yet connected to any of the cluster components.
UserParameter=galera.wsrep_connected,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_connected';" 2>/dev/null | awk '{print $2}' || echo 0

#Shows the internal state of the EVS Protocol
UserParameter=galera.wsrep_evs_state,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_evs_state';" 2>/dev/null | awk '{print $2}' || echo 0

#How much the slave lag is slowing down the cluster.
UserParameter=galera.wsrep_flow_control_paused,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_paused';" 2>/dev/null | awk '{print $2}' || echo 0

#FC_PAUSE events received
UserParameter=galera.wsrep_flow_control_recv,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_recv';" 2>/dev/null | awk '{print $2}' || echo 0

#FC_PAUSE events sent
UserParameter=galera.wsrep_flow_control_sent,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_flow_control_sent';" 2>/dev/null | awk '{print $2}' || echo 0

#UUID
UserParameter=galera.wsrep_gcom_uuid,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_gcomm_uuid';" 2>/dev/null | awk '{print $2}' || echo 0

#Seqno of last committed tx
UserParameter=galera.wsrep_last_committed,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_last_committed';" 2>/dev/null | awk '{print $2}' || echo 0

#FSM state number
UserParameter=galera.wsrep_local_state,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_state';" 2>/dev/null | awk '{print $2}' || echo 0

#Local tx aborted by slave
UserParameter=galera.wsrep_local_bf_aborts,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_bf_aborts';" 2>/dev/null | awk '{print $2}' || echo 0

#Recv queue length
UserParameter=galera.wsrep_local_recv_queue,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_recv_queue';" 2>/dev/null | awk '{print $2}' || echo 0

#Send queue length
UserParameter=galera.wsrep_local_send_queue,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_send_queue';" 2>/dev/null | awk '{print $2}' || echo 0

#State comment
UserParameter=galera.wsrep_local_state_comment,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_state_comment';" 2>/dev/null | awk '{print $2}' || echo 0

#UUID of node state
UserParameter=galera.wsrep_local_state_uuid,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_local_state_uuid';" 2>/dev/null | awk '{print $2}' || echo 0

#Node readiness
UserParameter=galera.wsrep_ready,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_ready';" 2>/dev/null | awk '{print $2}' || echo 0

#Bytes received
UserParameter=galera.wsrep_received_bytes,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_received_bytes';" 2>/dev/null | awk '{print $2}' || echo 0

#Bytes replicated
UserParameter=galera.replicated_bytes,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_replicated_bytes';" 2>/dev/null | awk '{print $2}' || echo 0

#Data replicated
UserParameter=galera.wsrep_repl_data_bytes,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_repl_data_bytes';" 2>/dev/null | awk '{print $2}' || echo 0

#Keys replicated
UserParameter=galera.wsrep_repl_keys,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_repl_keys';" 2>/dev/null | awk '{print $2}' || echo 0

#Keys bytes replicated
UserParameter=galera.wsrep_repl_keys_bytes,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_repl_keys_bytes';" 2>/dev/null | awk '{print $2}' || echo 0

#Other bytes replicated
UserParameter=galera.wsrep_repl_other_bytes,HOME=/var/lib/zabbix mysql -uzabbix -pFixNhanhPass123! -s -N -e "SHOW GLOBAL STATUS LIKE 'wsrep_repl_other_bytes';" 2>/dev/null | awk '{print $2}' || echo 0

#Port check (no timeout)
UserParameter=mysql.port_check,nc -zv localhost 3306 >/dev/null 2>&1 && echo 1 || echo 0

#Port check with 5s timeout
UserParameter=mysql.port_check_timeout,nc -zv -w5 localhost 3306 >/dev/null 2>&1 && echo 1 || echo 0

```

(Sử dụng file mẫu từ repo nếu cần)

### Bước 3: Khởi động lại Zabbix Agent

```bash
sudo systemctl restart zabbix-agent
```

### Bước 4: Kiểm tra key Zabbix nhận metric

```bash
zabbix_get -s <ip_node> -k galera.cluster_status
```

### Bước 5: Import Template thủ công vào Zabbix

- Vào Zabbix Web UI → Configuration → Templates → Import
- Dùng file XML: `template_galera_zabbix.xml` được xuất từ repo/script

---

✅ Sau khi hoàn tất, hệ thống sẽ bắt đầu nhận thông số Galera Cluster trong Zabbix trong vài phút.
