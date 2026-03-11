**multi-headach3**

第一次进去的话应该是这么一个界面：
<img width="752" height="455" alt="image" src="https://github.com/user-attachments/assets/3580ba4b-b773-43ca-b7a3-72b4aedb5097" />

很容易让人联想到robots.txt，访问看一下发现确实有hidden.php，但是它会进行跳转到index.php

<img width="1030" height="698" alt="image" src="https://github.com/user-attachments/assets/b204077f-e7d4-42d2-90a3-5c55cf643963" />

联想到302重定向，打开网络再尝试一次，发现flag：
<img width="1571" height="354" alt="image" src="https://github.com/user-attachments/assets/5df475a6-ecc3-40c4-99a1-811cdf8a93df" />


**宇宙的中心是 PHP**

进去是一个动画界面，f12，右键啥的按不出来，但是依旧可以手动调用开发者界面，或者ctrl+shift+I调出来，发现：
<img width="686" height="151" alt="image" src="https://github.com/user-attachments/assets/c4f4d47c-523a-4caa-9b7d-134d7b3bb8df" />

```
<?php
highlight_file(__FILE__);
include "flag.php";
if(isset($_POST['newstar2025'])){
    $answer = $_POST['newstar2025'];
    if(intval($answer)!=47&&intval($answer,0)==47){
        echo $flag;
    }else{
        echo "你还未参透奥秘";
    }
}
```
这里要同时满足：

intval($answer)不等于 47

intval($answer, 0)等于 47

关键点在于：intval($str, 0) 的 base=0 模式会自动检测前缀来决定进制：

0x 开头 → 按十六进制解析

0 开头 → 按八进制解析

其他 → 按十进制解析

因此，这里可以采用十六进制进行绕过--0x2f

<img width="1218" height="689" alt="image" src="https://github.com/user-attachments/assets/3c59013a-f5a0-4a32-8813-915aad542bf0" />




