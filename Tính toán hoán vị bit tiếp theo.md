Giả định chúng ta có một mẫu bit, chúng ta muốn tìm hoán vị thứ N tiếp theo.  
Ví dụ, nếu N là 3 và mẫu là 00010010, các hoán vị tiếp theo sẽ là 00010101, 00010110, 00011001, 00011010, 00011100, 00100011, và tiếp tục.  
Cách sau đây là nhanh nhất để tính ra hoán vị tiếp theo.
```
unsigned int v; // mẫu bit hiện tại
unsigned int w; // hoán vị bit tiếp theo

unsigned int t = v | (v - 1);
w = (t + 1) | (((~t & -~t) - 1) >> (__builtin_ctz(v) + 1));
```
Hàm __builtin_ctz(v) trong trình biên dịch GNU C cho x86 CPU trả về số chữ số 0. Nếu muốn sử dụng trình biên dịch Microsoft cho x86, hàm có sẵn là _BitScanForward.  
Cả hai đều sử dụng chỉ dẫn bsf, và mỗi loại là phù hợp với mỗi kiến trúc khác nhau. Nếu không, có thể cân nhắc sử dụng một phương pháp cho việc đếm các dãy số 0 đã trình bày trước đó.

Đây là một phiên bản khác chậm hơn vì nó sử dụng phép chia, nhưng không yêu cầu đếm số chữ số 0.
```
unsigned int t = (v | (v - 1)) + 1;
w = t | ((((t & -t) / (v & -v)) >> 1) - 1);
```
