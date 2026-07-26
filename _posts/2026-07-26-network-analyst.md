---
title: "Hành động nguy hiểm trên wireshark"
date: 2026-07-26 07:00:00 +0700
categories: [Web, Reverse Shell]
tags: [Malware, wireshark]
---
## Scenario

The SOC received an alert in their SIEM for ‘Local to Local Port Scanning’ where an internal private IP began scanning another internal system. Can you investigate and determine if this activity is malicious or not? You have been provided a PCAP, investigate using any tools you wish.

Wireshark đã capture được một cuộc tấn công để phân tích và làm rõ những điều này cần phải có những lưu ý sau, thứ nhất các cuộc tấn công có rất nhiều thể loại và rất nhiều điều xảy ra trên network chúng ta không thể nắm thóp hết được, nhưng để biết được đó là loại tấn công nào thì chúng ta phải thực hiện kiểm tra từng phần dù là nhỏ nhất trong cuộc tấn công đó, nó ảnh hưởng gì tới server hay mọi thứ của chúng ta hay không, với gợi ý trên là “Local to Local Port Scanning” điều này có thể cho thấy được rằng một user đã bị compromise và thực hiện active scanning port về máy chủ server. 

### Root Case

Nguồn gốc của điều này đã được xác định là xuất phát từ một máy nằm trong danh sách **local** và dưới đây chính là **conversation** của file pcap mà đã được capture.

!image.png

Điểm bất thường đã được xác định, có 2 IP có số lượng packet rất lớn và có lưu lượng bất thường so với những IP còn lại vậy nên sẽ thực hiện điều tra và phân tích hai IP để xác định được những gì đã xảy ra, sử dụng query dưới đây vào wireshark 

```jsx
ip.addr == 10.251.96.4 && ip.addr == 10.251.96.5
```

!image.png

Có hơn **15883** packet hiển thị, giao tiếp của hai IP này chiếm phần lớn packet trong file pcap này ngoài ra, từ hình ảnh trên xác định được IP **`10.251.96.4`** chính là IP bị compromise để thực hiện hành động **Scan Port** tới địa chỉ IP **`10.251.96.5`** và xác định được IP này chính là IP server. Quan sát được IP bị compromise đã sử dụng **`Nmap`** để thực hiện **Scan Port** các dấu hiệu và flag dưới đây chính là minh chứng và tôi còn biết rõ họ đã sử dụng option nào trong **`Nmap`** để thực hiện Scan

```jsx
**10.251.96.4 -> 10.251.96.5 SYN WIN=1024
10.251.96.5 -> 10.251.96.4 RST, ACK**  
```

Ở đây họ đã sử dụng option `-sS` trong `Nmap` để thực hiện Scan, còn phía tool thì Windown size đã chứng minh, có thể search internet để biết thêm chi tiết hơn, khi thực hiện scan mà được reply bằng hai flag RST và ACK thì điều đó có nghĩa là port đó đang đóng không còn để minh chứng về việc phát hiện cổng port nào mở thì xin mới mọi người xem dưới đây

!image.png

Dấu hiệu như sau và cũng giải thích cho mọi người dễ hiểu hơn về điều này

```jsx
**10.251.96.4 -> 10.251.96.5 SYN WIN=1024
10.251.96.5 -> 10.251.96.4 SYN, ACK port = 80**
```

Đó chính là những flag thiệt lập kết nối trên **Network** và nếu nó được open thì sẽ reply thành 2 flag điển hình của thiết lập connect đó chính là **SYN** và **ACK,** đây cũng chính là ROOT Cause cho cuộc attack này mà mình đã detect được. Attacker đã scan từ port 1-1024 để phát hiện xem có những cổng port nào được open và thật may mắn ch

### Phân Tích Diễn Biến

Nếu đã xác định được cổng port open, thì tiếp đến attacker sẽ thực hiện tấn công vào web server này, đây chính là điều chắc chắn vì dựa trên flow attack của attacker thì không phải dự đoán mà là nó sẽ chắc chắn xảy ra khi attacker detect được cổng port nào open thì họ sẽ thực hiện hoặc sử dụng tool để attack vào service đó. Điều gần như chắc chắn là attacker sẽ thực hiện scan resource của WEB server này và mình sẽ thực hiện anaylysts cũng như phát hiện thêm nhiều IOCs để take note lại.