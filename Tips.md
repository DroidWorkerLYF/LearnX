#Tips
## 最少知识原则
[Law of Demeter](https://hackernoon.com/object-oriented-tricks-2-law-of-demeter-4ecc9becad85)  
得墨忒耳定律，亦称为最少知识原则  

```
obj.getX().getY().getZ().doSomething()
	
obj.doSomething()
```
每个模块应该对其他模块只有有限的了解，只了解和他相关的模块。don't talk to strangers。  

Tell don't ask.我们应该调用这些对象的方法的方法：

* 作为参数传递的对象
* 方法内生成的对象
* 对象的变量
* 全局的对象

*Example*

```
class User {
	Account accout;
	
	double discountedPlanPrice(String discountCode) {
		Coupon coupon = Coupon.create(discountCode);
		return counpon.discount(account.getPlan().getPrice());
	}
}
	
class Account {
	Plan plan;
}
```
```
class User {
	Account accout;
	
	double discountedPlanPrice(String discountCode) {
		return account.discountedPlanPrice(discountCode);
	}
}
	
class Account {
	Plan plan;
	
	double discountedPlanPrice(String discountedCode) {
		Coupon coupon = Coupon.create(discountCode);
		return coupon.discount(plan.getPrice());
	}
}
```

## Command Query Separation
[链接](https://hackernoon.com/oo-tricks-the-art-of-command-query-separation-9343e50a3de0)
> Functions that change state should not return values and functions that return values should not change state.

```
User u = UserService.login(username, password);
```
改变了系统状态，同时有返回值，带来了side effect。
```
User u = UserService.getUser();
```
大部分情况下CQS都是work的，但是对像`Stack.pop`也是例外的。  
Commands return void and queries return values.  
Use exceptions rather than returning and checking for error states.


## Ripple效果
[链接](http://michaelevans.org/blog/2015/05/07/android-ripples-with-rounded-corners/)  
给View设置ripple效果时，需要让波纹是view background的形状，可以在ripple的xml中添加一个item和view background是同样的形状，并指定id为@android:id/mask。

```
<?xml version="1.0" encoding="utf-8"?>
<ripple xmlns:android="http://schemas.android.com/apk/res/android"
    android:color="?android:attr/colorControlHighlight">
    <item android:id="@android:id/mask">
        <shape android:shape="rectangle">
            <solid android:color="#000000" />
            <corners android:radius="15dp" />
        </shape>
    </item>
    <item android:drawable="@drawable/rounded_corners" />
</ripple>
```