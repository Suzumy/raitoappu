# keygenme-py
## reversing
## picoCTF2021
***

初めからpythonのソースコードが与えられる。  
コードを見ていると、check_keyという怪しげな関数があるため、
これを注意して見てみる。  

プログラムを見てみると、どうやら入力された値と本来の正しい値を比較する関数っぽい。  
コードの中に、
> if key[i] != hashlib.sha256(username_trial).hexdigest()[4]:

と書かれた部分がある。username_trialには"SCHOFIELD"が入っているため、このコードは
SCHOFIELDをハッシュ化した値と、keyの特定の場所を比較するコードとなる。  

SCHOFIELDで生成したハッシュは
>66b8e54339dc8b7ff42eb6ea4d95561c67ad1192de710700b57c98e47c8af4b5

次にこれの4番目をとるため、ここで比較される正しい値は'e'ということがわかる。  

これを繰り返すコードをいかに示す
```
import hashlib

username_trial = b"SCHOFIELD"
flag = "picoCTF{1n_7h3_|<3y_of_"
flag += hashlib.sha256(username_trial).hexdigest()[4] + \
        hashlib.sha256(username_trial).hexdigest()[5] + \
        hashlib.sha256(username_trial).hexdigest()[3] + \
        hashlib.sha256(username_trial).hexdigest()[6] + \
        hashlib.sha256(username_trial).hexdigest()[2] + \
        hashlib.sha256(username_trial).hexdigest()[7] + \
        hashlib.sha256(username_trial).hexdigest()[1] + \
        hashlib.sha256(username_trial).hexdigest()[8]
flag += "}"
print(flag)
```

これを実行するとフラグがでるため、これをsubmitする。