Chào bạn, việc nâng cấp từ zabbix-agent lên zabbix-agent2 trên Ubuntu 22.04 là một bước đi rất hợp lý. zabbix-agent2 được viết bằng Go, hỗ trợ nhiều plugin hiện đại hơn và xử lý các check giám sát hiệu quả hơn hẳn bản cũ.

Dưới đây là quy trình chuẩn chỉnh từng bước để xóa sạch bản cũ và cài bản mới mà không làm mất cấu hình quan trọng của bạn.

Bước 1: Sao lưu cấu hình cũ (Quan trọng)
Trước khi xóa, bạn nên backup file cấu hình cũ để kiểm tra lại các thông số như Server, ServerActive, hoặc Hostname nếu cần.

Bash
sudo cp /etc/zabbix/zabbix_agentd.conf /etc/zabbix/zabbix_agentd.conf.bak
Bước 2: Dừng và gỡ bỏ hoàn toàn zabbix-agent
Dừng dịch vụ hiện tại:

Bash
sudo systemctl stop zabbix-agent
sudo systemctl disable zabbix-agent
Gỡ bỏ gói cài đặt và xóa file cấu hình cũ (purge):

Bash
sudo apt-get purge zabbix-agent -y
sudo apt-get autoremove -y
Bước 3: Cài đặt và cấu hình zabbix-agent2
Do bạn dùng Ubuntu 22.04 (Jammy Jellyfish), chúng ta sẽ đảm bảo repository của Zabbix đã được cập nhật phiên bản mới nhất (ví dụ bản LTS phổ biến hiện tại là Zabbix 6.0 hoặc 7.0).

Cập nhật Zabbix Repository (Nếu chưa làm trước đó):
(Ví dụ dưới đây áp dụng cho Zabbix 7.0 LTS, bạn có thể thay đổi link repo tùy thuộc vào phiên bản Zabbix Server bạn đang dùng)

Bash
wget https://repo.zabbix.com/zabbix/7.0/ubuntu/pool/main/z/zabbix-release/zabbix-release_latest_7.0+ubuntu22.04_all.deb
sudo dpkg -i zabbix-release_latest_7.0+ubuntu22.04_all.deb
sudo apt update
Cài đặt zabbix-agent2:

Bash
sudo apt install zabbix-agent2 zabbix-agent2-plugin-* -y
Cấu hình zabbix-agent2:
Mở file cấu hình mới lên để chỉnh sửa:

Bash
sudo nano /etc/zabbix/zabbix_agent2.conf
Tìm và cập nhật các tham số chính tương thích với Zabbix Server của bạn:

Server=<IP_CỦA_ZABBIX_SERVER>

ServerActive=<IP_CỦA_ZABBIX_SERVER>

Hostname=<Tên_Máy_Chủ_Của_Bạn> (Phải khớp với tên Host trên giao diện Web Zabbix).

Khởi động và kích hoạt dịch vụ:

Bash
sudo systemctl start zabbix-agent2
sudo systemctl enable zabbix-agent2
Bước 4: Kiểm tra trạng thái
Đảm bảo rằng zabbix-agent2 đã chạy tracking mượt mà:

Bash
sudo systemctl status zabbix-agent2
Mẹo DevOps: Bạn có thể kiểm tra xem Agent2 phản hồi tốt không bằng lệnh kiểm tra nội bộ:
zabbix_agent2 -t agent.ping
Nếu kết quả trả về [s|1] là hệ thống đã sẵn sàng kết nối lên Server!

Nếu bạn gặp khó khăn ở bước cấu hình hay đồng bộ với Zabbix Server, cứ nhắn mình hỗ trợ tiếp nhé!
