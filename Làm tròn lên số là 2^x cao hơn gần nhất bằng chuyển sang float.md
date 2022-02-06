```
unsigned int const v; // số nguyên 32-bit
unsigned int r;       // giá trị trả về (3 -> 4, 8 -> 8)

if (v > 1)
{
  float f = (float) v;
  unsigned int const t = 1U < ((*(unsigned int *) &f >> 23) - 0x7f);
  r = t << (t < v);
}
else
{
  r = 1;
}
```
Đoạn mã trên sử dụng 8 phép tính, nhưng hoạt động với tất cả v <= (1<<31);
```
Một phiên bản khác cho 1 <v < (1<<25) là:

float = (float)(v - 1);
r = 1U << ((*(unsigned int*)(&f) >> 23) - 126);
```
Mặc dù cả 2 phiên bản đều sử dụng khoảng 6 phép tính, nhưng nó có thể 3x lần chậm hơn so với kỹ thuật bên dưới (với 12 phép tính) khi tính điểm trên Anthlon(TM)XP2100+CPU.

Làm tròn lên số là 2^x cao hơn gần nhất
```
unsigned int v;
v--;
v|= v >> 1;
v|= v >> 2;
v|= v >> 4;
v|= v >> 8;
v|= v >> 16;
v++;
```
Với 12 phép toán, đoạn mã trên tính số tiếp theo mà lũy thừa của 2 cho số 32-bit. Kết quả có thể được thể hiện từ công thức 1U << (lg(v - 1) + 1). Chú ý rằng trong trường hợp khi v = 0, nó trả về 0, nó không phải là lũy thừa của 2, bạn có thể muốn thêm biểu thức v += (v == 0) để chữa cho trường hợp đó.  
Nó có thể nhanh hơn bởi 2 phép toán sử dụng công thức và phương pháp log base 2 bằng bảng tham chiếu, nhưng có một vài tình huống, tìm kiếm bảng là không linh động, và đoạn code trên có thể tốt hơn.
