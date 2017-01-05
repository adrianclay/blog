---
layout: post
title:  "Upgrading to PHP 7.0 and MCrypt"
date:   2017-01-03 11:00:00 +0000
categories: PHP
author: Adrian Clay
image: roadworks.jpg
---
During the summer of 2016 I was part of a team that migrated a large software project written in PHP from version 5.5 to 7.0.
As part of that migration, we had to analyse each of the breaking changes and fix any we were affected by.
One such change, which was particularly tricky to fix, was in the MCrypt library for PHP.

For those unfamiliar with MCrypt, it is a library which exposes encryption and decryption functions.

```php
<?
function mcrypt_encrypt(string $cipher,
                        string $key,
                        string $data,
                        string $mode,
                        string $iv): string
{

```

In version 5.6 of PHP, the encryption functions stopped accepting invalid sized ```$key``` and ```$iv``` parameters.
In the project we were migrating, invalid ```$iv``` lengths had been passed in and used to encrypt/decrypt data at rest and in transit.
To allow the project to continue decrypting this data while on PHP 7.0, we had to replicate the behaviour of PHP MCrypt prior to 5.6.

The documentation for PHP MCrypt states that prior to 5.6 “keys and IVs were padded with '\0' bytes to the next valid size.”
Testing this claim is relatively easy, you just need a copy of PHP 5.5 and some scenarios which will invoke this behaviour.
When given an IV with 15 bytes, and given the same IV padded with null bytes up to 16 bytes, they should both produce the same ciphertext.
We tested this claim, using prefix and suffix padding, and failed to find the padding scheme that PHP claimed to be using.

The black box investigation of the behaviour wasn’t proving successful, so I decided to look inside the PHP interpreter source code to find out what was really happening.

``` php_mcrypt_do_crypt ``` is where the validation of the IV happens. If you like reading C code, I have reproduced the relevant validation block below (in accordance with the [PHP License](https://github.com/php/php-src/blob/PHP-5.5.38/LICENSE) of course).  During the positive validation path, ```iv``` gets copied to ```iv_s``` using memcpy.  Down the negative path, ```iv_s``` is unmodified and so retains its initialisation value of ```NULL```.  A few lines below ```iv_s``` then gets passed through, unmodified, into the mcrypt library.

```c
/* IV is required */
if (mcrypt_enc_mode_has_iv(td) == 1) {
    if (argc == 5) {
        if (iv_size != iv_len) {
            php_error_docref(NULL TSRMLS_CC, E_WARNING, MCRYPT_IV_WRONG_SIZE);
        } else {
            iv_s = emalloc(iv_size + 1);
            memcpy(iv_s, iv, iv_size);
        }
    } else if (argc == 4) {
        php_error_docref(NULL TSRMLS_CC, E_WARNING, "Attempt to use an empty IV, which is NOT recommend");
        iv_s = emalloc(iv_size + 1);
        memset(iv_s, 0, iv_size + 1);
    }
}
```


Reiterating the above, MCrypt receives a ```NULL``` IV in the case that an invalid IV was supplied.
Taking this information into account, in regards to our upgrade, we knew we had to pass in a valid length IV to meet the validation requirements, but additionally it had to be an empty IV to replicate the PHP 5.5 behaviour.
Calling ```mcrypt_encrypt``` with an IV consisting of ```NULL``` bytes passed the checks and we were satisfied we knew what PHP was doing.

In conclusion, don’t trust the PHP manual to behave exactly as it claims.
If you find yourself maintaining a project using PHP MCrypt, then be aware that before PHP 5.6 the IV passed through to MCrypt was equal to all NULL bytes.
I’ve provided a quick example function below to be more explicit.

```php
<?
function extend_iv_length($cipher, $mode, $iv) {
    $blockSize = mcrypt_get_block_size($cipher, $mode);
    if(strlen($iv) != $blockSize) {
      $iv = str_repeat("\0", $blockSize);
    }
    return $iv;
}
```