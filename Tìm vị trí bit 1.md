Tìm vị trí bit 1 cao nhất với MSB trong O(N) phép tính
```
unsigned int v;     // số 32-bit cần tìm log base 2
unsigned int r = 0; // r sẽ lưu kết quả

while (v >>= 1)
{
  r++;
}
```
Log base 2 của số nguyên là giống với vị trí của bit cao nhất.

Tìm vị trí bit 1 cao nhất với số float 64-bit IEEE
```
int v;
int r;
union { unsigned int u[2]; double d; } t;

t.u[__FLOAT_WORD_ORDER==LITTLE_ENDIAN] = 0x43300000;
t.u[__FLOAT_WORD_ORDER!=LITTLE_ENDIAN] = v;
t.d -= 4503599627370496.0;
r = (t.u[__FLOAT_WORD_ORDER==LITTLE_ENDIAN] >> 20) - 0x3FF;
```
Đoạn mã trên load số double 64-bit với số nguyên 32-bit (không thêm các bits) bằng cách lưu số nguyên trong phần nguyên trong khi phần mũ đặt là 2^52. Từ một số double mới, 2^52 sẽ được trừ, nơi lưu kết quả của phép mũ thành giá trị log base 2 của giá trị nhập vào. Tất cả các bits sẽ được dịch phải 20 và trừ đi 0x3FF (1023 thập phân).  
Kỹ thuật này chỉ cần 5 phép tính, nhưng nhiều CPU sẽ chậm hơn khi sử lý số double, và kiến trúc cần có những chỉ thị đặc biệt.
  
Tìm vị trí bit 1 cao nhất với số float 32-bit IEEE
```
const float v;  // tìm int(log2(v)) với v > 0.0 và finite(v) && isnormal(v)
int c;          // kết quả lưu ở đây

c = *(const int *) &v;  // hoặc, để linh động: memcpy(&c, &v, sizeof c);
c = (c >> 23) - 127;
```
Phép tính trên là nhanh, nhưng mẫu IEEE 754 có kiến trúc khác cho số float. Nó có một hàm lũy thừa bit tới 0, và phần số nguyên không được chuẩn hóa, nó có thể thêm nhưgnx số 0 và log2 cần phải được tính từ phần số nguyên. Để xử lý với số bất thường, chúng ta sử dụng:
```
const float v;
int c;
int x = *(const int *) &v;

c = x >> 23;
if (c)
{
  c -= 127;
}
else
{
  register unsigned int t;
  if (t = x >> 16)
  {
    c = LogTable256[t] - 133;
  }
  else
  {
    c = (t = x >> 8) ? LogTable256[t] - 141 : LogTable256[x] - 149;
  }
}
```
Tìm vị trí bit 1 cao nhất với số float 32-bit IEEE (cho số nguyên dương)
```
const int r;
const float v;
int c;

c = *(const int *) &v;
c = ((((c - 0x3f800000) >> r) + 0x3f800000) >> 23) - 127;
```
Nếu r là 0, chúng ta có c = int(log2((double)v)). Nếu r là 1, chúng ta có c = int(log2(sqrt(double)v)). Nếu r là 2, chúng ta có c =int(log2(pow((double)v, 1.0/4)))

Tìm vị trí bit 1 cao nhất bằng bảng tham chiếu
```
static const char LogTable256[256] = 
{
#define LT(n) n, n, n, n, n, n, n, n, n, n, n, n, n, n, n, n
    -1, 0, 1, 1, 2, 2, 2, 2, 3, 3, 3, 3, 3, 3, 3, 3,
    LT(4), LT(5), LT(5), LT(6), LT(6), LT(6), LT(6),
    LT(7), LT(7), LT(7), LT(7), LT(7), LT(7), LT(7), LT(7)
};

unsigned int v;               // số 32-bit
unsigned r;                   // r là kết quả
register unsigned int t, tt;  // lưu giá trị tạm

if (tt = v >> 16)
{
  r = (t = tt >> 8) ? 24 + LogTable256[t] : 16 + LogTable256[tt];
}
else
{
  r = (t = v >> 8) ? 8 + LogTable256[t] : LogTable256[v];
}
```
Dùng bảng tham chiếu chỉ cần 7 phép tính để tìm ra kết quả của số 32-bit. Nếu mở rộng ra 64-bit, thì sẽ cần 9 phép tính. Một phép tính khác có thể được lược bỏ bằng cách sử dụng 4 bảng, với những giá trị thêm cho mỗi bảng. Sử dụng bảng có thể nhanh, phụ thuộc vào kiến trúc máy.

Nếu giá trị nhập vào luôn là 32-bit, có thể cân nhắc để sử dụng đoạn mã sau.
```
if (tt = v >> 24)
{
  r = 24 + LogTable256[tt];
}
else if (tt = v >> 16)
{
  r = 16 + LogTable256[tt];
}
else if (tt = v >> 8)
{
  r = 8 + LogTable256[tt];
}
else
{
  r = LogTable256[v];
}
```
Để khởi tạo bảng một cách tự động:
```
LogTable256[0] = LogTable256[1] = 0;
for (int i=2; i<256; i++)
{
  LogTable256[i] = 1 + LogTable256[i / 2];
}
LogTable256[0] = -1;  // nếu bạn muốn log(0) return -1
```
Tìm vị trí bit 1 cao nhất của số nguyên N-bit trong O(lg(N)) phép tính
```
unsigned int v; // giá trị 32-bit muốn tìm log2
const unsigned int b[] = {0x2, 0xC, 0xF0, 0xFF00, 0xFFFF0000};
const unsigned int S[] = {1, 2, 4, 8, 16}
int i;

regiser unsigned int r = 0;
for (i=4; i>=0; i--)
{
  if (v & b[i])
  {
    v >>= S[i];
    r |= S[i];
  }
}
```
Hoặc nếu CPU mà rẽ nhánh chậm
```
unsigned int v;
register unsigned int r;
register unsigned int shift;

r =     (v > 0xFFFF) << 4; v >>= r;
shift = (v > 0xFF  ) << 3; v >>= shift; r |= shift;
shift = (v > 0xF   ) << 2; v >>= shift; r |= shift;
shift = (v > 0x3   ) << 1; v >>= shift; r |= shift;
                                        r |= (v >> 1);
```
Hoặc nếu bạn biết rằng v là lũy thừa của 2
```
unsigned int v;
static const unsigned int b[] = {0xAAAAAAAA, 0xCCCCCCCC, 0xF0F0F0F0, 0xFF00FF00, 0xFFFF0000};
register unsigned int r = (v & b[0]) != 0;
for (i=4; i>0; i--)
{
  r |= ((v & b[i]) != 0) << i;
}
```
Tất nhiên, để mở rộng đoạn mã để tìm log của số có 33 đến 64 bit, chúng ta sẽ thâm các phần tử như 0xFFFFFFFF00000000, vào b, thêm 32 vào S, và lặp từ 5 đến 0. Phương pháp này là chậm hơn nhiều so vưới phiên bản bảng tham chiếu, nhưng nếu bạn không muốn một bảng lớn hoặc kiến trúc máy là chậm khi truy cập bộ nhớ, nó là một sự lựa chọn. Biến thể thứ 2 là cần nhiều phép tính hơn, nhưng nó có thể nhanh trên máy mà rẽ nhánh chậm.

Tìm vị trí bit 1 cao nhất của số nguyên N-bit trong O(lg(N)) phép tính với phép nhân và tham chiếu
```
uinit32_t v;
int r;

static const int MultiplyDeBruijnBitPosition[32] =
{
  0, 9, 1, 10, 13, 21, 2, 29, 11, 14, 16, 18, 22, 25, 3, 30,
  8, 12, 20, 28, 15, 17, 24, 7, 19, 27, 23, 6, 26, 5, 4, 31
}

v |= v >> 1;
v |= v >> 2;
v |= v >> 4;
v |= v >> 8;
v |= v >> 16;

r = MultiplyDeBruijnBitPosition[(uint32_t)(v * 0x07C4ACDDU) >> 27];
```
Đoạn mã trên tính log base 2 của số nguyên 32-bit vưới một bảng tham chiếu nhỏ và phép nhân. Nó cần 13 phép tính, so với tối đa 20 phép tính của phiên bản trước đó. Phương pháp bảng cần ít tính toán hơn, nhưng nó cũng lý do để phân vân giữa kích thước bảng và tốc độ.

Nếu bạn biết rằng v là lũy thừa của 2, bạn có thể chỉ cần như sau:
```
static const int MultiplyDeBruijnBitPosition2[32] =
{
  0, 1, 28, 2, 29, 14, 24, 3, 30, 22, 20, 15, 25, 17, 4, 8,
  31, 27, 13, 23, 21, 19, 16, 7, 26, 12, 18, 6, 11, 5, 10, 9
}

r = MultiplyDeBruijnBitPosition2[(uint32_t)(v * 0x077CB531U) >> 27]
```
Tìm số nguyên là 10^x nhỏ hơn gần nhất của một số nguyên
```
unsigned int v; // giá trị không âm cần tìm
int r;          // kết quả
int t;          // biến tạm thời

static unsigned int const PowersOf10[] =
{1, 10, 100, 1000, 1000, 100000, 1000000, 10000000, 100000000, 1000000000};

t = (IntegerLogBase2(v) + 1) * 1233 >> 12;  // sử dụng hàm lg2 ở trên
r = t - (v < PowersOf10[t]);
```
Tính log base 10 của số nguyên đầu tiên sử dụng kỹ thuật tìm log base 2 ở trên. Bằng quan hệ log10(v) = log2(v) / log2(10), chúng ta cần nhân nó với 1/log2(10), mà sấp xỉ là 1233/4096, hoặc 1233 và dịch phải 12. Thêm 1 là cần thiết vì IntegerLogBase2 làm tròn xuống. Cuối cùng, giá trị của t là sấp xỉ và có thể lệch 1, giá trị chính xác được tìm thấy bằng cách trừ đi v < PowersOf10[t].
  
Tìm số nguyên là 10^x nhỏ hơn gần nhất của một số nguyên với cách rõ ràng
```
unsigned int v; // số không âm cần tính
int r;          // kết quả

r = (v >= 1000000000) ? 9 : (v >= 100000000) ? 8 : (v >= 10000000) ? 7 : 
  (v >= 1000000) ? 6 : (v >= 100000) ? 5 : (v >= 10000) ? 4 : 
  (v >= 1000) ? 3 : (v >= 100) ? 2 : (v >= 10) ? 1 : 0;
```
phương pháp này hoạt động tốt khi giá trị nhập vào là phân bố đồng đều trên 32-bit bởi vì 76% giá trị nhập vào là gặp ở lần so sánh đầu tiên, 21% ở lần so sánh thứ 2, 2% ở lần thứ 3, và tiếp tục, ít hơn 2.6 phép tính cần thiết ở mức trung bình.
