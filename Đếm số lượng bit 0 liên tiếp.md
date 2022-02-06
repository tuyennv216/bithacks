Đếm số lượng bit 0 liên tiếp bên phải tuần tự
```
unsigned int v;
int c;
if (v)
{
  v = (v ^ (v - 1)) >> 1;
  for (c=0; v; c++)
  {
    v >>= 1;
  }
}
else
{
  c = CHAR_BIT * sizeof(v);
}
```
Trung bình số lượng số bit 0 trong 1 số ngẫu nhiên là 0, nên thời gian O(số lượng bit 0) là không tệ so với những phương pháp nhanh hơn dưới đây.

Đếm số lượng bit 0 liên tiếp bên phải bằng đa tác vụ
```
unsigned int v;
unsigned int c = 32;
v &= -signed(v);
if (v) c--;
if (v & 0x0000FFFF) c -= 16;
if (v & 0x00FF00FF) c -= 8;
if (v & 0x0F0F0F0F) c -= 4;
if (v & 0x33333333) c -= 2;
if (v & 0x55555555) c -= 1;
```
Ở đây, chúng ta cơ bản làm các phép toán giống với tìm log base 2 bằng đa tác vụ, nhưng chúng ta tách bit thấp nhất, và sau đó xử lý từ cao tới thấp. Số lượng các phép tính là khoảng 3 * lg(N) + 4, với số N bit.

Đếm số lượng bit 0 liên tiếp bằng tìm kiếm nhị phân
```
unsigned int v;
unsigned int c;

// nếu v == 0, thì c = 31
if (v & 0x1)
{
  c = 0;
}
else
{
  c = 1;
  if ((v & 0xffff) == 0)
  {
    v >>= 16;
    c += 16;
  }
  if ((v & 0xff) == 0)
  {
    v >>= 8;
    c += 8;
  }
  if ((v & 0xf) == 0)
  {
    v >>= 4;
    c += 4;
  }
  if ((v & 0x3) == 0)
  {
    v >>= 2;
    c += 2;
  }
  
  c -= v & 0x1;
}
```
Đoạn mã trên là khá giống với phương pháp trước đó, nhưng nó tính số lượng các bit 0 ở cuối bằng cách tổng c tượng tự như cách tìm kiếm nhị phân. Ở bước đầu tiên, nó kiểm tra nếu 16 bit cuối là 0, thì sẽ thêm 16 vào c và dịch chuyển số sang bên phải 16, nó làm số lượng bit của v giảm đi một nửa. Tương tự cho đến khi số chỉ còn lại 1 bit.  
Phương pháp này là nhanh hơn phương pháp đầu khoảng 33% vì nội dung trong hàm rẽ nhánh thực hiện ít hơn.

Đếm số lượng bit 0 liên tiếp bằng chuyển sang số float
```
unsigned int v;
int r;
float f = (float)(v & -v);
r = (*(uint32_t *)&f >> 23) - 0x7f;
```
Mặc dù nó chỉ có 6 phép toán, thời gian chuyển từ số nguyên sang float có thể cao ở một số máy, phần mũ của số float 32-bit được làm tòn xuống, và phần nguyên được trừ bởi vị trí bit 1 thấp nhất trong v. Nếu v là 0 thì kết quả là -127.

Đếm số lượng bit 0 liên tiếp bằng chia dư và tham chiếu
```
unsigned int v;
int r;
static const int Mod37BitPosition[] =
{
  32, 0, 1, 26, 2, 23, 27, 0, 3, 16, 24, 30, 28, 11, 0, 13, 4,
  7, 17, 0, 25, 22, 31, 15, 29, 10, 12, 6, 0, 21, 14, 9, 5,
  20, 8, 19, 18
};

r = Mod37BitPosition[(-v & v) % 37];
```
Đoạn mã trên tìm kiếm số những số 0 mà ở bên phải, nên số 0100 sẽ có kết quả là 2. Nó sử dụng một điều là 32-bit vị trí sẽ liên hệ với số nguyên tố 37, nên phép chia dư cho 37 sẽ tạo ra 1 số duy nhất từ 0 đến 36. Và những số này có thể map cho những số 0 sử dụng bảng tham chiếu.

Đếm số lượng bit 0 liên tiếp bằng nhân và tham chiếu
```
unsigned int v;
int r;
static const int MultiplyDeBruijnBitPosition[32] = 
{
  0, 1, 28, 2, 29, 14, 24, 3, 30, 22, 20, 15, 25, 17, 4, 8, 
  31, 27, 13, 23, 21, 19, 16, 7, 26, 12, 18, 6, 11, 5, 10, 9
};
r = MultiplyDeBruijnBitPosition[((uint32_t)((v & -v) * 0x077CB531U)) >> 27];
```
Chuyển vector bit sang chỉ số của tập bit là một ví dụ sử dụng nó. Nó yêu cầu 1 hoặc nhiều phép tính hơn phiên bản chia dư trước đó, nhưng phép nhân có thể nhanh hơn. Biểu thức (v & -v) sẽ bổ sung thêm những bit 1 từ v.  
Hằng số 0x077CB531UL là một mã Bruijn, nó cũng cấp một mẫu duy nhất gồm 5 bit cao của các bit mà nó được nhân. Khi không có bit 1 nào, nó trả về 0.
