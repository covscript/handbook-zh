# 3.6 covscript-gmssl - 国密算法支持

`covscript-gmssl` 提供了国密（GM/T）加密算法的支持，包括 SM2、SM3、SM4 等算法。

### 安装

```bash
cspkg install gmssl
```

### SM3 哈希算法

SM3 是中国国家密码管理局发布的密码杂凑算法。

```covscript
import gmssl

# 计算 SM3 哈希（返回字节数组）
var text = "Hello, CovScript!"
var hash_bytes = gmssl.sm3(gmssl.bytes_encode(text))
var hash_hex = gmssl.hex_encode(hash_bytes)
system.out.println("SM3 哈希: " + hash_hex)

# SM3 HMAC
var key = "secret_key"
var hmac = gmssl.sm3_hmac(gmssl.bytes_encode(key), gmssl.bytes_encode(text))
system.out.println("SM3 HMAC: " + gmssl.hex_encode(hmac))

# SM3 PBKDF2 - 基于密码的密钥派生
# 安全提示：实际应用中应为每个密码生成唯一的随机 salt
# 并使用较高的迭代次数（如 100000 及以上），以防止暴力破解
var password = "my_password"
var salt = gmssl.rand_bytes(16)  # 生成 16 字节的随机 salt
var iterations = 100000  # 推荐使用 100,000 次及以上迭代
var key_bytes = gmssl.sm3_pbkdf2(password, gmssl.bytes_decode(salt), iterations, gmssl.sm4_key_size)
system.out.println("派生密钥: " + gmssl.hex_encode(key_bytes))
system.out.println("Salt (需保存): " + gmssl.hex_encode(salt))
```

### SM4 对称加密

SM4 是一种分组密码算法，分组长度为 128 位，密钥长度为 128 位。

```covscript
import gmssl

# 生成密钥和初始化向量
var key = gmssl.rand_bytes(gmssl.sm4_key_size)
var iv = gmssl.rand_bytes(gmssl.sm4_key_size)

# 使用 CTR 模式加密
var plaintext = "机密信息"
var plaintext_encoded = gmssl.base64_encode(plaintext)
var ciphertext = gmssl.sm4(gmssl.sm4_mode.ctr_encrypt, key, iv, gmssl.bytes_encode(plaintext_encoded))

system.out.println("加密后: " + gmssl.hex_encode(ciphertext))

# 解密数据
var decrypted_bytes = gmssl.sm4(gmssl.sm4_mode.ctr_decrypt, key, iv, ciphertext)
var decrypted = gmssl.base64_decode(gmssl.bytes_decode(decrypted_bytes))
system.out.println("解密后: " + decrypted)

# 使用 CBC 模式加密
var ciphertext_cbc = gmssl.sm4(gmssl.sm4_mode.cbc_encrypt, key, iv, gmssl.bytes_encode(plaintext_encoded))
var decrypted_cbc = gmssl.base64_decode(gmssl.bytes_decode(gmssl.sm4(gmssl.sm4_mode.cbc_decrypt, key, iv, ciphertext_cbc)))
system.out.println("CBC 解密: " + decrypted_cbc)
```

### SM2 非对称加密

SM2 是基于椭圆曲线的公钥密码算法，包括数字签名、密钥交换和公钥加密。

```covscript
import gmssl

# 生成 SM2 密钥对
var keypair = gmssl.sm2_keygen()
var publicKey = keypair.public_key
var privateKey = keypair.private_key

# 将密钥保存为 PEM 格式
gmssl.sm2_pem_write("public.pem", gmssl.pem_name_pbk, publicKey)
gmssl.sm2_pem_write("private.pem", gmssl.pem_name_pvk, privateKey)

# 从 PEM 文件读取密钥
var loaded_pubkey = gmssl.sm2_pem_read("public.pem", gmssl.pem_name_pbk)
var loaded_privkey = gmssl.sm2_pem_read("private.pem", gmssl.pem_name_pvk)

# 使用公钥加密
var message = "重要消息"
var encrypted = gmssl.sm2_encrypt(loaded_pubkey, gmssl.bytes_encode(message))
system.out.println("加密结果: " + gmssl.base64_encode(encrypted))

# 使用私钥解密（需要密码保护）
var keypass = "password"
var decrypted_bytes = gmssl.sm2_decrypt(loaded_privkey, keypass, encrypted)
var decrypted = gmssl.bytes_decode(decrypted_bytes)
system.out.println("解密结果: " + decrypted)

# 数字签名
var data = gmssl.bytes_encode("需要签名的数据")
var signature = gmssl.sm2_sign(loaded_privkey, keypass, data)
system.out.println("签名: " + gmssl.base64_encode(signature))

# 验证签名
var isValid = gmssl.sm2_verify(loaded_pubkey, data, signature)
if isValid
    system.out.println("签名验证成功")
else
    system.out.println("签名验证失败")
end
```

### 实际应用：基于国密的 TLS 实现

基于 `simple_tls.csp` 包的安全通信实现：

```covscript
import simple_tls
import network

# 服务器端
class TLSServer
    var server = null
    
    function start(pkey_path, vkey_path, keypass, addr, port)
        this.server = new simple_tls.server
        this.server.listen(pkey_path, vkey_path, keypass, addr, port)
        
        system.out.println("TLS 服务器启动")
        system.out.println("指纹: " + this.server.fingerprint())
        
        if this.server.accept()
            system.out.println("客户端已连接")
            return true
        else
            system.out.println("连接失败")
            return false
        end
    end
    
    function send(data)
        this.server.send(data)
    end
    
    function receive()
        return this.server.receive()
    end
    
    function close()
        this.server.close()
    end
end

# 客户端
class TLSClient
    var client = null
    
    function connect(addr, port, authorized_fingerprint)
        this.client = new simple_tls.client
        # 添加授权的服务器指纹
        this.client.authorized_keys.insert(authorized_fingerprint)
        
        if this.client.connect(addr, port)
            system.out.println("已连接到服务器")
            return true
        else
            system.out.println("连接失败")
            return false
        end
    end
    
    function send(data)
        this.client.send(data)
    end
    
    function receive()
        return this.client.receive()
    end
    
    function close()
        this.client.close()
    end
end

# 使用示例（服务器）
# 首先生成密钥对：
# var keypair = gmssl.sm2_keygen()
# gmssl.sm2_pem_write("server_pub.pem", gmssl.pem_name_pbk, keypair.public_key)
# gmssl.sm2_pem_write("server_priv.pem", gmssl.pem_name_pvk, keypair.private_key)

var server = new TLSServer{}
if server.start("server_pub.pem", "server_priv.pem", "password", "0.0.0.0", 8443)
    # 接收并回显消息
    var message = server.receive()
    system.out.println("收到: " + message)
    server.send("服务器收到: " + message)
    server.close()
end

# 使用示例（客户端）
var client = new TLSClient{}
var server_fingerprint = "..." # 从服务器获取的指纹
if client.connect("127.0.0.1", 8443, server_fingerprint)
    client.send("Hello, TLS Server!")
    var response = client.receive()
    system.out.println("响应: " + response)
    client.close()
end
```

### 编码工具函数

GMSSL 提供了多种编码/解码工具：

```covscript
import gmssl

# Base64 编码/解码
var text = "Hello, World!"
var encoded = gmssl.base64_encode(text)
system.out.println("Base64: " + encoded)
var decoded = gmssl.base64_decode(encoded)
system.out.println("解码: " + decoded)

# Hex 编码/解码
var data = gmssl.bytes_encode("test data")
var hex = gmssl.hex_encode(data)
system.out.println("Hex: " + hex)
var original = gmssl.hex_decode(hex)
system.out.println("原始: " + gmssl.bytes_decode(original))

# 字节与字符串转换
var str = "测试文本"
var bytes = gmssl.bytes_encode(str)
var str_back = gmssl.bytes_decode(bytes)
system.out.println("转换回: " + str_back)

# 生成随机字节
var random_bytes = gmssl.rand_bytes(16)
system.out.println("随机字节: " + gmssl.hex_encode(random_bytes))

# 使用种子生成字符
var seed = 2333
var random_chars = gmssl.rand_chars(16, seed)
system.out.println("随机字符: " + random_chars)
```

