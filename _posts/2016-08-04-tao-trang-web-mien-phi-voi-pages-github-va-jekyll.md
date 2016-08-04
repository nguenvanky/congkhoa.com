---
layout: post
title:  "Tạo trang blog cá nhân miễn phí với Github.io và Jekyll"
date: 2016-08-04 13:00:05 +0700
categories: [mariadb]
---

Chào anh em, hôm trước nổi hứng viết blog nên ngồi nghịch __pages.github.com__, tranh thủ nghỉ trưa mình đã tạo được trang blog này.
Thấy khá hay nên hôm nay mình sẽ hướng dẫn anh em tạo 1 trang blog cá nhân hoặc trang giới thiệu sản phẩm hay bạn làm trang CV đều được.
Mình chọn [https://pages.github.com/](https://pages.github.com/) vì nó thỏa mãn mình một số lý do sau:

* *Miễn phí* - tất nhiên rồi, càng rẻ càng tốt, dùng em này khỏi phải mua hosting, nếu anh em thích domain riêng thì tự mua. *(Ngoài lề chút: mình là thằng bán host nhé :D, anh em cần mua host cứ liên hệ với mình)*
* *Cho phép custom domain* - nhiều trang cho miễn phí nhưng trang của bạn phải dùng subdomain của họ, mình không thích.
* *Nhanh* - hơi xính ngoại chút nhưng cơ mà dùng hàng của các bạn này thì khỏi nghĩ.
* *Ổn định* - (như trên)
* *Quản lý bài viết như 1 Github project* - cái này ưng nhất này, mình lười nên muốn viết bài cũng ngại đăng nhập rồi chọn create post các kiểu, mặc dù có tài khoản của [http://how.vndemy.com/](http://how.vndemy.com/) để viết, cơ mà chẳng viết bao giờ :v. 
Với tính năng này, quy trình viết bài của mình rút gọn như sau: mở editer bất kỳ trên máy (vim, sublimetext...), tạo một file Markdown, save vào thư mục của project này, `commit`, `push`, done! 
Nói thì vẫn còn dài, nhưng dùng VScode chẳng hạn, thì chỉ trong 1 nốt nhạc là đã xong (ko tính thời gian viết nhé).
Sửa bài cũng tương tự, lại có git để xem sửa những chỗ nào. Nói chung là awnsome!

Tuy nhiên, có một số hạn chế mà các bạn phải cân nhắc, nếu sử dụng thằng này:

* Cần biết `Markdown` - `Markdown` là gì thì các bạn tự GG, bài viết ở đây hoàn toàn viết bằng `Markdown`. Bạn có thể viết bằng `html` nếu thích vì github hỗ trợ editor ngon mà =))
* Nên có một chút kỹ năng cơ bản về kỹ thuật - à, sao mình nói cái này, vì tốt nhất bạn phải biết `Git` để viết và public bài, ngoài ra sẽ cần biết `Ruby, Gem, Jekyll` (biết thôi) để setup và chạy trang trên máy cá nhân phục vụ việc xem trước bài viết.
Tuy nhiên bạn có thể không cần quan tâm đến `Git, Ruby, Gem, Jekyll` nếu bạn dùng editor của __Github__ cơ mà như thế lông rân lắm ^^,

Rồi dài dòng rồi, vào vấn đề:

**Step 1: Mua domain**

Bạn có thể bỏ qua bước này nếu không cần domain riêng, khi đó trang của bạn sẽ có domain `yourusername.github.io`.
Cơ mà như thế nó không có oai và chả có gì khác biệt ^^!. 

Bạn có thể mua domain từ các nhà cung cấp Goddady, Matbao ... hoặc muốn mua của mình thì liên hệ :D

**Step 2: Đăng ký tài khoản**

Bạn [vào đây](https://pages.github.com/ "Github Page's Homepage") đọc hướng dẫn và bỏ qua bài viết này của mình nếu muốn tự tìm hiểu ;)


Theo mình bạn chỉ cần làm hết `bước 1` ở đó là đủ.
Ví dụ mình tạo được project tên là [demoblog](https://github.com/congkhoa/demoblog)

**Step 3: Tạo Github Page cho project của bạn**

Như ví dụ trên thì sẽ vào  [demoblog](https://github.com/congkhoa/demoblog)

* Chọn mục `Setting`. 
* Trong `Setting` kéo xuống phần `GitHub Pages`.
* Chọn *Launch automatic page generator.*
* Click Click theo step trong đó.
Kết thúc bước này bạn sẽ có 1 trang blog cơ bản, ví dụ:
[https://congkhoa.github.io/demoblog/](https://congkhoa.github.io/demoblog/)
và một Branch mới trong project `demoblog` tên là `gh-pages`

**Step 4: Chọn theme**

Bạn [vào đây](http://jekyllthemes.org/ ) để chọn theme muốn dùng: 

Thực chất mỗi theme đều là 1 trang __Github Pages__ cũng là 1 project trên Github nên đều có icon hiển thị số __Fork__, __Star__ của project trên Github. 
Bạn click vào icon đó nó sẽ dẫn bạn đến trang Github của project đó. 
Ví dụ Theme này: [https://github.com/mattvh/jekyllthemes](https://github.com/mattvh/jekyllthemes)

Click vào `Clone or Download` chọn `DownloadZIP`

**Step 5: Clone project về máy tính**

Với project bạn vừa tạo được ở `bước 2` (của mình là project `demoblog`) bạn clone về máy tính.
Ví dụ: `git clone git@github.com:congkhoa/demoblog.git`

**Step 6: Sử dụng theme**

* Check out sang nhánh `gh-pages` trong project của bạn. Xóa sạch các file trong đó.
* Giải nén file theme vừa download ở bước 4, copy và ghi đè toàn bộ vào thư mục gốc của project
* Commit và push lên github.
 Sau bước này bạn đã được trang github page với theme ưa thích và project của bạn đã được dựng sẵn với layout của 1 Jekyll project

 **Step7: Chạy Jekyll để xem thử trang và voọc trên máy local**

* Ở đoạn này bạn cần phải cài đặt Ruby, gem, bundle những cái này bạn tra GG giúp mình nhé, viết ra dài dòng lắm.
* `Jekyll` là gì thì mời các bạn GG nhé :D

Bạn để ý nó có 1 file tên là `Gemfile` ở thư mục gốc của project file đó để bạn `buid` và chạy `Jekyll` đấy.

* Bạn chỉ cần gõ:
`bundle install` trong thư mục gốc project - thư mục chứa `Gemfile` _(mình không dùng Window nên ko biết chạy Window thế nào đâu nhé)_

Chạy lệnh này có thể xuất hiện lỗi tùy thuộc vào các thư viện Ruby của bạn đã được cài hay chưa. Cố gắng fix hết nếu gặp nhé.

* Sau khi chạy thành công, bạn chạy tiếp lệnh sau để start server local:

`bundle exec jekyll serve`

OK rồi, giờ thì vào [http://127.0.0.1:4000/](http://127.0.0.1:4000/) để xem kết quả.

**Step 8: Add domain**

* Trong phần `Setting` của project trên github, bạn nhập `domain` của bạn vào phần `Custom domain` và `Save` lại.
Khi này nó sẽ tạo 1 file tên là CNAME trong project. Nhớ `pull` về để update project nhé.

* Trỏ DNS theo như hướng dẫn sau khi bạn add domain.

**Step 9: Viết bài**

Viết bài bằng cách tạo file mới ở thư mục `_posts` nhớ để ý cách đặt tên file `[YYYY-MM-DD-ten-file]` nhé, nó là `url` của bài viết và `Jekyll` cũng lấy ngày đó đặt tên cho thư mục chứa html sau khi render đấy.
`commit` và `push` lên __Github__ sẽ tự động public bài của bạn.
Bây giờ bạn có thể voọc thoải mái theme và chỉnh sửa blog tùy thích, nhớ để `Themes by` ở footer nhé ;)

Tạm thời mới làm đến đây và cũng được trang như bạn đang xem, khi nào mình voọc thêm được phần gì sẽ viết tiếp nhé, bye!

_PS: bài viết đầu tiên, có gì sai sót xin được góp ý [tại đây](https://github.com/congkhoa/congkhoa.com/issues)^^_
