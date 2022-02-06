Đếm số bits bằng cách bình thường  
```
unsigned int v; // đếm số lượng bit trong v
unsigned int c; // c là tổng số lượng bit đã đếm

for (c = 0; v; v >>= 1)
{
  c += v & 1;
}
```
Cách tiếp cận này yêu cầu 1 phép tính cho mỗi bit, nên 1 số 32-bit sẽ có 32 phép tính.  
Đếm số bits 1 bằng bảng tham chiếu
```
static const unsigned char BitsSetTable256[256] =
{
# defined B2(n) n,      n+1,      n+1,      n+2
# defined B4(n) B2(n),  B2(n+1),  B2(n+1),  B2(n+2)
# defined B6(n) B4(n),  B4(n+1),  B4(n+1),  B4(n+2)
  B6(0),  B6(1), B6(1),  B6(2)
};

unsigned int v; // đếm số lượng bit 1 của số 32-bit
unsigned int c; // c là tổng số bit 1 trong v

// cách 1
c = BitsSetTable256[v & 0xff] + 
    BitsSetTable256[(v >>8) & 0xff] + 
    BitsSetTable256[(v >> 16) & 0xff] + 
    BitsSetTable256[v >> 24];

// cách 2
unsigned char * p = (unsigned char *) &v;
c = BitsSetTable256[p[0]] + 
    BitsSetTable256[p[1]] + 
    BitsSetTable256[p[2]] + 
    BitsSetTable256[p[3]];
    
// để khởi tạo bảng tham chiếu một cách tự động
BitsSetTable256[0] = 0;
for (int i=0 i<256; i++)
{
  BitsSetTable256[i] = (i & 1) + BitsSetTable256[i / 2];
}
```
Đếm số bits theo Brian Kernighan  
```
unsigned int v; // đếm số bit 1 của v
unsigned int c; // c lưu tổng số bit 1 của v
for (c = 0; v; c++)
{
  v &= v - 1;
}
```
Phương pháp Brian Kernighan đi qua vòng lặp với số lần bằng với số bit.  
Đếm số bits của 14, 24, 32-bit sử dụng mã chỉ thị 64-bit
```
unsigned int v; // đếm số bit 1 của v
unsigned int c; // c lưu tổng số bit 1 của v

// cách 1, cho 14 bit trong v
c = (v * 0x200040008001ULL & 0x111111111111111ULL) % 0xf;

// cách 2, cho 24 bit trong v
c = ((v & 0xfff) * 0x1001001001001ULL & 0x84210842108421ULL) % 0xf;
c += (((v & 0xfff000) >> 12) * 0x1001001001001ULL & 0x84210842108421ULL) % 0x1f;

// cách 3, cho 32 bit trong v
c = ((v & 0xfff) * 0x1001001001001ULL & 0x84210842108421ULL) % 0x1f;
c += (((v & 0xfff000) >> 12) * 0x1001001001001ULL & 0x84210842108421ULL) % 0x1f;
c += ((v >> 24) * 0x1001001001001ULL & 0x84210842108421ULL) % 0x1f;
```
Phương pháp này yêu cầu CPU 64-bit với phép chia dư nhanh để hiệu quả. Cách 1 cần 3 phép tính, cách 2 cần 10, và cách 3 cần 15.  
Đếm số bits bằng đa tác vụ
```
unsigned int v; // đếm số bit 1 của v
unsigned int c; // c lưu tổng số bit 1 của v
static const int S[] = {1, 2, 4, 8, 16};  // con số ma thuật
static const int B[] = {0x55555555, 0x33333333, 0x0F0F0F0F, 0x00FF00FF, 0x0000FFFF};

c = v - ((v >> 1) & B[0]);
c = ((c >> S[1]) & B[1]) + (c & B[1]);
c = ((c >> S[2]) + c) & B[2];
c = ((c >> S[3]) + c) & B[3];
c = ((c >> S[4]) + c) & B[4];

Mảng B, biểu diễn nhị phân là:
B[0] = 0x55555555 = 01010101 01010101 01010101 01010101
B[1] = 0x33333333 = 00110011 00110011 00110011 00110011
B[2] = 0x0F0F0F0F = 00001111 00001111 00001111 00001111
B[3] = 0x00FF00FF = 00000000 11111111 00000000 11111111
B[4] = 0x0000FFFF = 00000000 00000000 11111111 11111111
```
Chúng ta có thể thay đổi thuật toán cho số nguyên với nhiều bit hơn bằng cách tiếp tục với con số ma thuật S và B. Nếu chúng ta có k bits, chúng ta cần một mảng S và có độ dài ceil(lg(k)), và chúng ta tính toán giống như biểu thức cho c như S và B dài hơn. Cho số 32-bit, 16 phép tính được sử dụng.

Một cách nhanh hơn để đếm trên số nguyên 32-bit là:
```
v = v - ((v >> 1) & 0x55555555);                    // sử dụng lại input v
v = (v & 0x33333333) + ((v >> 2) & 0x33333333);     // tính trung gian
c = ((v + (v >> 4) & 0xF0F0F0F) * 0x1010101) >> 24; // đếm ra kết quả
```
Thuật toán trên sử dụng 12 phép tính, giống như với bảng tham chiếu, nhưng tránh truy cập bộ nhớ và tiềm năng bộ nhớ tạm bị trượt. Nó là lai của phương pháp đa tác vụ và phương pháp phép nhân, nhưng nó không sử dụng chỉ thị 64-bit. Số lượng bit 1 được đặt trong đa tác vụ, và tổng số lượng bit 1 được tính bằng nhân với 0x10101010 và dịch phải 24 bits.

Một mã phát sinh của thuật toán đếm bit cho số nguyên với tối đa 128 bit (kiểu dữ liệu đặt là T) là:
```
v = v - ((v >> 1) & (T)~(T)0/3);
v = (v & (T)~(T)0/15*3) + ((v >> 2) & (T)~(T)0/15*3);
v = (v + (v >> 4)) & (T)~(T)0/255*15;
c = (T)(v * ((T)~(T)0/255)) >> (sizeof(T) - 1) * CHAR_BIT;
```
Đếm số bits 1 từ bit quan trọng nhất đến một vị trí
```
uint 64_t v;      // tính số bit 1 trong v từ MSB đến pos
unsigned int pos; // vị trí bit muốn đếm đến
uint64_t r;       // kết quả trả về số 1 ở đây

// dịch bit sau khi có vị trí
r = v >> (sizeof(v) * CHAR_BIT - pos);
// đếm số bit bằng đa tác vụ
r = r - ((r >> 1) & ~0UL/3);
r = (r & ~0UL/5) + ((r >> 2) & ~0UL/5);
r = (r + (r >> 4)) & ~0UL/17;
r = (r * (0UL/255)) >> ((sizeof(v) - 1) * CHAR_BIT);
```
Chọn vị trí bit từ bit quan trọng nhất với một số đếm cho trước
Đoạn mã 64-bit dưới đây sẽ tìm vị trí của bit 1 thứ r khi đếm từ bên trái, và trị trí tìm thấy được trả về. Nếu r lớn hơn độ dài của số thì sẽ trả về 64. Đoạn mã có thể được chỉnh sửa để chạy cho 32-bit hoặc chạy từ bên phải.
```
uint64_t v;           // số input cần tìm
unsigned int r;       // số thứ tự của bit 1
unsigned int s;       // kết quả trả về là vị trí của bit 1 thứ r
uint64_t a, b, c, d;  // biến tạm
unsigned int t;       // biến tạm

a = v - ((v >>1) & ~0UL/3);
b = (a & ~0UL/5) + ((a >> 2) & ~0UL/5);
c = (b + (b >> 4)) & ~0UL/0x11;
d = (c + (c >> 8)) & ~0UL/0x101;
t = (d >> 32) + (d >> 48);

s = 64;
s -= ((t - r) & 256) >> 3; r -= (t & ((t - r) >> 8));
t = (d >> (s - 16)) & 0xff;
s -= ((t - r) & 256) >> 4; r -= (t & ((t - r) >> 8));
t = (d >> (s - 8)) & 0xf;
s -= ((t - r) & 256) >> 5; r -= (t & ((t - r) >> 8));
t = (d >> (s - 4)) & 0x7;
s -= ((t - r) & 256) >> 6; r -= (t & ((t - r) >> 8));
t = (d >> (s - 2)) & 0x3;
s -= ((t - r) & 256) >> 7; r -= (t & ((t - r) >> 8));
t = (d >> (s - 1)) & 0x1;
s -= ((t - r) & 256) >> 8;
s = 65 - s;
```
