```
unsigned int a;     // giá trị hợp mà không dùng mặt nạ
unsigned int b;     // giá trị hợp mà có dùng mặt nạ
unsigned int mask;  // 1 nếu bits từ b nên được chọn; 0 nếu từ a
unsigned int r;     // kết quả result of (a & ~mask) | (b & mask)

r = a ^ ((a ^ b) & mask);
```