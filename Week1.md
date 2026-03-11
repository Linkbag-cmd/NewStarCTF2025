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





**strange_login**

<img width="1150" height="730" alt="image" src="https://github.com/user-attachments/assets/fa023f9e-e93f-49d2-a864-87a6172d7438" />

很明显是SQL注入，输一下就能得到flag  admin' or 1=1 #   

<img width="1300" height="626" alt="image" src="https://github.com/user-attachments/assets/f66f0030-38f3-4fc4-95f3-abeee2961a0d" />





**别笑，你也过不了第二关**

竟然还是魔丸....  这种题目一般正常完肯定是不行的，看一下源代码发现这么一个东西：

<img width="771" height="329" alt="image" src="https://github.com/user-attachments/assets/5a94b315-a917-4dcc-9232-1bff3e122084" />

第二关要100000分肯定是不可能的，我们肯定要改数据，抓个包看看....然后发现抓不到，那就尝试改数据：

<img width="1373" height="330" alt="image" src="https://github.com/user-attachments/assets/bddcee7a-b4ee-486b-af3b-00c7870f4117" />

然后就能拿到我们的flag了：

<img width="907" height="149" alt="image" src="https://github.com/user-attachments/assets/7115651a-12de-4737-8244-301032301a0e" />





**黑客小 W 的故事（1）**

要800块钱，但是容易被干掉，看到提示有显示：

<img width="2111" height="573" alt="image" src="https://github.com/user-attachments/assets/7224d5c1-f31f-48b7-95e6-2a582a30012a" />

那就抓个包：

<img width="2313" height="647" alt="image" src="https://github.com/user-attachments/assets/bbed017a-48d6-48ff-bd6e-b150affffb34" />

可以看到直接转到level2去了，依旧看提示：

<img width="1983" height="499" alt="image" src="https://github.com/user-attachments/assets/814dcd55-03bd-440d-9594-f876b9e27aa9" />

按照上述要求输入即可：

<img width="2642" height="715" alt="image" src="https://github.com/user-attachments/assets/edf2c2f1-539b-493c-927d-bb222287e205" />

提示有新要求，这个得抓包了：

<img width="1434" height="674" alt="image" src="https://github.com/user-attachments/assets/ace34879-01b1-4e3a-afac-e5b60dedb410" />

后面再回到页面就能看到显示/Level2_END，那么我们再访问这个界面，经过一系列的对话后查看相应提示，也算是对http基础的一个熟悉吧：

<img width="1927" height="321" alt="image" src="https://github.com/user-attachments/assets/3241af2b-74a6-4ad0-a9e4-73ad47c4d2a6" />

然后它这里有个抽象的地方是一个技能还有升级，直接输入CycloneSlash显示盗版(这脑洞也是逆天)，得提升版本：

<img width="3012" height="703" alt="image" src="https://github.com/user-attachments/assets/d234a07b-1766-4712-91d2-ae9c99f981a4" />

最后再访问一下得到我们的flag，这里就不贴出来了。

week1至此结束
