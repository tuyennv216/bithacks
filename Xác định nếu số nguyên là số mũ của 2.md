```
unsigned int v; // chúng ta muốn thấy nếu là số mũ của 2
bool f;         // kết quả lưu ở đây

f = (v & (v - 1)) == 0;
```
Nếu v = 0 thì biểu thức là chưa chính xác, vì vậy sẽ đổi lại thành:
```
f = v && !(v & (v - 1));
```
