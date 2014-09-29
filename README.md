datetime_mongodb
================
###Xử lý datetime trong MongoDB

Khi muốn sắp xếp một trường trong MongoDB thì ta sử dụng $sort trong toán tử aggregation pipeline như sau 
```ruby
def sort sorted_field, sort_type
  {"$sort" => {sorted_field => sort_type}}
end
```
Trong MongoDB khi tạo document sẽ luôn tự động có các trường created_date và updated_at. Kiểu dữ liệu của 2 trường này là Datetime ví dụ: "created_at: 2014-09-26 02:18:09 UTC". Nếu sử dụng $sort với trường created_date thì kết quả sẽ sắp xếp theo thời gian như thông thường. Vậy nếu muốn group trường created_at theo thứ trong tuần, hoặc theo tháng rồi sắp xếp hoặc chọn ra kết quả là thứ 2 trong trường created_at thì sẽ phải làm như thế nào?

Nhắc lại về toán tử aggregation pipeline trong MongoDB thì  aggregation pipeline sẽ thực hiện các method $match, $sort, $group, $project, $limit.. một cách tuần tự theo thứ tự gọi các method khi sử dụng toán tử aggregation. Có nghĩa là có thể nhóm các phần tử, sau đó sắp xếp kết quả nhận được. Tiếp tục nhóm các kêt cả sau sắp xếp rồi giới hạn số kết quả in ra… Đây là điểm mạnh của toán tử aggregation. Method nào được gọi trước sẽ được xử lý trước, kết quả của method trước sẽ là input của method sau.

Ví dụ với vấn đề lấy ra kết quả mà trường created _at vào thứ 2 trong tuần thì làm như sau
Sử dụng $project để lấy kết quả có trường day_of_week lưu thứ trong tuần được chuyển đổi từ trường  created _at 
def project_day_of_week
  {
    “$project” => {
      dayweek: {“$dayOfWeek” => “$created_at”},
      name: 1
    }
  }
end

$dayOfWeek là 1 method của MongoDB hỗ trợ việc lấy thứ trong tuần. Ngoài ra MongoDB có rất nhiều hàm hỗ trợ xử lý ngày tháng như $dayOfMonth trả về giá trị ngày trong tháng từ 1 đến 31, $hour trả về giá trị giờ trong ngày từ 0đến 23. Có thể tham khảo trong link:
http://docs.mongodb.org/manual/reference/operator/aggregation-date/

Sau khi sử dụng method  project_day_of_week thì ta sẽ có kết quả trả về là một bảng có trường dayweek lưu ngày trong tuần. Sử dụng method $match để lấy tiếp các kết quả vào ngày thứ 2 như sau

def match_day_of_week
 {
    “$match” => {
      dayweek: 2
    }
  }	
end
Cần lưu ý rằng kết quả trả về khi sử dụng $dayOfWeek của ngày chủ nhật sẽ là 1, thứ 2 là 2... cho đến thứ 7 là 7.
Sử dụng toán tử aggreation pipeline như sau
db.sample_db.aggregate.(project_day_of_week, match_day_of_week)
sẽ cho kết quả là ngày thứ 2 theo trường created_at. 
Với toán tử aggregation pipeline thì có thể sắp xếp tiếp kết quả in ra theo name một cách đơn giản như sau
def sort_name
 {
    “$sort” => {
      name: 1
    }
  }	
và aggregate method là 
db.sample_db.aggregate.(project_day_of_week, match_day_of_week, sort_name)
Tương tự như trên thì khi cần group trường created_at theo thứ trong tuần rồi sắp xếp thì chỉ cần thêm 2 hàm group
def group_day_of_week
 {
    “$group” => {
      id: {dayofweek: "$dayweek"}
    }
  }

và hàm sort
def sort_day_of_week
	
{
    “$sort” => {
      "id.dayofweek" : 1
    }
  }
end

Sử dụng toán tử aggregation pipeline :

db.sample_db.aggregate.(project_day_of_week, group_day_of_week, sort_day_of_week
)

 Sử dụng mongoid trong rails để có thể dùng các hàm xử lý trên mongodb như active record là where, group... thì cũng có thể xử lý dễ dàng các vấn đề trên nhưng sẽ cho performance không cao bằng việc sử dụng aggregation pipeline trong trường hợp database là big data.



