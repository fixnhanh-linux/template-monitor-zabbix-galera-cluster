# template-monitor-zabbix-galera-cluster
Dưới đây là hướng dẫn chi tiết từng bước để tải và cấu hình toàn bộ thư mục giám sát Galera Cluster từ GitHub:
Bước 1: Tải và áp dụng cấu hình

<div class="code-block">
  <button class="copy-btn" onclick="copyToClipboard(this)">Copy</button>
  <pre><code>
git clone https://github.com/fixnhanh-linux/template-monitor-zabbix-galera-cluster.git
cd template-monitor-zabbix-galera-cluster
sudo cp userparameter_galera.conf /etc/zabbix/zabbix_agentd.d/
sudo systemctl restart zabbix-agent
  </code></pre>
</div>

Bước 2: Import template
Vào Zabbix Web: Configuration → Templates → Import, chọn file galera_cluster_template.xml từ thư mục vừa tải rồi Click Import

Bước 3: Thêm host trên web zabbix server, ta Chọn host → Templates → Thêm "Template Galera Cluster" → Update
