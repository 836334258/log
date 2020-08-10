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

3. Php7.3json处理

   ```php
   $json = '{"a":1,"b":2,"c":3,"d",4,"e":5}';
   
   try {
       var_dump(json_decode($json,true,512,JSON_THROW_ON_ERROR));
   }catch (JsonException $e){
       echo $e->getCode();
   }
   ```

4. curl 并发请求

   ```php
   function curl(string $url, bool $isPost, int $threads = 1, array $postData = [])
   {
       $mh = curl_multi_init();
       $ch_array = array();
   
       for ($i = 0; $i < $threads; $i++) {
           $ch_array[$i] = curl_init();
   
           curl_setopt($ch_array[$i], CURLOPT_URL, $url);
           curl_setopt($ch_array[$i], CURLOPT_RETURNTRANSFER, 1);
           curl_setopt($ch_array[$i], CURLOPT_POST, $isPost);
           curl_setopt($ch_array[$i], CURLOPT_SSL_VERIFYHOST, false);
           curl_setopt($ch_array[$i], CURLOPT_SSL_VERIFYPEER, false);
           curl_setopt($ch_array[$i], CURLOPT_HTTPHEADER, [
               'User-Agent' => 'Mozilla/5.0 (Macintosh; Intel Mac OS X 10_15_6) AppleWebKit/537.36 (KHTML, like Gecko) Chrome/84.0.4147.89 Safari/537.36'
           ]);
           if ($isPost) {
               curl_setopt($ch_array[$i], CURLOPT_POSTFIELDS, $postData);
           }
   
           curl_multi_add_handle($mh, $ch_array[$i]);
       }
   
       $running = null;
       do {
           $mrc = curl_multi_exec($mh, $running);
   
       } while ($mrc == CURLM_CALL_MULTI_PERFORM);
   
   
       while ($running && $mrc == CURLM_OK) {
           usleep(5000);
           if (curl_multi_select($mh) != -1) {
               do {
                   $mrc = curl_multi_exec($mh, $running);
               } while ($mrc == CURLM_CALL_MULTI_PERFORM);
           }
       }
   
       for ($i = 0; $i < $threads; $i++) {
           $data = curl_multi_getcontent($ch_array[$i]);
           var_dump($data);
           curl_multi_remove_handle($mh, $ch_array[$i]);
       }
   
   
       curl_multi_close($mh);
   }
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

- 关于stdclass，`$arr2=(object)$arr;` 这种stdclass是基类，正常的class不是

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

