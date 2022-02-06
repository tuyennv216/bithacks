Nếu bạn cần dấu âm chỉ khi cờ là false, thì hãy sử dụng như sau để tránh rẽ nhánh:
```
bool fDontNegate; // cờ chỉ rằng chúng ta không nên đổi dấu v
int v;            // giá trị nhập vào thành âm nếu fDontNegate là false
int r;            réult = fDontNegate ? v : -v;

r = (fDontNegate ^ (fDontNegate - 1)) * v;
```
Nếu bạn muốn chỉ thành âm khi cờ là true, thì hãy sử dụng:
```
bool fNegate; // cờ chỉ nếu chúng là nên negate v
int v;        // giá trị nhập vào là âm nếu fNegate là true
int r;        // result = fNegate ? -v : v

r = (v ^ -fNegate) + fNegate;
```
