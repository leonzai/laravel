> 配套视频地址 [https://www.bilibili.com/video/av70545323](https://www.bilibili.com/video/av70545323 "https://www.bilibili.com/video/av70545323")

---

###### **Laravel 是什么**

一个基于 php 的 MVC web 开发框架。

---

###### **MVC 原理**

![](http://qianjinyike.com/wp-content/uploads/2019/12/2056a5fbf247ae7a00a6a0f21efb205d.png)
用户通过在浏览器地址栏中输入链接，将请求发送到我们的应用程序：

（1）应用程序将请求匹配到某个特定路由。
（2）该路由映射到特定的控制器，控制器开始处理请求。
（3）如果控制器中需要操作数据，则把需要操作的内容交给模型。
（4，5）模型与数据库交互后，
（6）把搞定的数据返回给控制器。
（7）控制器最后把数据交给视图美化，
（8）视图美化完毕后返回给用户。

---


###### **一个简单的例子**

```php
&lt;?php
	
// 路由文件：routes/web.php

Route::get(&#039;/students&#039;, &#039;StudentController@index&#039;);
```

```php
&lt;?php
	
// 控制器文件：app/Http/Controllers/StudentController.php

namespace App\Http\Controllers;

use App\Student;

class StudentController extends Controller
{
	public function index()
	{
		$students = Student::all();
		return view(&#039;students.index&#039;, compact(&#039;students&#039;));
	}
}
```

```php
&lt;?php
	
// 模型文件：app/Student.php
	
namespace App;

use Illuminate\Database\Eloquent\Model;

class Student extends Model
{

}
```

```php
// 视图文件：resources/views/students/index.blade.php

@foreach($students as $student)
	&lt;p&gt;{{ $student-&gt;name }}&lt;/p&gt;
@endforeach
```