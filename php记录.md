### 过滤器

1. 字符串过滤器

```php
$fp=fopen("php://output",'w');
stream_filter_append($fp,'string.toupper');
fwrite($fp,"abcd");

echo fgets($fp);
ABCD
```

2. base64过滤

```php
$fp=fopen("php://output",'w');
stream_filter_append($fp,'convert.base64-encode');
fwrite($fp,"abcd");
fclose($fp);

echo fgets($fp);
YWJj

```

### 概念

- Trait

  - 当trait中的方法和类重名时，使用类中的方法

  - `use AA{test as protected;}`修改trait中方法的的访问级别

  - ```php
    use AA,AAA{
            AAA::test insteadof AA;
        }
    ```

    当trait方法名冲突时，使用AAA的test方法替换AA的test方法

- 静态方法可以通过一个实例化的对象访问，静态属性不行

-  spl_autoload_register 类的自动加载

  ```php
  spl_autoload_register(function ($className){
     require_once $className;
  });
  ```

- 

### 函数

- password_hash($a,PASSWORD_DEFAULT) 加密
- password_verify($a,'$2y$10$AhtReg1LlRQLqa7LhGXYdeOdIMaWfuDCzTfdz.5oHPp8iM3sC48yO') 解密
- serialize()
- unserialiaze()
- addslashes() `"a'bcd"` ->`a\'bcd`
-  parse_str 将字符串解析成多个变量 first=value
-  pathinfo  pathinfo('https://www.php.net/manual/zh/book.strings.php',PATHINFO_DIRNAME);
-  print 是语言结构，只支持一个参数，总是返回1
-  similar_text 计算字符串相似度(统计有几个字符串相同，从左往右)
-  error_reporting(-1)报告所有错误

