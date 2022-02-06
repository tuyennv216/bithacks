```
int v;          // chúng ta muốn tìm giá trị tuyệt đối của việc
unsigned int r; // kết quả lưu ở đây
int const mask = v >> sizeof(int) * CHAR_BIT - 1;

r = (v + mask) ^ mask;
```
Phiên bản được cấp sáng chế:
```
r = (v ^ mask) - mask;
```
Một vài CPU không có chỉ thị cho phép tính giá trị tuyệt đối (hoặc trình biên dịch không sử dụng chúng). Trên máy tính mà rẽ nhánh là đắt, biểu thức trên có thể nhanh hơn cách tiếp cận thông thường r = (v < 0) ? -(unsigned)v : v, dù số lượng của phép toán là như nhau.