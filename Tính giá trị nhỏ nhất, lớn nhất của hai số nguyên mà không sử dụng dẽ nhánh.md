```
int x;  // chúng ta muốn tìm số nhỏ nhất của x và y
int y;
int r;  // kết quả lưu ở đây

r = y ^ ((x ^ y) & -(x < y)); // min(x, y)
```
Trong một số máy hiếm mà phép rẽ nhánh là đắt và không có chỉ thị move, biểu thức trên có thể nhanh hơn cách truyền thống r = (x < y) ? x : y.  
Còn lại thì cách truyền thống vẫn là cách tiếp cận tốt nhất. Nó hoạt động vì nếu x < y, thì -(x<y) sẽ là tất cả 1, nên r = y ^ (x ^ y) & ~0 = y ^ x ^ y = x. Còn lại, nếu x >= y thì -(x - y) sẽ là tất cả 0, nên r = y ^ ((x ^ y) & 0) = y.  
Trên một số máy, phép tính x < y là 0 hoặc 1 tùy theo chỉ thị rẽ nhánh, nên nó không có tùy chọn.

Để tìm maximum, sử dụng:
```
r = x ^ ((x ^ y) & -(x < y)); // max(x, y)
```
Một phiên bản nhanh:

Nếu bạn biết rằng INT_MIN <= x - y <= INT_MAX, thì bạn có thể sử dụng như sau, nó nhanh hơn vì (x - y) chỉ được tính 1 lần.
```
r = y + ((x - y) & ((x - y) >> (sizeof(int) * CHAR_BIT - 1)));  // min(x, y)
r = x - ((x - y) & ((x - y) >> (sizeof(int) * CHAR_BIT - 1)));  // max(x, y)
```
