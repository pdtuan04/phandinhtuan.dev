---
title: "UDP: Gã Giao Hàng Siêu Tốc Nhưng... Dễ Mất Hàng."
summary: "Giải thích UDP bằng ẩn dụ 'gửi bưu thiếp': nhanh, không kết nối, nhưng không đáng tin cậy. Trái ngược với TCP ('cuộc điện thoại' tin cậy). Hoàn hảo cho các ứng dụng ưu tiên tốc độ hơn sự hoàn hảo như livestream hay game online."
categories: ["Post","Blog",]
tags: ["network"]
#externalUrl: ""
#showSummary: true
date: 2025-10-14
draft: false
---
{{< lead >}}

**Chào bạn! Trong thế giới mạng máy tính, có hai "anh em" giao hàng nổi tiếng mà bạn sẽ nghe đi nghe lại: TCP và UDP.**
![AnhMinhHoa](nvgh.png)
**Nếu TCP là một nhân viên giao hàng cực kỳ cẩn thận, luôn gọi điện xác nhận trước khi đi, giao đúng thứ tự, và yêu cầu ký nhận... thì UDP chính là gã giao hàng trái ngược hoàn toàn: phóng khoáng, siêu tốc, nhưng hơi đoảng.**
**Hôm nay, chúng ta hãy tìm hiểu về gã "não cá vàng" thú vị này nhé!**
{{< /lead >}}
## UDP (User Datagram Protocol) giống như gửi một tấm bưu thiếp
{{< lead >}}

**Không cần gọi trước: Bạn chỉ cần viết địa chỉ người nhận lên bưu thiếp rồi thả vào hòm thư. Bạn không cần gọi điện hỏi "Ê, tôi sắp gửi bưu thiếp cho ông nhé?". Đây được gọi là "không kết nối" (connectionless).**

**Không đảm bảo:**

**Bưu thiếp có thể bị thất lạc trên đường đi.**

**Nếu bạn gửi 3 tấm bưu thiếp liên tiếp, chúng có thể đến nơi không theo thứ tự. Tấm thứ 3 có thể đến trước tấm thứ 1.**

**Không ai gọi lại báo cho bạn là "Tôi nhận được bưu thiếp rồi nhé!".**

**Siêu nhẹ, siêu nhanh: Vì bỏ qua hết các thủ tục xác nhận rườm rà, việc gửi một tấm bưu thiếp cực kỳ nhanh và đơn giản.**

**=> Đặc điểm của UDP: Siêu nhanh, gọn nhẹ, nhưng không đảm bảo dữ liệu sẽ đến, không đảm bảo đến đúng thứ tự.**

{{< /lead >}}
## Trường Hợp Sử Dung Giao Thức UDP
{{< lead >}}
**Khoan Đã... Vậy Ai Lại Dùng Một Dịch Vụ "Không Đáng Tin Cậy"?**
**Nghe thì có vẻ UDP rất tệ, nhưng nó lại là người hùng trong rất nhiều trường hợp mà tốc độ được ưu tiên hơn sự hoàn hảo.**

**Hãy nghĩ xem, bạn có cần sự cẩn thận của một cuộc điện thoại trong các tình huống sau không?**

**Xem Livestream, Gọi Video:**

**Vấn đề: Khi bạn đang xem bóng đá trực tiếp, việc hình ảnh bị vỡ một vài khung hình (do mất gói tin UDP) trong một giây còn hơn là cả video bị đứng hình 2-3 giây để chờ tải lại gói tin bị mất đó (nếu dùng TCP).**

**Giải pháp UDP: Cứ liên tục gửi dữ liệu hình ảnh mới nhất. Mất một vài khung hình cũ cũng không sao, miễn là bạn đang xem được diễn biến hiện tại.**

**Chơi Game Online:**

**Vấn đề: Trong một game bắn súng, vị trí của đối thủ phải được cập nhật ngay lập tức. Dữ liệu về vị trí của họ 1 giây trước đã hoàn toàn vô dụng.**

**Giải pháp UDP: Gửi liên tục vị trí mới nhất của người chơi. Nếu một gói tin vị trí bị mất, không sao cả, vì gói tin ngay sau đó sẽ cập nhật vị trí mới hơn. Chậm trễ một chút thôi là bạn đã "lên bảng đếm số" rồi!**

**Hệ thống phân giải tên miền (DNS):**

**Vấn đề: Khi bạn gõ google.com vào trình duyệt, máy tính cần hỏi máy chủ DNS: "IP của `https://www.google.com/urlsa=Esource=gmailq=google.com` là gì?". Đây là một câu hỏi rất nhỏ.**

**Giải pháp UDP: Gửi đi một câu hỏi nhỏ và nhận về một câu trả lời nhỏ. Dùng UDP cực nhanh. Nếu lỡ gói tin bị mất, máy tính chỉ đơn giản là hỏi lại. Nhanh hơn nhiều so với việc thiết lập cả một kết nối TCP chỉ để hỏi một câu đơn giản.**

**Nguyên tắc vàng: Nếu dữ liệu mới nhất luôn quan trọng hơn dữ liệu cũ, và việc mất mát một chút dữ liệu có thể chấp nhận được, hãy dùng UDP!**
{{< /lead >}}
## Ví Dụ Code: Gửi và Nhận Tin Nhắn Bằng UDP
{{< lead >}}

**Hãy xem cách "gửi bưu thiếp" trong Java hoạt động như thế nào. Chúng ta sẽ có một Server (người nhận thư) và một Client (người gửi thư).**
{{< /lead >}}
### 1. UDPServer.java (Người nhận thư)
Java
```
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPServer {
    public static void main(String[] args) throws Exception {
        // 1. Mở một "hòm thư" (socket) tại cổng 9876
        DatagramSocket serverSocket = new DatagramSocket(9876);
        System.out.println("Server đang chạy và chờ nhận bưu thiếp...");

        while (true) {
            // 2. Chuẩn bị một chỗ trống để chứa bưu thiếp sắp tới
            byte[] receiveData = new byte[1024];
            DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);

            // 3. Chờ và nhận bưu thiếp
            serverSocket.receive(receivePacket);

            // 4. Mở bưu thiếp ra xem
            String sentence = new String(receivePacket.getData()).trim();
            System.out.println("ĐÃ NHẬN: " + sentence);

            // (Tùy chọn) Gửi lại thư cảm ơn. Lấy địa chỉ từ chính bưu thiếp vừa nhận.
            InetAddress IPAddress = receivePacket.getAddress();
            int port = receivePacket.getPort();
            String capitalizedSentence = "Server da nhan duoc: " + sentence.toUpperCase();
            byte[] sendData = capitalizedSentence.getBytes();
            DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, IPAddress, port);
            serverSocket.send(sendPacket);
        }
    }
}
```
### 2. UDPClient.java (Người gửi thư)
Java
```
import java.io.BufferedReader;
import java.io.InputStreamReader;
import java.net.DatagramPacket;
import java.net.DatagramSocket;
import java.net.InetAddress;

public class UDPClient {
    public static void main(String[] args) throws Exception {
        // 1. Lấy một cái bút và giấy để viết thư
        BufferedReader inFromUser = new BufferedReader(new InputStreamReader(System.in));
        
        // 2. Tạo một hòm thư của riêng mình để gửi và có thể nhận lại thư trả lời
        DatagramSocket clientSocket = new DatagramSocket();

        // 3. Tìm địa chỉ của người nhận (ở đây là máy của mình - localhost)
        InetAddress IPAddress = InetAddress.getByName("localhost");

        System.out.print("Nhập tin nhắn để gửi: ");
        String sentence = inFromUser.readLine();
        byte[] sendData = sentence.getBytes();

        // 4. Tạo một tấm bưu thiếp chứa: nội dung, độ dài, địa chỉ và cổng của người nhận
        DatagramPacket sendPacket = new DatagramPacket(sendData, sendData.length, IPAddress, 9876);

        // 5. Thả vào hòm thư và quên nó đi!
        clientSocket.send(sendPacket);
        System.out.println("Đã gửi bưu thiếp!");

        // (Tùy chọn) Chờ nhận thư cảm ơn
        byte[] receiveData = new byte[1024];
        DatagramPacket receivePacket = new DatagramPacket(receiveData, receiveData.length);
        clientSocket.receive(receivePacket);
        String modifiedSentence = new String(receivePacket.getData()).trim();
        System.out.println("THƯ TRẢ LỜI TỪ SERVER: " + modifiedSentence);
        
        // 6. Đóng hòm thư
        clientSocket.close();
    }
}
```
## Kết Luận
{{< lead >}}

**UDP không phải là phiên bản "lỗi" của TCP. Nó là một công cụ chuyên dụng, được thiết kế cho những công việc mà tốc độ là tất cả. Giống như trong cuộc sống, đôi khi bạn cần một cuộc gọi điện thoại nghiêm túc (TCP), nhưng cũng có lúc, một tấm bưu thiếp nhanh gọn (UDP) lại là lựa chọn hoàn hảo.**

**Hy vọng qua ví dụ này, bạn đã không còn thấy UDP xa lạ nữa!**
{{< /lead >}}