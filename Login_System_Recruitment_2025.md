### Recon
- Đầu tiên khi mở chall lên, ta sẽ được giao diện login cơ bản như thế này
  <img width="837" height="395" alt="image" src="https://github.com/user-attachments/assets/2c040546-808c-40a9-b6de-9ac7a7bfa5f4" />

- Quan sát source code, ta thấy được đoạn code chứa thông tin users và mật khẩu
  ```
  $users = [
    'nightcore' => 'nightcore',
    'tadokun468' => 'tadokun468',
    'admin' => $flag
  ];
  ```
- Ta sẽ thử đăng nhập bằng username nightcore để xem có được gì không?
  <img width="861" height="175" alt="image" src="https://github.com/user-attachments/assets/5da701f8-919a-4aa2-a2b1-28a89a96e8d5" />
  
- Có vẻ như không có gì ở đây cả. Vậy nên tôi sẽ bắt request từ burpsuite để xem có gì có thể khai thác được không nhé.
  <img width="1487" height="596" alt="image" src="https://github.com/user-attachments/assets/2c73aff8-f89e-4275-9582-0552d01195fb" />
  <img width="1490" height="595" alt="image" src="https://github.com/user-attachments/assets/8277cd62-c135-40c8-8608-20ef5ff1d708" />

- Vẫn không có gì cả, vậy nên ta sẽ mở lại source để xem có thể khai thác được gì không.
### Exploit
- Ta sẽ khai thác nó theo chiều ngược từ chỗ in ra flag rồi đi ngược lên để tìm cách giải.
  ```
  if (isset($_GET['dashboard'])) {
    $current_username = $_SESSION['user'] ?? null;

    if ($current_username) {
        echo "<h1>Welcome to the Dashboard, $current_username!</h1>";
        if ($current_username === 'admin') {
            echo "<p>Flag: $flag</p>";
        }
  ```
- Trong đoạn code này, để in ra flag, biến current_username phải có giá trị là admin. Ở đoạn code trên, biến current_username sẽ có giá trị là biến SESSION['user'] nếu tồn tại biến GET['dashboard']. Lúc nãy, ta đã có một request GET['dashboard'] nên có vẻ ta sẽ cần biến đổi một chút gì đó. Tiếp tục đọc lên trên thì ta thấy được đoạn code này
  ```
      } elseif ($users[$username] == $password) {
        $_SESSION['user'] = $username;
        echo json_encode(['success' => true, 'redirect' => '?dashboard']);
        exit;
  ```
- Trong đoạn code này, biến SESSION['user'] sẽ có giá trị là biến username nếu biến users[$username] == biến password. Và khi xem lại đoạn code ở đầu thì điều đó hoàn toàn đúng. Ở đây tôi để ý một điều đặc biệt là biến users[$username] dùng `==` để so sánh với biến password, vậy nên tôi sẽ sử dụng `Type Juggling` (ép kiểu dữ liệu) để lấy flag. 
- Vì tên đăng nhập là admin và mật khẩu chính là flag mà chúng ta cần lấy nên ta không thể nào đăng nhập bình thường được mà phải dùng `Type juggling`.
- Tôi thử đăng nhập với tên người dùng là admin và mật khẩu nhập vào là 0 và dùng burpsuite thì có được request và response sau
  <img width="1495" height="647" alt="image" src="https://github.com/user-attachments/assets/5958850c-23d6-4b5a-9432-7208c32baf0f" />

- Tôi không đăng nhập được vì tên đăng nhập đang là string và password là chuỗi số, đây là hai kiểu dữ liệu khác nhau và không nằm trong trường hợp xảy ra Type Juggling nên không thể đăng nhập được. Vì thế nên tôi sẽ phải chuyển password sang kiểu dữ liệu boolean để so sánh với tên đăng nhập là string để thực hiện Type Juggling.
  <img width="1491" height="797" alt="image" src="https://github.com/user-attachments/assets/8fc3cd23-e3f8-494c-99d7-5e0c2f8b97ec" />

- F5 lại trang web, ta nhận được flag

  <img width="665" height="247" alt="image" src="https://github.com/user-attachments/assets/0c631927-7f97-42bd-9cea-a495b45bc796" />

