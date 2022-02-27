# pwsh
## waniCTF2021fall
## rev
***

.ps1ファイルが配布される。検索してみるとpowershellで動くファイルっぽい
普通にダブルクリックで実行できたためとりあえず実行してみると
```
Welcome to the world of PowerShell!
Password:
```
とでてくるため、適当な文字を入力すると、一瞬で画面が消えた。
これでは何もわからないため、問題文に書いてあったhttps://docs.microsoft.com/en-us/powershell/scripting/install/install-ubuntu?view=powershell-7.1#installation-via-package-repository
にアクセスして、WSLにpowershellをインストールした。

WSL上のpowershellでもう一度実行してみる
``
PS /rev-pwsh> ./pwsh.ps1
Welcome to the world of PowerShell!
Password: hoge
Incorrect
```
どうやらパスワードが間違っているとIncorrectと出るっぽい。
諦めてソースコードを読んでみることにした。

コードを見ると、
```
(("{39}{4}{12}{45}{21}{0}{36}{25}{26}{27}{7}{13}{30}{16}{31}{48}{23}{18}{19}{20}{24}{28}{3}{38}{11}{5}{2}{8}{46}{34}{29}{1}{35}{15}{10}{33}{9}{32}{22}{37}{40}{6}{43}{17}{47}{44}{14}{41}{42}"-f ' world of PowerShe','d_p','cl','d3','ch','1n_','else','ost cW4Passwo','34r1n68r30b','{
  Writ','l}','_','o ','r','W4Incor','w3r5h3l','W','
 ','t ','-eq c','W4FLAG{','he','t c','(fj7inpu','y0u_','fj7input =',' ','Read-H','5ucc33','473','dc','4','e-Outpu','cW4) ','u5c','0','ll!cW4

','W4Co','d','e','rrect!cW4
} ','rec','tcW4
}
',' {','tput c','cW4Welcome to t','f',' Write-Ou','

if ')).replACe('cW4',[STRiNg][CHAr]34).replACe('8r3',[STRiNg][CHAr]95).replACe('fj7',[STRiNg][CHAr]36) |& ( $VErboSEPReFErencE.TostRIng()[1,3]+'x'-Join'')

```
と書かれている。ところどころ読めるが、全体的に意味がわからないためいろいろ調べてみると、どうやらpowershellの難読化という処理がされているっぽいため、元のコードを復号して読む必要があるっぽい。

眺めてみると、world of PowerSheなど実行時に出てきた文字やreplACeやelseなどプログラムっぽいこと、W4FLAGなどとも書かれてあり、ここにフラグが隠れていそうだと考えられる。

一番上を見てみると、{39}{4}{12}{...と数字が並んでいる。フラグっぽい文字列とこの数字から、文章がばらばらに書かれており、並び替えることで元の文章が出てくるのではないかと考えられた。

眺めてみると、world of PowerSheなど実行時に出てきた文字やreplACeやelseなどプログラムっぽいこと、W4FLAGなどとも書かれてあり、ここにフラグが隠れていそうだと考えられる。"-f ' world of PowerS...と続く文章は'と,で区切られているため、頭から39番目、4番目、12番目と並び替えればよいと考えられる。
とりあえずコードを書いて並び替える
コード
```
str1 = "{39}{4}{12}{45}{21}{0}{36}{25}{26}{27}{7}{13}{30}{16}{31}{48}{23}{18}{19}{20}{24}{28}{3}{38}{11}{5}{2}{8}{46}{34}{29}{1}{35}{15}{10}{33}{9}{32}{22}{37}{40}{6}{43}{17}{47}{44}{14}{41}{42}"
str1 = str1.replace('{', '')
s = str1.replace('}', ',')
list2 = s.split(',')
del list2[-1]

for i in range(len(list2)):
    list2[i] = int(list2[i])

list1 = [' world of PowerShe','d_p','cl','d3','ch','1n_','else','ost cW4Passwo','34r1n68r30b','{  Writ','l}','_','o ','r','W4Incor','w3r5h3l','W',' ','t ','-eq c','W4FLAG{','he','t c','(fj7inpu','y0u_','fj7input =',' ','Read-H','5ucc33','473','dc','4','e-Outpu','cW4) ','u5c','0','ll!cW4','W4Co','d','e','rrect!cW4} ','rec','tcW4}',' {','tput c','cW4Welcome to t','f',' Write-Ou','if ']


for i in list2:
    print(list1[i], end="")
    
```

出力はこうなる
> echo cW4Welcome to the world of PowerShell!cW4fj7input = Read-Host cW4PasswordcW4if (fj7input -eq cW4FLAG{y0u_5ucc33d3d_1n_cl34r1n68r30bfu5c473d_p0w3r5h3ll}cW4) {  Write-Output cW4Correct!cW4} else {  Write-Output cW4IncorrectcW4}

見てみるとcW4FLAG{y0u_5ucc33d3d_1n_cl34r1n68r30bfu5c473d_p0w3r5h3ll}といういかにもフラグっぽいものがある。
これをパスワードに入力してみる。最初のcW4は多分いらないため、省く。
```
PS /rev-pwsh> ./pwsh.ps1
Welcome to the world of PowerShell!
Password: FLAG{y0u_5ucc33d3d_1n_cl34r1n68r30bfu5c473d_p0w3r5h3ll}
Incorrect
```
ダメっぽい。
一番はじめのプログラムに戻りreplACeとあったのを思い出す。調べてみるとどうやらpowershellで文字列の置換を
行うプログラムらしい。replACeの処理を見てみると、
```
.replACe('cW4',[STRiNg][CHAr]34)
.replACe('8r3',[STRiNg][CHAr]95)
.replACe('fj7',[STRiNg][CHAr]36)
```
となっている。
cW4を何かに置換しているのはわかるが、何に置換しているかわからないため、実際にpowershellで動かしてみる。
```
PS C:\Users> [string][char]34
"
```
ダブルクォートが出てきた。同様に残りの二つもやると、_と$が出てきた。どうやらここではASCII文字を変換しているっぽい。
まとめると、cW4->", 8r3->_, fj7->$と変換されるらしい。このルールをもとに先ほどのフラグを見直す。
先ほどのフラグがこれ
> FLAG{y0u_5ucc33d3d_1n_cl34r1n68r30bfu5c473d_p0w3r5h3ll}

ルール通りに直す
> FLAG{y0u_5ucc33d3d_1n_cl34r1n6_0bfu5c473d_p0w3r5h3ll}

これを入力してみる。
```
PS /rev-pwsh> ./pwsh.ps1
Welcome to the world of PowerShell!
Password: FLAG{y0u_5ucc33d3d_1n_cl34r1n6_0bfu5c473d_p0w3r5h3ll}
Correct!
```
どうやらOKらしい。これをsubmitしてフラグを得る。