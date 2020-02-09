> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=8](https://www.bilibili.com/video/av70545323?p=8 "https://www.bilibili.com/video/av70545323?p=8")

---

## **知识点**

1. 返回视图
2. 给视图传递数据
3. 模版语法

#### **1. 返回视图**

```php
// resources/views/test.blade.php
return view(&#039;test&#039;);            

// resources/views/parent/test.blade.php
return view(&#039;parent.test&#039;);     
```

#### **2. 给视图传递数据**

```php
return view(&#039;greetings&#039;, [&#039;name&#039; =&gt; &#039;Victoria&#039;]);
return view(&#039;greeting&#039;)-&gt;with(&#039;name&#039;, &#039;Victoria&#039;);

return view(&#039;student&#039;, compact(&#039;names&#039;, &#039;ages&#039;));
// 相当于 return view(&#039;student&#039;, [&#039;names&#039; =&gt; $names, &#039;ages&#039; =&gt; $ages]);
```

#### **3. 模版语法**

###### **注释**

```php
{{-- 浏览器查看源代码不会显示这条注释 --}}
&lt;!-- 浏览器查看源代码会显示这条注释 --&gt;
```

###### **php 标签**

```php
@php
    
@endphp
    
// 等同于
&lt;?php 

?&gt;
```

###### **表单伪造**

```html
&lt;form method=&quot;POST&quot; action=&quot;/profile&quot;&gt;
    @method(&#039;PUT&#039;)     // put delete 模拟
    @csrf              // method=&quot;POST&quot; 必须加这行，防止跨站请求伪造
    ...
&lt;/form&gt;    
   
&lt;!-- 等同于 --&gt;
&lt;form method=&quot;POST&quot; action=&quot;/profile&quot;&gt;
    &lt;input type=&quot;hidden&quot; name=&quot;_token&quot; value=&quot;zou4UiPPJQ6MmwKWL&quot;&gt;     
    &lt;input type=&quot;hidden&quot; name=&quot;_method&quot; value=&quot;PUT&quot;&gt;   
&lt;/form&gt;
```

###### **输出变量**

```php
{{ $name }} // 等同于 &lt;?=htmlspecialchars($name)?&gt; 避免 xss 攻击

{!! $name !!}  // 不带 htmlspecialchars


@{{ $name }}

@verbatim
    &lt;div class=&quot;container&quot;&gt;
        Hello, {{ $name }}.
    &lt;/div&gt;
@endverbatim
```

###### **模版继承**

```php
块两种定义方式：
    1. 使用 section
        @section(&#039;def1&#039;)
        @show

        @section(&#039;def2&#039;)
            一些内容
        @show
    
    2. 使用 yield
    	@yield(&#039;def3&#039;)
    
    	@yield(&#039;def4&#039;, &#039;一些内容&#039;)
    
块的使用方式
    @section(&#039;def2&#039;, &quot;&lt;p&gt;def2&lt;/p&gt;&quot;)   // def2的内容会 htmlspecialchars
    
    @section(&#039;def1&#039;)
        正在使用
    @endsection
```

```css
// parent.blade.php
@section(&#039;header&#039;)
    &lt;p&gt;This is header.&lt;/p&gt;
@show

@yield(&#039;nav&#039;, &#039;&lt;p&gt;I am nav.&lt;/p&gt;&#039;)

@yield(&#039;content&#039;)

@section(&#039;footer&#039;)
    &lt;p&gt;父 footer.&lt;/p&gt;
@show
    
    
// son.blade.php 继承 parent.blade.php
@extends(&#039;extend.father&#039;)


@section(&#039;header&#039;, &quot;&lt;p&gt;新的头部&lt;/p&gt;&quot;)


@section(&#039;content&#039;)
    &lt;p&gt;body 主体&lt;/p&gt;
@endsection


@section(&#039;footer&#039;)
    @parent
    &lt;p&gt; 子 footer &lt;/p&gt;
@endsection
```

###### **模版包含**

```php
@include(&#039;shared.errors&#039;)
```

###### **模版组件**

```php
&lt;!-- /resources/views/alert.blade.php --&gt;
&lt;div class=&quot;alert alert-danger&quot;&gt;
    {{ $slot }}
&lt;/div&gt;

@component(&#039;alert&#039;)
    &lt;strong&gt;Whoops!&lt;/strong&gt; Something went wrong!
@endcomponent

    
@componentFirst([&#039;custom.alert&#039;, &#039;alert&#039;])
    &lt;strong&gt;Whoops!&lt;/strong&gt; Something went wrong!
@endcomponent
    
    
&lt;!-- /resources/views/alert.blade.php --&gt;

&lt;div class=&quot;alert alert-danger&quot;&gt;
    &lt;div class=&quot;alert-title&quot;&gt;{{ $title }}&lt;/div&gt;
    {{ $slot }}
&lt;/div&gt;

@component(&#039;alert&#039;)
    @slot(&#039;title&#039;)
        Forbidden
    @endslot
    You are not allowed to access this resource!
@endcomponent

   
@component(&#039;alert&#039;, [&#039;foo&#039; =&gt; &#039;bar&#039;])
    ...
@endcomponent
```

###### **错误消息**

```php
&lt;input id=&quot;title&quot; type=&quot;text&quot; class=&quot;@error(&#039;title&#039;) is-invalid @enderror&quot;&gt;

@error(&#039;title&#039;)
    &lt;div class=&quot;alert alert-danger&quot;&gt;{{ $message }}&lt;/div&gt;
@enderror    

@error(&#039;email&#039;, &#039;login&#039;) is-invalid @enderror
```

###### **json**

```php
&lt;script&gt;
    var app = @json($array);  // &lt;?php echo json_encode($array); ?&gt;

    var app = @json($array, JSON_PRETTY_PRINT);
&lt;/script&gt;
&lt;example-component :some-prop=&#039;@json($array)&#039;&gt;&lt;/example-component&gt;
# 在元素属性中使用@json要求它用单引号括起来。
```

###### **控制与循环**

```css
@if (count($records) === 1)
    I have one record!
@elseif (count($records) &gt; 1)
    I have multiple records!
@else
    I don&#039;t have any records!
@endif

@unless (Auth::check())
	// 未登录
@endunless

@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is &quot;empty&quot;...
@endempty

@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest

@auth(&#039;admin&#039;)   // 自己写的 admin 认证方式
    // The user is authenticated...
@endauth

@guest(&#039;admin&#039;)
    // The user is not authenticated...
@endguest

@switch($i)
    @case(1)
        First case...
        @break

    @case(2)
        Second case...
        @break
    
    @default
        Default case...
@endswitch

@for ($i = 0; $i &lt; 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    &lt;p&gt;This is user {{ $user-&gt;id }}&lt;/p&gt;
@endforeach

@forelse ($users as $user)
    &lt;li&gt;{{ $user-&gt;name }}&lt;/li&gt;
@empty
    &lt;p&gt;No users&lt;/p&gt;
@endforelse

@while (true)
    &lt;p&gt;I&#039;m looping forever.&lt;/p&gt;
@endwhile

@foreach ($users as $user)
    @if ($user-&gt;type == 1)
        @continue
    @endif

    &lt;li&gt;{{ $user-&gt;name }}&lt;/li&gt;
    
    @if ($user-&gt;number == 5)
        @break
    @endif
@endforeach

@foreach ($users as $user)
    @continue($user-&gt;type == 1)

    &lt;li&gt;{{ $user-&gt;name }}&lt;/li&gt;
    
    @break($user-&gt;number == 5)
@endforeach

@foreach ($users as $user)
    @if ($loop-&gt;first)
        This is the first iteration.
    @endif

    @if ($loop-&gt;last)
        This is the last iteration.
    @endif
    
    &lt;p&gt;This is user {{ $user-&gt;id }}&lt;/p&gt;
@endforeach

@foreach ($users as $user)
    @foreach ($user-&gt;posts as $post)
        @if ($loop-&gt;parent-&gt;first)
            This is first iteration of the parent loop.
        @endif
    @endforeach
@endforeach
```

属性	|描述
--|--
$loop->index	|当前迭代的索引（从 0 开始计数）。
$loop->iteration|	当前循环迭代 (从 1 开始计算）。
$loop->remaining|	循环中剩余迭代的数量。
$loop->count|	被迭代的数组元素的总数。
$loop->first|	是否为循环的第一次迭代。
$loop->last	|是否为循环的最后一次迭代。
$loop->even	|是否为循环的偶数次迭代。
$loop->odd	|是否为循环的奇数次迭代。
$loop->depth|	当前迭代的嵌套深度级数。
$loop->parent|	嵌套循环中，父循环的循环变量

## **拓展（用的并不多，想用时候能想起来就行）**

#### **基础**

```php
if (\View::exists(&#039;emails.customer&#039;)) 
return view()-&gt;first([&#039;custom.admin&#039;, &#039;admin&#039;], $data);  
return \View::first([&#039;custom.admin&#039;, &#039;admin&#039;], $data);   // 同上


public function boot()
{
    View::share(&#039;key&#039;, &#039;value&#039;);  // class AppServiceProvider
    // 在所有视图中都可以使用 {{ $key }}
}
```

#### **返回视图预操作**

###### **视图 composers**

```php
// service provider
public function boot()
{
    View::composer(
        &#039;profile&#039;, &#039;App\Http\ViewComposers\ProfileComposer&#039;
    );            // ProfileComposer@compose 在 profile 视图生成前调用

    View::composer(&#039;dashboard&#039;, function ($view) {
        //
    });
}


class ProfileComposer
{
    public function compose(View $view)
    {
        $view-&gt;with(&#039;key&#039;, &#039;value&#039;);
    }
}
```

```php
# 多个视图
View::composer(
    [&#039;profile&#039;, &#039;dashboard&#039;],
    &#039;App\Http\ViewComposers\MyViewComposer&#039;
);
```
```php
# 所有
View::composer(&#039;*&#039;, function ($view) {
    //
});
```

###### **视图 creator**

```php
# 视图 creators 和视图合成器非常相似。唯一不同之处在于：视图构造器在视图实例化之后立即执行，而视图合成器在视图即将渲染时执行。
# 简单的说，creator 在 composer 之前执行
View::creator(&#039;profile&#039;, &#039;App\Http\ViewCreators\ProfileCreator&#039;);
```


#### **更多语法**

###### **全局取消 htmlspecialchars 转化**

```php
public function boot()
{
    \Blade::withoutDoubleEncoding();
}
```

```php
@includeIf(&#039;view.name&#039;, [&#039;some&#039; =&gt; &#039;data&#039;])

@includeWhen($boolean, &#039;view.name&#039;, [&#039;some&#039; =&gt; &#039;data&#039;])

@includeFirst([&#039;custom.admin&#039;, &#039;admin&#039;], [&#039;some&#039; =&gt; &#039;data&#039;])
```

```php
# resources/views/includes/input.blade.php
&lt;input type=&quot;{{ $type ?? &#039;text&#039; }}&quot;&gt; // isset($type) ? $type : &#039;text&#039;;
    
// AppServiceProvider 
\Blade::include(&#039;includes.input&#039;, &#039;input&#039;);   // 别名

@input([&#039;type&#039; =&gt; &#039;email&#039;])  
// 等价于 @include(&#039;includes.input&#039; ,[&#039;type&#039; =&gt; &#039;email&#039;])
```

```php
@each(&#039;view.name&#039;, $jobs, &#039;job&#039;)
@each(&#039;view.name&#039;, $jobs, &#039;job&#039;, &#039;view.empty&#039;)  // 数组是空，则渲染 view.empty
```

###### **堆栈**

```php
&lt;!-- common.blade.php --&gt;
@stack(&#039;scripts&#039;)
@stack(&#039;scripts&#039;)
```
```php
&lt;!-- extend.blade.php --&gt;
@push(&#039;scripts&#039;)
    &lt;script src=&quot;/example.js&quot;&gt;&lt;/script&gt;
@endpush

/**  输出 start
&lt;script src=&quot;/example.js&quot;&gt;&lt;/script&gt;
&lt;script src=&quot;/example.js&quot;&gt;&lt;/script&gt;
     输出 end  **/
```
```php
@push(&#039;scripts&#039;)
    显示为第二个
@endpush


@prepend(&#039;scripts&#039;)
    显示为第一个
@endprepend
```

###### **服务注入**

```php
@inject(&#039;metrics&#039;, &#039;App\Services\MetricsService&#039;)

&lt;div&gt;
    Monthly Revenue: {{ $metrics-&gt;monthlyRevenue() }}.
        // (new App\Services\MetricsService())-&gt;monthlyRevenue()
&lt;/div&gt;
```

###### **拓展 Blade**

```php
# class AppServiceProvider 中 boot 方法
\Blade::directive(&#039;datetime&#039;, function ($expression) {
	return &quot;&lt;?php echo ($expression)-&gt;format(&#039;m/d/Y H:i&#039;); ?&gt;&quot;;
});
```

> 更新`Blade`指令的逻辑之后，您需要删除所有缓存的`Blade`视图(`php artisan view:clear`)。 
```html
&lt;!-- some.blade.php --&gt;
@datetime($var)
// &lt;?php echo ($var)-&gt;format(&#039;m/d/Y H:i&#039;); ?&gt;
```

###### **自定义模版语法**

```php
public function boot()
{
    \Blade::if(&#039;env&#039;, function ($environment) {
        return app()-&gt;environment($environment);
    });
}
```

```php
@env(&#039;local&#039;)
    // The application is in the local environment...
@else
    // The application is not in the local environment...
@endenv
```