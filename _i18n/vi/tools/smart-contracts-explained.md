### Những gì đang diễn ra ở đó?
Một ngày, ai đó đã hỏi tôi về tất cả những chuyện về eosDAC, những gì rất riêng biệt về tất cả công việc mà chúng tôi đang làm để xây dựng việc kỳ lạ này mà chúng tôi đang gọi là một bộ công cụ DAC. Làm thế nào mà nó khác biệt với các giải pháp tập trung cho quản trị và hoạt động của tổ chức? An toàn như thế nào? Làm thế nào mà chúng tôi thiết kế và xây dựng một bộ các hợp đồng thông minh để đảm bảo rằng không một cá thể nào có thể gây hại cho DAC thông qua các hợp đồng thông minh? 

Đã có một số bài báo và bài đăng trên blog thảo luận về niềm tin cốt lõi và cách thức vận hành một DAC bằng cách sử dụng bộ mã trong bài báo này, tôi sẽ tập trung vào những gì mà các hợp đồng thông minh đang làm và cách chúng làm việc cùng nhau để đạt được sự quản trị tự động an toàn. Tất cả bộ mã có mã nguồn mở và có sẵn để xem/sử dụng/phân tách trên Github nhưng rồi, không phải ai cũng hiểu C++.

Mục tiêu chính trong tất cả các hợp đồng thông minh của chúng tôi là việc tái sử dụng cho các DAC khác để có thể sao chép và sử dụng mã nguồn của chúng tôi. Đây là một trong những niềm tin cốt lõi của chúng tôi ngay từ đầu và là lý giải vì sao một số quyết định thiết kế lại phức tạp hơn so với các trường hợp khác nếu nó chỉ dành cho một trường hợp sử dụng. Mục tiêu tương lai của chúng tôi là làm cho những thứ này thậm chí được tái sử dụng để nhiều DAC có thể sử dụng các phiên bản hợp đồng thông minh được cài đặt và có thể điều chỉnh các cài đặt cho mục đích riêng của họ mà không cần phải duy trì bộ mã riêng của họ. Bằng cách này, họ sẽ nhận được các bản cập nhật cùng với đội ngũ eosDAC cho các bản cập nhật chức năng và sửa lỗi chung. 

Các hợp đồng mã nguồn bao gồm một hợp đồng token (`eosdactokens`), một hợp đồng quản lý bầu cử và giám hộ (`dacustodian`), một hợp đồng đề xuất công việc để quản lý tất cả các đề xuất công việc và phối hợp thanh toán và một tài khoản ký quỹ riêng (`dacescrow`) để giữ nguồn quỹ một cách an toàn cho các đề xuất công việc đang diễn ra để bảo vệ cả DAC lẫn nhân viên khỏi khả năng mất các khoản thanh toán đã hứa cho công việc của họ.

## EOSDACTOKENS

Đây là nơi tất cả bắt đầu. Token EOSDAC được giữ trong hợp đồng thông minh này. Nó bắt đầu như một bản sao từ bộ mã hợp đồng eosio.token chính được sử dụng cho token EOS và rất có thể là điểm khởi đầu cho tất cả các token đang chạy trên các tất cả các chuỗi EOS. Chúng tôi đã thêm vào một vài chức năng cho hợp đồng này đẻ phù hợp với nhu cầu của mình cho việc ra mắt DAC và airdrop đầu tiên bao gồm những điều sau:

### Khả năng tạo token trong trạng thái khóa.
Mục đích của việc này là để khi chúng tôi thực hiện airdrop đầu tiên đến những người sở hữu token gốc trên Ethereum ban đầu, vào tháng 6 năm 2018, chúng tôi có thể kiểm tra kỹ lưỡng số dư của các tài khoản nhận để đảm bảo rằng chúng khớp với số dư dự kiến từ điểm chụp trên Ethereum trước khi người dùng có thể bắt đầu giao dịch token. Chúng tôi kiểm tra rất nghiêm túc :) .

EOSDAC là một trong những token đầu tiên thực hiện airdrop trên chuỗi nên có nhiều điều còn chưa biết. Một khi đợt airdrop hoàn thành và được kiểm tra, chúng tôi có thể mở khóa token để cho phép giao dịch. Quan trọng là không có khả năng khóa token lại một khi nó được mở khóa bởi vì chức năng này có thể cho phép những người tạo ra token có thể thao túng giá thông qua sự kiểm soát tập trung việc thanh khoản token.

### Điều khoản thành viên và sự chấp thuận.
Việc đồng ý với các điều khoản sử dụng như một phần của DAC được chúng tôi xem là một chức năng cốt lõi của các hợp đồng thông minh của DAC và do đó, việc đưa các chức năng vào hợp đồng là điều hợp lý. Các điều khoản sẽ được người dùng chấp thuận để họ trở thành thành viên thông qua việc thực hiện lệnh `memberreg` trên hợp đồng. 

Để thực hiện lệnh này, người dùng cần kiểm tra toàn bộ các điều khoản mà họ đã thực sự đồng ý với nó và việc thực hiện hợp đồng sẽ đảm bảo việc tổng kiểm tra đó khớp với việc tổng kiểm tra các điều khoản mới nhất trước khi chấp nhận thỏa thuận và tư cách thành viên của họ. Logic của việc tổng kiểm tra này được trừu tượng hóa khỏi người dùng cuối cùng thông qua giao diện người dùng được xử lý trước nhưng cung cấp bằng chứng mã hóa mà người dùng đồng ý với một bộ điều khoản cụ thể và sẵn sàng cho những bộ khác thêm vào giao diện người dùng của họ để tương tác với bộ mã hợp đồng. Miễn là họ có thể cung cấp việc tổng kiểm tra tương tự cấc điều khoản và quy định đã biết trước thì họ sẽ có thể thực hiện thành công lệnh `memberreg`. Mặc dù điều này trông có vẻ dài dòng để đồng ý các quy định và điều khoản, chúng tôi đã thực hiện phương pháp này bởi vì nó giống như nói "Tôi đồng ý với các điều khoản và quy định *này*" hơn là hỏi người dùng "Bạn có đồng ý với các điều khoản và quy định mới nhất không" với việc người dùng phản hồi "có" hoặc "không". Chúng tôi đã nghĩ cách sau sẽ yếu hơn để đảm bảo người dùng không vô tình chấp thuận các điều khoản sai hoặc thông qua một giao diện người dùng đã hết hạn hợp đồng. 

Để tiết kiệm việc sử dụng RAM cho tất cả các người dùng cuối, hợp đồng chỉ lưu trữ một tham chiếu đến mã hash của các điều khoản được chấp thuận một lần trong hợp đồng thay vì lưu trữ một bản sao mã hash được chấp thuận của mỗi người dùng. Điều này có nghĩa là mỗi người dùng chỉ cần lưu trữ phiên bản của các điều khoản được chấp thuận (8 bytes) chứ không phải toàn bộ mã hash (32 bytes) và vì RAM trên chuỗi khá tốn kém nên mỗi lần tiết kiệm đều rất có ích. 

Trạng thái chấp thuận các điều khoản mới nhất của mỗi thành viên được đối chiếu nhiều lần trong các hợp đồng thông minh của EOSDAC để đảm bảo người dùng không thể thực hiện các lệnh trên DAC nếu không đồng ý với các điều khoản mới nhất, kể cả trong tương lai khi cấc điều khoản được cập nhật.

## DACCUSTODIAN

Hợp đồng này quản lý việc đặt cọc, đề cử, bỏ phiếu, kiểm phiếu và bổ nhiệm người giám hộ vào cuối mỗi cuộc bầu cử. Kết quả cuối cùng từ các lệnh của hợp đồng này là để quản lý các quyền hạn trên tài khoản quản lý của DAC (`dacauthority`) dưới dạng cấu hình thông qua các điều chỉnh trên hợp đồng này vì tài khoản quản lý có quyền được xác định trước để làm bất cứ gì trong DAC bao gồm thay đổi bộ mã và chuyển nguồn quỹ. Các hành động được phép sẽ là các giao dịch đa chữ ký khác nhau được điều khiển tận dụng các công cụ quản lý cấp phép phức tạp được tích hợp sẵn trong phần mềm giao thức EOSIO. Các tham số cho các hoạt động bầu cử này có thể được thay đổi thông qua một đối tượng cấu hình được thiết lập trong hợp đồng qua lệnh `updateconfig`. Chức năng của hợp đồng này có thể bị chia thành các thành phần mở rộng sau đây:

### Đặt cọc	
Trước khi có thể đề cử thành công là một ứng viên, một thành viên phải chuyển EOSDAC trước dưới dạng một khoản đặt cọc bằng cách sử dụng lệnh `transfer` trên hợp đồng token vào tài khoản này. Ghi chú phải đúng với tên tài khoản hợp đồng (ví dụ: "daccustodian") để việc giao dịch được xem là một giao dịch đặt cọc và số lượng đặt cọc bắt buộc có thể được cấu hình bởi trường (field) `lockupasset` trong đối tượng cấu hình. Sau khi đặt cọc thành công, sẽ có một lượng đặt cọc được chờ xử lý trong hợp đồng cho ứng viên này sẵn sàng cho lệnh đề cử thực tế.

### Đề cử ứng viên - `nominatecand`:
Để trở thành một giám hộ được bầu cho DAC, một thành viên hợp lệ phải đề cử làm ứng viên trước bằng cách sử dụng lệnh `nominatecand`. Việc này yêu cầu tên tài khoản của tài khoản đề cử và số tiền yêu cầu thanh toán bằng EOS. lệnh này kiểm tra người dùng là một thành viên hợp lệ và đã đặt cọc đủ số token EOSDAC. Nó cũng kiểm tra xem số tiền yêu cầu thanh toán không vượt quá `requested_pay_max` từ cấu hình và nếu các điều kiện này được đáp ứng thì sẽ thêm người dùng này vào với tư cách ứng viên có thể bỏ phiếu cho các lệnh bỏ phiếu tương lai.

### Bỏ phiếu - `votecust`:
Việc bỏ phiếu được thực hiện bằng lệnh `votecust` và yêu cầu tài khoản người bỏ phiếu và danh sách các ứng viên mà người bỏ phiếu muốn bầu. Các lá phiếu tuân theo các qui tắc sau: 

* Việc bỏ phiếu yêu cầu tư cách thành viên hợp lệ và đồng ý với các điều khoản thành viên hiện tại.
* Trọng lượng của mỗi lá phiếu được xác định bởi số dư EOSDAC mà người bỏ phiếu có trong tài khoản của họ. (Hiện tại không có yêu cầu đặt cọc để được bỏ phiếu)
* Một khi đã bỏ phiếu, phiếu bầu đó vẫn có hiệu lực cho đến khi người bỏ phiếu thay đổi phiếu bầu. Điều này có nghĩa rằng nếu số dư tài khoản giảm xuống còn 0 và sau đó số dư được thêm vào hoặc một trong số các ứng viên từ chức và sau đó phiếu bầu được trả lại sẽ có hiệu lực và được áp dụng trong việc kiểm phiếu.
* Tất cả các ứng viên được bỏ phiếu bởi một người bỏ phiếu nhận được cùng số phiếu bầu dựa trên số dư EOSDAC.
* Có số lượng ứng viên tối đa mà một người bỏ phiếu có thể bầu chọn, được xác định bởi cấu hình cài đặt.
* Để xóa phiếu bầu, người bỏ phiếu cần bỏ phiếu với một danh sách ứng viên trống.
* Việc bỏ phiếu và chuyển token vào và từ một tài khoản bỏ phiếu sẽ có tác động trực tiếp và ngay lập tức vào sức nặng của mỗi phiếu bầu có hiệu lực nhưng thời điểm duy nhất mà trọng lượng phiếu bầu có ảnh hưởng nằm ở khối mà lệnh `newperiod` được thực hiện (giải thích bên dưới).

### Chu kỳ bỏ phiếu mới - `newperiod`:
Đối với mỗi ứng viên hợp lệ, trọng lượng phiếu bầu sẽ được theo dõi liên tục dựa trên các thay đổi được kích hoạt từ lệnh `votecust` hoặc lệnh `transfer` trên hợp đồng token nếu có một phiếu bầu hoạt động cho các tài khoản `from` hoặc `to` trên lệnh chuyển. Với những giá trị này liên tục được theo dõi, cuộc bỏ phiếu thực tế (tại thời điểm diễn ra `newperiod`) chỉ cần lấy điểm chụp của các ứng viên được sắp xếp theo số phiếu nhiều nhất tại thời điểm đó. Số lượng ứng viên có thể được cấu hình thông qua trường (field) `numelected` trên cấu hình. Có những kiểm tra cần phải được đáp ứng để chạy thành công lệnh `newperiod` bao gồm:

* Đảm bảo số lượng người bỏ phiếu nhiều hơn cấu hình `initial_vote_quorum_percent`. 
* Đảm bảo trong các cuộc bỏ phiếu tiếp theo, số lượng người bỏ phiếu nhiều hơn cấu hình `vote_quorum_percent`.
* Đảm bảo thời gian giữa các chu kỳ đến `newperiod` nhiều hơn thời gian trì hoãn `periodlength`.

Nếu tất cả cá cuộc kiểm tra thành công, các sự kiện sau đây được thực thi:

* Tiền lương của người giám hộ được phân phối theo số tiền trung bình cho mỗi người từ số lương yêu cầu `requestedpay` của những người giám hộ hiện tại.
* Những người giám hộ mới được sắp xếp dựa trên thứ tự xếp hạng từ số phiếu tích lũy cho chu kỳ mới.
* Các quyền hạn cho phép vận hành các tài khoản DAC được cập nhật bao gồm những người giám hộ mới được bổ nhiệm.
* Thời gian hiện tại được lưu để sử dụng làm tham chiếu cho các chu kỳ tương lai `newperiod` để đảm bảo nó không được gọi quá sớm cho chu kỳ tiếp theo.
* Đối với mỗi ứng viên được bầu làm giám hộ, số token đặt cọc của họ sẽ bị khóa vào thời điểm này để cho thấy họ có một khoản đầu tư cá nhân. Họ sẽ có thể lấy token lại một khi họ không còn được bầu sau thời gian khóa được xác định bởi `lockup_release_time_delay` từ cấu hình.

### Các lệnh khác:
Ngoài con đường hạnh phúc được giải thích ở trên, còn có những lệnh khác được hỗ trợ trong hợp đồng này bao gồm:

* Rút ứng cử (`withdrawcand`): Lệnh này có thể được gọi bởi một ứng viên hiện hữu, bao gồm cả người hiện đang là người giám hộ được bầu chọn khi họ muốn rút khỏi các chu kỳ bỏ phiếu tiếp theo. Token của họ vẫn sẽ được đặt cọc và nếu số token này cũng bị khóa thì chúng vẫn sẽ bị khóa cho đến khi thời gian khóa hết hạn. Một trường hợp ứng dụng khá hay cho lệnh này có thể là một người giám hộ đang nghỉ lễ với ý định sớm quay lại (đó là lý do giữ cho chức năng hoàn cọc tách biệt). Mặt khác, lệnh này có thể được sử dụng cho bất kỳ lý do nào khác cho một ứng viên tự nguyện rút khỏi việc tham gia vào các hoạt động của DAC.
* Sa thải một ứng viên (`firecand`): Lệnh này sẽ được gọi bởi những người giám hộ được bầu thông qua tài khoản chính thức đa chữ ký để loại bỏ một ứng viên làm sai. Lệnh này có tùy chọn khóa số token đặt cọc của ứng viên nếu chúng vẫn chưa bị khóa.
* Sa thải một người giám hộ (`firecust`): Tương tự như lệnh `firecand`, lệnh này sẽ loại bỏ một người giám hộ hiện được bầu chọn như hành động của các người giám hộ khác. Lệnh này sẽ loại bỏ người giám hộ đó khỏi nhóm người giám hộ được bầu, loại bỏ họ như một ứng viên tiềm năng cho các cuộc bỏ phiếu tương lai, thay thế người giám hộ đó bằng ứng viên nhận được bầu chọn nhiều nhất tiếp theo và cuối cùng cập nhật các quyền tài khoản để đúng với nhóm người giám hộ được bầu mới.
* Từ chức giám hộ (`resigncust`): Lệnh này phải được thực hiện bởi một người giám hộ đang hoạt động, người muốn từ chức khỏi vị trí giám hộ hiện tại. Lệnh này sẽ loại bỏ họ khỏi vị trí ứng viên đủ điều kiện và thay thế người giám hộ này bằng ứng viên được bầu chọn nhiều nhất tiếp theo.
* Cập nhật tiền lương yêu cầu (`updatereqpay`): Một ứng viên có thể cập nhật tiền lương yêu cầu cho vị trí giám hộ. Lệnh này sẽ không có hiệu lực cho đến chu kỳ mới để ngăn chặn những thay đổi giữa chừng đối với tiền lương giám hộ.
* Yêu cầu thanh toán (`claimpay`): Đây là lệnh được gọi bởi một người giám hộ đang hoạt động để nhận được khoản tiền lương mà họ phải được nhận. Do hệ thống pháp lý cũ mà chúng ta đang sống trong đó, khoản thanh toán được thực hiện thông qua một công ty dịch vụ do các lý do pháp lý vượt quá quyền hạn của tài liệu này. Từ quan điểm của hợp đồng, đây là lệnh thanh toán cho một người giám hộ cho các dịch vụ mà họ làm cho DAC với tư cách là một người giám hộ và sau khi lệnh này được thực hiện, người giám hộ được xem là đã được thanh toán.
* Cập nhật cấu hình (`updateconfig`): Có một số tùy chọn điều chỉnh cấu hình trên bộ mã hợp đồng để cho phép nó được tùy chỉnh để sử dụng mà không cần phải biên dịch lại và triển khai bộ mã. Kế hoạch tương lai này là điều này cho phép các DAC khác hoạt động bằng cách sử dụng cùng một bộ mã được triển khai nhưng với cấu hình tùy chỉnh của riêng họ. Các thay đổi được thực hiện bằng cách cài đặt đối tượng cấu hình mới thông qua lệnh này với các tùy chọn sau:

	* `lockupasset`: Số lượng token EOSDAC bị khóa bởi mỗi ứng viên tham gia cuộc bỏ phiếu.
	* `maxvotes` : Số phiếu tối đa mà mỗi thành viên có thể bỏ phiếu cho một ứng viên. Mặc định là 5.
	* `numelected` : Số người giám hộ được bầu cho mỗi lần bỏ phiếu.
	* `periodlength` : Thời gian của một chu kỳ được tính bằng giây. Được sử dụng để ngăn chặn các cuộc bỏ phiếu sớm khỏi thực hiện trên lệnh `newperiod`. Mặc định là 7 ngày.
	* `authaccount` : Kiểm soát tài khoản để có quyền thiết lập với các người giám hộ dược bầu.
	* `tokenholder` : Hợp đồng nắm giữ các nguồn quỹ của DAC. Nó được dùng như nguồn để thanh toán của người giám hộ.
	* `serviceprovider` :  Hợp đồng này sẽ đóng vai trò là tài khoản nhà cung cấp dịch vụ cho DAC. Nó được dung như nhà cung cấp tiền lương cho các người giám hộ và nhân viên theo các đề xuất công việc.
	* `should_pay_via_service_provider` : Nếu thiết lập true, hợp đồng này sẽ chỉ đạo tất cả các khoản thanh toán thông qua nhà cung cấp dịch vụ hơn là thanh toán trực tiếp.
	* `initial_vote_quorum_percent` : Giá trị số lượng token trong bỏ phiếu được yêu cầu để kích hoạt đội ngũ giám hộ ban đầu.
	* `vote_quorum_percent` : Giá trị số lượng token trong bỏ phiếu được yêu cầu để cho phép một đội ngũ giám hộ mới được thành lập sau khi ngưỡng ban đầu đã đạt được - chu kỳ bỏ phiếu lần 2 trở đi.

		Số lượng người giám hộ cần thiết để phê duyệt các hành động xác thực ở các cấp độ khác nhau trên các hợp đồng thông minh của DAC:	

	* `auth_threshold_high`
	* `auth_threshold_mid`
	* `auth_threshold_low`
	* `lockup_release_time_delay` : Thời gian trước khi khóa cọc có thể được trả lại cho ứng viên sử dụng lệnh hoàn cọc.
	* `requested_pay_max` : Số tiền thanh toán tối đa mà một người giám hộ có thể yêu cầu thanh toán.

## DACPROPOSALS

Hợp đồng này chịu trách nhiệm quản lý các đề xuất công việc liên quan đến DAC. Một lần nữa nó được xây dựng cho khả năng thay đổi cấu hình thay vì chỉ để phù hợp với nhu cầu trước mắt trong eosDAC.

Ý tưởng chung là nhân viên tiềm năng sẽ có một phần việc mà họ muốn đề xuất để tăng thêm giá trị cho DAC để đổi lấy việc được thanh toán bằng một khoản token dựa tren EOS. Đề xuất sẽ được bỏ phiếu để phê duyệt ban đầu và sau đó hoàn thành bởi các người giám hộ hiện tại và các hành động này sẽ kích hoạt thanh toán cho người đề xuất. 


### Tạo đề xuất `createprop` 
Một nhân viên đề xuất sẽ tạo một đề xuất và gửi nó lên blockchain để xem xét và bỏ phiếu bởi những người giám hộ DAC hiện tại. Để đề xuất, sẽ cần bao gồm những điều sau:

* `title` (Sring - Chuỗi): xác định đề xuất.
* `summary` (String - Chuỗi): tóm tắt ngắn gọn về mục đích của đề xuất công việc.
* `arbitrator` (tên tài khoản EOS): tên tài khoản của trọng tài độc lập, người có thể được gọi để giải quyết các tranh chấp trong việc hoàn thành một đề xuất công việc.
* `pay_amount` (EOSAsset - Tài sản EOS): số lượng token dựa trên EOS được yêu cầu như tiền lương cho đề xuất công việc.
* `content_hash` (ChecksumHash - kiểm tra toàn diện mã hash): mã hash nội dung để đảm bảo chi tiết của đề xuất được lưu trữ ngoài chuỗi không được sửa đổi sau khi một đề xuất được chấp thuận. Điều này cho phép nhiều chi tiết mở rộng hơn sẽ không được lưu trữ trên chuỗi trong khi vẫn duy trì tính toàn vẹn của dữ liệu.

Đối với mỗi đề xuất, dữ liệu nội dung tối thiểu được yêu cầu được lưu trữ trong một trạng thái hợp đồng và thay vào đó chỉ được chuyển qua đẻ đảm bảo tính toàn vẹn của dữ liệu thông qua nhật ký giao dịch. Chỉ tài khoản và dữ liệu thanh toán được lưu trữ để sử dụng trong các hành động sau trong hợp đồng này.

### Bỏ phiếu cho một đề xuất `voteprop`
Khi một đề xuất được tạo ra, nó sẽ ở trạng thái chờ các người giám hộ hiện tại bỏ phiếu phê duyệt 'proposal_approve' hay từ chối 'proposal_deny' cho một đề xuất với số phiếu cần thiết và số lượng phiếu 'yes' (đồng ý) có thể được cấu hình trong hợp đồng. Tại thời điểm này, có thể có những sàn lọc đối với đề xuất với việc hủy bỏ `cancel` các đề xuất hiện hữu và gửi lại các thay đổi dựa trên phản hồi từ các người giám hộ cho đến khi các đề xuất đó sẵn sàng và có vị trí bỏ phiếu tích cực.

### Bắt đầu làm việc trên đề xuất được chấp thuận `startwork`
Khi có đủ lá phiếu tích cực cho một đề xuất, người đề xuất sẽ có thể gọi hành động này để xác nhận rằng họ sẽ đồng ý làm việc trên đề xuất đã thỏa thuận với các điều khoản đã thỏa thuận, tiền lương, v.v... Tại thời điểm này, tiền lương đã thỏa thuận được chuyển vào một tài khoản ký quỹ để đảm bảo nguồn quỹ có thể và sẽ được thanh toán cho người đề xuất khi công việc hoàn thành khi được phê duyệt bởi những người giám hộ hoặc trọng tài thỏa thuận cho đề xuất đó.

Việc khóa nguồn quỹ trong một tài khoản ký quỹ là một bước quan trọng để bảo vệ nhân viên khỏi những người giám hộ có ác ý tiềm tàng, những người có thể đảo ngược một sự chấp thuận đề xuất trước đó bởi vì họ có một trọng tài tin cậy trên đề xuất này để có thể giải phóng nguồn quỹ.

### Dấu hiệu hoàn thành công việc `completework`
Sau khi một nhân viên đã hoàn thành công việc của mình, họ sẽ báo cho những người giám hộ rằng công việc được cho là hoàn thành và các người giám hộ sẽ cần đánh giá công việc trước khi phê duyệt thông qua một cuộc bỏ phiếu khác bằng cách sử dụng `voteprop`, tuy nhiên lần này là một lựa chọn khác về giá trị lá phiếu để duyệt `claim_approve` hay từ chối `claim_deny` yêu cầu thanh toán.

### Yêu cầu thanh toán cho công việc đã hoàn thành `claim`
Nếu đã có đủ lá phiếu tích cực cho công việc đã hoàn thành từ các người giám hộ dựa trên các cấu hình hiện tại thì người đề xuất có thể gọi hành động sẽ kích hoạt chuyển khoản token thanh toán cho nhân viên. Trong thực tế cho eosDAC, nguồn quỹ được gửi đến một tài khoản công ty dịch vụ nhưng hiệu quả là chúng được trả trực tiếp đến nhân viên cho các dịch vụ của họ.

### Cấu hình hợp đồng `updateconfig`
Các tùy chọn khác nhau có thể cấu hình cho hợp đồng này bao gồm:

* `service_account` (Tên tài khoản EOS) 
* `proposal_threshold`: số lượng lá phiếu cần thiết để tham gia bỏ phiếu cho một đề xuất.
* `proposal_approval_threshold_percent`: Tỉ lệ phiếu bầu tích cực cần thiết để phê duyệt một đề xuất.
* `claim_threshold`: Số phiếu cần thiết để tham gia bỏ phiếu trong việc hoàn thành một đề xuất.
* `claim_approval_threshold_percent`: Tỉ lệ phiếu bầu tích cực cần thiết để phê duyệt một yêu cầu thanh toán đề xuất.
* `escrow_expiry`: Thời gian hết hạn được thiết lập trên giao dịch ký quỹ được tạo ra (số giây). Giá trị mặc định là 30 ngày.

## DACESCROW
Hợp đồng này có trách nhiệm giữ nguồn quỹ an toàn cho các đề xuất công việc cho đến khi những người giám hộ được bầu hoặc một trọng tài được thỏa thuận giải phóng nguồn tiền đến người nhận hoặc hết thời hạn ký quỹ thì nó sẽ cho phép trả lại tiền cho người gửi. Mục đích là hợp đồng này sẽ bị khóa để bảo vệ việc thay đổi mã nguồn từ những người giám hộ ác ý. Nó sẽ đạt được thông qua việc xóa bỏ các khóa owner và active trên tài khoản sau khi mã nguồn hợp đồng được thiết lập lại. Yêu cầu bất biến này là một lý do khác cho hợp đồng này để nó tách biệt và đơn giản với các hợp đồng mã nguồn khác trong bộ mã. Hy vọng là mã nguồn này không bao giờ cần sử đổi. Trong trường hợp thiếu may mắn (và hy vọng sẽ không xảy ra) thì mã nguồn này không cần phải sửa đổi, các người giám hộ sẽ cần phải có thỏa thuận bắt buộc từ các nhà sản xuất khối để thiết lập lại khóa owner cho tài khoản này.

### Khởi tạo một giao dịch ký quỹ  - `init`
Một giao dịch ký quỹ phải được tạo ra để chỉ rõ tất cả các trường (field) yêu cầu bao gồm người gửi, người nhận dự định, thời gian hết hạn, trọng tài, ghi chú cho hành động chuyển tiền cuối cùng. Ngoài ra còn có một khóa tùy chọn bên ngoài có thể được sử dụng như một tham chiếu hợp đồng chéo thay vì chỉ dựa vào khóa tăng tự động bên trong, nếu không điều này sẽ dẫn đến việc xung đột khóa trong một thời điểm.

### Chuyển tiền cho một giao dịch ký quỹ - `transfer`
Tiền dành cho một giao dịch ký quỹ sẽ cần được chuyển sang hợp đồng ký quỹ bằng cách sử dụng lệnh `transfer` (chuyển) thông thường như được thấy và được sao chép bởi hầu hết các hợp đồng token dựa trên EOS. Bộ mã của hợp đồng này được xây dựng trên các thông báo mà hành động chuyển tiền hướng đến tài khoản chuyển tiền của cả người gửi và người nhận. Khi nhận một thông báo chuyển khoản bởi hợp đồng ký quỹ, việc thực hiện lệnh `transfer` sẽ xác minh người gửi có một bản ghi ký gửi trống và chỉ định số tiền được chuyển vào bản ghi tài khoản đó để xử lý sau bằng cách lệnh khác trong bộ mã hợp đồng ký quỹ. Một bản ghi ký quỹ được khởi tạo có thể bị hủy bằng lệnh `cancel` được cung cấp, nó được gọi trước khi hành động chuyển được điền vào quỹ.

### Phê duyệt hay từ chối một giao dịch ký quỹ - `approve` và `unapprove`
Khi một giao dịch ký quỹ được khởi tạo và được điền vào với một hành động chuyển, bước tiếp theo sẽ là phê duyệt ký quỹ bởi người gửi hoặc trọng tài hay người nhận được chỉ định. Cần có 2 sự phê duyệt từ bất kỳ tài khoản nào trong 3 tài khoản này để cho phép hành động `claim` được thực hiện. Lệnh `unapprove` sau đó được gọi để loại bỏ một phê duyệt hiện có bởi tác nhân liên quan. 

### Yêu cầu thanh toán ký quỹ được phê duyệt - `claim`
Hành động khiếu nại chỉ có thể được thực hiện bởi người nhận dự định cho một khoản thanh toán ký quỹ và sẽ chỉ thành công với trạng thái phê duyệt chính xác cho bản ghi ký quỹ đó. Tại thời điểm này, khoản tiền ký quỹ sẽ được chuyển đến tài khoản công ty dịch vụ được chỉ định để các khoản thanh toán có thể được xử lý đến người nhận dự định.

_Lưu ý: Chi tiết trong công ty dịch vụ không được đề cập vì các lý do kỹ thuật nhưng là một yêu cầu pháp lý để có đủ can thiệp với thế giới pháp lý truyền thống. Chúng tôi muốn thực hiện tất cả các hành động nhiều nhất có thể trong sự an toàn của môi trường hợp đồng thông minh được bảo mật bằng mã hóa, nhưng thế giới vẫn chưa sẵn sàng :( ._

### Hoàn tiền sau khi hết hạn - `refund`
Mặc dù tiền trong tài khoản ký quỹ phải bị khóa trong một khoảng thời gian nhất định, chúng cũng phải có sẵn sau khi hết thời gian nếu không có đủ chấp thuận từ người gửi hay trọng tài, nếu không nguồn tiền có thể bị khóa trong tài khoản vĩnh viễn. Lệnh `refund` cung cấp cơ chế này và chỉ có thể được gọi bởi người gửi nếu thời gian hết hạn đã qua. Sau đó, số tiền được ký quỹ sẽ được chuyển ngược lại đến người gửi và bản ghi sẽ được xóa bỏ để ngăn chặn một kịch bản hoàn trả gấp đôi.  

[Quay trở lại]({% translate_link tools %})
