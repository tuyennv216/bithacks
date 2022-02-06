Xác định nếu kiểu word có chứa byte 0
```
// phiên bản ít phép tính
unsigned int v; // số 32-bit cần check nếu 8-byte trong đó có chứa 0
bool hasZeroByte = ~((((v & 0x7F7F7F7F) + 0x7F7F7F7F) | v) | 0x7F7F7F7F);
```
Đoạn mã trên hữu ích khi sao chép chuỗi ký tự trong trường hợp 1 từ được 1 sao chép tại 1 thời điểm, nó sử dụng 5 phép toán.
```
// nhiều phép tính hơn
bool hasNoZeroByte = ((v & 0xff) & (v & 0xff00) && (v & 0xff0000) && (v & 0xff000000));
// hoặc
unsigned char * p = (unsigned char *) &v;
bool hasNoZeroByte = *p && *(p + 1) && *(p + 2) && *(p + 3);
```
Để thêm cải thiện, tốt nhất là thêm 4 phép tính để xác định nếu số có thể có byte 0. Kết quả cũng trả về true nếu byte cao là 0x80, nên để chữa tình huống đó, phiên bản chậm hơn nhưng đáng tin cậy hơn dưới đây có thể là một ứng viên để xem xét.
```
bool hasZeroByte = ((v + 0x7efefeff) ^ ~v) & 0x81010100;
if (hasZeroByte)
{
  hasZeroByte = ~((((v & 0x7F7F7F7F) + 0x7F7F7F7F) | v) | 0x7F7F7F7F)
}
```
Đây là một phiên bản nhanh nhất - sử dụng hasless(v, 1), được định nghĩa dưới đây; nó cần 4 phép toán.
```
#define haszero(v) (((v) - 0x01010101UL) & ~(v) & 0x80808080UL);
```
Biểu thức (v - 0x01010101UL) sẽ tính các bit cao của byte khi một byte tương ứng là zero hoặc lớn hơn 0x80. Biểu thức ~v & 0x80808080UL tính những bit cao của byte khi giá trị của v không được set 8-bit cao, nên byte là nhỏ hơn 0x80.  
Cuối cùng, thêm phép AND 2 biểu thức để ra kết quả là tập bits có chứa byte 0 hay không.

Xác định nếu kiểu word có chứa byte có giá trị là n
Chúng ta muốn biết nếu bất kỳ một byte nào trong số đó là một giá trị đặc biệt. Để làm điều đó, chúng ta sẽ XOR giá trị để kiểm tra với số chỉ gồm những byte mà chúng ta quan tâm. Bởi vì XOR sẽ trả về chính là nó 0, ta có thể chuyển kết quả cho hàm kiểm tra byte 0 ở trên.
```
#define hasvalue(x, n) (haszero((x) ^ (~0UL/255 * (n))))
```
Xác định nếu kiểu word có chứa byte có giá trị lớn hơn n
Kiểm tra nếu số x chứa một byte có giá trị < n. Đặc biệt với n = 1, nó có thể sử dụng để tìm kiếm các byte-0 bằng cách kiểm tra 1 loạt các byte một thời điểm, hoặc bất kỳ byte nào bởi XOR x với mặt nạ. Sử dụng 4 phép toán khi n là hằng số.
Yêu cầu x >= 0 và 0 <= n <= 128
```
#define hasless(x, n) (((x)-~0UL/255*(n))&~(x)&~0UL/255*128)
```
Để đếm số lượng byte trong x mà nhỏ hơn n với 7 phép toán, sử dụng
```
#define countless(x, n) (((~0UL/255*(127+(n))-((x)&~0UL/255*127))&~(x)&~0UL/255*128)/128%255);
```
Xác định nếu kiểu word có chứa byte có giá trị trong khoảng m và n
Khi m < n, phương pháp này kiểm tra nếu số x chứa một byte mà m < value < n. Nó sử dụng 7 phép toán khi n và m là hằng số.  
Ghi chú: Byte có giá trị bằng n xử lý bởi likelyhasbetween cho kết quả không đúng, nên cần kiểm tra lại nếu một kết quả chắc chắn là cần thiết.
Yêu cầu: x >= 0, 0 <= m <= 127, 0 <= n <= 128
```
#define likelyhasbetween(x, m, n) ((((x)-~0UL/255*(n))&~(x)&((x)&~0UL/255*127)+~0UL/255*(127-(m)))&~0UL/255*128)
```
Kỹ thuật này có thể phù hợp cho một giải pháp đẹp nhanh chóng. Một biến thể khác nhiều hơn phép tính hơn (tất cả 8 phép tính cho hằng số m và n) nhưng cung cấp kết quả chính xác hơn là:
```
#define hasbetween(x,m,n)((~0UL/255*(127+(n))-((x)&~0UL/255*127)&~(x)&((x)&~0UL/255*127)+~0UL/255*(127-(m)))&~0UL/255*128)
```
Để đếm số lượng byte trong x nằm giữa m và n (không tính n) trong 10 phép toán, sử dụng:
```
#define countbetween(x,m,n) (hasbetween(x,m,n)/128%255)
```
