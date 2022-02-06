Dấu mở rộng của một hằng số
Mở rộng dấu là kiểu tự động có sẵn, giống như char và int. Nhưng giả định bạn có 2 chữ số có dấu, được lưu trữ chỉ sử dụng b bits.  
Tiếp theo, bạn muốn convert nó sang int, thứ có nhiều hơn b bits. Cách copy đơn giản sẽ hoạt động nếu như x là số dương, còn nếu âm, dấu cần được mở rộng.  
Ví dụ, nếu bạn chỉ có 4 bits để lưu trữ số, thì -3 sẽ là 1101 trong mã nhị phân. Nếu bạn có 8 bits, thì -3 sẽ là 11111101. Khi đó, bit chứa dấu của số sẽ được lặp lại để fill với những bit mở rộng. Ví dụ convert từ 5 bit sang integer đầy đủ.
```
int x;  // chuyển đổi nó từ 5 bits sang integer đầy đủ
int r;  // kết quả lưu ở đây
struct { signed int x:5;} s;
r = s.x = x;
```
Mã mẫu C++ dưới đấy sử dụng cùng cách thức để convert từ B bits trong 1 phép tính (tất nhiên trình biên dịch sẽ tạo ra nhiều hơn).
```
template <typename T, unsigned B>
inline T signextend(const T x)
{
  struct {T x:B;} s;
  return s.x = x;
}

int r = signextend<signed int,5>(x);  // mở rộng của số 5 bit gán cho r
```
Dấu mở rộng của một biến  
Đôi khi chúng ta cần mở rộng dấu cho một số mà chúng ta không biết số lượng bits mà nó được biểu diễn. (Hoặc chũng ta đang lập trình ngôn ngữ như Java, nó bị thiếu trường bits)
```
unsigned b; // số lượng bit mà biểu diễn số x
int x;      // mở rộng dấu số b-bit gán cho r
int r;      // kết quả mở rộng dấu

x = x & ((1U << b) - 1);  // bỏ quả nếu bits của x trên vị trí b đã là 0
r = (x ^ m) - m;
```
Mã trên yêu cầu 4 phép tính, nhưng khi số lượng bit là hằng số hơn là biến số, thì nó yêu cầu chỉ 2 phép tính, vì những bit cao hơn đã là 0.  
Một phiên bản nhanh hơn nhưng ít linh động hơn, nó không phụ thuộc vào số lượng bits của x trên vị trí b cần là 0;
```
int const m = CHAR_BIT * sizeof(x) - b;
r = (x << m) >> m;
```
Dấu mở rộng của một biến với 3 phép toán  
Đoạn mã sau có thể chậm trên một số máy, phụ thuộc vào yêu cầu cho phép nhân và chia.
```
usigned b;  // số lượng bit để biểu diễn số x
int x;      // mở rộng số b-bit sang số r
int r;      // kết quả lưu ở đây
#define M(B) (1U << ((sizeof(x) * CHAR_BIT) - B))   // CHAR_BIT là số lượng bit của 1 byte
static int const multipliers[] = 
{
  0,      M(1),   M(2),   M(3),   M(4),   M(5),   M(6),
  M(8),   M(9),   M(10),  M(11),  M(12),  M(13),  M(15),
  M(16),  M(17),  M(18),  M(19),  M(20),  M(21),  M(23),
  M(24),  M(25),  M(26),  M(27),  M(28),  M(29),  M(31),
  M(32)
};  // thêm nếu bạn muốn sử dụng nhiều hơn 64 bit

static int const divisors[] =
{
  0,     ~M(1),   M(2),   M(3),   M(4),   M(5),   M(6),
  M(8),   M(9),   M(10),  M(11),  M(12),  M(13),  M(15),
  M(16),  M(17),  M(18),  M(19),  M(20),  M(21),  M(23),
  M(24),  M(25),  M(26),  M(27),  M(28),  M(29),  M(31),
  M(32)
};  // thêm cho 64-bit
undef M

r = (x * multipliers[b]) / divisors[b];
```
Đoạn dưới đây là không linh động, nhưng trên các kiến trúc mà phép có phép tính dịch phải bit, vẫn giữ dấu, thì nó sẽ nhanh.
```
const int s = -b;   // hoặc: sizeof(x) * CHAR_BIT - b;
r = (x << s) >> s;
```
