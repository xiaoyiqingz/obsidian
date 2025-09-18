php 8.x 后默认编译的 openssl 3.x 的 版本，导致一些老的加密 摘要算法 （如 des-cbc、 sha1） 已经不支持了，如果使用，需要打开 legacy 模式

1.  openssl.cnf 如下
```
# 路径 /etc/pki/tls/openssl.cnf

[default_sect]
activate = 1
[legacy_sect]
activate = 1

[provider_sect]
default = default_sect
legacy = legacy_sect
```

这样, 'des-cbc' 可以工作了
```
 $data = openssl_decrypt($input, 'des-cbc', $desKey, OPENSSL_RAW_DATA | OPENSSL_ZERO_PADDING, $iv)
```

2.  /etc/crypto-policies/back-ends/opensslcnf.config
```
路径 /etc/crypto-policies/back-ends/opensslcnf.config

Options = UnsafeLegacyRenegotiation
```

   /etc/crypto-policies/config
```
LEGACY
```

执行 `update-crypto-policies --set LEGACY`

openssl_sign($data, $signture, $pkey)  默认  OPENSSL_ALGO_SHA1  可以工作