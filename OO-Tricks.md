#OO Tricks
## 最少知识原则 Law of Demeter
[链接](https://hackernoon.com/object-oriented-tricks-2-law-of-demeter-4ecc9becad85)  
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

## Death by arguments
* Argument Objects：如果传递了三个或更多的参数，那么这些变量能否合并成一个对象。
* Overloading：
* Builder Pattern：Effective Java

*boolean*  
参数列表中包含boolean，如果函数名和参数不能准确的传递意图，那么每次都需要查看实际的实现，来理解需要传递的正确值。如果不止一个boolean，那么更加麻烦

*null*  
公开的api使用@NonNull这样的注解，收到null，就抛出异常`IllegalArgumentException `