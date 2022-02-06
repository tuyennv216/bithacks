Kiểm tra số bit 1 là chẵn lẻ theo cách bình thường
```
unsigned int v;       // số sẽ được tính chẵn lẻ
bool parity = false;  // kết quả tính chẵn lẻ của v

while (v)
{
  parity = !partity;
  v = v & (v - 1);
}
```
Đoạn mã trên sử dụng cách tiếp cận đếm bit giống Brian Kernigan. Nó có thời gian chạy bằng với số bit 1.

Kiểm tra số bit chẵn lẻ bằng bảng tham chiếu
```
static const bool ParityTable256[256] =
{
# define P2(n) n,     n^1,      n^1,      n
# define P4(n) P2(n), P2(n^1),  P2(n^1),  P2(n)
# define P6(n) P4(n), P4(n^1),  P4(n^1),  P4(n)
  P6(0),              P6(1),    P6(1),    P6(0)
}

unsigned char b;                  // giá trị byte muốn kiểm tra số bit chẵn lẻ
bool parity = ParityTable256[b];

// hoặc cho số 32-bit
unsigned int v;
v ^= v >> 16;
v ^= v >> 8;
bool parity = ParityTable256[v & 0xff];

// biến thể
unsigned char * p = (unsigned char *) &v;
parity = ParityTable256[p[0] ^ p[1] ^ p[2] ^ p[3]];   
```
Kiểm tra số bit chẵn lẻ của một byte sử dụng phép nhân 64-bit và phép chia dư
```
unsigned char b;
bool parity = (((b * 0x0101010101010101ULL) & 0x8040201008040201ULL) & 0x1FF) & 1;
```
Phương pháp trên sử dụng 4 phép tính, nhưng chỉ hoạt động với kiểu byte.

Kiểm tra số bit chẵn lẻ của biến word sử dụng phép nhân
Phương pháp sau đây tính số bit chẵn lẻ của số 32-bit với 8 phép tính nhân
```
unsigned int v;   // số 32-bit
v ^= v >> 1;
v ^= v >> 2;
v = (v & 0x11111111U) * 0x11111111U;
return (v >> 28) & 1;

Với số 64-bit thì 8 phép tính cũng có thể tính toán được
unsigned long long v;   // số 64-bit
v ^= v >> 1;
v ^= v >> 2;
v = (v & 0x1111111111111111UL) * 0x1111111111111111UL;
return (v >> 60) & 1;

Kiểm tra số bit chẵn lẻ bằng đa tác vụ
unsigned int v;
v ^= v >> 16;
v ^= v >> 8;
v ^= v >> 4;
v &= 0xf;
return (0x6996 >> v) & 1;
```
Phương pháp trên trong khoảng 9 phép tính, và hoạt động với số 32-bit. Nó có thể tối ưu để hoạt động với kiểu byte trong 5 phép tính bằng cách loại bỏ 2 dòng phía sau dòng "unsigned int v;". Đầu tiên dịch chuyển và XOR với 8 bit của số 32-bit với nhau, sau đó lấy kết quả là 8-bit thấp nhất của v.  
Tiếp theo dãy số ma thuật 0110 1001 1001 0110 (0x6996) sẽ dịch chuyển sang phải bởi giá trị biểu diễn bởi những bit thấp nhất. Con số giống như chữ ký 16-bit của bảng tham chiếu vị trí chẵn lẻ tính bởi 4-bit thấp nhất của v.  
Kết quả sẽ trả về tính chẵn lẻ của các bit 1.
