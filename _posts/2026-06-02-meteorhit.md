---
title: "Compromise Active Directory"
date: 2026-04-08 07:00:00 +0700
categories: [AD]
tags: [Compromise]
---
## Scenario

A critical network infrastructure has encountered significant operational disruptions, leading to system outages and compromised machines. Public message boards displayed politically charged messages, and several systems were wiped, causing widespread service failures. Initial investigations reveal that attackers compromised the Active Directory (AD) system and deployed wiper malware across multiple machines.

Fortunately, during the attack, an alert employee noticed suspicious activity and immediately powered down several key systems, preventing the malware from completing its wipe across the entire network. However, the damage has already been done, and your team has been tasked with investigating the extent of the compromise.

You have been provided with forensic artifacts collected via KAPE SANS Triage from one of the affected machines to determine how the attackers gained access, the scope of the malware's deployment, and what critical systems or data were impacted before the shutdown.

Đây là một bài lab liên quan về AD ( Active Directory ) thì bản chất của AD là hệ thống quản lý danh tính ( Identify Management ) và quản lý tài nguyên tập trung do Microsoft phát triển cho doanh nhiệp, bản chất của nó là vậy nó là đầu não quản lấy về tài nguyên và danh tính. Thì hacker đã xâm nhập được vào hệ thống và đã compromise được system AD. Bên cạnh đó hacker đã lợi dụng để triển khai phần mềm độc hại và nhiệm vụ của mình là tìm ra những IOCs của nó và report lại với họ. Để bên cạnh thiết lập Policy thì còn rút kinh nghiệm cho những lần sau, kèm theo đó mình sẽ trả lời câu hỏi và giải thích chi tiết về những vấn đề đó. 

## Practice

Ở đây họ có đưa cho mình một folder C, mà folder này chính là disk của victim đã bị compromise bởi hacker, mình sẽ sử dụng tool như Registry Explorer, bên cạnh đó mình sẽ đọc event trong logs eventview để kiểm tra, ở đây mình không chỉ làm để biết mà mình làm còn để hiểu hành động của nó là như thế nào trên đó và attack chain của hacker là như thế nào khi compromise được

### The attack began with using a Group Policy Object (GPO) to execute a malicious batch file. What is the name of the malicious GPO responsible for initiating the attack by running a script?

Ở đây attacker đã sử dụng Group Policy Object để thực hiện thực thi file malicious, họ muốn mình đưa ra được bằng chứng là tên của GPO mà chịu trách nhiệm cho việc này, thì dành cho những ai chưa biết GPO trong AD là cơ chế ép toàn bộ máy tính và người dùng trong domain phải tuân theo một bộ cấu hình tập trung, nói cách khác nó là một tập hợp các cấu hình được lưu trong AD.

!image.png

Trước hết mình sẽ kiểm tra cấu hình của nó và mình sẽ kiểm tra hive, thì tại sao mình lại phải kiểm tra hive bởi vì bản chất các cấu hình về hệ thống của nó sẽ được lưu ở trên hive của nó bởi vì ở đây sẽ lâu cấu hình state sẽ được lưu trên hive và mình sẽ sử dụng Register Explorer để check nó. Ở đây có rất nhiều hive nhưng mình sẽ kiểm tra hive SOFTWARE tại vì đây chính là nơi lâu cấu hình hình của nó, trong hive có rất nhiều register và một nó sẽ có mỗi role khác nhau trong đó nên là mình sẽ kiểm tra file register này.

!image.png

Đây chính là những gì mình nhận được sau khi mình dump nó ra và mình sẽ vào path này để check 
`Microsoft\Windows\CurrentVersion\Group Policy\Scripts\Startup\0`

!image.png

Đây chính là những gì mà mình nhận được và kết quả đó là DeploySetup. Như các bạn đã nhìn trên path của nó có phần Group Policy và mình còn kiểm chứng thấy được rằng họ đã dùng script để thực hiện file malicious này trên đó.

### During the investigation, a specific file containing critical components necessary for the later stages of the attack was found on the system. This file, expanded using a built-in tool, played a crucial role in staging the malware. What is the name of the file, and where was it located on the system? Please provide the full file path.

Ở câu hỏi trên mình đã xác định họ đã run script để attack system vậy nên ở phần này thì mình sẽ xác định xem tên file script mà họ đã chạy là gì và full path của nó là như thế nào. Dựa vào câu hỏi trên thì mình sẽ kiểm tra trong eventview của victim có chứa logs của powershell hay là cmd gì hay không.

!image.png

Thì mình có phát hiện được là ở đây có một file event mang tên là powershell, mình sẽ kiểm tra logs và phân tích logs ở trong đó và hacker đã làm những điều gì.

!image.png

Thì ở đây mình đây mình đã xác định được file nào đã được họ run và các bạn có thể quan sát ở phía trên đó là file setup.bat và ngoài ra mình còn phát hiện được họ còn có sysmon thì mình sẽ check xem trong file này nó sẽ rõ hơn bởi vì sysmon là nơi tổng hợp các loại logs trong đó mà system input được. Khi mình kiểm tra trong file logs sysmon thì mình có phát hiện được khoảng thời gian đầu tiên mà hacker đã run file setup.bat đó.

!image.png

Vào lúc 4:04:+ ngày 24/09/2024 hacker đã dùng cmd để run file setup.bat đó ngoài ra khi run file setup.bat đó xong thì đây là children của file setup.bat đó.

!image.png

Vào lúc 4:04:+ PM cùng ngày thì sau khi run file setup.bat đó xong một process có tên là schtasks đã thực hiện một instruction như sau :

```jsx
schtasks  /CREATE /SC ONCE /ST 09:08:13 /TN "mstask" /RL HIGHEST /RU SYSTEM /TR "\""C:\ProgramData\Microsoft\env\env.exe"\" C:\temp\msconf.conf"
```

Schtasks chính là process lập lịch của windows và điều này có nghĩa là hacker thực hiện lập lịch để run file có tên là env.exe vào lúc 09:08:13 điều này có nghĩa malware không chạy tức thì mà nó schtasks để chạy sau đó điều này giúp hacker và tool của họ tránh detection khi AV hoạt động hay khi có dấu hiệu gì đó. Câu trả lời của câu hỏi này là đây C:\ProgramData\Microsoft\env\env.exe, đây chính là full path và tên file malware mà hacker đã thực hiện.

### The attacker employed password-protected archives to conceal malicious files, making it important to uncover the password used for extraction. Identifying this password is key to accessing the contents and analyzing the attack further. What is the password used to extract the malicious files?

Đây là một phần khá là hay, bởi vì hacker đã dùng password để archives file mà họ đã steal được. Ở đây họ muốn mình cung cấp password mà hacker đã dùng để compress file, dưới đây chính là cách mình check bên cạnh đó đây là tư duy của mình để xác định như thế nào để loại bỏ false positive trong logs. Đầu tin khi đến điều này thì mình còn xác định được một điều khá là đáng nghi ngờ khi file script được run đó là

```jsx
C:\Windows\system32\cmd.exe /c powershell -command "(Get-Date).AddMinutes(3.5).ToString('HH:mm:ss')"
```

Đây chính là một điều khá là đáng báo động khi malware lập lịch sau 3.5 phút để run một file hoặc script trong malware đó. Bên cạnh đó dưới đây cũng là một reg flag mà mình cần phải để ý 

```jsx
reg add "HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Windows\CurrentVersion\PersonalizationCSP" /v LockScreenImagePath /t REG_SZ /d C:\temp\mscap.jpg /f
```

Và không lăn tăn và lu mờ nữa thì mình sẽ tiếp tục thực hiện analysts logs để tìm ra pass mà malware đã dùng để compress data là 

```jsx
"Rar.exe"  x "C:\ProgramData\Microsoft\env\ms.rar" -p hackemall
```

Đó chính là hackemall và họ dùng RAR để compress file đó. Tất cả đều có process parent là setup.bat hết nên các bạn chỉ cần tìm những thứ liên quan với nó là các bạn có thể phát hiện được những gì file này đang làm và những điều gì đang xảy ra trong real time.

### Several commands were executed to add exclusions to Windows Defender, preventing it from scanning specific files. This behavior is commonly used by attackers to ensure that malicious files are not detected by the system's built-in antivirus. Tracking these exclusion commands is crucial for identifying which files have been protected from antivirus scans. What is the name of the first file added to the Windows Defender exclusion list?

Ở câu hỏi này họ muốn mình tìm ra file mà đã được add vào Windows Defender exclusion, bản chất của điều này không phải là run file hay gì hết mà là nó thực hiện hành động giúp cho file malware không bị xóa và tránh detection bởi Windows Defender và có thể liên tục nằm trong máy của victim nếu không phát hiện được.

```jsx
powershell  -Command "Add-MpPreference -Force -ExclusionPath '"C:\ProgramData\Microsoft\env"\update.bat'" 
```

trên đó chính là file đầu tiên mà malware đã add vào path exclusion để tránh việc bị xóa, vào lúc 16:04:+ thì file này đã được add vào đầu tiên và không có gì đáng nghi ngờ. Sau đó malware còn add thêm một vài file nữa vào path này và có khá  nhiều file xảy ra trong khoảng thời gian khá ngắn. Dưới đây là những minh chứng về điều đó :

```jsx
powershell  -Command "Add-MpPreference -Force -ExclusionPath '"C:\ProgramData\Microsoft\env\env.exe"'"
 powershell  -Command "Add-MpPreference -Force -ExclusionPath '"C:\ProgramData\Microsoft\env"\bcd.bat'"
 powershell  -Command "Add-MpPreference -Force -ExclusionPath '"C:\ProgramData\Microsoft\env"\msconf.conf'" 
 powershell  -Command "Add-MpPreference -Force -ExclusionPath '"C:\ProgramData\Microsoft\env"\mssetup.exe'" 
```

Đó chỉ là những số file điển hình mà malware đã thực hiện trên máy của victim. 

### After the malware execution, the wmic utility was used to unjoin the computer system from a domain or workgroup. Tracking this operation is essential for identifying system reconfigurations or unauthorized changes. What is the Process ID (PID) of the utility responsible for performing this action?

Ở câu hỏi này họ muốn mình tìm ra PID mà có trách nhiệm cho việc hoạt động trái quy định này và file này được WMIC khởi tạo cũng như dùng chính cái này để run, bản chất của WMIC là một process trong microsoft nó cho phép các admin quản  lý cũng như là truy vấn thông tin về Windows System bla bla, malware đã tận dụng điều này để thực hiện hành vi trái phép và mình sẽ tìm ra nó ở dưới đây. Bên cạnh đó dựa vào những gì đã có sẵn thì  mình xác định được PID của nó là 

!image.png

Như các bạn có thể thấy nó liên quan tới workgroup và ngoài ra nó còn liên quan tới configuring PC vậy nên mình xác định được PID mà mình đang cần tìm đó là 7492. Ngoài ra mình còn phát hiện được red flag tiếp thoe và hành động đáng nghi ngờ của họ 

!image.png

Đây chính là behavior nói lên rằng họ tạo scheduled task để persistence và maintain điều khiển hệ thống qua compromise, task mà họ đã tạo để scheduled là Aa153!EGzN, đây là task mà khi máy tính hoạt động thì nó sẽ run và cứ luân phiên nhau như vậy khiến cho việc mà phát hiện rất là khó.

## Conclusion

Đây là một bài LAB liên quan đến AD và điều này rất phổ biến ở trong các công ty và đặc biệt còn nghiêm trọng hơn khi hacker lại lợi dụng điều này để thực hiện những hành vi không đáng có. Nhưng sẽ có một số bạn đặt câu hỏi như thế này, initial access như thế nào và tại sao họ lạ compromise được AD thì mình sẽ nói như sau, khuôn mẫu của bài LAB này chỉ đến thế nên mình không thể truy lùng ra được việc đó là như thế nào, nên các bạn có thể thông cảm và bỏ qua cho mình chứ. Ở bài sau mình sẽ build project về AD và sẽ thực hiện kết hợp splunk để monitoring liên tục để theo dõi hoạt động liên tục của một endpoint vậy nên hẹn các bạn ở project sau nhé….
