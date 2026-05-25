---
title: "Malacryption"
date: 2026-05-25 07:00:00 +0700
categories: [Cybersecurity, malware]
tags: [malware]
---
## Scenario

During routine monitoring at MalaCrypt company, a suspicious binary named "malware.exe" was found on a device. Initial checks of its hash values against threat intelligence platforms yielded no results, suggesting the attacker may have altered the malware to evade detection. As a security analyst, the next action is to investigate further, using alternative methods beyond hash-based detection, considering that attackers often modify hashes to bypass initial security checks.

Đây là một bài lab mình nghĩ nó khá là hay và sự khởi đầu cho những ai thích malware analysts, bên cạnh đó còn giúp mình nhận biết được sự khởi đầu cho việc phân tích malware là như thế nào, bản chất của nó ra làm sao và nếu bạn muốn hiểu thêm về nó thì bạn phải chăm chỉ nghiên cứu cũng như là học hỏi về nó nhiều hơn nữa, thật sự malware analysts là một mảng khá khó và đầy tính thử thách cũng như khá nhiều thứ đáng để chúng ta học hỏi vậy nên mình sẽ thục hành bài lab này ngay dưới đây.

## Practice

### Understanding the type of binary architecture allows us to determine the types of registers being used. What architecture is this binary built for?

Ở đây mình sẽ xác định architecture của file này là gì, điều này rất quan trọng trong việc phân tích malware bởi nó ảnh hưởng đến cách mà mình xác định tool cần dùng và cấu trúc của mỗi loại nếu x64 thì cấu trúc khác mà x86 thì cấu trúc nó lại khác vậy nên để xác định điều này mình sẽ sử dụng tool PEstudio để xác định nó và dưới đây là thứ mà mình xác định được :

![image.png](./assets/malacrypt/1.png)

mình quan sát thấy được rằng nó có architecture là x64, đây là một architecture mới thiên về windows 7 trở đi, và đó chính là target mà malware này đang nhắm tới.

### Executables are sometimes renamed or altered to evade detection or disguise their true purpose. What is the original name of the executable?

Ở đây malware đã ngụy trang và điều họ mong muốn là xác định được original name của malware, bởi vì malware nó rất thông minh bên cạnh đó nó còn có rất nhiều method để lẫn tránh việc analysts nó để trả lời câu hỏi này thì mình vẫn lại dùng PEstudio để tiếp tục kiểm tra 

![image.png](./assets/malacrypt/2.png)

Mình đã thấy được rằng original name của nó là DefaultViewer.exe là tên gốc của nó.

### Some DLL files are responsible for accessing Windows registries. Which DLL file is utilized to manipulate the Windows Registry?

Câu hỏi này có nghĩa là file DLL nào được dùng để thao tác với Registry, điều này khá quan trọng trong việc phân tích vì malware thường rất hay dùng Registry để persistence hoặc lưu config các thứ để thực hiện điều này mình sẽ làm như sau.

![image.png](./assets/malacrypt/3.png)

Mình vào module import của PEstudio và mình quan sát được group registry được thực hiện bởi library ADVAPI32.dll, bên cạnh đó khi mình search ở external internet thì họ cũng trả về cho mình điều này

![image.png](./assets/malacrypt/4.png)

Vậy điều này có nghĩa là dll này phục vụ cho việc action với registry và thêm nữa mình lại có thêm một kiến thức mới về điều này

### Certain strings may reveal specific information. What is the name of the Chinese messaging app discovered in the basic static analysis?

Để trả lời câu hỏi này mình sẽ sử dụng tool floss, mình sẽ miêu tả tool này ở dưới đây.

**FLOSS (the FLARE Obfuscated String Solver) is a popular, free cybersecurity tool used by malware analysts to automatically detect, extract, and deobfuscate hidden text from malicious programs**.

Bên cạnh đó mình sẽ sử dụng nó để làm việc phân tích ra những text bị ẩn bên tronng file malicious đó và đây là cách mình thực hiện và làm nó :

```jsx
 ─────────────────────────
  FLOSS STACK STRINGS (3)
 ─────────────────────────

Software\DingTalk
Software\Tencent\WeChat
Software\TelegramDesktop\Capabilities\UrlAssociations
```

Đây chính là một phần nhỏ result mà mình nhận được và mình cũng xác định được là Wechat chính là app mà con malware này sử dụng để messaging. Đây chính là những Stack chứa strings để execution khi mà malware được run

### The Windows API can be used for malicious purposes. Which Windows API is used to destroy previously generated encryption keys?

Ở đây họ muốn hỏi Windows API nào đã được sử dụng để cung cấp encryption keys, nếu vậy có rất nhiều lý do để malware sử dụng API này bên cạnh đó để hiểu rõ nguyên nhân tại sao thì mình sẽ lần lại dấu vết từ mã hash của malware để xem nó là gì thì mình mới xác định được mục đích của nó là gì, mình đã kiểm tra signature của nó nhưng có lẽ nó khá mới nên chưa thể xác định được nhưng mình vẫn sẽ tiếp tục phân tích nó vầ để trả lời câu hỏi này thì mình sẽ show ở dưới đây

![image.png](./assets/malacrypt/5.png)

Đây là toàn bộ API mà liên quan đến điều mà đang cần và câu trả lời cho câu hỏi này đó chình là CryptDestroyKey

### Knowing the attacker's IP can help trace the source of the attack and gather information about their location and network. What IP address is found in the executable that belongs to Hong Kong?

Mình sẽ kiểm tra strings của nó trong PEstudio 

![image.png](./assets/malacrypt/6.png)

Đây chính là IP mà mình đang tìm kiếm ngoài ra khi mình check infor của ip này thì mình nhận được lại như thế này

![image.png](./assets/malacrypt/7.png)

Đây chính là câu trả lời của câu hỏi này.

### In dynamic analysis, we examine the behavior of the malware and identify any suspicious activities, What message is displayed on the screen when the binary is executed?

Đây chính là phần mà mình đang nhắm tới và cũng sẽ thực hiện tiếp theo đây mình khà là thích phân tích malware vậy nên mình sẽ chú tâm vào phần này khá nhiều và mình cũng khá thiên về DFIR nữa nên mình khá là thích nó.

![image.png](./assets/malacrypt/8.png)

Khi mình run malware thì mình nhận được kết quả như trên bên cạnh đó mình còn đoán được đây là một con malware mà được cyberdefender dev ra để thực hiện lab thôi nhưng đây cũng là một bài lab khá hữu dụng cho những ai mới vào analysts malware. Bên cạnh đó mình còn có process của malware khi run nó ở dưới đây 

![image.png](./assets/malacrypt/9.png)

Khi run malware thì chúng ta nên chờ vài phút để nó thực hiện hết được các function của nó, lúc đó chúng ta mới biết được hướng đi của nó là như thế nào và connect như thế nào, vì đây là con malware được các anh bên cyberdefend dev nên cũng chẳng có điều gì nguy hiểm lắm, hôm sau mình sẽ có bài lab về malware thật và run trên vm isolate hostt của mình.

### Identifying the executed DLLs gives us insight into the attacker's strategies and goals. What is the name of the first DLL file that is loaded after the binary is executed?

Ở đây họ muốn mình give cho họ câu trả lời sau khi run malware thì file dll đầu tiên được load là file nào, mình quan sát thấy được rằng đó là file ntdll.dll bạn có thể quan sát ở hình ảnh của câu hỏi trên khi file malware này được open thì ngay sau đó file dll này được load lên, bản chất của file này là gì thì mình sẽ để ở dưới đây các bạn có thể đọc

`ntdll.dll` (NT Layer DLL) is a critical core operating system file that acts as the lowest-level interface between user-mode applications and the Windows kernel. It handles fundamental system services like memory management, process/thread creation, and system calls

Nếu không hiểu thì  để mình giải thích một chút nó là một dll kết nối giữa user và kernel, vậy là xong câu hỏi này rồi đó.

### Tracing Windows API calls helps understand what the malware is intended to do by analyzing specific patterns or arguments used in these calls. What is the first Windows API call made from the function that is called from the **main function**?

Ở đây chính là phần hay mà khó nhất trong đây, nó liên quan đến code cũng như phải có kiến thức kha khá thì mới hiểu được, bên cạnh đó phải có niềm đam mê thì mới không nản khi không tìm ra được đáp án của nó, giờ thì hãy cùng mình thực hiện nó nào. Thay vì sử dụng IDA thì mình sẽ sử đụng cutter nó cũng tương tự như IDA đó.

Thì họ hỏi mình rằng API nào được call từ main function, để xác định được điều này trước tiên mình phải xác định được main function nằm ở đâu và mình đã tìm ra nó 

![image.png](./assets/malacrypt/10.png)

Đây chính là phần chứa main function của con malware này và bên trong main fucntion nó sẽ chứa những điều như sau

![image.png](./assets/malacrypt/11.png)

mình phát hiện main function còn call một function khác thì mình sẽ đi tiếp để phân tích thằng mà được main function call là API nào

![image.png](./assets/malacrypt/12.png)

Đây chính là một phần có trong function này và API được call trong function này đó là GetEnvironmentVariableA chính là điều mà mình tìm kiếm.

Đó là những gì mà mình tích hợp được trong khi làm bài lab này cám ơn các bạn đã giành thời gian theo dỡi mình phân tích.
