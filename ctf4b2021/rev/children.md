# children
## Beginners_CTF_2021
## reversing
***

ダウンロードして実行するとプロセス番号を入力するよう言われるため、言われた通りプロセス番号を調べて入力していく。
まず発生したプロセス番号を監視する。
> watch --interval=2 "ps auxw"

次に別端末を開いてプログラムを実行する。後は指示通りにプロセス番号を入力する。
プロセス番号は先ほど監視を開始した端末から見ることが出来る。
下はプログラム実行画面
```
$ ./children
I will generate 10 child processes.
They also might generate additional child process.
Please tell me each process id in order to identify them!

Please give me my child pid!
11233
ok
Please give me my child pid!
11246
ok
Please give me my child pid!
11256
ok
Please give me my child pid!
11264
ok
Please give me my child pid!
11274
ok
Please give me my child pid!
11282
ok
Please give me my child pid!
11298
ok
Please give me my child pid!
11308
ok
Please give me my child pid!
11318
ok
Please give me my child pid!
11332
ok
How many children were born?
12
ctf4b{p0werfu1_tr4sing_t0015_15_usefu1}
```
プロセス監視画面
```
USER       PID %CPU %MEM    VSZ   RSS TTY      STAT START   TIME COMMAND
suzumy     462  0.0  0.0      0     0 ?        Z    17:39   0:00 [children] <defunct>
suzumy   12501  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12511  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12518  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12525  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12526  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12536  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12537  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12546  0.0  0.0      0     0 tty3     Z    22:57   0:00 [children] <defunct>
suzumy   12560  0.0  0.0      0     0 tty3     Z    22:58   0:00 [children] <defunct>
suzumy   12567  0.0  0.0      0     0 tty3     Z    22:58   0:00 [children] <defunct>
suzumy   12577  0.0  0.0      0     0 tty3     Z    22:58   0:00 [children] <defunct>
```
最後に生成されているプロセスは12個のため、最後に12と入力して終了