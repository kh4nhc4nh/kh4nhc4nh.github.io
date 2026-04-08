---
title: "Analyze Hash for Malware"
date: 2026-04-07 09:00:00 +0700
categories: [Cybersecurity]
tags: [malware]
---

## Scenario :

You are an SOC analyst on the SOC team at Managed Server Provider TrySecureMe. Today, you are supporting an L3 analyst in investigating flagged IPs, hashes, URLs, or domains as part of IR activities. One of the L1 analysts flagged two suspicious findings early in the morning and escalated them. Your task is to analyse these findings further and distil the information into usable threat intelligence.

Flagged IP: **101[.]99[.]76[.]120**

Flagged SHA256 hash: **5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f**

Mình sẽ mô tả lại một chút về điều này : thì họ muốn mình phải phân tích được threat có ẩn chứa những gì và trả lời một số câu hỏi của họ, bên cạnh đó mình đóng vai là một SOC L1 để phân tích cũng như hỗ trợ cho L3 về những vấn đề này thì họ có give cho mình 2 flag trên và muốn mình tìm ra cũng như báo cáo lại những IOC mà mình đã tìm thấy được về sự nguy hiểm của nó.

## Thực Hành

### SHA256 Hash : **5d0509f68a9b7c415a726be75a078180e3f02e59866f193b0a99eee8e39c874f**

Đầu tiên mình sẽ thực hiện kiểm tra xem hash này thuộc về cái gì và nó thuộc loại malware nào, sâu xa hơn nữa là hành động của nó như thế nào để biết được có những gì liên quan tới nó

![image.png](./assets/invite/1.png)

Khi mình đưa hash này lên virustotal thì mình nhận lại được kết quả khá là đáng kinh ngạc, nó là một loại  malware trojan và có score khá là cao ở trên đánh giá của virustotal. Dưới đây là những câu hỏi mà họ đặt ra và muốn mình giải quyết cũng như báo cáo lại với họ

#### What is the name of the file identified with the flagged SHA256 hash?

Tên của nó đã được xác định ở trên và nó có tên là syshelpers.exe. Khi mình search nó ở trên [any.run](http://any.run) thì kết quả họ trả về cho mình 

![image.png](./assets/invite/2.png)

Nó là một dạng malware mà thực hiện remote access control có nghĩa là nó cho phép hacker điều khiển từ xa máy của mình hơn nữa bên cạnh đó mình còn phát hiện được family malware của nó chính là **AsyncRAT**.

#### What are the execution parents of the flagged hash?

Họ muốn mình xác định được parents của nó là những gì, thì mình có câu trả lời khá rõ ràng khi mình vào relationship của nó

![image.png](./assets/invite/3.png)

thì đây chính là hai process parents của nó, bạn có nghĩ tại sao họ lại bắt mình xác định điều này không ? . Có thể bạn chưa biết nhưng để mình giải thích kĩ hơn về nó, nếu mình xác định được parents của nó thì sẽ biết trong hệ thống malware này được khởi tạo từ đâu và đó gọi là initial access chính là cuộc khởi đầu của attack việc xác định điều này không chỉ giúp mình nhận diện được file gốc của nó trong hệ thống mà còn biết malware sẽ làm gì tiếp theo với điểm khởi đầu mình đã có được.

#### What is the name of the file being dropped?

Họ muốn mình xác định được file đã được drop, điều này có nghĩa là các file mà malware đã tạo trên hệ thống của chúng ta là file gì, bản chất của nó là vậy cái này rất quant rọng trong việc phân tích bởi vì nếu malware nó tạo ra một file khác mà cũng là malware có chức năng khác thì việc nhận biết điều này sẽ giúp ích cho chúng ta rất nhiều trong việc phân tích nó.

![image.png](./assets/invite/4.png)

đây là những file mà malware đã dropped trong hệ thống khi nó initial được hệ thống của mình.

#### Research the second hash in question 3 and list the four malicious dropped files in the order they appear (from up to down), separated by commas.

đây là một câu hỏi khá là thú vị và rất quan trọng trong việc phân tích của mình, có nghĩa là dựa vào file parent của nó để nhận biết các file đã được dropped trong hệ thống, ở đây họ muốn mình kiểm tra file parent thứ 2 của nó đó là **installer.exe** 

![image.png](./assets/invite/5.png)

Đây cũng là một file malware trojan, nó ngụy trang thành một file install của một app hay một process nào đó để đánh lừa người dùng. 

![image.png](./assets/invite/6.png)

Đây chính là những file mà được dropped khi file malware này run trên system, có khá nhiều children file malware khác được drop, file đầu tiên thì vừa mới được phát hiện nên là không thể xác định được tên của nó như thế nào, thì mình đã xác định tên của nó ở dưới đây

![image.png](./assets/invite/7.png)

đây là tên của nó, bởi vì report vừa mời được gần đây nên là cách họ viết name cũng khá là dài và thú vị nhưng thực chất thì đây chính là tên của nó =))

### Kết Luận :

Sau khi có được những thứ mà họ mong muốn thì mình sẽ kết luận một vài vấn đề tổng quan như sau. Malware này là một loại thuộc AsyncRAT malware nó là một loại malware trojan thường dùng để tạo backdoor cho hacker hoặc một vài thứ khác, bên cạnh đó hệ thống file parent của nó rất phức tạp khi được cài và run vào hệ thống thì sẽ tạo ra rất nhiều children malware khác và có nhiều chức năng khác nhau, bên cạnh đó đây được nhận xét là một loại malware khá mới nó bởi vì file children được respond gần đây là searchHost.exe vào ngày 21/2/2026. Ngoài ra khi nó vào được hệ thống thì nó sẽ cho cấp quyền cho hacker remote máy của user từ xa đây là một loại tấn công **Remote Access Tools** được kí hiệu là T1219. 

### Flagged IP: **101[.]99[.]76[.]120 check IP**

đền phần tiếp theo chúng ta sẽ qua tới check IP và tìm kiếm IOC để báo cáo cho họ về những gì mình đã tìm thấy được ở trong phần này có nhiều câu hỏi khá hay và một vài câu hỏi rất khó nếu mình không thực sự hiểu được bản chất của nó là gì.

![image.png](./assets/invite/8.png)

Khi mình kiểm tra IP này thì thấy đây là một IP của server khá là nguy hiểm và các vedor của Virustotal đánh giá khá là kinh khủng về IP này, bên cạnh đó mình còn check xem nó được tạo ở đâu và những thứ liên quan tới nó 

![image.png](./assets/invite/9.png)

Mình có check ra được IP này và nó được tạo bởi công ty 

![image.png](./assets/invite/10.png)

Đó chính là những gì liên quan tới IP này thì không lan man nữa bây giờ mình sẽ đi vào  phần yêu cầu của họ chứ nói nhiều quá thì xao nhãng điều này mất tiêu =))

#### Analyse the files related to the flagged IP. What is the malware family that links these files?

Làm sao để mình xác định được file và biết được file malware family gắn với file đó thì điều này mình sẽ đi kiểm tra những gì có liên qua tới IP này và check xem nó có liên quan tới những file nào

![image.png](./assets/invite/11.png)
Đây chính là những file liên quan tới địa chỉ IP này, thật bất ngờ đó là có file installer.exe một file parent của file mà họ đã cung cấp hash cho mình, để xác thực điều này mình sẽ đi kiểm tra xem có thật sự như vậy không và hai file này liệu có trùng hash với nhau hay không.

Sha256 : 918b56cdda24395066e38da635914bcef50bada2bc03b5c1342c3411b70678ad ( hash của file trên )

Sha256 : fa102d4e3cfbe85f5189da70a52c1d266925f3efd122091cdc8fe0fc39033942 ( hash của file parent )

thì thật bất ngờ chúng không như mình nghĩ nhưng có một điều khá thú vị đó là khi mình kiểm tra file instraller.exe này thì mình phát hiện file drop của nó đều có giống nhau và file parent của nó cũng giống nhau bên cạnh đó nó còn có IP giống connect tới cũng như nhau, vậy nên nó thuộc asyncrat

#### What is the title of the original report where these flagged indicators are mentioned? Use Google to find the report.

Ở câu hỏi này mình thấy khá là hay đó chính là đã có những báo cáo về con malware này nhưng họ muốn mình tìm ra báo cáo đó cũng như chi tiết những gì có trong báo cáo đó

![image.png](./assets/invite/12.png)

đây chính là bản báo cáo mà mình nhận được ở phần community của IP này trên virustotal

⇒ From Trust to Threat: Hijacked Discord Invites Used for Multi-Stage Malware Delivery đây chính là câu trả lời, khi mình đọc bản báo cáo này thì có rất nhiều thứ đáng để mình suy nghĩ và phân tích sâu hơn dưới đâu là những gì mình kết luận được từ bản báo cáo này và cách thức hành động của con hacker nó sẽ là gì 

Hacker đã sử dụng nền tảng discord để điều hướng người dùng, bằng cách họ sẽ gửi link mời vào discord hoặc là kết bạn discord gì đó… không có gì đáng nói ở đây nhưng link discord mà họ gửi đã bị Hijecked có nghĩa là khi bạn nhấn vào link đó nó bí mật đẩy bạn sang một server malicious mà hacker đã chuẩn bị từ trước. Bạn cứ hiểu nô nam là như thế đó là những gì mà hacker đã dùng để đánh lừa một cách rất tự nhiên và hết sức tinh vi không có sự nguy hiểm nào ở đây.

#### Which tool did the attackers use to steal cookies from the Google Chrome browser?

Tool mà hacker đã dùng để steal cookies đó chính là ChromeKatz, thì tool này có thể giúp hacker bypass được và steal được cookies của Chrome

#### Which phishing technique did the attackers use?

Technique mà hacker đã dùng đó chính là ClickFix, đây là phương thức khá là phổ biến ở trong phishing khi mà độ tin tưởng của người dụng bị lạm dụng thì đó chính là lúc mà hacker có thể thực hiện được điều này, có rất nhiều người mắc sai lầm đó chính là tin quá nhiều vào thế giới ảo. Niềm tin của họ không có sự nghi ngờ hay bất cứ điều gì khác, có thể là họ không biết những điều này, có thể họ là những người tốt và muốn giúp đỡ người khác hoặc muốn kết nối cộng đồng qua cách này cách khác nhưng sự tin tưởng và nồng nhiệt của họ đã bị chính những con người tràn đầy sự thông minh và tinh vi đánh lừa một cách quá dễ dàng ở thế giới ảo. 

### Tổng Kết :

Thực chất thì IP mà họ give cho mình chính là server của những file đó, bạn phải hiểu là malware không bao giờ hành động một mình, nó có rất nhiều domain và rất nhiều IP khác nhau tràn lan trên toàn thế giới việc tìm ra được IP và domain của nó thì chúng ta sẽ đi tới những bài lab sau. Bên cạnh những điều này thì việc có kỹ năng threat intelligent cũng rất quan trọng đối với mình và bạn. Giúp cho chúng ta điều tra được rất nhiều thứ mà chúng ta chưa biết tới. Dưới đây là các đường link giúp bạn tìm ra những điều này :

https://www.nslookup.io/ 

https://ipinfo.io/

https://client.rdap.org/

https://www.shodan.io/

Đến đây tay cũng mệt rồi, các bạn làm việc tiếp đi, tôi đi học đâyyyyyy.
