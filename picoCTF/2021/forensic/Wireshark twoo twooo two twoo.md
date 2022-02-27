# Wireshark twoo twooo two twoo
## picoCTF2021
## Forensics
***

始めにpcapファイルが渡される。ipアドレスを見てみると、  
・8.8.8.8 - googleドメイン  
・18.217.1.57 - ?  
・169.254.169.254 - ローカルip  
・192.168.38.x - ローカルip  
こう見ると、18.217.1.57だけよくわからないところと通信していて怪しそうに見える。そのため、まずはipアドレスをこれに絞ってフィルタリングする。  
> ip.dst == 18.217.1.57
  
また、dnsのinfoの欄を見てみると、サブドメインが怪しげな文字列になっている。これを抜き出すために以下の条件を付与する。  
ついでに邪魔なamazonawsとかwindomainとかも除外した。  
> dns and ip.dst == 18.217.1.57 and !(dns.qry.name contains "amazonaws.com") and !(dns.qry.name contains "windomain.local")
  
これで抜き出されたパケットのサブドメインをみて、必要な部分をくっつけると、base64エンコードされた文字列が出てくるため、これをデコードしてsubmitする。