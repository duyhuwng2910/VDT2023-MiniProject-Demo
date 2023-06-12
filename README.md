# VDT2023-MiniProject-Demo
Trong phần demo này, ta sẽ sử dụng giải pháp Delta Lake trên giao diện của databricks là Databricks Lakehouse Platform, xây dựng cụm tính toán và lưu trữ dữ liệu, sau đó tiến hành một số bước xử lý dữ liệu để minh hoạ tính năng của Data Lakehouse. Ta sẽ cùng xem qua một số hình ảnh minh hoạ cho demo.
Sau khi đăng kí và đăng nhập tài khoản cộng đồng của databricks, ta sẽ có giao diện quản lý hệ thống của databricks như sau:
![Hình 7.1: Giao diện của Databricks Lakehouse Platform](/Demo/7.1.png)
Tiếp theo, ta tiến hành tạo một Notebook mới trong phần Workspace (bấm chuột phải, chọn “Create”, sau đó chọn “Notebook”) 
![Hình 7.2: Hướng dẫn tạo notebook để tiến hành demo hệ thống](/Demo/7.2.png)
Ở đây, phần demo của em gồm hai notebook, nằm trong folder “Demo Data Lakehouse”. Trong notebook “1. Tổng quan”, em tiến hành một số thao tác cơ bản với cụm tính toán, bao gồm việc thêm, sửa, xoá thư mục; đọc, ghi tệp và thực hiện truy vấn đơn giản. Đối với lựa chọn xây dựng như trên, ta có một cụm tính toán sử dụng Apache Spark và Scala. Vì đây chỉ là phiên bản cộng đồng nên ta chỉ sử dụng được một vài tính năng nhất định, tuy nhiên ta vẫn có thể sử dụng các công nghệ khác kết nối với cụm này để có thể phục vụ cho công việc. Nếu sử dụng phiên bản dành cho doanh nghiệp, ta sẽ được hỗ trợ tối đa, cả về hệ sinh thái lẫn khả năng tính toán và lưu trữ.
Phần demo chính về các tính năng nằm ở notebook “2. Một số tính năng chính”. Để phục vụ cho việc minh hoạ các tính năng, em có sử dựng hai tệp dữ liệu về thông tin laptop năm 2020 và số liệu khách du lịch đến các nước Đông Nam Á (nguồn được ghi trong phần Tham khảo). 
## 7.1	Hỗ trợ giao dịch ACID
Đối với tính năng này, ta sẽ sử dụng định bảng delta để minh hoạ. Ta sẽ tiến hành ghi dataframe và lưu dưới dạng bảng trong metastore.
Sau khi đã ghi thành công, ta sẽ tiến hành các hành động DML (Data Manipulation Language operation) bao gồm thêm bản ghi, cập nhật bản ghi và xoá bản ghi để chứng minh tính chất ACID của giao dịch trong Data Lakehouse.
![Hình 7.3: DML Operations trong Data Lakehouse](/Demo/7.3.png)
![Hình 7.4: Danh sách các tệp lưu thông tin về siêu dữ liệu và nội dung của giao dịch](/Demo/7.4.png)
Nếu như có thao tác ghi hay cập nhật nào bị lỗi, nó sẽ không gây ảnh hưởng đến phiên bản hiện tại của tệp Parquet, trong khi vẫn lưu lại những phiên bản cũ. Những định dạng tệp này bao gồm các tệp CRC và JSON dùng để lưu trữ siêu dữ liệu, các thông tin liên quan đến nội dung chỉnh sửa. Tất cả đều được lưu trên kho lưu trữ đám mây, giúp đảm bảo tính khả dụng và bền vững. 
![Hình 7.5: Các phiên bản thay đổi của dữ liệu được ghi lại thành từng tệp định dạng Parquet](/Demo/7.5.png)
Ban đầu khi tiến hành ghi dữ liệu từ dataframe, ta chỉ có một tệp Parquet, tuy nhiên sau khi tiến hành các bước cập nhật dữ liệu trên bảng, ta thấy đã có thêm ba tệp Parquet mới. Điều này có thể giải thích là vì, các thao tác đọc và ghi được tiến hành độc lập với nhau theo tuần tự, điều này để đảm bảo tính nhất quán và cô lập của giao dịch, từ đó mà thông lượng cho thao tác ghi sẽ lớn hơn.
## 7.2	 Thực thi và cải tiến lược đồ
### 7.2.1	Thực thi lược đồ (Schema Enforcement)
Đối với Data Lake mà điển hình khi ta sử dụng định dạng Parquet, ta sẽ không có tính năng này và nếu muốn đạt được hiệu quá thì sẽ phải tiến hành thiết lập thủ công. Tất nhiên không phải lúc nào ta cũng nhớ được điều đó, và với Data Lakehouse thì tính năng đã được xây dựng bên trong kiến trúc.
Lấy ví dụ cho tính năng này, ta sẽ tạo hai dataframe có cấu trúc khác nhau và cùng được ghi vào cùng một thư mục theo hình thức “append” như hình sau:
![Hình 7.6: Ví dụ về tính năng thực thi lược đồ](/Demo/7.6.png)
Như ta có thể thấy đã có lỗi xảy ra khi tiến hành ghi hai dataframe có cấu trúc bảng khác nhau dưới định dạng delta. Đây chính là tính năng thực thi lược đồ, giải thích dễ hiểu là khi ta đã tiến hành ghi một dataframe vào một thư mực dưới định dạng delta, nếu như có một dataframe khác cũng được ghi vào thư mục đó mà có cấu trúc bảng khác thì Delta Lake sẽ từ chối việc ghi dữ liệu vào. 
### 7.2.2	Cải tiến lược đồ (Schema Evolution)
Tính năng này đề cập đến khả năng thích ứng với các lược đồ khác nhau theo thời gian, thông thường sẽ là cập nhật nội dung hay là thêm các cột bổ sung vào bảng. Nếu như tính năng thực thi lược đồ giúp cho chúng ta kiểm tra sự xung đột giữa các lược đồ của các dataframe, nhằm tránh gây ra việc gộp các lược đồ không mong muốn thì với Schema Evolution, ta sẽ có giải pháp khác phục vấn đề AnalystException trong hình 7.4. 
Đầu tiên, ta có thể bỏ qua tính năng Schema Enforcement bằng cách thiết lập lựa chọn mergeSchema thành true để có thể ghi vào bảng delta dữ liệu của một lược đồ không khớp với lược đồ ban đầu:
![](/Demo/mergeSchema.png) 
Và đây là nội dung trên bảng sau khi ta tiến hành thiết lập:
![](/Demo/result.png)
Như ta có thể thấy, hai dataframe có lược đồ khác nhau đã được gộp lại, và bảng delta hiện đã có ba cột trong khi trước đó chỉ có hai cột. Bên cạnh đó, ta hoàn toàn có thể cho phép mặc định tính năng này được thực hiện bằng cách thiết lập cài đặt autoMerge thành true:
![](/Demo/autoSchema.png)
Thiết lập này cho phép ta tránh việc phải cài đặt mergeSchema mỗi lần ta thêm dữ liệu và bảng, đồng thời cho phép thêm những dataframe có số lượng cột ít hơn so với bảng delta hiện tại. 
![Hình 7.7: Schema Evolution cho phép thêm các dataframe khác nhau hoàn toàn về lược đồ](/Demo/7.7.png)
Kể cả với việc ta gộp một dataframe có các cột hoàn toàn khác với lược đồ trong bảng delta thì Schema Evolution cho phép điều đó mà không bị chồng chất dữ liệu. Một lý do khác mà giúp cho tính năng này hữu ích là khi cần phải thay đổi lược đồ của bảng mà không phải ghi lại toàn dữ liệu thì Schema Evolution cho phép điều đó.
## 7.3	Audit History
Đây là một tính năng được chính Delta Lake công bố, với tính năng này ta có thể xem thông tin về việc tạo ra bảng, cấu hình cấu trúc của bảng, xem các thông tin về siêu dữ liệu. Bên cạnh đó ta còn có thể xem các mô tả về lịch sử của bảng, các lần sửa đổi, bao gồm phiên bản, thời gian, thông tin người tiến hành, loại thao tác… Như trong ảnh, ta có thể xem đang thông tin về bảng laptop, cũng như xem được các thông tin về siêu dữ liệu liên quan đến bảng ở mỗi lần thao tác dữ liệu trên bảng:
![Hình 7.8: Tính năng Audit History trên Delta Lake](/Demo/7.8.png)
## 7.4	Time Travel
Như đã đề cập ở chương 3, thông qua tính năng này ta có thể truy cập, truy vấn và phân tích dữ liệu ở các phiên bản cũ hơn, tại thời điểm bất kì trong quá khứ đã được ghi lại. Trong phần demo, ta tiếp tục tiến hành ba thao tác với bảng laptop như sau:
![](/Demo/laptop.png)
Giống với phần demo cho tính năng giao dịch ACID, khi ta tiến hành thao tác như trong hình thì sẽ tạo ra ba phiên bản mới của dataframe ứng với một thao tác. Ta có thể truy cập đến ba phiên bản này cũng như có thể quay lại (rollback) với phiên bản ban đầu bằng việc sử dụng số phiên bản (version number) hoặc là bằng dấu thời gian (timestamp):
![Hình 7.9: Thực hiện tính năng Time Travel trong Data Lakehouse](/Demo/7.9.png)
Trên đây là toàn bộ phần demo hệ thống Data Lakehouse cơ bản, gồm tổng quan và một số tính năng nổi bật của kiến trúc này. Bên cạnh các tính năng được thể hiện rõ trong quá trình demo, ta còn có thể kể đến một số tính năng được thể hiện xuyên suốt đó là khả năng hỗ trợ các định dạng mở, khả năng lưu trữ và tính toán tách biệt. Có hai tính năng chưa được đề cập đến trong phần demo đó là khả năng hỗ trợ các dữ liệu phi cấu trúc và theo luồng, cùng với cả hỗ trợ BI. Trong thời gian tới, khi có nhiều điều kiện và thời gian hơn, em sẽ xây dựng một hệ thống Data Lakehouse hoàn thiện hơn, thể hiện rõ kiến trúc theo tầng và minh hoạ cụ thể hơn các tính năng của mô hình này. Phần demo vừa rồi, hy vọng có thể giúp mọi người có thể hình dung được dễ hiểu hơn về kiến trúc Data Lakehouse
