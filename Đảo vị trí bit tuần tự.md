Đảo vị trí bit với cách rõ ràng
```
unsigned int v;                   // giá trị nhập vào cần đảo vị trí
unsigned int r = v;               // r sẽ là kết quả đảo ngược bit của v
int s = sizeof(v) * CHAR_BIT - 1; // bổ sung thêm dịch bit sang trái

for (v >>= 1; v; v >>= 1)
{
  r <<= 1;
  r |= v & 1;
  s--;
}
r <<= s;  // dịch trái bit nếu bit cao nhất là 0
```
Đảo vị trí bit loại word bằng bảng tham chiếu
```
static const unsigned char BitReverseTable256[256] =
{
# defineR2(n)     n,  n + 2*64    , n + 1*64    , n + 3*64
# defineR4(n) R2(n),  R2(n + 2*64), R2(n + 1*16), R2(n + 3*16)
# defineR6(n) R4(n),  R4(n + 2*4) , R4(n + 1*4) , R4(n + 4*4)
  R6(0),              R6(2)       , R6(1)       , R6(3)
};

unsigned int v;
unsigned int c;

// cách 1
c = (BitReverseTable256[v & 0xff] << 24) |
    (BitReverseTable256[(v >> 8) & 0xff] << 16) |
    (BitReverseTable256[(v >> 16) & 0xff] << 8) |
    (BitReverseTable256[(v >> 24) & 0xff]);

// cách 2
unsigned char * p = (unsigned char *) &v;
unsigned char * q = (unsigned char *) &c;
q[3] = BitReverseTable256[p[0]];
q[2] = BitReverseTable256[p[1]];
q[1] = BitReverseTable256[p[2]];
q[0] = BitReverseTable256[p[3]];
```
Phương pháp 1 cần 17 phép tính, phương pháp 2 cần 12 phép tính, giả định CPU có thể lấy và lưu các byte dễ dàng.

Đảo vị trí bit loại byte với 3 phép tính (phép nhân và chia dư 64-bit)
```
unsigned char b;  // đảo bit của số kiểu byte

b = (b * 0x0202020202ULL) & 0x010884422010ULL) % 1023;
```
Phép nhân tạo ra 5 phần copy của 1 byte (8-bit) và lưu vào số 64-bit. Phép toán AND sẽ chọn các bit mà đã được đảo vị trí, liên quan đến mỗi 10-bit của nhóm các bits. Phép nhân và AND copy các bit từ byte gốc nên chúng xuất hiện với chỉ 1 của tập 10-bit. Giá trị đảo ngược bit từ byte gốc lặp lại với mỗi 10-bit.  

Bước cuối cùng, liên quan đến việc chia dư cho 2^10 - 1, là là việc lấy các tập 10-bits từ vị trí (0-9, 10-19, 20-29, ...) của số 64-bit. Nó sẽ không chồng lên nhau, và tổng hợp lại các bước ta sẽ có các phép tính như vậy.

Đảo vị trí bit loại byte với 4 phép tính (phép nhân 64-bit)
unsigned char b;// đảo bit của số kiểu byte
```
b = ((b * 0x80200802ULL) & 0x0884422110ULL) * 0x0101010101ULL >> 32;
```
Bước tiếp theo sẽ biểu diễn các phép tính với các bit là a, b, c, d, e, f, g và h, chúng tổng hợp lại thành 1 byte (8-bit). Chú ý rằng phép nhân đầu tiên sẽ sao chép byte nhiều lần, trong khi đó phép toán cuối cùng sẽ tổng hợp lại chúng từ các byte khác.
```
                                                                                        abcd efgh (-> hgfe dcba)
*                                                      1000 0000  0010 0000  0000 1000  0000 0010 (0x80200802)
-------------------------------------------------------------------------------------------------
                                            0abc defg  h00a bcde  fgh0 0abc  defg h00a  bcde fgh0
&                                           0000 1000  1000 0100  0100 0010  0010 0001  0001 0000 (0x0884422110)
-------------------------------------------------------------------------------------------------
                                            0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
*                                           0000 0001  0000 0001  0000 0001  0000 0001  0000 0001 (0x0101010101)
-------------------------------------------------------------------------------------------------
                                            0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
                                  0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
                      0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
            0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
0000 d000  h000 0c00  0g00 00b0  00f0 000a  000e 0000
-------------------------------------------------------------------------------------------------
0000 d000  h000 dc00  hg00 dcb0  hgf0 dcba  hgfe dcba  hgfe 0cba  0gfe 00ba  00fe 000a  000e 0000
>> 32
-------------------------------------------------------------------------------------------------
                                            0000 d000  h000 dc00  hg00 dcb0  hgf0 dcba  hgfe dcba  
&                                                                                       1111 1111
-------------------------------------------------------------------------------------------------
                                                                                        hgfe dcba
```
Ghi chú rằng 2 bước cuối có thể được kết hợp trên một số bộ vi xử lý bởi vì thành ghi có thể sử dụng như một số kiểu byte;  
Và phép nhân có thể lưu trữ kết quả trên thanh ghi 32-bit và lấy các byte thấp. Như vậy, nó có thể chỉ cần 6 phép tính.

Đảo vị trí bit loại byte với 7 phép tính (chỉ dành cho 32-bit)
```
b = ((b * 0x0802LU & 0x22110LU) | (b * 0x8020LU & 0x88440LU)) * 0x10101LU >> 16;
```
Kết quả nên được chuyển về unsigned int char * để loại bỏ những bit ngẫu nhiên.

Đảo vị trí số lượng N-bit bằng đa tác vụ với 5 * lg(N) phép tính
```
unsigned int v;

v = ((v >> 1) & 0x55555555) | ((v & 0x55555555) << 1);
v = ((v >> 2) & 0x33333333) | ((v & 0x33333333) << 2);
v = ((v >> 4) & 0x0F0F0F0F) | ((v & 0x0F0F0F0F) << 4);
v = ((v >> 8) & 0x00FF00FF) | ((v & 0x00FF00FF) << 8);
v = ((v >> 16)            ) | ((v             ) << 16);
```
Phương pháp trên và cách tốt nhất nếu N là một số lớn. Nếu sử dụng cho số 64-bit hoặc lớn hơn, thì sẽ cần thêm nhiều dòng hơn theo mẫu; còn như trên thì 32 bit thấp nhất được đảo vị trí.
