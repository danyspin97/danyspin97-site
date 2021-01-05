---
title: "Replacing Openssl, Part 1: wolfSSL"
date: 2021-01-05T16:00:00+01:00
---

This post highlight my experiment of replacing OpenSSL cryptographic library
with its embedded competitor: `wolfssl`.

***Prerequites***: _Basic knowledge about C/C++ programming_.

## A bit of background of OpenSSL

OpenSSL is one of the most crucial library on a Unix system: it performs
cryptography functions and it provides  Transport Layer Security (TLS) and Secure Sockets Layer (SSL) protocols to applications.

According to the _Arch Linux_ [OpenSSL package], **355** packages, out of the
_11523_ available, depend on it. You can find it installed on pretty much any
Unix system (and often on Windows too!).

[OpenSSL package]: https://archlinux.org/packages/core/x86_64/openssl/

It started in 1998 as a fork of [SSLeay] and has been in development since.
Two full-time developer work on it, as well an many volunteers.

[SSLeay]: https://en.wikipedia.org/wiki/SSLeay

Fast forward to 2014: a CVE had been issued regarding a high risk vulnerability
found in OpenSSL implementation. [Heartbleed] is its given name, because it has
been found in the TLS/DTLS heartbeat extension (RFC6520). It allowed attackers
to steal sensitive data, such as secret keys, user names and passwords.

[Heartbleed]: https://heartbleed.com/

All the community turned to the OpenSSL project, weighting its implementation
and security policy. Heartbleed have been promptly fixed, but it could happen
again, if security was not properly prioritized.

At this point, OpenBSD's folks forked OpenSSL and started a new project:
[LibreSSL]. It primary goals were to modernize the codebase and to improve
its security. This new project hasn't been adopted by big distributions such
Ubuntu and Arch Linux; instead smaller distributions (at that time) replaced
OpenSSL with LibreSSL on their default configuration, such as Alpine and Void.

[LibreSSL]: https://www.libressl.org/

In the last years, LibreSSL have seen a decline in its usage. Alpine switched
back to OpenSSL ([link to the thread]). Many are considering doing the same,
since OpenSSL got the improvment that LibreSSL aimed for. And it's still the
_de facto standard_ library.

[link to the thread]: https://lists.alpinelinux.org/~alpine/devel/%3C20181011171746.4c01f758%40ncopa-desktop.copa.dup.pw%3E

## Why replacing OpenSSL now?

OpenSSL works fine, there is no denying that. Every software has out of the box
support for it so it costs no effort in term of additional mainteinance.
However, there are newer and lighter TLS and SSL implementations, which many
folks might prefer over the heavy OpenSSL's one. I, too, prefer lightweight
libraries.

I am uninstalling LibreSSL on my systems, therefore this might be the perfect
time to experiment with something different. I have found an interesting (and
well done!) library called ***[wolfSSL]*** that claims to have an OpenSSL
compatibility layer. It also claims to be 20 times smaller than OpenSSL.

[wolfSSL]: https://wolfssl.com

## Installation

**wolfSSL** is a small C library; It uses _autotools_ and __CMake__ support
is being added (sigh!). Upon running `configure`, I am overwhelmed by the
quantity of compile options available.

![](/img/replacing-openssl-part-1-wolfssl/wolf-ssl-configure.jpg
        "some of the wolfSSL configure options")

Fortunately, there is one comfy option: `--enable-all`, which I've used
when compiling wolfSSL. Another useful option is `--enable-distro` which
allow package mainteiner to provide their own `option.h` header. Since
we don't know much about wolfSSL yet, we might just stick with the default
options.

Upon completing the installation (which takes just a couple of minutes),
a library `libwolfssl.so` will be installed into `/usr/lib`, as well as
a pkgconfig file (`wolfssl.pc`) and all the headers.

## Out of the box support

Key applications provide support for different SSL implementations other than
OpenSSL. For example _curl_ supports _mbedTLS_, _BearSSL_ and our **wolfSSL**.
To compile _curl_ using wolfSSL, we just need to add `--with-ssl=wolfssl` and
we're done. I am aware of only two packages that have out of the box support
for wolfSSL and they are curl and wget2 (not the legacy version!).

## OpenSSL compatibility

`--enable-all` configure option enabled the OpenSSL compatibility layer.
However, according to the [documentation], we need to link to wolfssl library
manually by adding `-lwolfssl` as link argument. We also need to include
`/usr/include/wolfssl` so that the includes in `/usr/include/wolfssl/openssl`
will be picked up.

[documentation]: https://www.wolfssl.com/docs/wolfssl-manual/ch13/

This way both wolfSSL and OpenSSL can coexist on the same system and the latter
can be replaced on a per project basis. However, that's not what we want since
want to replace OpenSSL at a system level.

Adding the link flag to each package on the system or patching one by one
is not an option as it would take too much time. Let's instead apply some
workarounds.

First, we create a symlink for the headers:
```bash
$ sudo ln -sf /usr/include/wolfssl/openssl /usr/include/openssl
```

Then we create a symlink for each library. OpenSSL provides the following
libraries:

- `libssl.so`
- `libcrypto.so`

So we can run:

```bash
$ sudo ln -sf /usr/lib/libwolfssl.so /usr/lib/libssl.so
$ sudo ln -sf /usr/lib/libwolfssl.so /usr/lib/libcrypto.so
```

### Fixing pkg-config

Some software rely on pkg-config for checking OpenSSL dependency since
it provides the following pkg-config files:

- `libssl.pc`
- `libcrypto.pc`
- `openssl.pc`

My system doesn't currently have either OpenSSL or LibreSSL installed, so any
attempt to include OpenSSL by using pkg-config will fail. To fix this, we
can add a pkg-config file in `/usr/lib/pkgconfig/openssl.pc`:

```
prefix=/usr/x86_64-pc-linux-musl
includedir=${prefix}/include/wolfssl

Name: openssl
Description: wolfssl C library.
Version: 4.6.0
Requires: wolfssl
Cflags: -I${includedir}
```

Our pkg-config file will include the correct directory containing OpenSSL
headers. It will also import cflags and link flags from the wolfssl pkg-config.
Let's try if it works:

``` bash
$ pkg-config --cflags --libs openssl
-I/usr/x86_64-pc-linux-musl/include/wolfssl -lwolfssl
```

Yes! Now we just need create symlinks the other two pkg-config files:

```bash
$ sudo ln -sf /usr/lib/pkgconfig/openssl.pc /usr/lib/pkgconfig/libssl.pc
$ sudo ln -sf /usr/lib/pkgconfig/openssl.pc /usr/lib/pkgconfig/libcrypto.pc
```

Now we're ready to compile system packages.

## Compiling system packages

Before actually going further into the various compile results, there are few
general changes that I've done:

1. Include the default `/usr/include/wolfssl/option.h` provided in
  `/usr/include/wolfssl/wolfcrypt/setting.h`. This way we will have a bunch of
  defines already included that enable many wolfSSL features.
2. Remove some defines that disables old OpenSSL features, like old SSL names.
  While I understand that these feature are plainly old (or even deprecated),
  projects will still use them as long as they haven't been removed upstream.
3. Define `OPENSSL_NO_WHIRPOOL` since this feature ins't implemented in
  wolfSSL. There are other `OPENSSL_NO_*` defines for unimplemented features,
  but this was missing.

### wget

_[wget]_, which comes installed on any unix system, is the first software we'll
try building with wolfSSL and its OpenSSL compatibility layer.

[wget]: https://www.gnu.org/software/wget/

![](/img/replacing-openssl-part-1-wolfssl/wget-error-1.jpg
        "wget compile error")

wget complains about two unresolved symbols:

- `CONF_MFLAGS_DEFAULT_SECTION`
- `CONF_MFLAGS_IGNORE_MISSING_FILE`

We can fix this by adding the defines from the OpenSSL file `openssl/conf.h`
to `/usr/include/wolfssl/option.h`:

```c
# define CONF_MFLAGS_IGNORE_MISSING_FILE 0x10
# define CONF_MFLAGS_DEFAULT_SECTION     0x20
```

![](/img/replacing-openssl-part-1-wolfssl/wget-error-2.jpg
        "wget link-time errors about missing symbols")

We arrived at link-time, that's huge! However, there are 8 undefined symbols:
6 symbols about unimplemented features in wolfSSL, such as `i2d_x509_PUBKEY`
and `a2i_IPADDRESS`; 2 symbols regarding wolfSSL functions. The latter could
probably be fixed by adding the proper compile time option to wolfSSL, like
`--enable-sslv3`.

### rhash

rhash is an utility that calculates and verify message digests, such as _SHA256_
and _MD5_. It is required by CMake, so we can consider it an essential software.
Compiling rhash lead to many errors. Let's look in depth on some of them.

![](/img/replacing-openssl-part-1-wolfssl/rhash-error-1.jpg
        "rhash compile error about RIPEMD160_CTX")

Here, _rhash_ uses `RIPEMD160_CTX` which is not implemented by wolfSSL.

![](/img/replacing-openssl-part-1-wolfssl/rhash-error-2.jpg
        "rhash compile error about WOLFSSL_MD5_CTX")

This time, rhash access a member of `MD5_CTX`, which wolfSSL has replaced by
`WOLFSSL_MD5_CTX`, using `offsetof`. The original `MD5_CTX` struct looks like
this:

```c
typedef struct MD5state_st {
    MD5_LONG A, B, C, D;
    MD5_LONG Nl, Nh;
    MD5_LONG data[MD5_LBLOCK];
    unsigned int num;
} MD5_CTX;
```

`WOLFSSL_MD5_CTX` instead looks like this:

```c
typedef struct WOLFSSL_MD5_CTX {
    /* big enough to hold wolfcrypt md5, but check on init */
#ifdef STM32_HASH
    void* holder[(112 + WC_ASYNC_DEV_SIZE + sizeof(STM32_HASH_Context)) / sizeof(void*)];
#else
    void* holder[(112 + WC_ASYNC_DEV_SIZE) / sizeof(void*)];
#endif
} WOLFSSL_MD5_CTX;
```

My educated guess it this incosistency between OpenSSL and wolfSSL structs
_will_ happen again with different data types.

### libssh2

From the official [libssh2] site:

> _libssh2 a client-side C library implementing the SSH2 protocol_

[libssh2]: https://www.libssh2.org/

curl has a hard dependency on libssh2, so we can consider it another essential
library.

![](/img/replacing-openssl-part-1-wolfssl/libssh2-error-1.jpg
        "libssh2 build errors")

This time there are only 2 errors, both about unimplemented features:
`EVP_bf_cbc` and `EVP_cast5_cbc`. The man pages are the best source of
information about OpenSSL functions:

```bash
$ man EVP
$ man EVP_bf_cbc
$ man EVP_cast5_cbc
```

Let's split the names and study each part:

- `EVP` is a high-level interface to cryptographic functions
- `bf` and `cast5` are the name of algorithms, _[blowfish]_ and _[CAST]_
  respectively.
- `cbc`, _[Cipher Block Chaining]_, is a mode of operation that tells OpenSSL
  to operato on each block seperataely from the others.

Operating cryptographic functions in _CBC_ mode isn't really safe nor the
two algorithms above are really popular, so I understand why wolfSSL developers
might have prioritized other things over implementing the two functions above.

### ffmpeg

_[ffmpeg]_ is a powerful library and collection of utilities for handling
images, audio and videos. It is required by _Firefox_, _Chromium_, _mpv_ and
another hundred and a half software (at least). We surely don't want a system
without this library.

[ffmpeg]: https://ffmpeg.org/

![](/img/replacing-openssl-part-1-wolfssl/ffmpeg-error.jpg
        "ffmpeg build error")

This time there is only one error (but others may pop up after fixing this
one!). The function `BN_sub_word` is missing. According to the man pages,
it is the arithmetic function that perform subtractions on big integers.
This seems pretty straightforward to add in wolfSSL.

## Conclusion

I've tried different essential software and none of them built using
wolfSSL and its OpenSSL compatibility layer. However, most of them had small
errors that can probably be fixed in a day or two. That's a shame because
wolfSSL is really close to replace OpenSSL at a distro level, and yet so distant
since it's not its priority. According to what I've seen and the documentation,
the OpenSSL compatibility layer is best used on a per-project basis, where
you can fix your code or replace missing features with modern ones (or
even better, you can pay them to implement the features you need).

