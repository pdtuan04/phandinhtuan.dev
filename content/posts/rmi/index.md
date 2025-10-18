---
title: "Java RMI: Gọi Phương Thức Từ Xa Dễ Dàng"
summary: "Lorem ipsum dolor sit amet, consectetur adipiscing elit. Aenean in eleifend justo, vestibulum congue lacus. Quisque est libero, lacinia sed placerat ac, interdum id urna."
categories: ["Post","Blog",]
tags: ["post","lorem","ipsum"]
#externalUrl: ""
#showSummary: true
date: 2025-09-04
draft: false
---
Chào bạn! Chúng ta đã biết cách gửi "bưu thiếp" (UDP) và thực hiện "cuộc gọi" (TCP) giữa các máy tính. Nhưng những cách đó giống như bạn phải tự mình viết thư tay và bỏ vào hòm thư.

Giờ hãy tưởng tượng một thế giới "vi diệu" hơn: bạn ngồi ở Sài Gòn, nhưng có thể cầm cái điều khiển từ xa và bấm nút "pha cà phê" trên một máy pha cà phê đặt ở Hà Nội, và nó thực sự hoạt động.

Đó chính xác là những gì Java RMI (Remote Method Invocation) cho phép bạn làm!

## RMI Là Gì? Một Cái "Điều Khiển Từ Xa" Cho Đối Tượng
RMI là một cơ chế của Java cho phép một đối tượng trên một máy tính (Client) có thể gọi các phương thức (method) của một đối tượng đang chạy trên một máy tính khác (Server) một cách trong suốt, cứ như thể hai đối tượng đó đang ở chung một máy.

Để dễ hình dung, hãy so sánh:

Lập trình thông thường: Bạn có một đối tượng mayPhaCaPhe và bạn gọi mayPhaCaPhe.phaEspresso(). Mọi thứ đều diễn ra trong cùng một chương trình.

Với RMI: Bạn ở máy A, có một cái "điều khiển từ xa" trỏ tới đối tượng mayPhaPhe thật đang chạy ở máy B. Bạn chỉ cần gọi dieuKhien.phaEspresso() trên máy A, và RMI sẽ lo hết phần việc "gửi tín hiệu" qua mạng để máy B thực hiện lệnh và trả kết quả về cho bạn.

Bạn, với tư cách là lập trình viên, không cần phải quan tâm đến việc mở socket, gửi byte hay đóng kết nối. Bạn chỉ cần "bấm nút" thôi!

Các Nhân Vật Chính Trong Vở Kịch RMI
Một hệ thống RMI đơn giản có 4 thành phần chính:

### Remote Interface (Bảng điều khiển):

Đây là một file interface trong Java, định nghĩa các "nút bấm" nào sẽ có trên điều khiển từ xa. Ví dụ: phaEspresso(), kiemTraLuongNuoc().

Nó là bản hợp đồng giữa Client và Server, quy định những gì Client có thể yêu cầu Server làm.

### Remote Object (Đối tượng thực thi ở Server):

Đây là một lớp (class) ở phía Server, implements cái Remote Interface ở trên.

Lớp này chứa đoạn code thực sự để "pha cà phê" hay "kiểm tra nước". Nó là cái máy pha cà phê thật sự.

### RMI Server (Người quản lý máy móc):

Đây là chương trình chính ở phía Server. Nhiệm vụ của nó là khởi tạo "máy pha cà phê" (Remote Object) và đăng ký nó với một dịch vụ đặc biệt.

Nó nói với dịch vụ rằng: "Này, tôi có một cái máy pha cà phê sẵn sàng phục vụ ở địa chỉ là /mayphaphe nhé!".

### RMI Registry (Tổng đài/Danh bạ):

Đây là một chương trình đặc biệt của Java, hoạt động như một cuốn danh bạ. Nó lắng nghe các yêu cầu đăng ký từ Server.

Khi Client muốn tìm máy pha cà phê, nó sẽ hỏi "Tổng đài" này: "Cho tôi xin cái điều khiển của /mayphaphe với".

### RMI Client (Người dùng):

Đây là chương trình ở máy Client. Nó sẽ kết nối tới "Tổng đài" (RMI Registry), hỏi xin "cái điều khiển" (một đối tượng proxy đặc biệt gọi là Stub).

Sau khi có được cái điều khiển này, nó có thể bắt đầu bấm nút (gọi phương thức) như bình thường.

## Ví Dụ Code: Máy Tính Từ Xa
Hãy tạo một chương trình RMI đơn giản cho phép Client gửi 2 số lên Server và nhận lại kết quả phép cộng.

### 1. Calculator.java (Remote Interface - Bảng điều khiển)
Java
```
import java.rmi.Remote;
import java.rmi.RemoteException;

// Phải kế thừa từ Remote để đánh dấu đây là một interface cho RMI
public interface Calculator extends Remote {
    // Mọi phương thức trong này đều phải ném ra RemoteException
    // để xử lý các lỗi liên quan đến mạng
    int add(int a, int b) throws RemoteException;
}
```
### 2. CalculatorImpl.java (Remote Object - Máy tính thật ở Server)
Java
```
import java.rmi.server.UnicastRemoteObject;
import java.rmi.RemoteException;

// Kế thừa từ UnicastRemoteObject để biến nó thành một đối tượng có thể truy cập từ xa
public class CalculatorImpl extends UnicastRemoteObject implements Calculator {

    // Constructor bắt buộc phải ném ra RemoteException
    protected CalculatorImpl() throws RemoteException {
        super();
    }

    @Override
    public int add(int a, int b) throws RemoteException {
        System.out.println("Server nhận được yêu cầu tính tổng: " + a + " + " + b);
        return a + b;
    }
}
```
### 3. Server.java (Người quản lý)
Java
```
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Server {
    public static void main(String[] args) {
        try {
            // 1. Tạo đối tượng thực thi
            CalculatorImpl calculator = new CalculatorImpl();

            // 2. Khởi động RMI Registry tại cổng 1099
            Registry registry = LocateRegistry.createRegistry(1099);

            // 3. Đăng ký đối tượng với một cái tên duy nhất
            registry.bind("CalculatorService", calculator);

            System.out.println("Server đã sẵn sàng!");
        } catch (Exception e) {
            System.err.println("Server exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```
### 4. Client.java (Người dùng)
Java
```
import java.rmi.registry.LocateRegistry;
import java.rmi.registry.Registry;

public class Client {
    public static void main(String[] args) {
        try {
            // 1. Tìm đến RMI Registry đang chạy trên máy chủ (localhost) ở cổng 1099
            Registry registry = LocateRegistry.getRegistry("localhost", 1099);

            // 2. Hỏi xin "cái điều khiển" có tên là "CalculatorService"
            // Lưu ý: Phải ép kiểu về đúng loại Interface
            Calculator calculator = (Calculator) registry.lookup("CalculatorService");

            // 3. Bấm nút! Gọi phương thức từ xa như bình thường
            int result = calculator.add(5, 3);
            System.out.println("Kết quả từ Server: " + result);

        } catch (Exception e) {
            System.err.println("Client exception: " + e.toString());
            e.printStackTrace();
        }
    }
}
```
Bạn sẽ thấy Server in ra thông báo nó nhận được yêu cầu, và Client in ra kết quả "8".

## Kết Luận
RMI là một công nghệ mạnh mẽ thuộc họ RPC (Remote Procedure Call), giúp đơn giản hóa việc lập trình mạng phân tán trong Java. Thay vì phải làm việc với các chi tiết phức tạp của socket, bạn có thể tập trung vào logic nghiệp vụ, gọi các đối tượng từ xa một cách tự nhiên và thanh lịch. Nó là nền tảng cho nhiều công nghệ Java phức tạp hơn như EJB (Enterprise JavaBeans).