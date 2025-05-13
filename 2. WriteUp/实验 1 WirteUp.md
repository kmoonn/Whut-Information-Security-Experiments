### 1. 编程实现 RSA 算法 

### 2. 编写 Base64 加解密函数 

### 3. 提取图片中隐写的密文

'''python
# base64 = "flag{VGVhY2hlciBaaGFvJ3MgY2xhc3MgaXMgcmVhbGx5IGludGVyZXN0aW5n77yB=}"
#
# for i in base64:
#     print(str(hex(ord(i)))[2:],end='')

ascii = "666c61677b564756685932686c63694261614746764a334d675932786863334d6761584d67636d56686247783549476c75644756795a584e306157356e373779423d7d"

for i in range(0, len(ascii), 2):
    # print(ascii[i:i + 2], end=' ')
    print(chr(int(ascii[i:i + 2], 16)), end='')