---
title: "Điều tra mã độc tống tiền TeslaCrypt: Từ quá trình tải xuống mã độc đến khôi phục tài liệu Tender"
date: 2026-07-23 07:00:00 +0700
categories: [Malware, Ransomware]
tags: [Malware, Tesla]
---
## Scenario

ABC Industries worked day and night for a month to prepare a tender document for a prestigious project that would secure the company’s financial future. The company was hit by ransomware, believed to be conducted by a competitor, and the final version of the tender document was encrypted. Right now they are in need of an expert who can decrypt this critical document. All we have is the network traffic, the ransom note, and the encrypted ender document. Do your thing Defender!

![image.png](./assets/ransomware2/1.png)

Hiện tại case này đã bị tấn công **ransomware** và mục tiêu của mình là phân tích bên cạnh đó trả lời và giải thích những câu hỏi của họ dành cho mình, dưới gốc nhìn của một điều tra viên mình sẽ thực hiện nó một cách chi tiết và cụ thể nhất không có sự sai lệch hay điều chỉnh gì ở đây hết, file bị encryption là file mình đã khoanh vùng màu đó, cón file pcap là file đã capture được cuộc tấn công này xảy ra. Còn những file còn lại là những file mà attacker để lại để victim liên hệ với attacker để tìm cách khôi phục, mình sẽ capture nó ở dưới đây cho các bạn quan sát và khoanh vùng những điều quan trọng.

![image.png](./assets/ransomware2/2.png)

Đây chính là những gì attacker đã để lại và nếu victim muốn recover được file đã bị encrypt đó thì phải contact với attacker hoặc làm theo những gì attacker chỉ dẫn. Giờ thì mình không vòng vo thêm một giây phút nào nữa mình sẽ bắt đầu vào thực hiện điều tra và trả lời câu hỏi mà họ đã đưa ra.

## Practice

### What is the operating system of the host from which the network traffic was captured?

Ở đây họ muốn biết được host nào đã capture được cuộc tấn công này và đặc biệt là OS mà host này sử dụng là gì, đặc biệt điều này là bước quyết định sống còn trong việc phân tích và ngăn chặn. Điều này giúp mình xác định được kỹ thuật tấn công, khoanh vùng payload độc hai của nó và ngăn chặn việc lây lan của nó, nói dễ hiểu là không phải malware nào cũng có thể run được trên tất cả OS, một malware được tạo ra chỉ đánh vào một OS duy nhất và những version duy nhất, ngoài ra điều này còn giúp mình xác định được những lỗ hổng trong OS đó để khoanh vùng các malware và những gì liên quan tới OS này.

![image.png](./assets/ransomware2/3.png)

Như bạn có thể thấy được ở trên đó chính là những gì liên quan tới câu hỏi và những gì liên quan tới host này **32-bit Windows 7 Service Pack 1, build 7601** đây chính là câu trả lời cho câu hỏi này ngoài ra mình xác định được thời gian mà cuộc attack này xảy ra vào khoảng thời gian nào..

### What is the full URL from which the ransomware executable was downloaded?

Để xác định điều này mình sẽ mở file pcap mà họ đã cung cấp lên và thực hiện việc điều tra về case này

![image.png](./assets/ransomware2/4.png)

có **766** package đã được capture nhưng để trả lời câu hỏi trên và tiếp tục phân tích thì mình sẽ filter package như thế này, mính sẽ sử dụng http để thực hiện filter và phân tích, dưới đây chính là kết quả mà mình nhận được 

![image.png](./assets/ransomware2/5.png)

mình chỉ phát hiện ra có 2 package liên quan đến protocol này thôi nên điều này có thể giúp điều tra nhanh hơn.

![image.png](./assets/ransomware2/6.png)

mình đã xác định được full url mà thực hiện download đó là **http://10.0.2.15:8000/safecrtypt.exe** và  ngoài ra mình còn xác định được thêm file mà đã được download đó là file safecrypt.exe, chưa thế khẳng định được nó có phải là file malware hay không, dưới những câu hỏi tiếp theo mình sẽ thực hiện phân tích về file này và đưa ra một kết luận chính xác nhất.

### What is the MD5 hash of the ransomware?

Mình đã export file malware đó về máy của mình giờ đây mình sẽ sử dụng virustotal để thực hiện scan nó và kiểm tra xem nó có phải là file malware hay không, dưới đây chính là kết quả mà mình nhận được sau khi mình dùng virustotal để scan nó.

![image.png](./assets/ransomware2/7.png)

Đây chính xác là một file malware và mình cũng xác định được nó là một file malware thuộc trojan malware bên cạnh đó con ransomware này có tên là **teslacrypt**.

![image.png](./assets/ransomware2/8.png)

Hình ảnh trên chính là properties basic của con malware này, nó được dev bằng ngôn ngữ C++, để xác thực điều này mình cũng đã sử dụng Die tool để kiểm tra và dưới đây chính là result của nó

![image.png](./assets/ransomware2/9.png)

Mã hash MD5 của nó là : **4a1d88603b1007825a9c6b36d1e5de44**, các bạn có thể sử dụng hash này và lên virustotal để check xem như thế nào.

### What is the domain beginning with ‘d’ that is related to ransomware traffic?

Đây cũng chính là một phần mấu chốt của case này, domain này có liên quan tới malware này và đặc biệt nó sẽ resolve tới khi traffic network được thiết lập, khi mình kiểm tra các domain liên quan tới malware này trong virustotal thì mình nhận được những domain dưới đây.

![image.png](./assets/ransomware2/10.png)

Tuy nhiên điều này chỉ giúp mình một phần nào đó khoanh vùng scope domain liên quan tới con malware này thôi, chứ trong case này thì hiện tại mình vẫn chưa phân tích nên là vẫn chưa thể xác định được đó là con domain nào, để tiếp tục mình sẽ thực hiện phân tích domain của nó trong case này bằng file pcap mà họ đã cung cấp cho mình.

![image.png](./assets/ransomware2/11.png)

Mình sử dụng protocol DNS để thực hiện phân tích và phát hiện domain của nó. Thì trên hình ảnh đó chính là domain mà mình đã khoanh vùng và xác định được điều này **dunyamuzelerimuzesi.com** đây chính là domain của malware này khi thiết lập traffic network.

### Decrypt the Tender document and submit the flag ?

Đây chính là phần kết cũng như là phần pivot của case này, mình đã research về loại malware **teslacrypt** này và thật bất ngờ mình không cần phải analysts file malware đó để nhận được key, nhưng mình có thể dùng tool để thực hiện decrypt file đã bị **ransomware** attack và nhận được content trong file đó.

![image.png](./assets/ransomware2/12.png)

Đây chính là bằng chứng cho việc đó, dưới đây chính là cách mà mình đã decrypt file đó các bạn có thể quan sát dưới đây.

![image.png](./assets/ransomware2/13.png)

Mình đã download tool về máy để thực hiện và mình sẽ run file TeslaDecrypter.exe, khi mình run file này thì mình có GUI mà họ cấp như sau

![image.png](./assets/ransomware2/14.png)

Tool đang thực hiện scan xem file đã bị encrypted là file nào, nó run khá lâu vì nó scan toàn bộ ổ C của mình để check từng folder một nên việc trả kết quả là khá lâu và có thể mất nhiều time về điều này.

![image.png](./assets/ransomware2/15.png)

Đây chính là file mà mình đã thực hiện decryption thành công và nhận lại được flag : “**BTLO-T3nd3r-Fl@g**”. Đó chính là những gì và xuyên suốt trong bài LAB này và mục đích của nó là như thế dưới đây chính là một vài nhận xét của mình về bài LAB này và một vài ý kiến của mình.

## Conclusion

Đầu tiên nếu đánh giá về mức độ của bài LAB đối với một người như mình thì mình thấy bài này khá hay và có nhiều tình tiết đáng chú ý như là download malware, hay là traffic network. Nhưng mình muốn hiểu sâu hơn về tại sao lại trong cùng một dải IP mà thực hiện download được, bên cạnh đó đây chỉ là phòng LAB nên traffic như thế là cũng đúng, không có external IP hay gì hết chỉ là trong phòng LAB thôi. Còn về phần malware thì mình đã thực hiện analysts malware và không nhận được điều gì hay ho nên mình không bàn đến với một điều nữa là malware type of teslacrypt này đã được public nên là không còn hấp dẫn đối với mình. Tới đây thôi là được rồi, see you lateeeeee….
