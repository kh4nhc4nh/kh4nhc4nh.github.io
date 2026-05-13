---
title: "Aruze Hunt"
date: 2026-05-13 07:00:00 +0700
categories: [Cloud, Aruze]
tags: [Aruze]
---
## Scenario :

A finance company's Azure environment has flagged multiple failed login attempts from an unfamiliar geographic location, followed by a successful authentication. Shortly after, logs indicate access to sensitive Blob Storage files and a virtual machine start action. Investigate authentication logs, storage access patterns, and VM activity to determine the scope of the compromise.

This is a attack and hacker was compromised user’s account and they login from an unfamiliar geographic location and they have some flags login failed and success. My task is analyze this case and give them IOC and proof to confirm this attack was occur. Other hand I will make it become clear and explain detail for it. The first thing you know needed is Azure is cloud computing such as AWS cloud and their logs are the same as the logs of AWS. Beside, Instead of analyze them i also answer question them give.. Let’s do it with me…..

Mình có thể dùng tiếng anh gà quá nên mình tạm bỏ qua viết bằng tiếng Anh nhaa các sếp, dưới đây là flow mà hacker đã thực hiện tấn công mà mình đã mô phỏng lại được ở dưới đây, các bạn có thể xem ở dưới đây….

```jsx
Attacker
   ↓
Compromise Azure account
   ↓
Enumerate cloud resources
   ↓
Access Blob Storage
   ↓
Find secrets / scripts
   ↓
Compromise thêm account
   ↓
Start VM
   ↓
Export database
   ↓
Create persistence
   ↓
Privilege escalation
```

### Q1 : As a US-based company, the security team has observed significant suspicious activity from an unusual country. What is the name of the country from which the attack originated?

Ở đây chính là điểm xuất phát của cuộc tấn công này, có một user đã bị compromise và login vào từ một country lạ, khác với vùng của login thông thường. Để xác định điều này mình sẽ thực hiện như sau, đầu tiên mình sẽ check những country có mặt trong đây :

![image.png](./assets/AzureHuntLab/1.png)

thì mình kiểm tra có hai country là hoạt có những hoạt động bình thường đó chính là US và France đều login rất bình thường

![image.png](./assets/AzureHuntLab/2.png)

Các bạn có thể quan sát thấy rằng đây là những hoạt động bình thường mà user đã thực hiện, nhưng vào ngày **5/10/2023** vào lúc **15:09:57** thì có một phát hiện bất thường như bạn có thể quan sát ở bên dưới đây : 

![image.png](./assets/AzureHuntLab/3.png)

hoạt động login từ country này và đã success trong việc login vào cloud này, tại sao mình lại xác định nó là bất thường mà không phải là France, đó chính là khi mình xem những hoạt động xảy ra từ trước thì đa số không có France hay là Germany, bên cạnh đó login từ France chỉ là hoạt động bình thường thôi và chỉ có hai có hai hành động được thực hiện ở country này, nhưng còn về Gemany thì sau khi login xong lai có nhiều hành động được thực hiện nên đây chính là country có hoạt động bất thường..

![image.png](./assets/AzureHuntLab/4.png)

User bị compromise và được sử dụng để login đó chính là alice.

### Q2 : To gain insights into the attacker's tactics and enumeration strategy, what is the name of the script file the attacker accessed within blob storage?

Thì bản chất của câu hỏi này là script file nào mà hacker đã access trong Blob storage, cho những ai chưa biết thì BLOB storage là các tài nguyên mà nằm trong các container của cloud này, bản chất của nó là như vậy thì các script file sẽ nằm trong các BLOB này nên là hacker sẽ thực hiện hành động access vào BLOB để xem có những gì xuất đó, dưới đây là query mình làm để kiểm tra xem là hacker đã access vào BLOD nào 

```jsx
azure.eventhub.category: "StorageRead" and azure.eventhub.operationName: "GetBlob"
```

![image.png](./assets/AzureHuntLab/5.png)

thì **service-config.ps1** chính là file mà hacker đã access vào để thực hiện những điều mờ ám sau đó, đây hình như là một file cấu hình của cloud này thì phải, nhưng file này có chức năng gì mình sẽ kiểm tra những điều sự kiện xảy ra tiếp theo. Vào lúc **15:14** ngày **5/10/2023** thì hacker đã thực hiện hành động read file config này ở dưới đây chính là active của chúng.

![image.png](./assets/AzureHuntLab/6.png)

bên cạnh đó hacker còn thực hiện những liệt kê sau các BLOB trong cloud này các bạn có thể quan sát ở dưới đây 

![image.png](./assets/AzureHuntLab/7.png)

có 3 script mà họ đã access vào trong đó salaries được hacker access rất nhiều và thực hiện nhiều hoạt động về BLOB salaries này. Còn về accountmetadata thì status của nó trả về là NotFound nên là scripts này không tồn tại. Những hành động này là thực hiện những list các scripts file có trong nó. Bên cạnh đó mình còn thấy được rằng hacker luôn access vào storage cactusstorage2023 để tìm các file nằm trong đó và liệt kê các contains trong đó, chắc hắn đây là một storage quan trọng.

### Q3 : Tracing the attacker's movements across our infrastructure, what is the User Principal Name (UPN) of the second user account the attacker compromised?

Để trả lời cho câu hỏi này thì mình đã quan sát hành động tiếp theo của hacker và thấy được rằng họ đã login vào một account có tên như sau ở dưới đây :

![image.png](./assets/AzureHuntLab/8.png)

đó chính là accout có tên là it.admin1, vào lúc **15:23** ngày **05/10/2023** thì hacker đã thành công login vào account này, để biết tại sao hacker có thể biết được password của account này thì mình đã xem các hoạt động trước đó và đặc biệt chú ý đến là hacker thực hiện behavior của họ với một file có tên là **service-cofig.ps1** rất nhiều lần và để compromise được account này thì hacker đã làm điều dưới đây:

![image.png](./assets/AzureHuntLab/9.png)

Vào lúc **15:18:41.543** ngày **05/10/2023** thì hacker đã thực hiện hành động GetBlobMetadata, Blob này là nơi chứa những dữ liệu nhạy cảm bên cạnh đó cũng chứa các user và các data sensitive khác mà không thể tiết lộ. Thật là đáng báo động cho việc này, hacker đã compromise được một account mới và có thể đây chính là một trong những account critical trong cloud này, vậy nên điều mình rút ra ở đây là không lưu những cái gì liên quan đến account hay liên quan đến hệ thống trên đây, vì có khả năng nếu bị attack thì bị leo thang và lateral movement chỉ còn lại là thời gian. Bên cạnh đó sau khi login success vào được account này thì mình còn phát hiện ra được rằng hacker đã ngay sau đó khởi chạy một VM :

![image.png](./assets/AzureHuntLab/10.png)

Vào lúc **15:24:53** ngày **05/10/2023** sau khi login thành công chỉ vọn vẹn 1p hacker đã thực hiện khởi chạy VM ngay lập tức và điều đó chứng minh rằng account này chính là một trong những account critical trong cloud này, bạn nên hiểu như thế này VM như kiểu một máy tính thật nhưng nó chỉ nằm trên cloud thôi và để khởi chạy nó thì account đó phải là account có permission thì mới làm được điều đó. VM mà hacker đã chạy đó là **DEV01VM** bên cạnh đó khi mình kiểm tra và soi kỹ log  này thì mình phát hiện được những điều còn đáng gờm hơn dưới đây :

![image.png](./assets/AzureHuntLab/11.png)

account này được xem là Owner của cloud này, nghĩa là họ quản lý cloud này và quản lý cloud của bộ phận IT, đây chính là điều đáng sợ nhất trong company đúng không. Còn ở phía dưới đây chính là toàn bộ alerts mà khi hacker start VM thì điều gì xảy ra sẽ nằm ở dười đây:

![image.png](./assets/AzureHuntLab/12.png)

Flow nó sẽ đi như sau, đầu tiên hacker sẽ gửi request để start VM này đó là 2 alerts dưới cùng các bạn có thể thấy, tiếp đến là 3 alerts tiếp theo chính là phần chứng minh rằng Resource mọi thứ đã sẵn sàng để chạy VM này rồi và alert tiếp đến là VM đã được chạy, cuối cùng không còn là hành động bình thường nữa mà hacker đã thực hiện xác nhận connect đó chính là những alerts cuối cùng thuộc về NETWORK, họ kiểm tra xem kết nối như thế nào, kết nối có được xác thực không, điều này dẫn đến việc hacker có thể duy trì và ở lâu được trong cloud vì thiết lập kết nối và xác nhận kết nối đã hoàn tất. Điều này chứng minh được hacker không chỉ có account để active nữa mà họ có thể điều khiển internal network rồi. 

Ngay sau khi access được account này thì lập tức hacker đã thực hiện những điều tiếp theo ở dưới đây như sau :

![image.png](./assets/AzureHuntLab/13.png)

Hacker đã thực hiện export một file DB ngay sau khi login success và thực hiện hành vi listkeys có trong Blob mà account này sở hữu, các bạn có thể quan sát bên dưới đây để xem hành động list key của hacker :

![image.png](./assets/AzureHuntLab/14.png)

vào lúc **15:33:24** cùng ngày hành động thực hiện list key của hacker đã success và điều này có nghĩa là hacker đã biết được trong đó chứa những cái gì…. Sau khi biết được có những file nào hay những script nào có ở trong đó thì ngay lập tức vào lúc **15:33:31** hacker đã thực hiện export file DB này với mục đích có thể là data exfiltration

![image.png](./assets/AzureHuntLab/15.png)

Như bạn có thể quan sát ở trên hacker đã thực hiện hành động export một file có tên là Customerdatadb, mình không quan tâm file này là gì nhưng đó là behavior bất thường và xác định được rằng họ đã thực hiện **exfiltratrion data**. Không dừng lại ở đó thì vào lúc **15:34:12.018** hacker đã thực hiện hành động **PUTBLOB** có thể vào file mà họ đã export, điều này có nghĩa là hacker đã chỉnh sửa content trong file đó, họ có thể overwrite hoặc là thêm những content khác vào file, không những thế vào lúc **15:34:12.072** thì hacker đã thực hiện xóa file này. Đây là một minh chứng rõ ràng có thể đang thực hiện dạng như ransomware để tống tiền bla các thứ.

![image.png](./assets/AzureHuntLab/16.png)

Vào lúc **15:42:53.536** hacker đã success trong việc add user vào cloud này để thực hiện hành vi duy trì của họ trong hệ thống. Vậy là đã xong quá trình hacker attack vào cloud này.

### Kết Luận

Đây chính là một bài LAB phân tích về cloud Azure, đây chỉ mới là một dạng phân tích tấn công, không không biết nó với các bạn là như thế nào nhưng đối với mình nó khá là một dạng có thể nói là rất mới và mình phải đào sâu hơn về nó nữa, không chỉ dừng lại ở việc phân tích logs mà còn phải hiểu rõ những function hoạt động và những điều kiện có trong đó như thế nào. Muốn phân tích một cái gì đó thì điều đầu tiên mình cần phải học đó chính là làm sao để hiểu nó hoạt động bình thường sẽ như thế nào thì khi đó mình mới hiểu được nó hoạt động bất thường là như thế nào. Tới đây cũng khá là mệt rồi hẹn các bạn ở các bài lab sau…..
