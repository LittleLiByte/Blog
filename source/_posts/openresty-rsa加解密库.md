---
title: openresty-rsa加解密库
categories: OpenResty
tags: 
 - RSA
 - Nginx
 - Lua
date: 2016-08-20 16:53:22
description: "最近在项目中用到OpenResty做服务端的接口，其中接口认证方面又用到了RSA算法进行加解密。但是目前OpenResty并没有官方的rsa模块，只有doujiang24前辈提供的lua-resty-rsa库，但是该库只支持公钥加密私钥解密，对私钥加密公钥解密却不支持，为了完善这一方面的缺陷，开发了自己的rsa加解密库"
---

因为lua相关的RSA加解密库比较少，而C和C++相关的RSA加解密库却比较多，因此萌生了通过LuaJit的FFI  拓展库来调用C或C++实现来封装成Lua 的RSA库。经过谷歌终于找到了一个很符合要求的实现方法：
http://hayageek.com/rsa-encryption-decryption-openssl-c/

下面就来说说整个开发详细流程
<!-- more -->
## 安装OpenSSL
RSA加密和解密的公钥私钥由OpenSSL生成。在Ubuntu下，安装OpenSSL通过以下两条命令可以完成：
``` bash
sudo apt-get install openssl
sudo apt-get install libssl-dev
```

## 生成公钥和私钥
``` bash
-- 生成 RSA 私钥（传统格式的,PKCS#1标准）
openssl genrsa -out rsa_private_key.pem 1024
-- 将传统格式的私钥转换成 PKCS#8 格式的（JAVA需要使用的私钥需要经过PKCS#8编码,而本次在项目作为接口使用Lua编写，如不需要这一步其实可以略过）
openssl pkcs8 -topk8 -inform PEM -in rsa_private_key.pem -outform PEM -nocrypt
-- 生成 RSA 公钥
openssl rsa -in rsa_private_key.pem -pubout -out rsa_public_key.pem
```

## 编译动态链接库
参考上面给的链接，要封装的RSA加解密C语言实现版如下：
rsa.c文件
``` c
#include <openssl/pem.h>
#include <openssl/ssl.h>
#include <openssl/rsa.h>
#include <openssl/evp.h>
#include <openssl/bio.h>
#include <openssl/err.h>
#include <stdio.h>
 
int padding = RSA_PKCS1_PADDING;
 
RSA * createRSA(unsigned char * key,int public)
{
    RSA *rsa= NULL;
    BIO *keybio ;
    keybio = BIO_new_mem_buf(key, -1);
    if (keybio==NULL)
    {
        printf( "Failed to create key BIO");
        return 0;
    }
    if(public)
    {
        rsa = PEM_read_bio_RSA_PUBKEY(keybio, &rsa,NULL, NULL);
    }
    else
    {
        rsa = PEM_read_bio_RSAPrivateKey(keybio, &rsa,NULL, NULL);
    }
    if(rsa == NULL)
    {
        printf( "Failed to create RSA");
    }
 
    return rsa;
}
 
int public_encrypt(unsigned char * data,int data_len,unsigned char * key, unsigned char *encrypted)
{
    RSA * rsa = createRSA(key,1);
    int result = RSA_public_encrypt(data_len,data,encrypted,rsa,padding);
    return result;
}
int private_decrypt(unsigned char * enc_data,int data_len,unsigned char * key, unsigned char *decrypted)
{
    RSA * rsa = createRSA(key,0);
    int  result = RSA_private_decrypt(data_len,enc_data,decrypted,rsa,padding);
    return result;
}
 
 
int private_encrypt(unsigned char * data,int data_len,unsigned char * key, unsigned char *encrypted)
{
    RSA * rsa = createRSA(key,0);
    int result = RSA_private_encrypt(data_len,data,encrypted,rsa,padding);
    return result;
}
int public_decrypt(unsigned char * enc_data,int data_len,unsigned char * key, unsigned char *decrypted)
{
    RSA * rsa = createRSA(key,1);
    int  result = RSA_public_decrypt(data_len,enc_data,decrypted,rsa,padding);
    return result;
}
 
void printLastError(char *msg)
{
    char * err = malloc(130);;
    ERR_load_crypto_strings();
    ERR_error_string(ERR_get_error(), err);
    printf("%s ERROR: %s\n",msg, err);
    free(err);
}
```
在ubuntu下通过以下命令来生成.so库
``` bash
# gcc rsa.c -fPIC -shared -o librsa.so
```

## 使用rsa.so库
生成的动态链接库放到**usr/lib**目录下就能使用了，以下使用rsa.so封装了一个lua版的rsa加解密库
``` lua
module("rsa", package.seeall)
local ffi = require('ffi')
 
local rsa = ffi.load('rsa')
 
ffi.cdef [[
int public_encrypt(unsigned char * data,int data_len,unsigned char * key, unsigned char *encrypted);
int private_decrypt(unsigned char * enc_data,int data_len,unsigned char * key, unsigned char *decrypted);
 
int private_encrypt(unsigned char * data,int data_len,unsigned char * key, unsigned char *encrypted);
int public_decrypt(unsigned char * enc_data,int data_len,unsigned char * key, unsigned char *decrypted);
]]
 
local _M = { _VERSION = '1.0' }
 
--公钥加密
function _M.public_encrypt(msg, publicKey)
    local c_str = ffi.new("char[?]", 1024 / 8)
    ffi.copy(c_str, msg)
    local pub = ffi.new("char[?]", #publicKey)
    ffi.copy(pub, publicKey)
    --存放加密结果
    local encrypt = ffi.new("char[?]", 2048)
    local ret = rsa.public_encrypt(c_str, #msg, pub, encrypt)
    if ret == -1 then
        return nil
    end
    return ffi.string(encrypt,ret)
end
 
--私钥解密
function _M.private_decrypt(msg, privateKey)
    local c_str = ffi.new("char[?]", 1024 / 8)
    ffi.copy(c_str, msg)
    local pri = ffi.new("char[?]", #privateKey)
    ffi.copy(pri, privateKey)
    --存放解密结果
    local decrypt = ffi.new("char[?]", 2048)
    local ret = rsa.private_decrypt(c_str, #msg, pri, decrypt)
    if ret == -1 then
        return nil
    end
    return ffi.string(decrypt,ret)
end
 
--私钥加密
function _M.private_encrypt(msg, privateKey)
    local c_str = ffi.new("char[?]", 1024 / 8)
    ffi.copy(c_str, msg)
    local pri = ffi.new("char[?]", #privateKey)
    ffi.copy(pri, privateKey)
    --存放加密结果
    local encrypt = ffi.new("char[?]", 2048)
    local ret = rsa.private_encrypt(c_str, #msg, pri, encrypt)
    if ret == -1 then
        return nil
    end
    return ffi.string(encrypt,ret)
end
 
--公钥解密
function _M.public_decrypt(msg, publicKey)
    local c_str = ffi.new("char[?]", 1024 / 8)
    ffi.copy(c_str, msg)
    local pub = ffi.new("char[?]", #publicKey)
    ffi.copy(pub, publicKey)
    --存放解密结果
    local decrypt = ffi.new("char[?]", 2048)
    local ret = rsa.public_decrypt(c_str, #msg, pub, decrypt)
    if ret == -1 then
        return nil
    end
    return ffi.string(decrypt,ret)
end
 
return _M
```
就这样，本人的openresty rsa加解密第三方库就制作完成了，支持公钥加密私钥解密，也支持私钥加密公钥解密。下面贴出本人的测试代码：
``` lua
local rsa = require "resty.rsa"
local RSA_PUBLIC_KEY = [[-----BEGIN PUBLIC KEY-----
MIGfMA0GCSqGSIb3DQEBAQUAA4GNADCBiQKBgQC3bTBJNQJjY6u7Y5b2eOWws0yW
CGuWPm6MGOSVan65wCrJa5p3q3sodQUDVPotjsknjLlje9E1F7Bx94ZuqTwkvAr6
ieLkgbbeqTCzeJ0AryUXiF3auxFSPdpBoD6nxtEeN8bZwfa/IYzdKyKlbhiQbUMN
qWgmxiPVwupwAML7RQIDAQAB
-----END PUBLIC KEY-----]]
local RSA_PRIV_KEY = [[-----BEGIN RSA PRIVATE KEY-----
MIICXAIBAAKBgQC3bTBJNQJjY6u7Y5b2eOWws0yWCGuWPm6MGOSVan65wCrJa5p3
q3sodQUDVPotjsknjLlje9E1F7Bx94ZuqTwkvAr6ieLkgbbeqTCzeJ0AryUXiF3a
uxFSPdpBoD6nxtEeN8bZwfa/IYzdKyKlbhiQbUMNqWgmxiPVwupwAML7RQIDAQAB
AoGAc4NXvUKc1mqWY9Q75cwNGlJQEMwMtPlsNN4YVeBTHjdeuqoBBQwA62GGXqrN
QpOBKl79ASGghob8n0j6aAY70PQqHSU4c06c7UlkeEvxJKlyUTO2UgnjjIHb2flR
uW8y3xmjpXAfwe50eAVMNhCon7DBc+XMF/paSVwiG8V+GAECQQDvosVLImUkIXGf
I2AJ2iSOUF8W1UZ5Ru68E8hJoVkNehk14RUFzTkwhoPHYDseZyEhSunFQbXCotlL
Ar5+O+lBAkEAw/PJXvi3S557ihDjYjKnj/2XtIa1GwBJxOliVM4eVjfRX15OXPR2
6shID4ZNWfkWN3fjVm4CnUS41+bzHNctBQJAGCeiF3a6FzA/0bixH40bjjTPwO9y
kRrzSYX89F8NKOybyfCMO+95ykhk1B4BF4lxr3drpPSAq8Paf1MhfHvxgQJBAJUB
0WNy5o+OWItJBGAr/Ne2E6KnvRhnQ7GFd8zdYJxXndNTt2tgSv2Gh6WmjzOYApjz
heC3jy1gkN89NCn+RrECQBTvoqFHfyAlwOGC9ulcAcQDqj/EgCRVkVe1IsQZenAe
rKCWlUaeIKeVkRz/wzb1zy9AVsPC7Zbnf4nrOxJ23mI=
-----END RSA PRIVATE KEY-----]]
 
ngx.say('-----公钥加密私钥解密 start------')
local str='i love lwl'
 
local encryptPubStr = rsa.public_encrypt(str, RSA_PUBLIC_KEY)
if not encryptPubStr then
ngx.say('pub encrypt failed')
end
local decryptPriStr=rsa.private_decrypt(encryptPubStr,RSA_PRIV_KEY)
if not decryptPriStr then
ngx.say('pri decrypt failed')
end
ngx.say('公钥加密私钥解密成功\n'..decryptPriStr)
ngx.say('-----公钥加密私钥解密 end------')
 
ngx.say('================================')
 
ngx.say('-----私钥加密公钥解密 start------')
local str='i like lwl'
 
local encryptPriStr = rsa.private_encrypt(str, RSA_PRIV_KEY)
if not encryptPriStr then
ngx.say('pri encrypt failed')
end
local decryptPubStr=rsa.public_decrypt(encryptPriStr,RSA_PUBLIC_KEY)
if not decryptPubStr then
ngx.say('pub decrypt failed')
end
ngx.say('公钥加密私钥解密成功\n'..decryptPubStr)
ngx.say('-----私钥加密公钥解密 end------')
```
项目已托管到Github，欢迎star或fork，也欢迎提出改进意见。
传送门：[lua-rsa](https://github.com/LittleLiByte/lua-rsa)
