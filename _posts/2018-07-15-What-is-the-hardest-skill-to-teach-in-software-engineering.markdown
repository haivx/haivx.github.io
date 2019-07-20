---
title: What is the hardest skill to teach in software engineering
date: 2018-07-15 10:46:12
tags:
  - skills
---

Bài viết lược dịch từ [quora.com](https://www.quora.com/What-is-the-hardest-skill-to-teach-in-software-engineering/answer/Brian-Knapp-1)

### <strong>Câu hỏi: Kỹ năng khó dạy nhất trong kỹ thật phần mềm?</strong>

Trả lời bởi Brian Knapp, Christian, Kỹ sư Phần mềm. I blog at Code Career Genius
<!-- more -->
Có một kỹ năng đặc biệt khó dạy và áp dụng trong tất cả tình huống phát triền phần mềm …

<!-- more -->
Tôi phải kể một câu chuyện vui để nói rõ điều này.

Nhiều năm trước, Google đã tung ra tính năng mới lạ này. Đó là danh sách thả xuống tự động hoàn tất khi bạn nhập liệu. Nó thật sự ấn tượng.

<b>Về mặt kỹ thuật, họ phải giải quyết một vài vấn đề lớn để cho tính năng hoạt động.</b>

Đầu tiên, họ phải có trải nghiệm giao diện người dùng (UI) và mã javascript liên quan để gợi ý tự động hoàn tất trong thời gian thực (hoặc tôi đoán là gần như vậy).

Thứ hai, họ cần danh sách các gợi ý tự động điền hợp lý dựa trên nội dung người dùng đang nhập. Vì người dùng có thể nhập bất kỳ thứ gì trên Google nên có RẤT NHIỀU biến thể.

Thứ ba, điều này có khả năng sẽ tạo ra một lượng lớn tải trên hệ thống. Theo tôi nhớ thì bài đăng thông báo tính năng này trên blog ước tính họ phải mở rộng cơ sở hạ tầng thêm 6 lần để hỗ trợ các HTTP request, truy vấn cơ sở dữ liệu, v.v…

Tại Google với quy mô hàng trăm triệu người dùng, thực hiện tất cả những điều đó và khiến nó “hoạt động được” thực đáng kinh ngạc.

Ngoài ra, cho những ai nghĩ rằng có thể thực hiện điều này trong một ngày cuối tuần với React và Elastic Search, là hồi đó chưa có mấy thứ này.

Dù sao đi nữa, như tôi đã nói, đây là một tính năng rất hay. Lúc đó tôi đang làm việc với một trang web gây quỹ phi lợi nhuận rất lớn, với hàng triệu đô la tiền đã đuợc quyên góp.

Có người thấy những gì Google đã làm và nghĩ rằng chúng tôi cần triển khai tính năng đó trên trang web của mình. Nó hữu ích cho việc tìm kiếm các sự kiện và cá nhân nhanh hơn một chút, nên có vẻ nó là một ý tưởng hợp lý vào lúc đó.

Chúng tôi có một chuyên gia JavaScript trong nhóm, vì vậy phần front-end chỉ có một ít vấn đề. Phần back-end, chúng tôi dùng MySQL, chúng tôi cùng xử lý phần bảng gợi ý. Hơi lộn xộn chút xíu, nhưng nó hoạt động khá tốt khi thử nghiệm.

Vì vậy, sau tầm một tháng phát triển, chúng tôi đã giới thiệu tính năng này. Có vẻ không có vấn đề gì to tát cho đến khi sau đó vài giờ…

<b>Trang web đã sập.</b>

Giống như là chúng tôi bị DDOS và mất khoảng một giờ để khôi phục mọi thứ và trực tuyến trở lại.

Hóa ra JS đã gửi một AJAX request với mỗi phím nhấn. Vì vậy, nếu người dùng gõ nhanh “John Doe” vào thanh tìm kiếm, nó sẽ kích hoạt 8 HTTP request. Tệ hơn nữa, 7 request đầu tiên ngay lập tức vô hiệu.

Lý do tại sao Google phải tăng cơ sở hạ tầng của họ để phù hợp với tính năng này trở nên rõ ràng. Trưởng nhóm đã không tính đến điều đó tí nào.

Giải pháp khá đơn giản, đặt thời gian chờ ít nhất nửa giây hoặc 1 giây sau lần nhấn phím cuối cùng trước khi hiển thị gợi ý tự động. Ngoài ra, sau đó chúng tôi đã hoàn thành bằng cách sử dụng ElasticSearch cho back-end tìm kiếm khi nó phát hành.

<b>Câu chuyện này có 2 điểm chính.</b>

Một, trang web sập trong một giờ ngày hôm đó có thể tổn thất từ $ 10,000 đến $ 100,000 tiền quyên góp. Đó xấp xỉ mức lương cả năm của một nhân viên toàn thời gian (FTE) (tùy thuộc vào vị trí và số tiền tổn thất thực tế).

Hai, toàn bộ tính năng không thực sự cần thiết. Đó là một thứ “có thì tốt” mà không đảm bảo làm ứng dụng tốt hơn. Tuy nhiên, điều này đi kèm với rủi ro thiệt hại lớn đã chơi chúng tôi rất đau.

Tất cả những điều này đã có thể tránh được nếu nhóm nghiên cứu tiếp cận kỹ thuật phần mềm từ phương pháp tối giản.

Code nhanh nhất là code không chạy.

Chạy ít truy vấn SQL tốt hơn là chạy nhiều truy vấn.

Ít tính năng hơn có nghĩa là ít phức tạp hơn.

Ít thư viện hơn có nghĩa là ít phức tạp hơn.

Ít công cụ hơn có nghĩa là ít phức tạp hơn.

Phức tạp tốn thời gian.

Phức tạp tốn tiền bạc.

“Không làm gì” tốt hơn là làm điều gì đó vô ích.

Một kỹ sư phần mềm già dơ sẽ làm mọi thứ trong khả năng của họ để tránh phức tạp và viết code nếu có thể.

Tối giản trong kỹ thuật phần mềm gần như không thể dạy được.

Thật khó để xây dựng một đội xung quanh. Hầu hết mọi người thà tập trung vào thư viện, gems, công nghệ mới, v.v. trong việc theo đuổi sự lười biếng.

Món nợ kỹ thuật là tiêu chuẩn mới và mọi người vui vẻ khi không bao giờ trả hết.

Vì vậy, trên thực tế, tối giản trong kĩ thuật phần mềm, đó là cách tiếp cận của việc phát triển phần mềm với ít công cụ nhất. Nó không phải là kĩ năng phổ biến và dễ dạy.

Các lập trình viên thích xếp chồng lớp các “thứ”. Thực tế là không thể vượt qua điều đó trong một nhóm lớn hơn 1 người.

Author: Trần Vũ

Copier: HaiVX :P
