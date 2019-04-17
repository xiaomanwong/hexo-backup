---
title: Java 判断字符串是否是数字的方法
date: 2019-04-16 17:47:06
tags:
---

* 使用 Java自带的函数

```
public static boolean isNumeric (String str) {
    for (int i = str.length(); --i >=0) {
          if (!Character.isDigit(str.charAt(i))) {
                return false;
          }
    }
    return true;
}
```

* 使用正则表达式

方法一:

```
public static boolean isNumeric(String str) {
    Pattern pattern = Pattern.compile("^[-\\+]?[\\d]*$");
    return pattern.matcher(str).matches();
}
```

方法二:

```
public static boolean isNumeric(String str) {
    if (str != null && !"".equals(str.trim())) {
          return s.matches("^[0-9]*$");
    }
    return false;
}
```

方法三:

```
 public static boolean isNumeric (String str) {
    Pattern pattern = Pattern.compile("[0-9]*");
    return pattern.matcher(str).matcher();
}
```

* 使用 ASCII 码

```
public static boolean isNumeric (String str) {
    for (int i = str.length(); --i>=0;) {
        int chr = str.charAt(i);
        if (chr < 48 || chr > 57) {
            return false;
        }
   }
   return true;
}
```

* 判断是不是浮点型数据

```
public static boolean isDouble(String str) {
    Pattern pattern = Pattern.compile("^[-\\+]?[.\\d]*$");
    return pattern.matcher(str).matches();
}
```