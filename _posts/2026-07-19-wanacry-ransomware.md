---
title: "PC Bị Wannacry tấn công"
date: 2026-07-19 07:00:00 +0700
categories: [Malware, Ransomware]
tags: [Malware, Wannacry]
---
## Scenario

The Account Executive called the SOC earlier and sounds very frustrated and angry. He stated he can’t access any files on his computer and keeps receiving a pop-up stating that his files have been encrypted. You disconnected the computer from the network and extracted the memory dump of his machine and started analyzing it with Volatility. Continue your investigation to uncover how the ransomware works and how to stop it!

Hiện tại có một victim đã bị attack ransomware và may mắn thay họ đã thực hiện capture RAM của victim đó và mình sẽ thực hiện analysts nó để xem đó là con ransomware gì và nó đã hành động như thế nào cũng như phân tích mọi thứ về con ransomware này là gì, nó thuộc họ hàng nào và mọi thứ về nó. Bên cạnh đó mình sẽ sử dụng Volatility 2 để thực hiện analysts file RAM này và trả lời những câu hỏi mà họ đã yêu cầu và đưa ra cho mình.

## Practice

![image.png](./assets/ransomware1/1.png)

Trên đây chính là file RAM đó, giờ mình sẽ sử dụng command dưới đây để xem information của file RAM này, xem nó thuộc OS nào và ngoài ra điều này còn giúp mình giảm được scope khi phân tích

![image.png](./assets/ransomware1/2.png)

Kết quả mình nhận được thì nó là Win7SP1x86 đây cũng chính là profile mà mình sẽ sử dụng để thực hiện phân tích nó.

### What is the name of the suspicious process?

Đầu tiên mình phải xác định được đâu là process mà mình đang nghi ngờ và đâu mới thật sự là process có thuộc về malware. Dưới đây chính là cách mà mình thực hiện và command mình thực hiện để làm điều này và các minh xác định nó như thế này.

```jsx
python2 volatility/vol.py -f infected.vmem --profile=Win7SP1x86 pslist
```

![image.png](./assets/ransomware1/3.png)

Ở đây file mà mình nghi là malware đó chính là process **WanaDecryptor**, bên cạnh đó mình còn khoanh vùng những processes liên quan tới nó. Đó là những process parents của nó thông qua **PID** và **PPID** để mình xác định được, thì dựa trên flow đó  mình xác định được là user đã open file **or4qtckT.exe** và sau đó chính file exe này đã drop file **WanaDecryptor** này, tại sao mình lại bảo là user open file, bởi vì **PPID** của **or4qtckT.exe** này là **explorer.exe** là một process liên quan tới FILE. Vậy nên có thể khẳng định được hành động này là gây ra bởi user open chứ không phải là tác động bên ngoài, không có traffic network nhưng user đã bị phishing hoặc là download nhầm file package gì đó.

### If you drill down on the suspicious PID, find the process used to delete files ?

Nếu ai chưa biết thì **WanaDecryptor** chính là malware **ransomware** của **WanaCry** và bản chất của nó là deleted file hay encryption file, thì ở đây họ có hỏi mình về điều này đó chính là phân tích process nào đã thực hiện deleted file ở trên máy nạn nhân. Để thực hiện điều này dưới đây chính là result cũng là mọi thứ mình thực hiên analysts về điều này.

![image.png](./assets/ransomware1/4.png)

như các bạn có thể thấy mình lấy chính PID của **or4qtckT.exe** để tìm kiếm việc này bởi vì đây chính là process nguồn gốc của mọi thứ xuất phát nên là điều này xác định được ngoài **WanaDecryptor** thì còn một process giả mạo các process schedule đó là **taskdl.exe**, thì không còn gì khác mình xác định đây là chính là file dùng để thực hiện deleted file của victim và file này mình sẽ thực hiện phân tích ở bên dưới.

### Find the path where the malicious file was first executed

Đây cũng chính là câu hỏi giúp mình củng cố được suy nghĩ của mình khi để xác định là user open file chứ không phải do tác động nào khác. Dưới đây chính là command và cũng chính là cách mà mình phân tích nó các bạn có thể quan sát.

```jsx
python2 volatility/vol.py -f infected.vmem --profile=Win7SP1x86 cmdline
```

![image.png](./assets/ransomware1/5.png)

Mình có thể chắc chắn được rằng user hacker đã open file or4qtckT.exe tại folder Desktop và điều này có thể khẳng định được những gì mình nói là đúng. Dưới đây mình sẽ thực hiện analysts file malware này và xem là behavior của nó như thế nào ngoài ra xem nó có relationship như thế nào, dưới đây chính là quá trình hay những gì mình đã thực hiện để analysts nó.

### Analysts Malware

Mình đã thực hiện command và dump file **or4qtckT.exe** ra disk của mình dưới đây chính là những điều đó.

```jsx
python2 volatility/vol.py -f infected.vmem --profile=Win7SP1x86 procdump -p 2732 -D dump/
```

![image.png](./assets/ransomware1/6.png)

Đây chính là file mà mình đã thực hiện dump ra disk của mình và mình sẽ thực hiện phân tích nó, mình chỉ thực hiện phân tích tĩnh về nó, dưới đây chính là cách mà mình thực hiện điều này. Mính sẽ sử dụng công cụng floss để phân tích và kèm theo rankstrings để sắp xếp những IOC nào quan trọng trong nó.

![image.png](./assets/ransomware1/7.png)

Mình cho output của floss ra một file có tên là mal.txt và sử dụng rank_string để tiếp tục điều tra về nó. Dưới đây chính là toàn bộ IOC mà mình capture và có giải thích dưới đây

### 1. Tên file và artifact

| Loại | IOC | Ý nghĩa |
| --- | --- | --- |
| File phân tích | `executable.2732.exe` | Tên file mẫu được đưa vào FLOSS |
| Executable | `tasksche.exe` | File thường liên quan đến thành phần thực thi của WannaCry |
| Data/resource file | `t.wnry` | File tài nguyên/mã hóa đặc trưng của WannaCry |
| Chuỗi tác vụ | `TaskStart` | Có thể liên quan đến quá trình khởi chạy tác vụ |

Các chuỗi này xuất hiện trực tiếp trong mẫu.

### 2. Bitcoin wallet addresses

```
115p7UMMngoj1pMvkpHijcRdfJNXj6LrLn
12t9YDPgwueZ9NyMgw519p7AA8isjr6SMw
13AM4VW2dhxYgXeQepoHkHSQuy6NgaEb94
```

Đây là ba địa chỉ ví Bitcoin có thể dùng làm IOC để tìm kiếm trong log, threat-intelligence platform hoặc dữ liệu giao dịch.

### 3. Mutex

```
Global\MsWinZonesCacheCounterMutexA
```

Mutex thường được malware sử dụng để kiểm tra xem một phiên bản của nó đã chạy trên máy hay chưa. Đây là IOC có giá trị cao khi hunting trên endpoint.

### 4. Chuỗi mật khẩu hoặc khóa nội bộ

```
WNcry@2ol7
```

Chuỗi này có thể được sử dụng để giải nén hoặc truy cập các thành phần tài nguyên của malware.

### 5. Lệnh hệ thống đáng ngờ

```
cmd.exe /c "%s"
icacls . /grant Everyone:F /T /C /Q
attrib +h .
```

Ý nghĩa:

- `cmd.exe /c "%s"`: thực thi một lệnh thông qua Command Prompt.
- `icacls . /grant Everyone:F /T /C /Q`: cấp toàn quyền cho nhóm `Everyone` trên thư mục hiện tại và toàn bộ thư mục con.
- `attrib +h .`: đặt thuộc tính ẩn cho thư mục hiện tại.

Đây không phải IOC định danh duy nhất, nhưng có thể dùng làm **behavioral IOC** hoặc chuỗi tìm kiếm trong EDR/Sysmon.

### 6. Đường dẫn đáng chú ý

```
%s\Intel
%s\ProgramData
```

Có thể cho thấy malware tạo hoặc sử dụng thư mục dưới `ProgramData`, có khả năng giả mạo tên thư mục `Intel`.

### 7. Crypto API

```
Microsoft Enhanced RSA and AES Cryptographic Provider
CryptGenKey
CryptDecrypt
CryptEncrypt
CryptDestroyKey
CryptImportKey
CryptAcquireContextA
```

Đây là các API mã hóa của Windows, thể hiện mẫu có khả năng tạo, nhập và sử dụng khóa để mã hóa hoặc giải mã dữ liệu. Chúng là **behavioral indicators**, không phải IOC định danh độc lập.

## Kết Luận

Đây là một loại malware rất nổi tiếng đó chính là ransomware và hơn nữa con ransomware này có tên là wannacry, các bạn có thể kiểm search về nó. Cái tên này rất nổi tiếng trong type ransomware, bài lab trên thật sự chỉ được run trong máy ảo và được dump ra thôi các bạn ạ, nhưng nó cũng khá hay đối với mình nên mình viết report lại thế thôi. Còn nữa, tất cả IOC thì mình lọc ra vào nhờ AI tổng hợp chứ thời đại công nghệ mà, không sài AI thì phí lắm các bạn ạ. Thế đến đây đủ rồi nhé, see  you lateeeeee.
