```
int v;    // chúng ta muốn tìm dấu của số v
int sign; // kết quả lưu ở đây

sign = -(v < 0);  // nếu v < 0 thì -1, còn lại 0

// hoặc, không sử dụng rẽ nhánh trên CPU với ghi IA32
// CHAR_BIT là độ dài bit của một byte, thường là 8
sign = -(int)((unsnigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));

//hoặc, với ít chỉ mã thị hơn, nhưng ít uyển chuyển hơn
sign = v >> (sizeof(int) * CHAR_BIT - 1);
```
Phép tính cuối cùng thực hiện sign = v >> 31 cho số nguyên 32-bit. Nó là 1 phép tính nhanh hơn cách thông thường, sign = -(v < 0). Thủ thuật này hoạt động vì dấu của số nguyên là giá trị của bit xa nhất bên trái được sao chép đến bit bên phải. Bit xa nhất bên trái là 1 khi số có giá trị âm và là 0 nếu ngược lại. Tuy nhiên, nó chỉ đúng cho kiến trúc cụ thể.

Ngoài ra bạn có thể muốn kết quả là -1 hoặc +1, hãy sử dụng
```
sign = +1 | (v >> (sizeof(int) * CHAR_BIT - 1));  // nếu v < 0 thì -1, còn lại +1
```
Một trường hợp khác, nếu bạn muốn kết quả là -1, 0 hoặc +1, hãy sử dụng
```
sign = (v != 0) | -(int)((unsigned int)((int)v) >> (sizeof(int) * CHAR_BIT - 1));

// hoặc, nhanh hơn nhưng ít linh động hơn
sign = (v != 0) | (v >> (sizeof(int) * CHAR_BIT - 1));  // -1, 0, hoặc +1

// hoặc, linh động, ngắn gọn, và rất có thể tốc độ:
sign = (v > 0) - (v < 0); -1, 0, hoặc +1

// nếu thay vào đó bạn muốn biết nếu số đó là không âm, trả về +1 hoặc còn lại 0, hãy sử dụng
sign = 1 ^ ((unsigned int)v >> (sizeof(int) * CHAR_BIT - 1)); // nếu v < 0 thì 0, còn lại 1
```
