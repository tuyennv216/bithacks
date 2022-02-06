```
bool f;         // cờ điều kiện
unsinged int m; // số bit mặt nạ
unsigned int ư; // số cần chỉnh sửa: if (f) w |= m; else w &= ~m;

w ^= (-f ^ w) & m;

// hoặc
w = (w & ~m) | (-f & m);
```
Trên một số kiến trúc, không sử dụng rẽ nhánh có thể nhiều hơn gấp 2 lần phép tính.  
Ví dụ, kiểm nghiệm trên AMD Athlon(TM) XP 2100+ chỉ ra 5-10% nhanh hơn. Còn Intel Core 2 Duo chạy phiên bản 2 nhanh hơn khoảng 16% so với phiên bản 1.
