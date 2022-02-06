Hoán đổi giá trị với phép trừ và cộng
```
# define SWAP(a, b) ((&(a) == &(b)) || (((a) -= (b)), ((b) += (a)), ((a) = (b) - (a))))
```
Giá trị hoán đổi của a và b mà không sử dụng biến trung gian. Bắt đầu kiểm tra nếu a và b cùng vị trí trong bộ nhớ thì 2 số sẽ không cần hoán đổi. Trình biên dịch có thể bỏ sót nó trong quá trình tối ưu. Nếu bạn bật cờ tràn bộ đệm, và truyền vào một giá trị không âm thì lỗi sẽ không xuất hiện.  
Phương pháp XOR tiếp theo có thể sẽ nhanh hơn trên một số máy. Không sử dụng nó với số kiểu float (trừ khi số float biểu diễn một số nguyên).

Hoán đổi giá trị với phép đảo bit
```
#define SWAP(a, b) (((a) ^= (b)), ((b) ^= (a)), ((a) ^= (b)))
```
Đây là một thủ thuật đã có từ lâu để hoán đổi giá trị của hai số a và b mà không sử dụng thêm bộ nhớ hoặc biến.

Hoán đổi 2 đoạn bit với phép đảo bit
```
unsigned int i, j;    // vị trí của bit cần hoán đổi
unsigned int n;       // số bit của mỗi đoạn
unsigned int b;       // bits cần hoán đổi trong b
unsigned int r;       // bit-đã hoán đổi lưu ở đây

unsigned int x = ((b >> i) ^ (b >> j)) & ((1U << n) - 1);
r = b ^ ((x << i) | (x << j));
```
Ví dụ hoán đổi đoạn bit bởi số có giá trị b = [001] 0 [111] 1 = 0010 1111 (số nhị phân, 3 bit 001 hoán đổi với 3 bit 111)
và chúng ta muốn hoán đổi bắt đầu với i = 1, và độ dài n = 3 (3 bit ở bên trái 001), với đoạn bắt bắt dầu từ j = 5, (3 bit tiếp theo có giá trị 111).  

Ta có kết quả r = [111] 0 [001] 1 = 1110 0011

Phương pháp này tương tự với thủ thuật hoán đổi sử dụng XOR thông thường, nhưng hoạt động với những đoạn bit riêng biệt.  
Giá trị của x lưu kết quả của phép XOR những cặp bit mà chúng ta muốn hoán đổi, và sau đó đoạn bit được đặt vào kết quả của chúng bằng phép XOR với x.  
Tất nhiên, kết quả sẽ là không xác định nếu đoạn bit bị tràn bộ đệm (vượt quá số lượng bit của kiểu dữ liệu).
