## AES/ECB/PKCS5Padding

### 1、加密（128位）

```php
// AES/ECB/PKCS5Padding 加密
// source: http://php.net/manual/en/ref.mcrypt.php#69782
function encrypt_aes_ecb_pkcs5padding($input, $key) {
  if (!$input || !$key) return '';

  //然后选择你的算法并生成IV:
  $alg = MCRYPT_RIJNDAEL_128; // AES
  $mode = MCRYPT_MODE_ECB; // not recommended unless used with OTP
  $iv_size = mcrypt_get_iv_size($alg, $mode);
  $block_size = mcrypt_get_block_size($alg, $mode);
  $iv = mcrypt_create_iv($iv_size, MCRYPT_DEV_URANDOM); // pull from /dev/urandom

  // 对输入做PKCS#5填充
  $pad = $block_size - (strlen($input) % $block_size);
  $input =  $input . str_repeat(chr($pad), $pad);

  // 加密
  return base64_encode(mcrypt_encrypt($alg, $key, $input, $mode, $iv));
}
```