---
title: "Malware được giấu trong sheet của file Excel"
date: 2026-06-27 07:00:00 +0700
categories: [Malware, Offical]
tags: [Excel, Sheets]
---
## Scenario :

Recently, we have seen a resurgence of Excel-based malicous office documents. Howerver, instead of using VBA-style macros, they are using older style Excel 4 macros. This changes our approach to analyzing these documents, requiring a slightly different set of tools. In this challenge, you, as a security blue team analyst will get hands-on with two documents that use Excel 4.0 macros to perform anti-analysis and download the next stage of the attack.

Đây là một trường hợp về malware phishing của office, ở trường hợp này họ có give cho mình 2 file xlsx và đây chính là định dạng hay còn gọi là extension của file excel, hai file này đều được nhúng sheet có chứa payload độc hại ở trong đó và việc của mình chỉ là đi phân tích tĩnh về nó mà thôi, nhưng để hiểu sâu hơn thì ngoài việc trả lời câu hỏi mà họ đưa ra mình sẽ giải đáp các phần thắc mắc về nó. Ngoài việc phân tích thì mình còn nhìn nhận ra được rằng các cuộc tấn công phishing đều tới từ file office và đặc biệt là các file có chưa VBA bản cũ, chúng rất dễ để attacker lợi dụng và nhấn payload vào trong đó.

## Thực Hành :

### Example 1 :

#### Sample1: What is the document decryption password?

Ở file này hacker hay author đã thực hiện hành động encryption content trong file hoặc các VBA bằng một password nhất định, hành động này nếu file bình thường thì có thể là cần sự xác thực hoặc giữ độ authentication cho file, còn nếu là một file chứa payload và đặc biệt là một file malware thì điều này có thể chứng minh rằng nó đã encryption payload, để crack được password file này mình sẽ sử dụng tool msoffcrypto-tool để crack nó và mình sẽ thực hiện như sau.

![image.png](./assets/xlmmacros/1.png)

Mình đã kiểm tra file này và file này thực sự bị encryption bây giờ mình sẽ thực hiện crack nó dưới đây.

![image.png](./assets/xlmmacros/2.png)

Tại sao mình có thể crack được và tìm ra được pass của file này, bởi vì đây chỉ là một bài lab nên password này public nằm trong list của file crack nên nó rất nhanh chóng tìm ra, chứ thực tế ở thì có thể không crack được mà phải sử dụng đến các cách phân tích khác mới thực hiện và hiều được password của nó.

#### Sample1: This document contains six hidden sheets. What are their names? Provide the value of the one starting with S.

Ở file này có một document chứa 6 trang tính ẩn và bị encode bằng password, 6 trang thì một hoặc nhiều trong số đó có thể là payload hoặc có thể là cả 6 trang đó đều là payload vậy nên để phân tích được thì mình phải xác định được đó là document nào, để xác định được thì mình sẽ thực hiện nó như sau.

![image.png](./assets/xlmmacros/3.png)

Thì mình xác định được có 2 đối tượng rất đáng nghi ngờ đó là **SOCWNEScLLxkLhtJp** và **OHqYbvYcqmWjJJjsF**, một phần là tên của các sheets này rất lạ so với thông thường, có thể chúng được chèn vào hoặc bằng cách khác nhưng để kiểm tra chính xác và để xác định “**chắc chắn**” được cái nào chứa payload thì mình sẽ tiếp tục phân tích ở dưới đây.

![image.png](./assets/xlmmacros/4.png)

Đây chính là IOC mà có thể xác định được sheet **SOCWNEScLLxkLhtJp**  chính là sheet chứa payload và sau khi sheet này được run thì payload được call và thực hiện Download file từ url “http://rilaer.com” này và thực hiện download file về, file này có thể là một file malware khác của hacker.  Khi thực hiện download các file từ URL đó về thì sẽ được lưu thành nhiều tên khác nhau, ngoài ra dưới đây là một số những IOC mà mình bắt được 

![image.png](./assets/xlmmacros/5.png)

Có khá nhiều IOC về url và đặc biệt được cảnh báo là XLM macro được phát hiện là chứa malicious code, đây chính là phần đáng để tâm và đặc biệt là nên chú ý payload của nó. Như các bạn đã thấy trên payload của con malware này nó sẽ tạo ra thư mục có name như sau **C:\jhbtqNj\IOKVYnJ** và sau khi tạo thực mục này thì nó sẽ thực hiện download file từ url và chuyển về thư mục này. Ngoài ra khi mình sử dụng hash để check OSINT của nó thì mình phát hiện ra được một số điểm như sau 

![image.png](./assets/xlmmacros/6.png)

Khi mình check url thì nó ra kết quả đây là một loại malware Dridex, dành cho những ai chưa biết thì malware dridex thì lên mạng mà tra nhaaaa.

### Example 2 :

#### Sample2: This document has a very hidden sheet. What is the name of this sheet?

Ở câu hỏi này cũng giống như câu hỏi đầu của example 1 thì sẽ có những sheet nó chứa payload và bên cạnh đó để xác định được điều này thì mình cũng sẽ làm theo cách tương tự và dưới đây là cách mình thực hiện nó.

![image.png](./assets/xlmmacros/7.png)

Đây như có bạn có thể thấy ở IOC này mình đã phát hiện ra được rằng CSHykdYHvi chính là sheet hidden đã bị ẩn và ở đây có một dạng payload được thực hiện nhưng cũng là download file từ url hay còn gọi là server của hacker để tải payload hoặc file nào khác dạng như exe. Bên cạnh đó payload này còn thực hiện hành động kiểm tra registry, có thể nói payload này sẽ thực hiện

![image.png](./assets/xlmmacros/8.png)

Họ thực hiện run process reg.exe, đây chính là process thực hiện về hành động for registry, và đây cũng chính là payload chính của malware này, tất cả đều được thực hiện từ sheet này và sẽ download payload.

#### Sample2: This document uses reg.exe. What registry key is it checking?

Ở đây thì đã xác định là họ sẽ thực kiện kiểm tra registry key, vậy nên để tiếp tục xác minh điều này mình sẽ check toàn bộ file để thực hiện kiểm tra toàn thể sheet cũng như file này xem có IOC nào chứng minh điều này hay không.

![image.png](./assets/xlmmacros/9.png)

Đây chính là key mà họ sẽ thực hiện kiểm tra VBAWarnings, tại sao họ lại kiểm tra điều này thì dươi đây chính là câu trả lời dành cho các bạn để hiểu được bản chất của nó cũng như chi tiết về nó.

VBAWarnings quyết định Excel xử lý VBA macro như thế nào, như các bạn có thể quan sát  thấy được rằng giá trị cươi cùng là 2 thì điều này có nghĩa tắt run macro nhưng sẽ hiện thông báo để user biết và bấm enable content, điều này khá là khó hiểu nhưng mình sẽ để giá trị của dưới đây để các bạn dễ hiểu hơn :

| `1` | Cho phép toàn bộ macro chạy, không cảnh báo 

| `2` | Tắt macro mặc định nhưng hiện cảnh báo để người dùng bấm **Enable Content** 

| `3` | Chỉ cho macro có chữ ký số đáng tin chạy 

| `4` | Tắt toàn bộ macro, không cảnh báo 

thì ở đây giá trị của nó là 2 thì đây là giá trị default của Excel nhưng nếu user nhấn enable được thông báo thì payload sẽ được thực thi. Còn về phần HKCU\Software\Microsoft\Office\16.0\Excel\Security\Trusted Documents thì sau khi user nhấn enable thì tài liệu sẽ được chuyển vào folder Trusted Documents và payload sẽ được thực thi một cách thoải mái mà không lo bị prevent hay detect.

Ở dươi đây có một vài IOC mà họ đã trích xuất giúp mình các bạn có thể quan sát và kèm theo đó mình sẽ giải thích chi tiết về flow của nó hơn

![image.png](./assets/xlmmacros/10.png)

Như các bạn có thể thấy được có khá nhiều IOC được phát hiện đặc biệt là về phần URL và các process được thực hiện kèm theo đó có một file cấu hình 1.reg. Bây giờ mình sẽ thực hiện giải thích bằng flow của nó như dưới đây

![image.png](./assets/xlmmacros/11.png)

Mình đã khoanh vùng những IOC đó ở trên các bạn có thể quan sát, khi open file này ra thì hành động đầu tiên sẽ là check key registry để xem là value của **VBA warning** là bao nhiêu, nếu là một thì payload sẽ được thực thi còn số 2 thì sẽ hiện thông báo và nếu user nhấn enable thì file sẽ được chuyển vào trust document và flow sẽ tiếp tục, con malware này khá là khôn nó còn thực hiện check **sandbox**, nó gắn giá trị **00001** để kiểm tra nếu là **sandbox** thì  payload không được run và nó như là môt file bình thường còn nếu là host thật thì nó sẽ download file từ url **https://ethelenecrace.xyz/fbb3** và sẽ được lưu file thành **bmjn5ef**.html, đây chắc chắn là một file dll nhưng được giải dạng làm extension html, tại sao mình lại nói như thế bởi vì ngay sau đó payload sử dụng **rundll32**.exe để thực hiện run file **bmjn5ef**.html này, **rundll32** là một process dùng để load các dll vào một process nào đó để thực thi, có rất nhiều kiểu attack liên quan đến nó vậy nên mình có thể khẳng định được là như thế. Còn sau đó nó như thế nào mình không rõ tại mình chỉ phân tích static về nó  mà thôi còn để run nó thì mình đã setup sẵn sandbox rồi và bài lab sau mình sẽ kiểm tra xem nó như thế nào.

### Kết luận :

Đây là một thể loại malware về microsoft office và đặc biệt liên quan đến excel thời đại cũ và đa số họ dùng để phishing và thực hiện những hành động mờ ám, muốn ngăn chặn được điều này thì trước tiên hết chúng ta cần phải training nhân viên của công ty bên cạnh đó mail server của company phải hết sức cẩn trọng và thiệt lập các rules rõ ràng để xác định được những domain nào đáng để cảnh báo, mọi cuộc tấn công phishing không hợp pháp đều mang tới mục đích xấu cho company và ảnh hưởng rất nhiều tới company nên bảo mật hay an toàn trong công thật sự rất cần thiết, ngoài ra cần phải monitoring liên tục mọi hành động đều phải có sự mạch lạc rõ ràng ngăn chặn mọi thủ đoạn của hacker cũng như của threat. Mình mỏi tay rồi mình nghỉ đây, see you lateeeeeee =))
