---
title: "Mail Phishing chưa thể pass SPF và DMARC"
date: 2026-07-27 07:00:00 +0700
categories: [Phishing, Reverse Shell]
tags: [Report, Mail Phishing]
---

## Scenario

A sales executive at Greenholt PLC has reported a suspicious email received from a known customer. The message raised several red flags: a generic greeting, an unexpected request for a money transfer, and an unsolicited attachment. According to the employee, this behavior does not align with the customer’s usual communication style. Concerned that the email may be malicious, the message has been escalated to the SOC (Security Operations Center) for further investigation. Your goal is to analyze the provided email sample and determine whether it is legitimate or part of a phishing attempt.

Miêu tả một chút thì với vai trò là một SOC Analysts sẽ thực hiện phân tích mail phishing và đặc biệt hơn nữa cần phải viết lại hết mọi thứ xuất hiện trong mail đó, bên cạnh đó investigation xem mail đó chứa cái gì và phishing như thế nào nó nhằm vào điều gì, bên cạnh đó ghi lại mọi thứ xuất hiện trong mail đó. Hiện nay trên thế giới hầu hết mọi thứ đều xuất phát từ **Phishing Mail** và đây cũng chính là vấn nạy hay còn gọi là **Big Problem** đang xảy ra rất nhiều trên thế giới, với vai trò là một Security thì sẽ giúp công ty đưa ra phương án hay cách thức để giảm điều này cũng như có khả năng ngăn chặn được nó xảy ra.

![image](./assets/phishing1/1.png)

## Analysts
Thực hiện phân tích về mail có thể quan sát ở trên mail được gửi từ `info@mutawamarine.com` tới người nhận có tên là **Mr.James Jackson** có mail là `info.mutawamarine@mail.com`. Bên cạnh đó ở trong file này có một attachment có tên là `SWT_#09674321_PDF_.CAB` và đó là một file có định dạng *RAR* dưới đây chính là minh chứng cho điều đó.
![image](./assets/phishing1/2.png)
đây chính là hash sha256 của file trên **2e91c533615a9bb8929ac4bb76707b2444597ce063d84a4b33525e25074fff3f** khi thực hiện tích hash. Để đi sâu vào phần phân tích thì sẽ xem source của mail này thì sẽ như thế nào và flow hoạt động của mail sẽ như thế nào.
![image](./assets/phishing1/3.png)
Ở đây sẽ vẽ lại flow hoạt động của email này và xác định được thế nào flow hoạt động của email như thế nào, điểm khởi đầu ra sao, IP của attacker là gì và dưới đây chính là minh chứng, mọi thống số hay số liệu đều được lấy từ hình ảnh trên.

```
192.119.71.157
hwsrv-737338.hostwindsdns.com
HELO: mutawamarine.com
        |
        | ESMTP
        v
sub.redacted.com
Exim 4.80
        |
        | SMTP
        v
mta4212.mail.bf1.yahoo.com
        |
        | SMTPs
        v
atlas125.free.mail.bf1.yahoo.com
        |
        v
Yahoo mailbox
redacted@yahoo.com
```

Theo flow hoạt động trên thì có thể xác định được IP của hacker là `192.119.71.157` bên cạnh đó thực hiện OSINT về IP này thì biết được là IP này trực thuộc như hình ảnh dưới đây.
![image](./assets/phishing1/4.png)
Hơn nữa khi flow hoạt động trên cho thấy được mail đã đi qua nhiều hop chứ không phải riêng một server, bên cạnh đó có thể xác định thông qua **hình ảnh 3** cho thấy được rằng *SPF* của sender bị fail còn *DMARC* thì xuất hiện *UNKNOW* dưới đây chính là phần giải thích về điều này.
![image](./assets/phishing1/5.png)
![image](./assets/phishing1/6.png)

Từ hai hình trên có thể rút ra kết luận rằng email này có dấu hiệu không vượt qua các kiểm tra xác thực bảo mật phổ biến. Hình 5 cho thấy SPF có thể bị fail, nghĩa là địa chỉ IP hoặc server gửi mail không nằm trong danh sách được phép của miền gửi. Hình 6 cho thấy DMARC ở trạng thái không rõ/unknown, cho thấy mail chưa được xác thực đầy đủ theo chính sách SPF/DKIM. Đây là một tín hiệu rất quan trọng trong phân tích phishing, vì các email lừa đảo thường cố tình bypass hoặc không tuân thủ các cơ chế xác thực này.

### Đánh giá nhanh
Đây là một trường hợp phishing điển hình với các dấu hiệu nhận diện rõ ràng: lời chào chung chung, yêu cầu chuyển tiền bất thường, attachment lạ và việc kiểm tra xác thực email không đạt. Mặc dù mail đã đến được hộp thư người nhận, các chỉ số SPF/DMARC cho thấy người gửi không được xác thực đầy đủ, nên đây là tín hiệu rất đáng ngờ. Nếu được phát hiện sớm, email này có thể được chặn trước khi ảnh hưởng đến người dùng hoặc hệ thống nội bộ.

### Gợi ý cho doanh nghiệp về việc ngăn chặn mail phishing từ domain này
Dựa trên phân tích trên, doanh nghiệp nên triển khai các biện pháp sau để giảm thiểu rủi ro từ các email phishing tương tự:

1. Chặn và giám sát domain nghi ngờ
   - Đưa tên miền như `mutawamarine.com` và các biến thể lặp lại, spoofing hoặc typo-squatting vào danh sách chặn trên gateway email.
   - Tạo rule chặn các mail gửi từ domain không thuộc hệ thống đáng tin cậy hoặc không vượt qua SPF/DKIM/DMARC.

2. Bật và kiểm tra xác thực email chuẩn
   - Cấu hình SPF, DKIM và DMARC cho toàn bộ domain doanh nghiệp.
   - Nếu có thể, thiết lập DMARC ở mức `quarantine` hoặc `reject` để ngăn mail giả mạo đi vào hộp thư người dùng.

3. Kiểm soát attachment và URL
   - Chặn các file có định dạng đáng ngờ như `.cab`, `.rar`, `.zip`, `.iso`, `.exe` hoặc các attachment bất thường từ người gửi chưa quen.
   - Áp dụng sandbox và URL scanning cho mọi attachment và link trước khi cho người dùng mở.

4. Tăng cường cảnh báo và đào tạo người dùng
   - Bật tính năng báo cáo phishing trong Outlook/Google Workspace để người dùng có thể phản hồi nhanh khi nhận thấy mail đáng ngờ.
   - Tổ chức đào tạo thường xuyên về các mẫu email lừa đảo, đặc biệt là các mail yêu cầu chuyển tiền, kèm attachment hoặc chứa lời kêu gọi khẩn cấp.

5. Bảo vệ tài khoản và endpoint
   - Bật MFA cho tài khoản email và các hệ thống quan trọng.
   - Giới hạn việc chạy macro trong Office, chặn PowerShell hoặc script từ email đối với các người dùng không cần thiết.

6. Theo dõi IOC và phản ứng sự cố
   - Ghi nhận các dấu hiệu như IP `192.119.71.157`, hash malware và các domain sender liên quan để đưa vào phòng ngừa và điều tra.
   - Khi phát hiện incident, cần block ở tầng email, proxy và endpoint để tránh lây lan.

### Attachment
Khi đưa hash của nó lên Virustotal thì nhận được kết quả như dưới đây.
![image](./assets/phishing1/7.png)
đây là một file malware trojan, tiếp tục thực hiện phân tích về malware này thì cần dùng hash của nó lên malwarebazzar và download malware về để thực hiện phân tích vì đây là file, có một chút vấn đề về file malware này, mặc dù mình đã truy cập mọi thứ liên quan tới malware này nhưng không có thông số về malware này, chỉ có virustotal detected được và ngoài ra mình còn sử dụng **any.run** để tìm kiếm và dưới đây chính là những gì thuộc về nó còn sót lại ở đây.
![image](./assets/phishing1/8.png)
đó là những gì mà mình đã phân tích và detected được, bên cạnh đó dựa vào hình ảnh trên có thể quan sát được rằng run chỉ mở nó lên thì không có hành vi gì bất thường vì đây chỉ là file RAR nên là chưa run malware và hành động của malware nằm bên trong chưa thể phát hiện được. Khi nào unzip file rar đó và run click được vào file nằm bên trong đó thì sẽ thực hiện detect được và ngoài ra có thể thực hiện phân tích static với file đó để xác định một vài behavior của nó như thế nào, từ đó đưa ra kết luận sát xao và cứng nhắc hơn về phần này.