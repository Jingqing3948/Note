---
title: 问题解决：ubuntu 安装 emscripten 时 install latest 报错
date: 2023-10-09
tags:
- Linux
categories:
- 破浪（遇到的bug及解决方案）
description: ubuntu 安装 emscripten 时 install latest 报错问题解决

---

学习官网参考：[Compiling a New C/C++ Module to WebAssembly - WebAssembly | MDN (mozilla.org)](https://developer.mozilla.org/en-US/docs/WebAssembly/C_to_wasm)

## 报错信息

形如：

```shell
Error: Downloading URL 'https://storage.googleapis.com/webassembly/emscripten-releases-builds/linux/b90507fcf011da61bacfca613569d882f7749552/wasm-binaries.tbz2': <urlopen error [Errno 104] Connection reset by peer>
error: installation failed!
```

OS：

```shell
$ uname -a
Linux jingqing 5.19.0-35-generic #36~22.04.1-Ubuntu SMP PREEMPT_DYNAMIC Fri Feb 17 15:17:25 UTC 2 x86_64 x86_64 x86_64 GNU/Linux
```

## 产生错误原因分析

> emsdk install latest报错（因为从谷歌中下载，cmd中命令形式访问不到google）
> 版权声明：本文为CSDN博主「小白啥时候能进阶成功」的原创文章，遵循CC 4.0 BY-SA版权协议，转载请附上原文出处链接及本声明。
> 原文链接：https://blog.csdn.net/qq_34754747/article/details/103815349

非常感谢博主的答疑解惑，不过我没有看得太懂原文中的解决方案，网上搜到的大多数方案也是 windows 环境下的解决方案，因此我决定自己写一个 ubuntu 系统下的补安装。

## 解决方案

说白了就是我自己复制链接到浏览器里，下载安装这几个包，放到 emsdk 的指定位置。

这里有两个要注意的点，这一部分主要是分析，不想看的同学可以直接跳到[具体步骤](#solution)处：

1. emsdk install 的默认安装规则是：不管你有没有安装过这些包，我 install latest 都是重新安装，保证最新版本。但是现在问题是 install latest 有问题，我要手动安装包放进去。
   我们打开 emsdk.py 通过搜索关键词可以找到报错信息的位置：

   ```python
   # ./emsdk.py:675
   
   # On success, returns the filename on the disk pointing to the destination file that was produced
   # On failure, returns None.
   def download_file(url, dstpath, download_even_if_exists=False, filename_prefix=''):
     debug_print('download_file(url=' + url + ', dstpath=' + dstpath + ')')
     file_name = get_download_target(url, dstpath, filename_prefix)
     if os.path.exists(file_name) and not download_even_if_exists:
       print("File '" + file_name + "' already downloaded, skipping.")
       return file_name
     try:
       u = urlopen(url)
       mkdir_p(os.path.dirname(file_name))
       with open(file_name, 'wb') as f:
         file_size = get_content_length(u)
         if file_size > 0:
           print("Downloading: %s from %s, %s Bytes" % (file_name, url, file_size))
         else:
           print("Downloading: %s from %s" % (file_name, url))
   
         file_size_dl = 0
         # Draw a progress bar 80 chars wide (in non-TTY mode)
         progress_max = 80 - 4
         progress_shown = 0
         block_sz = 256 * 1024
         if not TTY_OUTPUT:
             print(' [', end='')
         while True:
             buffer = u.read(block_sz)
             if not buffer:
                 break
   
             file_size_dl += len(buffer)
             f.write(buffer)
             if file_size:
                 percent = file_size_dl * 100.0 / file_size
                 if TTY_OUTPUT:
                     status = r" %10d  [%3.02f%%]" % (file_size_dl, percent)
                     print(status, end='\r')
                 else:
                     while progress_shown < progress_max * percent / 100:
                         print('-', end='')
                         sys.stdout.flush()
                         progress_shown += 1
         if not TTY_OUTPUT:
           print(']')
           sys.stdout.flush()
     except Exception as e:
       errlog("Error: Downloading URL '" + url + "': " + str(e))
       if "SSL: CERTIFICATE_VERIFY_FAILED" in str(e) or "urlopen error unknown url type: https" in str(e):
         errlog("Warning: Possibly SSL/TLS issue. Update or install Python SSL root certificates (2048-bit or greater) supplied in Python folder or https://pypi.org/project/certifi/ and try again.")
       rmfile(file_name)
       return None
     except KeyboardInterrupt:
       rmfile(file_name)
       exit_with_error("aborted by user, exiting")
     return file_name
   ```

   大致一看能看明白逻辑，如果 download_even_if_exists = True 那么无论包是否已经存在都要安装，否则为 False 就只安装不存在的包，我们需要为 False。

   搜索函数名查看在哪里使用了这个函数：

   ```python
   # ./emsdk.py:1411
   
   received_download_target = download_file(url, download_dir, not KEEP_DOWNLOADS, filename_prefix)
   ```

   这个 KEEP_DOWNLOADS 是一个环境变量，默认为0，我们需要他为1，传入函数的参数则为0（False），即已存在文件不再重复下载。

   在终端输入 `./emsdk --help` 可以看到提示信息如下：

   ```shell
   Environment:
         EMSDK_KEEP_DOWNLOADS=1     - if you want to keep the downloaded archives.
         EMSDK_NOTTY=1              - override isatty() result (mainly to log progress).
         EMSDK_NUM_CORES=n          - limit parallelism to n cores.
         EMSDK_VERBOSE=1            - very verbose output, useful for debugging.
   ```

   也就是说只要安装时单独指定此变量值为1即可。

2. 第二步就是如何下载文件了。下载什么文件？放到哪个目录下？

   这里大家可以通过 download_file 的 print debug 调试来查看他校验文件是否存在是去哪里校验的，我就不再具体展开讲调试步骤了，结论就是：他在 emsdk/downloads/ 目录下先查找一下待下载的压缩包是否存在，那么我们复制报错信息中的 url 下载文件到这个 downloads 文件夹下即可（没有就新建）。

## 具体步骤

1. 首先要安装所缺的所有包，一个个安装，报错信息里提示什么安装什么。比如文章开头的报错信息中下载链接是：https://storage.googleapis.com/webassembly/emscripten-releases-builds/linux/b90507fcf011da61bacfca613569d882f7749552/wasm-binaries.tbz2，就先安装这个。

   node: https://storage.googleapis.com/webassembly/emscripten-releases-builds/linux/b90507fcf011da61bacfca613569d882f7749552/wasm-binaries.tbz2

   wasm-binaries: https://storage.googleapis.com/webassembly/emscripten-releases-builds/linux/b90507fcf011da61bacfca613569d882f7749552/wasm-binaries.tbz2

   安装完成后要重命名 b90507fcf011da61bacfca613569d882f7749552-wasm-binaries.tbz2。

2. 移入 emsdk/downloads 文件夹下，不用解压。

3. 执行 `EMSDK_KEEP_DOWNLOADS=1` 变量赋值。

4. 执行 `./emsdk install latest`  。

```shell
jingqing3948@jingqing:~/Webassembly/emsdk$ ./emsdk install latest
Resolving SDK alias 'latest' to '3.1.44'
Resolving SDK version '3.1.44' to 'sdk-releases-b90507fcf011da61bacfca613569d882f7749552-64bit'
Installing SDK 'sdk-releases-b90507fcf011da61bacfca613569d882f7749552-64bit'..
Skipped installing node-16.20.0-64bit, already installed.
Skipped installing releases-b90507fcf011da61bacfca613569d882f7749552-64bit, already installed.
All SDK components already installed: 'sdk-releases-b90507fcf011da61bacfca613569d882f7749552-64bit'.
```

好哎，看来是自己单独安装的文件包都可以用，他会自己解压文件包后提示 All SDK components already installed。

接下来就是下一步：`./emsdk activate latest`.

最后是 `source ./emsdk_env.sh` 配置好环境变量。
