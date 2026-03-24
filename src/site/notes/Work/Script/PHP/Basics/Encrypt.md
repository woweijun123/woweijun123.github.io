---
{"dg-publish":true,"permalink":"/Work/Script/PHP/Basics/Encrypt/","title":"Encrypt","tags":["flashcards"],"noteIcon":"","created":"2025-03-18T17:39:39.767+08:00","updated":"2026-03-24T17:23:44.006+08:00"}
---

# 非对称加密
这种就是单项散列加密，单项散列加密是不可逆的。
## RSA
**RSA** 是一种**非对称加密算法**，它利用**大整数分解的数学难题**来确保安全。它使用一对密钥：**公钥**和**私钥**。
- **公钥加密**：任何人都可以使用你的公钥来加密数据。
- **私钥解密**：只有拥有相应私钥的人才能解密数据。
在实践中，由于 RSA 算法的**计算量大，效率较低**，它**很少**用于直**接加密大量数据**。它通常用于 **“密钥封装” (key wrapping)**，也就是用公钥加密一个对称密钥，然后用这个对称密钥去加密真正的数据。
### RSA 的限制
- **加密数据长度限制**：  
    RSA 加密的数据长度**不能超过密钥长度**（以字节为单位）减去 adding 的大小。例如 2048 位密钥（256 字节）的 RSA，使用 `RSA_PKCS1_OAEP_PADDING` 时，最大加密数据长度为 `256 - 42 = 214 字节`。
- **性能**：RSA 加密速度较慢，适合小数据（如加密对称密钥或签名）。
### 安全建议
- **密钥存储**：私钥应加密存储（如使用 `openssl_pkey_export` 的 `config` 参数指定加密密码）。
- **哈希算法**：优先使用 `sha256` 或更强的哈希算法（如 `sha512`）。
- **密钥长度**：RSA 至少 2048 位，ECC 至少 256 位（如 secp256k1）。
### 用途
生成 RSA 密钥对
```php
// RSA 示例
echo "=== RSA ===\n";
$rsaPrivateKey = null;
$rsaPublicKey  = null;
// 生成 RSA 密钥对（2048 位）
$bits   = 2048;
$config = [
    "digest_alg"       => "sha256",
    "private_key_bits" => $bits,
    "private_key_type" => OPENSSL_KEYTYPE_RSA,
];

// 创建密钥对
$res = openssl_pkey_new($config);

// 获取私钥和公钥
openssl_pkey_export($res, $privateKey);
$publicKey = openssl_pkey_get_details($res)['key'];

// 输出 PEM 格式的密钥
echo "Private Key:\n" . $privateKey . "\n";
echo "Public Key:\n" . $publicKey . "\n";
```
输出
```shell
=== RSA ===
Private Key:
-----BEGIN PRIVATE KEY-----
MIIEvQIBADANBgkqhkiG9w0BAQEFAASCBKcwggSjAgEAAoIBAQCWnVWhqnGCOkyr
MBVLvZ83CT9d+N0nmHGBvMh/V/zJ3IQnuAn4TITkS6lZfXFMC3co879PB2rVqq6l
76P1M4osolScQr4vQ+uBbTDhk7+sV2D+1a/V6NixP67x6BhhtJMqtOKRfZAy4pB0
QZXjaAyhpVQnvK+8pQM4kg3u1NDXrP0jVG6as/B99KZ3UgFY8hZcL925+Eus81IE
c0Mo11JDgKqz/zYe9CJs8ipUHGI0SSBvTyevn6VVss0P85qy54AYnS0l3aL/QoGe
oEVGsltdbLxEUM1Q0RlVd4HjulsbxRaDwwYkqZ210NhabFY58mMtLwaZXxAVV71m
uYqNkEkpAgMBAAECggEAAufxPYfM3d7jGNB6MLZtaoHuq5EAL2HlGsQ6OB7J/VkY
Ya3O33AWhlMhZt0hQP7dozgkwlEZ0hqTeRcpGjOO4HKXYFZ3VfEhC6PANDIGGjyr
Xe9gj6eI+s6IQRmh0szJpCLOVlFOZXTypZOYYUViLQJEH+onl+O1LrO+uhN4Hhks
SFz9y6WUDWY48aA0StSJtOCy6e2jLBZs2BLq5vWRsEU+gmxTZnPWS0/j9OnxetC/
vwfmV/yx68lnSYYZ5e5l0PcVbr4K1cGriDdqELK9wW1/o63t+VvsrLd03BmPTZz1
MmKffUrq/QDLb8rttv7q8ltVIGMAocJ5aHWKxpdLyQKBgQDPmCk3RfXWRy++PsWX
VHMoDDm5Ga6wutcZLi+xAtAtFDAuHoU68h2+MBfqe9IY+5qrqpohT1/HWwmIWiGO
aGEkRSUgnykenYcpeOXnSYL8X5BPEVaLHFAputQ26wZTFPNVDq8qkwVL2/BC51NL
eU4joUA+D0p0a5z0+dlEOFv8vQKBgQC5u+d2xYidaDaDohjUs9puezQ8l0Sf9iO6
AFIm+bJjrHFt+Tec4KrqbtIYz7TMDdEQF+KaFljJiQUqm9P8sh7BLknpIeOsNEfJ
ixDbsErp8MBfmU2JTvGx0+e0vbR0tUee+Js40J+BV4rblNVSB+CxEPz1VofMmmxM
idY0sU4i3QKBgB2tWg8N0FQ+vwOKg8Lbjs7l2IautjuLql5uLOE4TrxzC2Q8dr1z
iW77/x9FbkamCXmLHRev+jhMunMkt3FdWK3PuLwOJNm8mWsDXpKO/svHeaDkEKQ4
evlMPTRQqwnLj/HT9JS4ieRLX/Cgk1bR06riTYXRt8om7DxVT4siJ3xdAoGAVkPe
+rw+epWXlEXqcIhkcKIKngXIGt+wskhJ385ju4WxXVm+Kb/zwlTcgieempPkQSxG
1DiC3oAkhSjBKgH05NbB/2T9INNbcFGF7/OOp99pCj3i1F51RZndaYYe1YIJFN31
AktreiCV3uzes23zP2pbgvvAsRgcKuRuOCUN3IUCgYEAmHPj8kqRFtrz8vc16bwn
KZ5YJw7mME8nKVQwC3lLYaNGgFhwgMKx/woMknlNFPYwdsFWnFXEHaX49bIgFfhn
zd1kRalvKjvflnnP51OsX1LkSjy2x9iMLzcGX9TtasvbR61CFBTJBt9vg5fNKi8o
jAYviaq0iJclP8AoWCmC94k=
-----END PRIVATE KEY-----

Public Key:
-----BEGIN PUBLIC KEY-----
MIIBIjANBgkqhkiG9w0BAQEFAAOCAQ8AMIIBCgKCAQEAlp1VoapxgjpMqzAVS72f
Nwk/XfjdJ5hxgbzIf1f8ydyEJ7gJ+EyE5EupWX1xTAt3KPO/Twdq1aqupe+j9TOK
LKJUnEK+L0PrgW0w4ZO/rFdg/tWv1ejYsT+u8egYYbSTKrTikX2QMuKQdEGV42gM
oaVUJ7yvvKUDOJIN7tTQ16z9I1RumrPwffSmd1IBWPIWXC/dufhLrPNSBHNDKNdS
Q4Cqs/82HvQibPIqVBxiNEkgb08nr5+lVbLND/OasueAGJ0tJd2i/0KBnqBFRrJb
XWy8RFDNUNEZVXeB47pbG8UWg8MGJKmdtdDYWmxWOfJjLS8GmV8QFVe9ZrmKjZBJ
KQIDAQAB
-----END PUBLIC KEY-----
```
#### RSA 加密与解密
**SSL/TLS证书**：网站HTTPS通信中，RSA用于交换对称加密的会话密钥「证书加密」。
当你访问 `https://example.com` 时，服务器使用 RSA 公钥加密传输数据，你的浏览器用私钥解密。
```php
// 加密数据（使用公钥加密）
$data      = "Hello, RSA!";
$encrypted = '';
openssl_public_encrypt($data, $encrypted, $publicKey);
$encryptedBase64 = base64_encode($encrypted);
echo "Encrypted: " . $encryptedBase64 . PHP_EOL;

echo PHP_EOL;

// 解密数据（使用私钥解密）
$decrypted = '';
openssl_private_decrypt(base64_decode($encryptedBase64), $decrypted, $privateKey);
echo "Decrypted: " . $decrypted;
```
输出
```shell
=== RSA ===
Encrypted: eNme4JVOBAAyGBnEZSNSPsL+YJhvbqVzGY0WyonSL+pciBd940m8Rg/VY14MD9JTOzLTzAw9gcSs2VKBU636/0ZsnYCR7L2NjcJy1Z5upZijVN67ONs6Zlh20lK2Rn8lFe7FUeJ72EotvfKfLx38j6vxoNJJ7v2rrDMbK2YVbDgroaJCmMDl1SNdqY1tuhH3aFPAsmAk6dv9OSkGuxpBPIieb2Ksy8S6mDaubwkodCEhHbu7MuHwqGfOIPPRc8BDxNH4PmiFJwqAMVB+sVNWplcWRBan0bDiHSiyS+A8mf7j8hVIeiHGThpaYPCDSChqfz4aWZ4cwW6EHZEYGI4ebw==

Decrypted: Hello, RSA!
```
#### RSA 签名与验证
**数字签名**：用私钥对数据哈希值签名，公钥验证签名真实性（如软件更新包验证）「软件、邮件签名」。
GitHub 使用 RSA 签名验证代码提交的完整性。
```php
// 签名数据（使用私钥签名）
$signature = '';
openssl_sign($data, $signature, $privateKey, "sha256");
$signatureBase64 = base64_encode($signature);
echo "Signature: " . $signatureBase64 . PHP_EOL;

echo PHP_EOL;

// 验证签名（使用公钥）
$verified = openssl_verify($data, base64_decode($signatureBase64), $publicKey, "sha256");
echo "Verification Result: " . ($verified === 1 ? "Valid" : "Invalid") . PHP_EOL;
```
输出
```shell
=== RSA ===
Signature: ZKobg1x7PQf4NQGj+KGO0eIy5r0apOvDGVXa1oBCdSFuQkND1w3NbMP99o7KwMpkCy2SAv9EGOM8OfbtE0e/uZKXHbX7ijD5B6JwNTW//hCYJYkXYWu8sdKHX8+FZPgfbeREvOFq08yBODdojaXk+ZAeFcyLnz11GBr4xo3d83ar4a2PqAw+z3XwfIosBEKc/LrN+3A3foIVSvuXEjlrN7UO6mYsnDT3p9OpZYy2l+rDl2aBJYtrjm65j+pn/u8u+5UNLn7Et46XHIOeA4jF1qJ1sDDaxYH1+xEfrL2ezLU5EgBOetq2H1dvGUzrFTn6himZBa+0DqEj02HCzcszsg==

Verification Result: Valid
```
## ECC
### ECC 的限制
- **PHP 支持**：  
    ECC 在 PHP 中的实现依赖 OpenSSL 库版本，需确认服务器的 OpenSSL 版本支持**椭圆曲线**（如 secp256k1）。
- **加密方式**：  
    ECC 通常用于密钥交换（ECDH）或签名，**不直接用于加密大数据**。若需加密，需结合对称加密算法（如 AES）。
### 安全建议
- **密钥存储**：私钥应加密存储（如使用 `openssl_pkey_export` 的 `config` 参数指定加密密码）。
- **哈希算法**：优先使用 `sha256` 或更强的哈希算法（如 `sha512`）。
- **密钥长度**：ECC 至少 256 位（如 secp256k1）。
### 用途
生成 ECC 密钥对（以 secp256k1 曲线为例）
```php
echo "\n=== ECC ===\n";
$privateKey = null;
$publicKey  = null;
// 生成 ECC 密钥对（曲线：secp256k1）
$config = [
    "digest_alg"       => "sha256",
    "private_key_bits" => 384, // 将私钥长度增加到 384 位或更高
    "private_key_type" => OPENSSL_KEYTYPE_EC,
    "curve_name"       => "secp256k1", // 指定椭圆曲线
];

// 创建密钥对
$res = openssl_pkey_new($config);

// 获取私钥和公钥
openssl_pkey_export($res, $privateKey);
$publicKey = openssl_pkey_get_details($res)['key'];

// 输出 PEM 格式的密钥
echo "Private Key:\n" . $privateKey . "\n";
echo "Public Key:\n" . $publicKey . "\n";
```
输出
```
=== ECC ===
Private Key:
-----BEGIN PRIVATE KEY-----
MIGEAgEAMBAGByqGSM49AgEGBSuBBAAKBG0wawIBAQQgHTRFHMY5VMixSnitcaUi
NcFd71UH4C3ZY0SO+65v7mihRANCAAQuEkQgP0f1OIqSK3/wqZi9QB9NSdD+B2v1
H0No6uzLsreCgxbA2tfHmOmtKS9yYLkJYb/4mW5/yOhx7su/cBaM
-----END PRIVATE KEY-----

Public Key:
-----BEGIN PUBLIC KEY-----
MFYwEAYHKoZIzj0CAQYFK4EEAAoDQgAELhJEID9H9TiKkit/8KmYvUAfTUnQ/gdr
9R9DaOrsy7K3goMWwNrXx5jprSkvcmC5CWG/+Jluf8joce7Lv3AWjA==
-----END PUBLIC KEY-----
```
#### ECC 签名与验证
- **移动设备安全**：因密钥短、计算量小，适合资源受限的物联网设备
- **加密货币**：比特币地址生成中使用的ECDSA（椭圆曲线数字签名算法）
```php
// 签名数据（使用私钥签名）
$signature = '';
$data      = "Hello, ECC!";
openssl_sign($data, $signature, $privateKey, "sha256");
$signatureBase64 = base64_encode($signature);
echo "Signature: " . $signatureBase64 . PHP_EOL;
echo PHP_EOL;
// 验证签名（使用公钥）
$verified = openssl_verify($data, base64_decode($signatureBase64), $publicKey, "sha256");
echo "Verification Result: " . ($verified === 1 ? "Valid" : "Invalid") . PHP_EOL;
```
内容
```shell
=== ECC ===
Signature: MEQCIDz5JzI8w/X8weVox5+JSBa/GF5CeJbFP8UgZq7uhP+0AiAdBqMwro0136p/2NUM2r9csskuURYHKG7rJxJJ3WooyA==

Verification Result: Valid
```
## Password Hashing API 加密
password_hash是crypt的一个简单封装, 并且完全与现有的密码哈希兼容「php5.5引入」
**推荐用来加密密码**
### password_hash
创建密码的散列(hash)
```php
// 数据库里储存结果的列可超过60个字节(最好是255个字节)
echo password_hash('helloWorld', PASSWORD_DEFAULT);
// $2y$10$0Hw7gWzf4rClfPSSsqy5nuDnQNMPV.ikYHO6kRGgjOYy4hg8KzA72
```
### password_verify
验证密码是否和散列值匹配
```php
$hash = password_hash('helloWorld', PASSWORD_DEFAULT);
$password = 'helloWorld';
// 核验密码是否正确
echo password_verify($password, $hash) ? true : false;
```
### password_needs_rehash
给密码重新加密
```php
$password = 'rasmuslerdorf';
$hash = '$2y$10$YCFsG6elYca568hBi2pZ0.3LDL5wjgxct1N8w/oLR/jfHsiQwCqTS';

// 当硬件性能得到改善时，cost 参数可以再修改
$options = ['cost' => 11];

// 根据明文密码验证储存的散列
if (password_verify($password, $hash)) {
    // 检测是否有更新的可用散列算法
    // 或者 cost 发生变化
    if (password_needs_rehash($hash, PASSWORD_DEFAULT, $options)) {
        // 如果是这样，则创建新散列，替换旧散列
        $newHash = password_hash($password, PASSWORD_DEFAULT, $options);
    }

    // 使用户登录
}
```
### password_get_info
返回加密算法的名称和一些相关信息
```php
print_r(password_get_info('$2y$10$0Hw7gWzf4rClfPSSsqy5nuDnQNMPV.ikYHO6kRGgjOYy4hg8KzA72'));
/** 
Array
(
    [algo] => 1
    [algoName] => bcrypt
    [options] => Array
        (
            [cost] => 10
        )
) 
*/
```
## hash_hmac
使用 HMAC 方法生成带有密钥的散列值
### 参数介绍
`algo`
要使用的散列算法的名称（例如：`“sha256”`）
`data`
要进行散列运算的消息。
`key`
使用 HMAC 生成信息摘要时所使用的密钥。
`binary`
设置为 true 输出原始二进制数据，设置为 false 输出小写 16 进制字符串。
```php
echo hash_hmac('sha256', 'The quick brown fox jumped over the lazy dog.', 'secret'); // 9c5c42422b03f0ee32949920649445e417b2c634050833c5165704b825c2a53b
```
### 安全建议
- **密钥保密：** `secretKey`是客户端和服务器之间共享的秘密，**必须保密**，否则任何人都可以伪造 HMAC 值（只有拥有相同密钥的双方才能生成和验证正确的 HMAC 值）。
- **哈希算法：** `sha256` 是一种常用的哈希算法，您可以根据需要选择其他算法。
- **数据格式：** 请求数据或要验证的数据可以是任何格式，但必须在发送方和接收方之间达成一致。
- **错误处理：** 在实际应用中，您需要添加适当的错误处理机制。
### 用途
#### API 认证：请求合法验证
```php
// --- 客户端请求 ---
// 预共享密钥 (请务必保密)
$secretKey = "your_secret_key";
// 模拟 API 请求数据
$requestData = [
    "user_id" => 123,
    "timestamp" => time(),
    "data" => "some sensitive data"
];
// 将请求数据转换为 JSON 字符串 (或任何其他格式)
$requestString = json_encode($requestData);
// 计算 HMAC 值
$hmac = hash_hmac('sha256', $requestString, $secretKey);
// 将 HMAC 值添加到请求头 (或参数)
$headers = [
    'X-API-Signature: ' . $hmac,
    'Content-Type: application/json'
];
// 模拟发送 API 请求 (这里仅为演示，实际应用中需要使用 HTTP 客户端)
echo "Request Headers:\n";
print_r($headers);
echo "\nRequest Body:\n";
echo $requestString;

// --- 服务器端验证 ---
// 接收请求数据和 HMAC 值
$receivedRequestString = $requestString; // 假设从请求体中获取
$receivedHmac = $hmac; // 假设从请求头中获取
// 重新计算 HMAC 值
$calculatedHmac = hash_hmac('sha256', $receivedRequestString, $secretKey);
// 验证 HMAC 值
if ($receivedHmac === $calculatedHmac) {
    echo "\n\nAPI 请求认证成功！";
    // 处理 API 请求
} else {
    echo "\n\nAPI 请求认证失败！";
    // 返回错误响应
}
```
#### 数据完整性： 数据防篡改
```php
// 预共享密钥 (请务必保密)
$secretKey = "your_secret_key";
// 模拟数据
$data = "This is some important data.";
// 计算 HMAC 值
$hmac = hash_hmac('sha256', $data, $secretKey);

// --- 传输或存储数据和 HMAC ---
// 模拟接收数据和 HMAC 值
$receivedData = $data;
$receivedHmac = $hmac;
// 重新计算 HMAC 值
$calculatedHmac = hash_hmac('sha256', $receivedData, $secretKey);
// 验证 HMAC 值
if ($receivedHmac === $calculatedHmac) {
    echo "数据完整性验证成功！";
    // 使用数据
} else {
    echo "数据完整性验证失败！";
    // 处理错误
}
```
#### 用户认证： 令牌认证
```php
// 预共享密钥 (请务必保密)
$secretKey = "your_secret_key";
// 用户 ID
$userId = 123;
// 生成令牌
$timestamp = time();
$tokenPayload = $userId . ":" . $timestamp;
$tokenSignature = hash_hmac('sha256', $tokenPayload, $secretKey);
$token = base64_encode($tokenPayload . ":" . $tokenSignature);
// 输出令牌
echo "Generated Token: " . $token . "\n\n";

// --- 用户验证 ---
// 模拟接收令牌
$receivedToken = $token;
// 解码令牌
$decodedToken = base64_decode($receivedToken);
list($receivedUserId, $receivedTimestamp, $receivedSignature) = explode(":", $decodedToken);
// 验证令牌签名
$calculatedSignature = hash_hmac('sha256', $receivedUserId . ":" . $receivedTimestamp, $secretKey);
if ($receivedSignature === $calculatedSignature) {
    // 验证令牌时间 (例如，令牌有效期为 1 小时)
    if (time() - $receivedTimestamp <= 3600) {
        echo "Token verification successful! User ID: " . $receivedUserId;
        // 用户认证成功
    } else {
        echo "Token expired!";
        // 令牌过期
    }
} else {
    echo "Token verification failed!";
    // 令牌验证失败
}
```
#### 消息队列： 验证消息来源和完整性
```php
// 预共享密钥 (请务必保密)
$secretKey = "your_secret_key";
// 模拟消息
$message = [
    "type" => "order_created",
    "order_id" => 456,
    "user_id" => 789,
    "timestamp" => time()
];
// 将消息转换为 JSON 字符串
$messageString = json_encode($message);
// 计算 HMAC 值
$hmac = hash_hmac('sha256', $messageString, $secretKey);
// 将消息和 HMAC 值放入消息队列
$queueMessage = [
    "message" => $messageString,
    "signature" => $hmac
];
// 模拟将消息放入队列 (这里仅为演示)
echo "Message sent to queue:\n";
print_r($queueMessage);
echo "\n\n";

// --- 消息处理 ---
// 模拟从队列中获取消息
$receivedQueueMessage = $queueMessage;
// 提取消息和 HMAC 值
$receivedMessageString = $receivedQueueMessage["message"];
$receivedHmac = $receivedQueueMessage["signature"];
// 重新计算 HMAC 值
$calculatedHmac = hash_hmac('sha256', $receivedMessageString, $secretKey);
// 验证 HMAC 值
if ($receivedHmac === $calculatedHmac) {
    echo "Message integrity verification successful!\n";
    // 处理消息
    $receivedMessage = json_decode($receivedMessageString, true);
    print_r($receivedMessage);
} else {
    echo "Message integrity verification failed!";
    // 丢弃消息
}
```
## md5
### 计算字符串的MD5散列值
```php
echo md5('hello world'); // 5eb63bbbe01eeed093cb22bb8f5acdc3
// 返回32字节的十六进制表示的字符串。
echo strlen(md5('hello world', true)); // 16
// 返回16字节数据
```
### MD5 加密解密类
```php
class MyCrypt
{
    public static $defaultKey = '';
    /**
     * 字符加密，一次一密,可定时解密有效
     * @param string $string 原文
     * @param string $key 密钥
     * @param int $expiry 密文有效期,单位s,0 为永久有效
     * @return string 加密后的内容
     */
    public static function encode($string, $key = '', $expiry = 0)
    {
        [$keyC, $result] = self::interpreter($string, $key, $expiry);
        return str_replace(
            ['+', '/', '='], ['-', '_', '.'],
            $keyC . str_replace('=', '', base64_encode($result))
        );
    }
    /**
     * 字符解密，一次一密,可定时解密有效
     * @param string $string 密文
     * @param string $key 解密密钥
     * @return string 解密后的内容
     */
    public static function decode($string, $key = '')
    {
        [$keyB, $result] = self::interpreter(str_replace(['-', '_', '.'], ['+', '/', '='], $string), $key);
        if (
            (substr($result, 0, 10) == 0 || substr($result, 0, 10) - time() > 0)
            &&
            substr($result, 10, 16) == substr(md5(substr($result, 26) . $keyB), 0, 16)
        ) {
            return substr($result, 26);
        } else {
            return '';
        }
    }
    // 公共解释器
    public static function interpreter($string, $key, $expiry = 0): array
    {
        $cKeyLength = 4;
        $key = md5($key ? $key : self::$defaultKey); // 解密密匙
        $keyA = md5(substr($key, 0, 16)); //做数据完整性验证
        $keyB = md5(substr($key, 16, 16)); //用于变化生成的密文 (初始化向量IV)

        $prevFunc = debug_backtrace()['1']['function'];
        if ($prevFunc == 'encode') {
            $keyC = substr(md5(microtime()), -$cKeyLength);
            $string = sprintf('%010d', $expiry ? $expiry + time() : 0) . substr(md5($string . $keyB), 0, 16) . $string;
        } else {
            $keyC = substr($string, 0, $cKeyLength);
            $string = base64_decode(substr($string, $cKeyLength));
        }

        $cryptKey = $keyA . md5($keyA . $keyC);
        $keyLength = strlen($cryptKey);
        $stringLength = strlen($string);

        $rndKey = array();
        for ($i = 0; $i <= 255; $i++) {
            $rndKey[$i] = ord($cryptKey[$i % $keyLength]);
        }

        $box = range(0, 255);
        // 打乱密匙簿，增加随机性
        for ($j = $i = 0; $i < 256; $i++) {
            $j = ($j + $box[$i] + $rndKey[$i]) % 256;
            $tmp = $box[$i];
            $box[$i] = $box[$j];
            $box[$j] = $tmp;
        }
        // 加解密，从密匙簿得出密匙进行异或，再转成字符
        $result = '';
        for ($a = $j = $i = 0; $i < $stringLength; $i++) {
            $a = ($a + 1) % 256;
            $j = ($j + $box[$a]) % 256;
            $tmp = $box[$a];
            $box[$a] = $box[$j];
            $box[$j] = $tmp;
            $result .= chr(ord($string[$i]) ^ ($box[($box[$a] + $box[$j]) % 256]));
        }
        return $prevFunc == 'encode' ? [$keyC, $result] : [$keyB, $result];
    }
}

echo $code = MyCrypt::encode('hello world', '7580') . PHP_EOL; // d3ffpP5Wjzhz415P8hegjjEOmccJJrg4m9mKSYQrFhHSBnGwhQsjNg
echo MyCrypt::decode($code, '7580'); // hello```
```
## crypt
单向字符串散列
```php
// 需要加密的字符串
$str = "this is string";
// 指定盐值，但是盐值只能写两位，如果超过了则只会取前两位，
// 在某些系统中会直接返回FALSE
echo crypt($str,'jm'); // jmQUrN5p4VpCg
```
**注：太长的密码不支持，会生成相同的密码**
```php
$token = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0ZW5hbnRzIjpbInl1cGFvIl0sIm9wZW5JZCI6Im91XzVmYzYwMzM0YjBjNzBhMmJlMTRkNDYyOWM2MjhiMzg2IiwidXNlck5hbWUiOiLprY_miJDkv4oiLCJleHAiOjE2OTIwMDc1MTksInVzZXJJZCI6ImQxZ2NmOWMyIn0.uBVLk8Lc7ZJDlw2yGWujIHmCOcAVCkSR5FTIewYRysQ';
$token2 = 'eyJ0eXAiOiJKV1QiLCJhbGciOiJIUzI1NiJ9.eyJ0ZW5hbnRzIjpbInl1cGFvIl0sIm9wZW5JZCI6Im91XzVlOWNmMjczZWI0ODU2NzhlNzY0ZDU2ZWJiMWUwMDk1IiwidXNlck5hbWUiOiLmnajlkK8iLCJleHAiOjE2OTI0MTQxMDksInVzZXJJZCI6IjdmM2ExZDE3In0.BQtPEK54o9F5tZZMA5nZniwb_5ETlOkIlvcaletNcVw';

echo crypt($token, 'abc'), PHP_EOL; // abcmPcne0uh4A
echo crypt($token2, 'abc'), PHP_EOL; // abcmPcne0uh4A
```
## hash_equals
可防止时序攻击的字符串比较
>注：非常重要的一点是，用户提供的字符串必须是第二个参数
```php
$expected = crypt('12345', '$2a$07$usesomesillystringforsalt$');
$correct = crypt('12345', '$2a$07$usesomesillystringforsalt$');
$incorrect = crypt('apple', '$2a$07$usesomesillystringforsalt$');
var_dump(hash_equals($expected, $correct)); // true
var_dump(hash_equals($expected, $incorrect)); // false
```
# 对称加密
## openssl_encrypt
### 参数 [¶](https://www.php.net/manual/zh/function.openssl-encrypt.php#refsect1-function.openssl-encrypt-parameters)
```
data
待加密的明文信息数据。

cipher_algo
密码学方式。openssl_get_cipher_methods() 可获取有效密码方式列表。

passphrase
密码短语。若 passphrase 比预期长度短，将静默用 NUL 填充； 若比预期长度更长，将静默截断。

警告
正如其名称所示，passphrase 没有用于密钥导出函数。唯一的操作是用 NUL 字符填充，或者如果长度与预期不同则截断。

options
options 是以下标记的按位或： OPENSSL_RAW_DATA 和 OPENSSL_ZERO_PADDING 或 OPENSSL_DONT_ZERO_PAD_KEY。

iv
非 null 的初始化向量。如果 IV 比预期短，则用 NUL 字符填充并发出警告；如果密码短语比预期长，则将其截断并发出警告。

tag
使用 AEAD 密码模式（GCM 或 CCM）时传引用的验证标签。

aad
附加的验证数据。

tag_length
验证 tag 的长度。GCM 模式时，它的范围是 4 到 16。
```
**最佳实践：**
- 使用强密钥，并妥善保管。
- 为每次加密生成随机 IV。
- 选择合适的加密算法，并了解其安全性和性能特点。
- 使用认证加密模式（例如 AES-GCM），以同时提供机密性和完整性保护。
- 避免使用 ECB 模式，因为它不安全。
```php
// 明文数据
$plaintext = "This is some sensitive data.";
// 密钥 (请务必保密)
$key = "your_secret_key_1234567890"; // 至少 16 字节
// 加密算法
$cipher = "AES-256-CBC";
// 选项
$options = OPENSSL_RAW_DATA;
// 获取密码iv长度
$ivlen = openssl_cipher_iv_length($cipher);
// 初始化向量 (IV)
$iv    = openssl_random_pseudo_bytes($ivlen);
// 加密
$ciphertext_raw = openssl_encrypt($plaintext, $cipher, $key, $options, $iv);
// 将 IV 和密文连接起来，以便解密时使用
$ciphertext = base64_encode($iv . $ciphertext_raw);
// 输出密文
echo $ciphertext, PHP_EOL; // DcDM9vdIPWSIQ/JLzjvUlrQVLmuhaGA+FoP9kWO9eI0jbkFKOSUrNEtuOj3WjzCo

// --- 解密 ---
// 解码密文
$ciphertext_decoded = base64_decode($ciphertext);
// 提取 IV
$iv_dec = substr($ciphertext_decoded, 0, $ivlen);
// 提取密文
$ciphertext_dec = substr($ciphertext_decoded, $ivlen);
// 解密
$plaintext_dec = openssl_decrypt($ciphertext_dec, $cipher, $key, $options, $iv_dec);
// 输出解密后的明文
echo $plaintext_dec, PHP_EOL; // This is some sensitive data.
```
## base64
```php
# 加密
echo base64_encode('严').PHP_EOL; // 5Lil
# 解密
echo base64_decode('5Lil'); // 严
```
## URL编解码
### urlencode
编码 URL 字符串
此字符串中除了`-_.`之外的所有非字母数字字符,都将被替换成百分号 `%` 后跟两位十六进制数,**空格则编码为`+`**
```php
# 编码
echo urlencode('http://www.baidu.com?phone=11 2');
// http%3A%2F%2Fwww.baidu.com%3Fphone%3D11+2

// http://baidu.com?name=baidu&123,我们想把baidu&123作为参数传给后台
echo 'http://baidu.com?name=' . urlencode('baidu&123');
// http://baidu.com?name=baidu%26123

# 解码
echo urldecode('http%3A%2F%2Fwww.baidu.com%3Fphone%3D11+2');
// http://www.baidu.com?phone=11 2
```
### rawurlencode
按照 RFC 3986 对 URL 进行编码
此字符串中除了`-_.`之外的所有非字母数字字符,都将被替换成百分号 `%` 后跟两位十六进制数,**空格则编码为`%20`**
```php
# 编码
echo rawurlencode('http://www.baidu.com?phone=11 2');
// http%3A%2F%2Fwww.baidu.com%3Fphone%3D11%202

# 解码
echo rawurldecode('http%3A%2F%2Fwww.baidu.com%3Fphone%3D11%202');
// http://www.baidu.com?phone=11 2
```