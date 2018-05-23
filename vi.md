Giới thiệu
============

Trong thời hiện đại ngày nay, phần mềm thường được phân phối dưới dạng dịch vụ: được gọi là ứng dụng web hoặc phần mềm dưới dạng dịch vụ. Ứng dụng mười hai yếu tố là phương pháp để xây dựng các ứng dụng phần mềm như một dịch vụ:

*	Sử dụng các định dạng **khai báo** cho việc tự động hóa thiết lập, để giảm thiểu thời gian và chi phí cho các nhà phát triển mới tham gia dự án;
*	Có một **ràng buộc rõ ràng** với nền tảng hệ điều hành , cung cấp khả năng **linh động tối đa** giữa các môi trường thực thi;
*   Thích hợp cho việc **triển khai** trên **nền tảng cloud** hiện đại , giảm thiểu sự cần thiết của các máy chủ và những người quản trị hệ thống;
*	**Giảm thiểu sự khác biệt** giữa phát triển và sản xuất, cho phép **triển khai liên tục** nhanh  tối đa;
*	Và có thể **mở rộng quy mô** mà không cần thay đổi nhiều về công cụ, kiến ​​trúc hoặc thực tiễn phát triển.

Phương pháp 12 yếu tố có thể được áp dụng cho các ứng dụng được viết bằng bất kỳ ngôn ngữ lập trình nào và kết hợp sử dụng bất cứ dịch vụ sao lưu nào (cơ sở dữ liệu, hàng đợi, bộ nhớ cache, ...).

Background
==========

Những người đóng góp cho tài liệu này đã trực tiếp tham gia vào việc phát triển và triển khai hàng trăm ứng dụng, và gián tiếp giám sát ​​sự phát triển, vận hành và mở rộng hàng trăm nghìn ứng dụng thông qua công việc của chúng tôi trên nền tảng [Heroku](http://www.heroku.com/) .

Tài liệu này tổng hợp tất cả các kinh nghiệm và quan sát của chúng tôi về một loạt các phần mềm dưới dạng dịch vụ trong tự nhiên. Đó là một sự phân tích và đối chiếu về thực tiễn lý tưởng cho việc phát triển ứng dụng, đặc biệt chú ý đến sự linh động của  việc phát triển tự nhiên của ứng dụng theo thời gian, tính linh động của sự cộng tác giữa các nhà phát triển làm việc trên codebase của ứng dụng, và [việc tránh các chi phí phần mềm phát sinh](http://blog.heroku.com/archives/2011/6/28/the_new_heroku_4_erosion_resistance_explicit_contracts/).

Động lực của chúng tôi chính là nâng cao nhận thức về một số vấn đề hệ thống mà chúng tôi đã gặp trong việc phát triển ứng dụng hiện đại, để cung cấp vốn từ vựng chung để thảo luận những vấn đề đó và cung cấp một loạt giải pháp cho những vấn đề kèm theo các thuật ngữ. Định dạng này được lấy cảm hứng từ 2 cuốn sách của Martin Fowler là: _[Patterns of Enterprise Application Architecture](https://books.google.com/books/about/Patterns_of_enterprise_application_archi.html?id=FyWZt5DdvFkC)_ và _[Refactoring](https://books.google.com/books/about/Refactoring.html?id=1MsETFPD3I0C)_.

Ai nên đọc tài liệu này?
==============================

Bất kỳ nhà phát triển nào xây dựng phần mềm dưới dạng dịch vụ. Các kỹ sư triển khai hoặc quản lý các ứng dụng như vậy.

I. Codebase
-----------

### Một codebase được theo dõi trong việc kiểm soát thay đổi, nhiều triển khai

Một ứng dụng mười hai yếu tố luôn được theo dõi trong một hệ thống quản lý phiên bản, như là Git , Mercurial hoặc Subversion . Một bản sao của cơ sở dữ liệu theo dõi sửa đổi được gọi là _code repository_ , thường được rút ngắn thành _code repo_ hoặc chỉ _repo_ .

Một _codebase_ là bất kỳ repo đơn (trong một hệ thống điều khiển sửa đổi tập trung như Subversion), hoặc bất kỳ bộ repos nào chia sẻ root commit (trong một hệ thống kiểm soát sửa đổi phân cấp như Git).

![One codebase maps to many deploys](https://12factor.net/images/codebase-deploys.png)

Luôn có mối tương quan một-một giữa codebase và ứng dụng:

* Nếu có nhiều codebase, nó không phải là một ứng dụng - đó là một hệ thống phân tán. Mỗi thành phần trong một hệ thống phân tán là một ứng dụng và mỗi thành phần có thể tuân thủ riêng với mười hai yếu tố.
* Nhiều ứng dụng chia sẻ cùng một mã là vi phạm mười hai yếu tố. Giải pháp ở đây là chia sẻ code thành các thư viện có thể được bao gồm bởi các trình quản lý phụ thuộc .

Chỉ có một codebase cho một ứng dụng, nhưng sẽ có nhiều sự triển khai ứng dụng. Một lần triển khai là một phiên bản đang chạy của ứng dụng. Đây thường là một trang web sản phẩm và một hoặc nhiều trang web dàn dựng. Ngoài ra, mọi nhà phát triển đều có một bản sao của ứng dụng đang chạy trong môi trường phát triển local của họ, mỗi một trong số đó cũng đủ điều kiện để triển khai.

Các codebase là giống nhau trên tất cả các triển khai, mặc dù các phiên bản khác nhau có thể hoạt động trong mỗi triển khai. Ví dụ, một nhà phát triển có một số commit chưa triển khai để stage; stage có một số cam kết chưa được triển khai thành sản phẩm. Nhưng tất cả đều cùng một codebase, do đó làm cho chúng có thể nhận dạng như các triển khai khác nhau của cùng một ứng dụng.

II. Những dependency
----------------

### Khai báo rõ ràng và cô lập dependency

Hầu hết các ngôn ngữ lập trình đều cung cấp một hệ thống package để phân phối các thư viện hỗ trợ, chẳng hạn như CPAN cho Perl hoặc Rubygems cho Ruby. Các thư viện được cài đặt thông qua hệ thống package có thể được cài đặt trên toàn hệ thống (được gọi là “các package web”) hoặc được đưa vào thư mục chứa ứng dụng (được gọi là ““vendoring”” hoặc “bundling”).


**Một ứng dụng mười hai yếu tố không bao giờ dựa vào sự tồn tại ngầm của các package hệ thống.** Nó khai bá tất cả các dependency, hoàn toàn và chính xác, thông qua một **khai báo dependency** . Hơn nữa, nó sử dụng một công cụ cô lập dependency trong quá trình thực thi để đảm bảo rằng không có dependency ngầm nào bị "rò rỉ" từ hệ thống xung quanh. Đặc tả dependency đầy đủ và rõ ràng được áp dụng thống nhất cho cả sản xuất và phát triển.

Ví dụ: Bundler của Ruby cung cấp Gemfile định dạng manifest cho khai báo dependency và `bundle exec` cách ly dependency. Trong Python có hai công cụ riêng biệt cho các bước này - Pip được sử dụng để khai báo và Virtualenv để cô lập. Ngay cả C có Autoconf để khai báo dependency, và liên kết tĩnh có thể cung cấp sự cô lập dependency. Chuỗi các công cụ, khai báo phụ thuộc và cách ly phải luôn luôn được sử dụng cùng nhau - chỉ có một là không đủ để đáp ứng mười hai yếu tố.

Một lợi ích của khai báo phụ thuộc rõ ràng là nó đơn giản hóa việc thiết lập cho các nhà phát triển mới cho ứng dụng. Nhà phát triển mới có thể kiểm tra codebase của ứng dụng trên máy phát triển của họ, chỉ yêu cầu trình quản lý dependency và thời gian chạy ngôn ngữ được cài đặt làm điều kiện tiên quyết. Họ sẽ có thể thiết lập mọi thứ cần thiết để chạy mã của ứng dụng bằng `lệnh build` xác định . Ví dụ, lệnh xây dựng cho Ruby / Bundler là `bundle install`, trong khi cho Clojure / Leiningen là `lein deps`.

Ứng dụng 12 yếu tố cũng không phụ thuộc vào sự tồn tại tiềm ẩn của bất kỳ công cụ hệ thống nào. Các ví dụ bao gồm việc kích hoạt ImageMagick hoặc curl. Mặc dù các công cụ này có thể tồn tại trên nhiều hoặc thậm chí hầu hết các hệ thống, không có gì đảm bảo rằng chúng sẽ tồn tại trên tất cả các hệ thống mà ứng dụng chạy trong tương lai hoặc phiên bản trên hệ thống tương lai sẽ tương thích với ứng dụng. Nếu ứng dụng cần trình tích hợp một công cụ hệ thống, công cụ đó sẽ được đưa vào ứng dụng.


III. Cấu hình
-----------

### Lưu trữ cấu hình trong môi trường

Cấu hình của ứng dụng là mọi thứ có khả năng thay đổi giữa các triển khai (dàn dựng, sản xuất, môi trường nhà phát triển, ...). Điều nay bao gôm:

*   Tài nguyên xử lý cơ sở dữ liệu, Memcached và các dịch vụ sao lưu khác
*   Thông tin đăng nhập cho các dịch vụ bên ngoài như Amazon S3 hoặc Twitter
*   Các giá trị triển khai ví dụ như tên máy chủ chuẩn cho việc triển khai

Đôi khi, các ứng dụng lưu trữ cấu hình dưới dạng hằng số trong mã. Đây là một vi phạm của mười hai yếu tố, đòi hỏi phải **tách biệt cấu hình từ mã** . Cấu hình khác nhau đáng kể trên các triển khai, mã thì không.

Một bài kiểm tra litmus xem một ứng dụng có tất cả các cấu hình chính xác với code là liệu codebase có thể được trở thành mã nguồn mở tại bất kỳ thời điểm nào, mà không ảnh hưởng đến bất kỳ thông tin đăng nhập nào.

Lưu ý rằng định nghĩa "cấu hình" này **không** bao gồm cấu hình ứng dụng nội bộ, chẳng hạn như ``config/routes.rb`` trong Rails, hoặc cách các module được kết nối trong Spring . Loại cấu hình này không thay đổi giữa triển khai và do đó được thực hiện tốt nhất trong mã.

Một cách tiếp cận khác để cấu hình là sử dụng các tệp cấu hình không được kiểm tra trong điều khiển sửa đổi, chẳng hạn như `` config/database.yml`` trong Rails. Đây là một cải tiến lớn so với việc sử dụng các hằng số được kiểm tra trong repo code, nhưng vẫn có những điểm yếu: rất dễ nhầm lẫn khi kiểm tra tệp cấu hình repo; có một xu hướng là các tập tin cấu hình được phân tán ở những nơi khác nhau và các định dạng khác nhau, làm cho nó khó khăn để xem và quản lý tất cả các cấu hình ở một nơi. Hơn nữa, các định dạng này có xu hướng đặc biệt về ngôn ngữ hoặc framework.

**Ứng dụng mười hai yếu tố lưu trữ cấu hình trong các biến môi trường** (thường được rút gọn thành _env vars_ hoặc _env_ ). Env vars dễ thay đổi giữa các triển khai mà không thay đổi bất kỳ code nào; không giống như các tập tin cấu hình, có rất ít khả năng chúng được đưa vào mã repo một cách vô tình; và không giống như các tệp cấu hình tùy chỉnh, hoặc các cơ chế cấu hình khác như hệ thống Java, chúng là một tiêu chuẩn về ngôn ngữ và hệ điều hành.

Một khía cạnh khác của quản lý cấu hình là nhóm. Đôi khi các ứng dụng cấu hình hàng loạt thành các nhóm được đặt tên (thường được gọi là “các môi trường”) có tên sau khi triển khai cụ thể, chẳng hạn như môi trường ``development``, ``test`` và ``production``  trong Rails. Phương pháp này không quy định rõ ràng: khi nhiều triển khai ứng dụng được tạo, tên môi trường mới là cần thiết, chẳng hạn như ``staging`` hoặc ``qa``. Khi dự án phát triển hơn nữa, các nhà phát triển có thể thêm các môi trường đặc biệt của riêng mình như ``joes-staging``, dẫn đến một vụ lẫn lộn tổ hợp của cấu hình và làm cho quản lý triển khai của ứng dụng rất mỏng manh.

Trong một ứng dụng mười hai yếu tố, env vars là những điều khiển chi tiết, mỗi chúng đều độc lập với các env vars khác. Chúng không bao giờ được nhóm lại với nhau thành “môi trường”, mà thay vào đó được quản lý độc lập cho từng triển khai. Đây là một mô hình có quy trình thuận lợi khi ứng dụng mở rộng với nhiều triển khai hơn trong suốt lifetime của nó.

