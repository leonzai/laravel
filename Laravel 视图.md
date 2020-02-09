> 配套视频地址 [https://www.bilibili.com/video/av70545323?p=8](https://www.bilibili.com/video/av70545323?p=8 "https://www.bilibili.com/video/av70545323?p=8")

---

## **知识点**

1. 返回视图
2. 给视图传递数据
3. 模版语法

#### **1. 返回视图**

```php
// resources/views/test.blade.php
return view('test');            

// resources/views/parent/test.blade.php
return view('parent.test');     
```

#### **2. 给视图传递数据**

```php
return view('greetings', ['name' => 'Victoria']);
return view('greeting')->with('name', 'Victoria');

return view('student', compact('names', 'ages'));
// 相当于 return view('student', ['names' => $names, 'ages' => $ages]);
```

#### **3. 模版语法**

###### **注释**

```php
{{-- 浏览器查看源代码不会显示这条注释 --}}
<!-- 浏览器查看源代码会显示这条注释 -->
```

###### **php 标签**

```php
@php
    
@endphp
    
// 等同于
<?php 

?>
```

###### **表单伪造**

```html
<form method="POST" action="/profile">
    @method('PUT')     // put delete 模拟
    @csrf              // method="POST" 必须加这行，防止跨站请求伪造
    ...
</form>    
   
<!-- 等同于 -->
<form method="POST" action="/profile">
    <input type="hidden" name="_token" value="zou4UiPPJQ6MmwKWL">     
    <input type="hidden" name="_method" value="PUT">   
</form>
```

###### **输出变量**

```php
{{ $name }} // 等同于 <?=htmlspecialchars($name)?> 避免 xss 攻击

{!! $name !!}  // 不带 htmlspecialchars


@{{ $name }}

@verbatim
    <div class="container">
        Hello, {{ $name }}.
    </div>
@endverbatim
```

###### **模版继承**

```php
块两种定义方式：
    1. 使用 section
        @section('def1')
        @show

        @section('def2')
            一些内容
        @show
    
    2. 使用 yield
    	@yield('def3')
    
    	@yield('def4', '一些内容')
    
块的使用方式
    @section('def2', "<p>def2</p>")   // def2的内容会 htmlspecialchars
    
    @section('def1')
        正在使用
    @endsection
```

```css
// parent.blade.php
@section('header')
    <p>This is header.</p>
@show

@yield('nav', '<p>I am nav.</p>')

@yield('content')

@section('footer')
    <p>父 footer.</p>
@show
    
    
// son.blade.php 继承 parent.blade.php
@extends('extend.father')


@section('header', "<p>新的头部</p>")


@section('content')
    <p>body 主体</p>
@endsection


@section('footer')
    @parent
    <p> 子 footer </p>
@endsection
```

###### **模版包含**

```php
@include('shared.errors')
```

###### **模版组件**

```php
<!-- /resources/views/alert.blade.php -->
<div class="alert alert-danger">
    {{ $slot }}
</div>

@component('alert')
    <strong>Whoops!</strong> Something went wrong!
@endcomponent

    
@componentFirst(['custom.alert', 'alert'])
    <strong>Whoops!</strong> Something went wrong!
@endcomponent
    
    
<!-- /resources/views/alert.blade.php -->

<div class="alert alert-danger">
    <div class="alert-title">{{ $title }}</div>
    {{ $slot }}
</div>

@component('alert')
    @slot('title')
        Forbidden
    @endslot
    You are not allowed to access this resource!
@endcomponent

   
@component('alert', ['foo' => 'bar'])
    ...
@endcomponent
```

###### **错误消息**

```php
<input id="title" type="text" class="@error('title') is-invalid @enderror">

@error('title')
    <div class="alert alert-danger">{{ $message }}</div>
@enderror    

@error('email', 'login') is-invalid @enderror
```

###### **json**

```php
<script>
    var app = @json($array);  // <?php echo json_encode($array); ?>

    var app = @json($array, JSON_PRETTY_PRINT);
</script>
<example-component :some-prop='@json($array)'></example-component>
# 在元素属性中使用@json要求它用单引号括起来。
```

###### **控制与循环**

```css
@if (count($records) === 1)
    I have one record!
@elseif (count($records) > 1)
    I have multiple records!
@else
    I don't have any records!
@endif

@unless (Auth::check())
	// 未登录
@endunless

@isset($records)
    // $records is defined and is not null...
@endisset

@empty($records)
    // $records is "empty"...
@endempty

@auth
    // The user is authenticated...
@endauth

@guest
    // The user is not authenticated...
@endguest

@auth('admin')   // 自己写的 admin 认证方式
    // The user is authenticated...
@endauth

@guest('admin')
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

@for ($i = 0; $i < 10; $i++)
    The current value is {{ $i }}
@endfor

@foreach ($users as $user)
    <p>This is user {{ $user->id }}</p>
@endforeach

@forelse ($users as $user)
    <li>{{ $user->name }}</li>
@empty
    <p>No users</p>
@endforelse

@while (true)
    <p>I'm looping forever.</p>
@endwhile

@foreach ($users as $user)
    @if ($user->type == 1)
        @continue
    @endif

    <li>{{ $user->name }}</li>
    
    @if ($user->number == 5)
        @break
    @endif
@endforeach

@foreach ($users as $user)
    @continue($user->type == 1)

    <li>{{ $user->name }}</li>
    
    @break($user->number == 5)
@endforeach

@foreach ($users as $user)
    @if ($loop->first)
        This is the first iteration.
    @endif

    @if ($loop->last)
        This is the last iteration.
    @endif
    
    <p>This is user {{ $user->id }}</p>
@endforeach

@foreach ($users as $user)
    @foreach ($user->posts as $post)
        @if ($loop->parent->first)
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
if (\View::exists('emails.customer')) 
return view()->first(['custom.admin', 'admin'], $data);  
return \View::first(['custom.admin', 'admin'], $data);   // 同上


public function boot()
{
    View::share('key', 'value');  // class AppServiceProvider
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
        'profile', 'App\Http\ViewComposers\ProfileComposer'
    );            // ProfileComposer@compose 在 profile 视图生成前调用

    View::composer('dashboard', function ($view) {
        //
    });
}


class ProfileComposer
{
    public function compose(View $view)
    {
        $view->with('key', 'value');
    }
}
```

```php
# 多个视图
View::composer(
    ['profile', 'dashboard'],
    'App\Http\ViewComposers\MyViewComposer'
);
```
```php
# 所有
View::composer('*', function ($view) {
    //
});
```

###### **视图 creator**

```php
# 视图 creators 和视图合成器非常相似。唯一不同之处在于：视图构造器在视图实例化之后立即执行，而视图合成器在视图即将渲染时执行。
# 简单的说，creator 在 composer 之前执行
View::creator('profile', 'App\Http\ViewCreators\ProfileCreator');
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
@includeIf('view.name', ['some' => 'data'])

@includeWhen($boolean, 'view.name', ['some' => 'data'])

@includeFirst(['custom.admin', 'admin'], ['some' => 'data'])
```

```php
# resources/views/includes/input.blade.php
<input type="{{ $type ?? 'text' }}"> // isset($type) ? $type : 'text';
    
// AppServiceProvider 
\Blade::include('includes.input', 'input');   // 别名

@input(['type' => 'email'])  
// 等价于 @include('includes.input' ,['type' => 'email'])
```

```php
@each('view.name', $jobs, 'job')
@each('view.name', $jobs, 'job', 'view.empty')  // 数组是空，则渲染 view.empty
```

###### **堆栈**

```php
<!-- common.blade.php -->
@stack('scripts')
@stack('scripts')
```
```php
<!-- extend.blade.php -->
@push('scripts')
    <script src="/example.js"></script>
@endpush

/**  输出 start
<script src="/example.js"></script>
<script src="/example.js"></script>
     输出 end  **/
```
```php
@push('scripts')
    显示为第二个
@endpush


@prepend('scripts')
    显示为第一个
@endprepend
```

###### **服务注入**

```php
@inject('metrics', 'App\Services\MetricsService')

<div>
    Monthly Revenue: {{ $metrics->monthlyRevenue() }}.
        // (new App\Services\MetricsService())->monthlyRevenue()
</div>
```

###### **拓展 Blade**

```php
# class AppServiceProvider 中 boot 方法
\Blade::directive('datetime', function ($expression) {
	return "<?php echo ($expression)->format('m/d/Y H:i'); ?>";
});
```

> 更新`Blade`指令的逻辑之后，您需要删除所有缓存的`Blade`视图(`php artisan view:clear`)。 
```html
<!-- some.blade.php -->
@datetime($var)
// <?php echo ($var)->format('m/d/Y H:i'); ?>
```

###### **自定义模版语法**

```php
public function boot()
{
    \Blade::if('env', function ($environment) {
        return app()->environment($environment);
    });
}
```

```php
@env('local')
    // The application is in the local environment...
@else
    // The application is not in the local environment...
@endenv
```
