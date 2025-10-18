---
title: "Đa Luồng: Khi Máy Tính Của Bạn Mọc Thêm Tay"
summary: "Giải thích Java Multithreading bằng ẩn dụ nhà bếp với đầu bếp (luồng) làm việc song song. Hướng dẫn tạo luồng (Thread/Runnable) và dùng synchronized để tránh tranh giành tài nguyên (dữ liệu chung)."
categories: ["Post","Blog",]
tags: ["network","java"]
#externalUrl: ""
#showSummary: true
date: 2025-10-17
draft: false
smartTOC: true
---

**Chào bạn! Hôm nay chúng ta sẽ khám phá một khái niệm nghe có vẻ phức tạp trong Java: Multithreading (lập trình đa luồng). Nhưng đừng lo, tôi sẽ giải thích nó theo cách đơn giản nhất, như thể bạn đang xem một bộ phim hoạt hình vậy.**

## Đơn Luồng | Bạn Là Một Đầu Bếp Đơn Độc
![lonely chef](chef.jpg)

**Hãy hình dung bạn là một đầu bếp trong một nhà hàng nhỏ, và bạn phải làm tất cả mọi việc.**

**Khi có khách gọi món, quy trình của bạn là:**

**- Nhận đơn hàng.**

**- Đi vào bếp, chuẩn bị và nấu món ăn.**

**- Mang món ăn ra cho khách.**

**- Quay lại bếp dọn dẹp.**

**Mọi thứ diễn ra tuần tự. Nếu có 3 bàn khách cùng gọi món, bạn phải làm xong cho bàn thứ nhất rồi mới đến bàn thứ hai, rồi mới đến bàn thứ ba. Các vị khách đến sau sẽ phải chờ dài cổ. Đây chính là cách một chương trình đơn luồng (single-thread) hoạt động. Nó chỉ làm được một việc tại một thời điểm.**

## Đa Luồng | Bạn Tuyển Thêm Nhân Viên

**Giờ thì nhà hàng của bạn phát đạt hơn. Bạn quyết định thuê thêm 2 phụ bếp nữa. Mọi chuyện bây giờ hoàn toàn khác:**

**Bạn (Đầu bếp chính): Chuyên nhận đơn hàng và giám sát.**

**Phụ bếp A: Chuyên sơ chế nguyên liệu.**

**Phụ bếp B: Chuyên nấu nướng và trình bày.**

**Khi 3 bàn khách cùng gọi món, bạn có thể nhận cả 3 đơn gần như cùng lúc, phụ bếp A bắt đầu sơ chế ngay lập tức, và phụ bếp B thì nấu ngay khi có nguyên liệu. Mọi thứ diễn ra song song. Nhà hàng của bạn phục vụ nhanh hơn gấp nhiều lần.**

**Đó chính là Multithreading!**

**Tóm lại: Một chương trình đa luồng giống như một nhà bếp có nhiều đầu bếp. Mỗi "đầu bếp" được gọi là một Thread (luồng). Toàn bộ chương trình (nhà hàng) có thể xử lý nhiều công việc (nấu nhiều món) cùng một lúc, giúp tăng hiệu suất và tốc độ đáng kể.**

## Làm Sao Để "Thuê Phụ Bếp" (Tạo Thread) Trong Java?

**Trong Java, bạn có thể "thuê phụ bếp" (tạo một thread) theo hai cách phổ biến.**

### Cách 1: "Truyền nhân" - Kế thừa từ lớp Thread

**Cách này giống như bạn dạy nghề cho con trai mình. Bạn tạo một lớp mới và cho nó "kế thừa" mọi kỹ năng của một Thread cha.**

**Cách làm:**

**Tạo một class mới kế thừa từ java.lang.Thread.**

**Ghi đè (override) phương thức run(). Đây là nơi bạn định nghĩa những việc mà "phụ bếp" này sẽ làm.**

**Để bắt đầu, bạn gọi phương thức start().**

**Ví dụ: Hãy tạo một "phụ bếp" có nhiệm vụ thái 5 củ cà rốt.**

Java
```
// Phụ bếp này tên là KitchenHelper, được đào tạo từ lớp Thread cha
class KitchenHelper extends Thread {
    private String name;

    public KitchenHelper(String name) {
        this.name = name;
    }

    // Đây là công việc của phụ bếp: thái cà rốt
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(this.name + " đã thái xong củ cà rốt thứ: " + i);
            try {
                // Giả vờ nghỉ 1 giây để thái củ tiếp theo
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // Xử lý nếu có lỗi
            }
        }
    }
}

public class Restaurant {
    public static void main(String[] args) {
        // Thuê 2 phụ bếp
        KitchenHelper helper1 = new KitchenHelper("Tuấn");
        KitchenHelper helper2 = new KitchenHelper("Đình");

        // Yêu cầu họ bắt đầu làm việc!
        helper1.start();
        helper2.start();
    }
}
```

**Nếu bạn chạy đoạn code trên, bạn sẽ thấy Tuấn và Đình thái cà rốt xen kẽ nhau. Họ đang làm việc song song!**

### Cách 2: "Người làm thuê" - Implement interface Runnable**

**Cách này linh hoạt hơn. Thay vì tạo ra một "phụ bếp" chính hiệu, bạn chỉ cần viết ra một "bản mô tả công việc" (gọi là Runnable), rồi đưa bản mô tả này cho một Thread bất kỳ để họ thực hiện.**

**Đây là cách được khuyến khích sử dụng nhiều hơn vì Java không cho kế thừa từ nhiều lớp, nên việc "implement" sẽ giúp code của bạn linh hoạt hơn.**

**Ví dụ: Cùng công việc thái 5 củ cà rốt.**

Java
```
// Đây là "bản mô tả công việc"
class TaskThaiCarrot implements Runnable {
    private String tenNguoiLam;

    public TaskThaiCarrot(String tenNguoiLam) {
        this.tenNguoiLam = tenNguoiLam;
    }

    // Công việc cần làm được viết ở đây
    @Override
    public void run() {
        for (int i = 1; i <= 5; i++) {
            System.out.println(this.tenNguoiLam + " đang thực hiện công việc, củ thứ: " + i);
            try {
                Thread.sleep(1000);
            } catch (InterruptedException e) {
                // Xử lý lỗi
            }
        }
    }
}

public class Restaurant {
    public static void main(String[] args) {
        // Tạo ra 2 bản mô tả công việc
        TaskThaiCarrot task1 = new TaskThaiCarrot("Nhiệm vụ của Tuấn");
        TaskThaiCarrot task2 = new TaskThaiCarrot("Nhiệm vụ của Đình");

        // Thuê 2 "Thread" và giao việc cho họ
        Thread worker1 = new Thread(task1);
        Thread worker2 = new Thread(task2);

        // Bắt đầu làm việc
        worker1.start();
        worker2.start();
    }
}
```

**Kết quả cũng tương tự, nhưng cách tổ chức code này gọn gàng và dễ mở rộng hơn.**

## Synchronized: Khi Các Phụ Bếp Tranh Giành Một Cái Chảo

**Quay lại căn bếp. Giả sử bạn chỉ có một cái chảo chống dính duy nhất. Cả Tuấn và Đình đều cần nó để chiên trứng.**

**Tình huống xấu:**

**Tuấn kiểm tra, thấy cái chảo đang rảnh.**

**Ngay lúc đó, Đình cũng kiểm tra, cũng thấy cái chảo rảnh.**

**Tuấn đặt chảo lên bếp và đập trứng vào.**

**Đình không biết Tuấn vừa lấy, cũng chạy tới và đập trứng của mình vào cùng cái chảo đó.**

**Kết quả: Món trứng chiên hỗn loạn, không ra hình thù gì!**

**Vấn đề này trong lập trình gọi là Race Condition (Tranh chấp tài nguyên). Nó xảy ra khi nhiều luồng cùng truy cập và thay đổi một tài nguyên dùng chung (biến, đối tượng, file...).**

### Giải Pháp: "Ai Dùng Thì Khóa Lại" (Synchronized)

![AnhMinhHoa](lock.jpg)
**Để giải quyết, bạn ra quy định: "Ai muốn dùng cái chảo thì phải cầm lấy nó và khóa cửa bếp lại. Dùng xong, rửa sạch rồi mới được mở khóa cho người khác vào".**

**Trong Java, cơ chế khóa đó chính là từ khóa synchronized.**

**Khi một phương thức hoặc một khối lệnh được đánh dấu là synchronized, nó đảm bảo rằng tại một thời điểm, chỉ có duy nhất một luồng được phép thực thi nó trên cùng một đối tượng. Các luồng khác muốn vào phải xếp hàng chờ đến lượt.**

**Ví dụ: Quản lý số lượng món ăn đã hoàn thành.**

Java
```
class CounterMonAn {
    private int soMonDaHoanThanh = 0;

    // Chỉ một người được vào đây cập nhật số lượng tại một thời điểm
    public synchronized void hoanThanhThemMon() {
        int hienTai = soMonDaHoanThanh;
        System.out.println(Thread.currentThread().getName() + " thấy có " + hienTai + " món đã xong, chuẩn bị thêm 1.");
        soMonDaHoanThanh = hienTai + 1;
        System.out.println("=> Tổng cộng đã xong: " + soMonDaHoanThanh + " món.");
    }

    public int getSoMonDaHoanThanh() {
        return soMonDaHoanThanh;
    }
}

public class Main {
    public static void main(String[] args) throws InterruptedException {
        CounterMonAn counter = new CounterMonAn();

        Runnable task = () -> {
            for (int i = 0; i < 100; i++) {
                counter.hoanThanhThemMon();
            }
        };

        Thread dauBepA = new Thread(task, "Đầu bếp A");
        Thread dauBepB = new Thread(task, "Đầu bếp B");

        dauBepA.start();
        dauBepB.start();

        // Chờ cả 2 đầu bếp làm xong việc
        dauBepA.join();
        dauBepB.join();

        System.out.println("Cuối ngày, tổng số món đã hoàn thành là: " + counter.getSoMonDaHoanThanh()); // Kết quả sẽ luôn là 200
    }
}
```

**Nếu bạn bỏ từ khóa synchronized đi, kết quả cuối cùng có thể sẽ không phải là 200, vì hai luồng sẽ "giẫm chân" lên nhau khi cập nhật biến soMonDaHoanThanh.**

## Kết Luận

**Vậy là bạn đã hiểu những ý tưởng cốt lõi của multithreading rồi đấy!**

**Multithreading là cho phép chương trình làm nhiều việc cùng lúc, như có nhiều đầu bếp trong bếp.**

**Một Thread giống như một đầu bếp.**

**Bạn có thể tạo Thread bằng cách extends Thread hoặc implements Runnable (khuyến khích dùng cách thứ hai).**

**Khi nhiều thread dùng chung tài nguyên (cái chảo), hãy dùng synchronized để tránh tranh giành và gây ra lỗi.**
**Hy vọng qua ví dụ về nhà bếp, bạn đã thấy multithreading không hề đáng sợ. Nó là một công cụ cực kỳ mạnh mẽ để làm cho ứng dụng của bạn chạy nhanh và hiệu quả hơn rất nhiều!**
