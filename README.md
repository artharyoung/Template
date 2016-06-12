# 使用Pattern与Matcher进行用户名与手机号校验

Pattern|[Android API](https://developer.android.com/reference/java/util/regex/Pattern.html)
Matcher|[Android API](https://developer.android.com/reference/java/util/regex/Matcher.html)
## UserName校验
这样的功能需求多用于账号注册，可以通过如下的正则表达式实现支持字母、数字、下划线的6~16位字符账号注册
```java
public static boolean isValidUserName(String username){
       Pattern p = Pattern.compile("(^[a-zA-Z0-9_]{6,16})");
       Matcher m = p.matcher(username);
       return m.matches();
  }
```
## 手机号校验
手机号校验首先要了解手机号有哪些，以国内的手机号为例：
> 移动：134、135、136、137、138、139、150、151、157(TD)、158、159、187、188

> 联通：130、131、132、152、155、156、185、186

> 电信：133、153、180、189、（1349卫通）

编写正则表达式进行实现：
```java
public static boolean isValidMobile(String mobile){
        Pattern p = Pattern.compile("^((13[0-9])|(15[^4,\\D])|(18[0,5-9]))\\d{8}$");
        Matcher m = p.matcher(mobiles);
        return m.matches();
    }
```
以上对于手机号的校验非常严格，在Android的Patterns类中封装了手机号校验的正则表达式：
```java
public static final Pattern PHONE
        = Pattern.compile(                      // sdd = space, dot, or dash
                "(\\+[0-9]+[\\- \\.]*)?"        // +<digits><sdd>*
                + "(\\([0-9]+\\)[\\- \\.]*)?"   // (<digits>)<sdd>*
                + "([0-9][0-9\\- \\.]+[0-9])"); // <digit><digit|sdd>+<digit>
```
可见，对于手机号只要求是0~9的数字。使用SDK中的正则表达式：
```java
public static boolean isValidMobile(String phone) {
        return Patterns.PHONE.matcher(phone).matches();
  }
```
## 其他字符串校验
除此之外，Patterns中还封装了Email、IP-address、WEB_URL等正则表达式。

校验Email地址：
```java
 private boolean isValidMail(String email) {
    return Patterns.EMAIL_ADDRESS.matcher(email).matches();
  }
```
校验URL:
```java
  private boolean isValidWebUrl(String url) {
    return Patterns.WEB_URL.matcher(url).matches();
  }
```
## 参考
[Java正则表达式：Pattern类和Matcher类](http://www.cnblogs.com/lonelysharer/archive/2012/03/08/2384773.html)
