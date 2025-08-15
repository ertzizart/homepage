+++
title = '通过PGP保护自己的Git记录与邮件'
date = 2024-09-01T02:07:00+08:00
article_author = "Hao Wu [ Ertzizart ]"
preface = ""
preface-author = ""
license = "CC BY-NC-SA 4.0"
license_url = "https://creativecommons.org/licenses/by-nc-sa/4.0/"
novel = false
draft = false
+++

PGP（Pretty Good Privacy）是由Philip R. Zimmermann于1991年6月发布的一款加密软件。在PGP诞生之前的1991年1月24日，美国当局参议院下彼时还是个议员的拜登提出了著名的S.266《综合反恐法案》，其中提到了所有加密软件都必须为政府提供后门以便读取任何用户的信息，不过在学界激烈的声讨之下S.266法案最终未能通过。站在今天的角度回看过去，便会发现S.266法案仅仅只是个开始而已。

1993年2月，Zimmermann被FBI指控违反《武器出口管制法（Arms Export Control Act，AECA）》，因其开发的PGP加密软件的加密强度超过规定的40位密钥，被认定为受到管制的军用技术。但Zimmermann十分滑头，在当局对其进行刑事调查期间将PGP的源代码编成书籍并委托麻省理工学院出版社发行并绕过管制作成图书分发到世界各地。由于各种复杂的情况加上书籍被视为作者的言论且美国宪法第一修正案声明保护言论自由，直到FBI意识到问题的严重性时PGP的源代码已经遍布海外，最终无奈撤销了指控。

Zimmermann在FBI指控撤销后创立了PGP公司，并且在1998年与互联网工程任务组（IETF）一起推出了OpenPGP标准，现在PGP也多是单指OpenPGP标准。

接下来会用到一款名为GnuPG（简称GPG）的开源加密软件，这是由Werner Koch于1999年9月7日发布的遵循OpenPGP标准实现而来的加密软件，目前是自由软件基金会的一部分，也是时下最流行的PGP加密工具之一，除某些特殊的Linux发行版之外其他所有发行版均预装了该软件。在Windows平台上也有Gpg4win之类的带界面操作的GPG移植版，但在此不作赘述，Windows用户可以安装并借用Git软件([https://git-scm.com/downloads](https://git-scm.com/downloads))自带的MinGW环境通过Bash进行后续操作。

首先使用`gpg --full-gen-key`生成主密钥，回车后会按序出现加密方案选择、密钥长度设定、密钥过期时间设定、名字邮箱设定和密码设定，根据不同的需求有不同的设定方案，全看各自取舍。

以下是在Arch Linux的生成记录：

```bash
gpg --full-gen-key
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Please select what kind of key you want:
   (1) RSA and RSA
   (2) DSA and Elgamal
   (3) DSA (sign only)
   (4) RSA (sign only)
   (9) ECC (sign and encrypt) *default*
  (10) ECC (sign only)
  (14) Existing key from card
Your selection? 1 # 这里根据需要选择加密技术，我常用第一项，具体看情况选择

RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096 # 加密长度，数字越大越难破解，但使用时更吃性能

Requested keysize is 4096 bits

Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0) 0 # 设定何时过期，0为永不过期
Key does not expire at all

Is this correct? (y/N) y

GnuPG needs to construct a user ID to identify your key.

Real name: Xiao Ming # 姓名，可以用任何你想用的真名假名绰号代号
Email address: xiaoming@email.com # 电子邮箱，如果用来给Git签名，建议用Git设定的同一邮箱，除此以外随意
Comment:
You selected this USER-ID:
    "Xiao Ming <xiaoming@email.com>"

Change (N)ame, (C)omment, (E)mail or (O)kay/(Q)uit? o # 确认无误输入“o”并回车确认

We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

gpg: directory '/home/ming/.gnupg/openpgp-revocs.d' created
gpg: revocation certificate stored as '/home/ming/.gnupg/openpgp-revocs.d/6B6C2FACF234F2F2F37F2602A66DAC863D954000.rev'
public and secret key created and signed.

pub   rsa4096 2024-09-01 [SC]
      6B6C2FACF234F2F2F37F2602A66DAC863D954000
uid                      Xiao Ming <xiaoming@email.com>
sub   rsa4096 2024-09-01 [E]
```

以上操作记录为生成主密钥，同时也会自动生成一个用于加密的子密钥，现在还需要再创建一个用于签名的子密钥：

```bash
gpg --edit-key 6B6C2FACF234F2F2F37F2602A66DAC863D954000
gpg (GnuPG) 2.4.5; Copyright (C) 2024 g10 Code GmbH
This is free software: you are free to change and redistribute it.
There is NO WARRANTY, to the extent permitted by law.

Secret key is available.

gpg: checking the trustdb
gpg: marginals needed: 3  completes needed: 1  trust model: pgp
gpg: depth: 0  valid:   2  signed:   0  trust: 0-, 0q, 0n, 0m, 0f, 2u
sec  rsa4096/6B6C2FACF234F2F2
     created: 2024-09-01  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/F2F37F2602A66DA8
     created: 2024-09-01  expires: never       usage: E
[ultimate] (1). Xiao Ming <xiaoming@email.com>

gpg> addkey
Please select what kind of key you want:
   (3) DSA (sign only)
   (4) RSA (sign only)
   (5) Elgamal (encrypt only)
   (6) RSA (encrypt only)
  (10) ECC (sign only)
  (12) ECC (encrypt only)
  (14) Existing key from card
Your selection? 4
RSA keys may be between 1024 and 4096 bits long.
What keysize do you want? (3072) 4096
Requested keysize is 4096 bits
Please specify how long the key should be valid.
         0 = key does not expire
      <n>  = key expires in n days
      <n>w = key expires in n weeks
      <n>m = key expires in n months
      <n>y = key expires in n years
Key is valid for? (0)
Key does not expire at all
Is this correct? (y/N) y
Really create? (y/N) y
We need to generate a lot of random bytes. It is a good idea to perform
some other action (type on the keyboard, move the mouse, utilize the
disks) during the prime generation; this gives the random number
generator a better chance to gain enough entropy.

sec  rsa4096/6B6C2FACF234F2F2
     created: 2024-09-01  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb  rsa4096/F2F37F2602A66DA8
     created: 2024-09-01  expires: never       usage: E
ssb  rsa4096/85DF98DEA096A5CD
     created: 2024-09-01  expires: never       usage: S # 这就是刚刚创建的用于签名的子密钥
[ultimate] (1). Xiao Ming <xiaoming@email.com>

gpg> save # 不要忘记保存
```

所有密钥创建完成后首先生成吊销凭证并导出为revoke.key：

```bash
gpg --gen-revoke -ao revoke.key 6B6C2FACF234F2F2F37F2602A66DAC863D954000

sec  rsa4096/6B6C2FACF234F2F2 2024-09-01 Xiao Ming <xiaoming@email.com>

Create a revocation certificate for this key? (y/N) y
Please select the reason for the revocation:
  0 = No reason specified
  1 = Key has been compromised
  2 = Key is superseded
  3 = Key is no longer used
  Q = Cancel
(Probably you want to select 1 here)
Your decision? 3
Enter an optional description; end it with an empty line:
>
Reason for revocation: Key is no longer used
(No description given)
Is this okay? (y/N) y
Revocation certificate created.

Please move it to a medium which you can hide away; if Mallory gets
access to this certificate he can use it to make your key unusable.
It is smart to print this certificate and store it away, just in case
your media become unreadable.  But have some caution:  The print system of
your machine might store the data and make it available to others!
```

吊销凭证创建完成后查看一下主密钥和子密钥的id，并为下面导出公钥、主密钥、子密钥做准备：

```bash
gpg -k --fingerprint --keyid-format long
[keyboxd]
---------

pub   rsa4096/6B6C2FACF234F2F2 2024-09-01 [SC]
      Key fingerprint = 6b6C 2FAB BADC 22E1 F345  DF33 8E8E 3CAA B324 F2F2
uid                 [ultimate] Xiao Ming <xiaoming@email.com>
sub   rsa4096/F2F37F2602A66DA8 2024-09-01 [E]
sub   rsa4096/85DF98DEA096A5CD 2024-09-01 [S]

gpg -K --fingerprint --keyid-format long
[keyboxd]
---------

sec   rsa4096/6B6C2FACF234F2F2 2024-09-01 [SC]
      Key fingerprint = 6b6C 2FAB BADC 22E1 F345  DF33 8E8E 3CAA B324 F2F2
uid                 [ultimate] Xiao Ming <xiaoming@email.com>
ssb   rsa4096/F2F37F2602A66DA8 2024-09-01 [E]
ssb   rsa4096/85DF98DEA096A5CD 2024-09-01 [S]
```

导出（不要忘记感叹号后缀）：

```bash
gpg -ao public-key --export 6B6C2FACF234F2F2F37F2602A66DAC863D954000
gpg -ao secret-key --export-secret-key 6B6C2FACF234F2F2!
gpg -ao encrypt-subkey --export-secret-subkeys F2F37F2602A66DA8!
gpg -ao sign-subkey --export-secret-subkeys 85DF98DEA096A5CD!
```

目前应该已经导出了这些文件：

```bash
-rw------- 1 ming ming 6748 Sep 1 03:48 encrypt-subkey
-rw-r--r-- 1 ming ming 5377 Sep 1 03:47 public-key
-rw------- 1 ming ming  893 Sep 1 03:40 revoke.key
-rw------- 1 ming ming 3453 Sep 1 03:48 secret-key
-rw------- 1 ming ming 4516 Sep 1 03:49 sign-subkey
```

当某个子密钥泄露后及时吊销泄露的子密钥并签发一个新的子密钥并重新公开公钥即可，以下是吊销操作：

```bash
gpg --edit-key 6B6C2FACF234F2F2F37F2602A66DAC863D954000  
  
gpg> list  # 列出所有子密钥

sec  rsa4096/6B6C2FACF234F2F2
     created: 2024-09-01  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb rsa4096/F2F37F2602A66DA8
     created: 2024-09-01  expires: never       usage: E
ssb  rsa4096/85DF98DEA096A5CD
     created: 2024-09-01  expires: never       usage: S
[ultimate] (1). Xiao Ming <xiaoming@email.com>

gpg> key 1  # 选择你要吊销的子密钥序号，被选中的子密钥会有星号

sec  rsa4096/6B6C2FACF234F2F2
     created: 2024-09-01  expires: never       usage: SC
     trust: ultimate      validity: ultimate
ssb* rsa4096/F2F37F2602A66DA8
     created: 2024-09-01  expires: never       usage: E
ssb  rsa4096/85DF98DEA096A5CD
     created: 2024-09-01  expires: never       usage: S
[ultimate] (1). Xiao Ming <xiaoming@email.com>
gpg> revkey
gpg> save # 不要忘记保存
```

如果十分不幸主密钥失窃或泄漏，只能通过导入吊销凭证将其废除。当然，无论主密钥还是子密钥失窃都需要将新的公钥发给公钥服务器，下面是本地吊销与重新将新的公钥发送到公钥服务器的两条命令：

```bash
gpg --import revoke.key
gpg --keyserver hkps://keys.openpgp.org --send-keys 6B6C2FACF234F2F2F37F2602A66DAC863D954000
```
如果之前没有上传到公钥服务器，仅在熟人范围内交换过公钥，则直接把新的公钥发给之前拥有你的公钥的人即可，无论是手写抄到本子上还是直接快讯发送都行。

接下来便可将拥有签名功能的子密钥导入Git，主要给提交信息签名。众所周知Git Commit所需的user.name和user.email可以随便填，导致仿冒他人提交代码的事件曾出现过，为了防止这种情况发生，必须对自己的所有提交签名。

下列命令是配置Git全局设置签名：

```bash
git config --global user.signingkey 85DF98DEA096A5CD!
```

到提交代码的时候即可使用`git commit -S`为提交记录增加签名和署名，我使用的Arch Linux则要执行`git commit -sS`。

具体效果如下：

```bash
ming@~/workspace/project > git log --show-signature -1
commit 5c3386cf54bba0a22a32da706aa52bc0155503c2 (HEAD -> trunk, origin/trunk)
gpg: Signature made Sun 1 Sep 2024 01:57:13 AM CST
gpg:                using RSA key 6B6C2FACF234F2F2F37F2602A66DAC863D954000
gpg: Good signature from "Xiao Ming <xiaoming@email.com>" [ultimate]
Author: Xiao Ming <xiaoming@email.com>
Date:   Sun Sep 1 01:57:13 2024 +0800

    Init commit

    Signed-off-by: Xiao Ming <xiaoming@email.com>
```

最后就是保护邮件内容，在邮件交流时你和对方需要互换公钥。内容均使用公钥加密，私钥解密。对邮件自动加密是很多客户端的自带功能，在此不作赘述，仅演示手动对文件内容加密：

```bash
gpg -e -o  output.txt -r 6B6C2FACF234F2F2F37F2602A66DAC863D954000 input.txt # 加密文件并作为附件发送
gpg -d output.txt -o content.txt # 解密
gpg -d output.txt # 直接查看

echo "hello" | gpg --encrypt --armor -r 6B6C2FACF234F2F2F37F2602A66DAC863D954000 >> encrypted_message # 直接加密纯文本，生成的加密文本完整内容复制到邮件正文中即可
-----BEGIN PGP MESSAGE-----

hQIMA2rPRx1meROYAQ/8CnKHVgQgPhNTGJb7zc0nQpFrHemUWMCTKb+wjJxlXqqX
helJMzuChVDg8egR92wr4obKtXyylYRqWDMJLp1Excezyv53OMQN5bHkQ4B3JMVZ
rknolXJfJItSc5aSBvMC23hvtHgeMO8aVp+FFxOIj0tO2MLXOFqlSqfDzZnXTrhB
zC6yivX63++x0GfFfYFTE+mM3fsviUWKQ2X5/qKOe6GytONdpxI+KUIySaQCrBF4
kNWxxAcbF9b7tdpIZI+bIHU4NFm1R5Sqhi/m2ZOoQz4QCDFK0wNB8xnGlX0aeTI/
JwnYJZAhB6lACXuKBYUst3fBF2MTSGXgn02mmF62ZqtuH46lwzb/MUcgofNpB05j
rboBTOmkpEZhNlYIB5BLmrm5WObPSMNIVGBMHhsDekkM6mUA49+NJWFeJasNrIQ6
568sP9ddyItbZQI6ygNDoxLpHcbKhCZw7+0DxnETpnt2EgtQ9j3UGS9jFLbju8Fi
QKFDJRLDN95v9wKiGxZ/g628NuxgfN7ZulYbc9064FonyEg1IficbhvHwfuhE/bV
lvNaF3fCq/NIEdXxyA0mH9B9eOHTqI7+uZfcnDPyb938Kub4DaU+ilj2X3lGjmiN
ZKZ6BxNRaKpIqyKfgJwu0T0OyMmfDpprd+WmcPA3dqCD4pfFI4XZAziqUJnTy1jS
QQHADpUU0pe8daSUqEoHBAxbsBDRqsdN1/cw473YeT5/ohsWvhwdbHwZ4SWJjv4y
UwOrzeq1J9yINrtQh435rhd9
=cNzz
-----END PGP MESSAGE-----

cat encrypted_message | gpg # 解密encrypted_message文件中的密文
gpg: encrypted with rsa4096 key, ID F2F37F2602A66DA8, created 2024-09-01
      "Xiao Ming <xiaoming@email.com>"
hello # 成功解密
```

有时候可能会出现如下报错：

```bash
gpg: public key decryption failed: Inappropriate ioctl for device
gpg: decryption failed: Inappropriate ioctl for device
```

这里需要在`.zshrc`或`.bashrc`里加入`export GPG_TTY=$(tty)`

密钥需要始终处于安全位置，多准备几个备份，可以存在U盘、卡片、甚至手机里，而且所有导出的密钥凭证都需要分开放（特别是主密钥与吊销凭证），避免失窃泄漏时被一锅端。