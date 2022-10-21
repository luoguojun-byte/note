## MD5

```
题目 e00cf25ad42683b3df678c61f42c6bda
网址https://www.cmd5.com/
flag{admin1}
```

## URL编码

```
题目 %66%6c%61%67%7b%61%6e%64%20%31%3d%31%7d
网址https://tool.chinaz.com/tools/urlencode.aspx
flag{and 1=1}
```

## 一眼就解密

```
题目 ZmxhZ3tUSEVfRkxBR19PRl9USElTX1NUUklOR30=
网址https://www.cmd5.com/
flag{THE_FLAG_OF_THIS_STRING}
```

## 看我回旋踢

```
题目 synt{5pq1004q-86n5-46q8-o720-oro5on0417r1}
网址http://www.atoolbox.net/Tool.php?Id=778，每次解密完成后复制再解密直到出现flag
flag{5cd1004d-86a5-46d8-b720-beb5ba0417e1}
```

## 摩丝

```
题目 .. .-.. --- ...- . -.-- --- ..-
对照摩丝图
flag{ILOVEYOU}
```

## password

```
题目 姓名：张三 
生日：19900315
key格式为key{xxxxxxxxxx}
这个题的格式为 flag{名字首字母小写+生日}
flag{zs19900315}
```

## 变异凯撒

```
题目 加密密文：afZ_r9VYfScOeO_UL^RWUc
格式：flag{ }

对照ASSIC表现吧afZ_转换成flag{，发现规律为5，6，7...，逐步增长
flag{Caesar_variation}
```

## Quoted-printable编码

```
题目 =E9=82=A3=E4=BD=A0=E4=B9=9F=E5=BE=88=E6=A3=92=E5=93=A6
网址 http://web.chacuo.net/charsetquotedprintable
flag{那你也很棒哦}
```

## Rabbit

```
题目 U2FsdGVkX1/+ydnDPowGbjjJXhZxm2MP2AgI
网址 http://www.jsons.cn/rabbitencrypt/
flag{Cute_Rabbit}
```

## 篱笆墙的影子（阑珊密码）

```
题目 felhaagv{ewtehtehfilnakgw}

解题过程，将felhaagv{ewtehtehfilnakgw}拆分，然后在合并
flag{wethinkw
ehavetheflag}

flag{wethinkwehavetheflag}
```

## RSA

```
题目 在一次RSA密钥对生成中，假设p=473398607161，q=4511491，e=17，求解出d作为flga提交
过程
https://www.lfd.uci.edu/~gohlke/pythonlibs/#pygame 下载对应python的包
pip3 install rsa和gmpy2....
import gmpy2
import rsa

e = 17
p = 473398607161
q = 4511491
phin = (q-1)*(p-1)
d = gmpy2.invert(e, phin)
print(d)
flag{125631357777427553}
```

## 丢失的MD5

```
题目 
import hashlib   
for i in range(32,127):
    for j in range(32,127):
        for k in range(32,127):
            m=hashlib.md5()
            m.update('TASC'+chr(i)+'O3RJMV'+chr(j)+'WDJKX'+chr(k)+'ZM')
            des=m.hexdigest()
            if 'e9032' in des and 'da' in des and '911513' in des:
                print des
                
解题过程 在进行md5哈希运算前，需要对数据进行编码，原题没有指定编码格式
import hashlib   
for i in range(32,127):
    for j in range(32,127):
        for k in range(32,127):
            m=hashlib.md5()
            upwd='TASC'+chr(i)+'O3RJMV'+chr(j)+'WDJKX'+chr(k)+'ZM'
            m.update(upwd.encode("utf8"))
            des=m.hexdigest()
            if 'e9032' in des and 'da' in des and '911513' in des:
                print (des)
flag{e9032994dabac08080091151380478a2}
```

## Alice与Bob

```
题目 下面是一个大整数:98554799767,请分解为两个素数，分解后，小的放前面，大的放后面，合成一个新的数字，进行md5的32位小写哈希
101999 · 966233
拆分网址 http://www.factordb.com/index.php?query=98554799767
flag{d450209323a847c8d01c6be47c81811a}
```

## rsarsa

```
题目给出 pqe
解题过程
m=pow(c,d,n)
返回 c 的 d 次方取余 n（等同于 (c^d) % n）：
import gmpy2
import rsa

e = xxx
p = xxx				
q = xxx
c= xxx
n=p*q
phin = (q-1)*(p-1)
d = gmpy2.invert(e, phin)
m=pow(c,d,n)
print(m)

flag{5577446633554466577768879988}
```

## 大帝的密码武器

```
题目 给出 FRPHEVGL解密
过程，下列代码，解出单词为security，偏移量是13，将密文ComeChina偏移量为13，解密
str1 = 'FRPHEVGL'
str2 = str1.lower()                                 #转换为小写方便识别
num = 1                                             #偏移量
for i in range(26):
    print("{:<2d}".format(num),end = ' ')
    for temp in str2:
        if(ord(temp)+num > ord('z')):               #如果超出'z',需要重新映射会a~z这26个字母上
            print(chr(ord(temp)+num-26),end = '')
        else:
            print(chr(ord(temp)+num),end = '')
    num += 1
    print('')
flag{PbzrPuvan}
```

## Windows 系统密码

```
题目
Administrator:500:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
ctf:1002:06af9108f2e1fecf144e2e8adef09efd:a7fcb22a88038f35a8f39d503e7f0062:::
Guest:501:aad3b435b51404eeaad3b435b51404ee:31d6cfe0d16ae931b73c59d7e0c089c0:::
SUPPORT_388945a0:1001:aad3b435b51404eeaad3b435b51404ee:bef14eee40dffbc345eeb3f58e290d56:::
过程
使用 MD5将：后面的一窜数字加字母一个一个复制上去
flag{good-luck}
```

## 传统知识+古典密码

```
题目 
小明某一天收到一封密信，信中写了几个不同的年份
          辛卯，癸巳，丙戌，辛未，庚辰，癸酉，己卯，癸巳。
          信的背面还写有“+甲子”，请解出这段密文。
过程
对照60甲子顺序表然后题目要求“+甲子”再加上60，，拿到对应的ASSCI码，XZSDMFLZ，然后进行凯撒解密然后是阑珊解密
str1 = 'XZSDMFLZ'
str2 = str1.lower()                                 #转换为小写方便识别
num = 1                                             #偏移量
for i in range(26):
    print("{:<2d}".format(num),end = ' ')
    for temp in str2:
        if(ord(temp)+num > ord('z')):               #如果超出'z',需要重新映射会a~z这26个字母上
            print(chr(ord(temp)+num-26),end = '')
        else:
            print(chr(ord(temp)+num),end = '')
    num += 1
    print('')
    
拿到凯撒密码：SUNYHAGU
栅栏密码1栏  SNHG      SNHGUYAU
            NYAU
栅栏密码2栏  SHUA      SHNANGYU
            NGYU     
```

## 信息化时代的步伐

```
题目
606046152623600817831216121621196386
网址中文电码 http://code.mcdvisa.com/
flag{计算节要从娃娃抓起}
```

## 凯撒？替换？呵呵！

```
题目
MTHJ{CUBCGXGUGXWREXIPOYAOEYFIGXWRXCHTKHFCOHCFDUCGTXZOHIXOEOWMEHZO}
过程
在网址https://quipqiup.com/输入 MTHJ=flag
flag{substitutioncipherdecryptionisalwayseasyjustlikeapieceofcake}
```

## old-fashion

```
题目
Os drnuzearyuwn, y jtkjzoztzoes douwlr oj y ilzwex eq lsdexosa kn pwodw tsozj eq ufyoszlbz yrl rlufydlx pozw douwlrzlbz, ydderxosa ze y rlatfyr jnjzli; mjy gfbmw vla xy wbfnsy symmyew (mjy vrwm qrvvrf), hlbew rd symmyew, mebhsymw rd symmyew, vbomgeyw rd mjy lxrzy, lfk wr dremj. Mjy eyqybzye kyqbhjyew mjy myom xa hyedrevbfn lf bfzyewy wgxwmbmgmbrf. Wr mjy dsln bw f1_2jyf-k3_jg1-vb-vl_l
网址 https://quipqiup.com/
flag{n1_2hen-d3_hu1-mi-ma_a}
```

## 权限第一步

```
题目
Administrator:500:806EDC27AA52E314AAD3B435B51404EE:F4AD50F57683D4260DFD48AA351A17A8:::
网址 https://www.cmd5.com/
flag{3617656}
```

## 萌萌哒的猪八戒

```
对照猪圈密码表
flag{whenthepigwanttoeat}
```

## 异性相吸

```
题目
ἇ̀Ј唒ဃ塔屋䩘卖剄䐃堂ن䝔嘅均ቄ䩝ᬔ
key asadsasdasdasdasdasdasdasdasdasdqwesqf
```

## RSA1

```
题目
p=xxx
q=xxx
dp=xxx
dq=xxx
c=xxx
n=p*q
过程
m1≡c^{dq }mod q
m2≡c^{dp} mod p
m≡(((m2−m1)∗p−1 mod q)p+m1) mod n

import libnum
def egcd(a, b):
    if (b == 0):
        return 1, 0, a
    else:
        x, y, q = egcd(b, a % b)  # q = GCD(a, b) = GCD(b, a%b)
        x, y = y, (x - (a // b) * y)
        return x, y, q


def mod_inv(a, b):
    return egcd(a, b)[0] % b  # 求a模b得逆

invq=mod_inv(p,q)
mp=pow(c,dp,p)
mq=pow(c,dq,q)
m=((mp-mq)*invq%p)*q+mq
print(libnum.n2s(m))

flag{W31c0m3_70_Ch1n470wn}
```

## Unencode取消编码

```

```

