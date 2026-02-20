虽然mit官方的教程让你在linux下运行，但是我不喜欢WSL（还是太穷了，内存不够吃）

# qemu
首先确保你安装了mysy2环境，
接着每日一滚`pacman -Syyu`
![](https://linux.do/uploads/default/original/4X/f/7/e/f7e12b0eec6e73b420baa9ad385a469d26113eaa.png)
随后使用`pacman -S mingw-w64-x86_64-qemu`安装qemu
![image](https://linux.do/uploads/default/original/4X/f/b/7/fb79cfcc423b259e9d4210cf8c48066dc99d7c98.png)
随后打开你的环境变量，在Path最后上添加一行，是qemu存放的位置
这里为
`C:\msys64\mingw64\bin`


# riscV gcc windows
先安装一个make `pacman -S make`
然后追加一个普通gcc `winget install BrechtSanders.WinLibs.POSIX.MSVCRT`
下载预构建的gcc工具链（其他的
https://gnutoolchains.com/risc-v/
![image](https://linux.do/uploads/default/original/4X/8/0/6/806ba2137eefa538001aa86b50d3d446053d7811.png)
选择你心仪的版本，随后下载
![](https://linux.do/uploads/default/original/4X/1/a/a/1aafb9d348d57943c46e249471490c246b22e40c.png)
打开
![image](https://linux.do/uploads/default/original/4X/9/7/d/97dd85b444661233fc9276828315d3fa4369460b.jpeg)
自行安装（默认会勾选自动配置环境变量）

# 启动xv6
克隆仓库
https://github.com/mit-pdos/xv6-riscv
```
git clone https://github.com/mit-pdos/xv6-riscv
cd xv6-riscv
```
此时不要急，可能会报错，我们将mkfs.c进行修改
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <fcntl.h>
#include <assert.h>
```
修改为
```
#include <stdio.h>
#include <unistd.h>
#include <stdlib.h>
#include <string.h>
#include <strings.h>
#include <fcntl.h>
#include <assert.h>
#include <errno.h>

#ifndef O_BINARY
#define O_BINARY 0
#endif
```
`fsfd = open(argv[1], O_RDWR|O_CREAT|O_TRUNC, 0666);`修改为`  fsfd = open(argv[1], O_RDWR|O_CREAT|O_TRUNC|O_BINARY, 0666);`

`bzero(*, sizeof(de));`修改为`memset(*, 0, sizeof(de));`对，这个*代表任意长度字符串，让ai自己该也行

`assert(index(shortname, '/') == 0);`修改为`assert(strchr(shortname, '/') == 0);`

`open(argv[i], 0))`修改为`open(argv[i], O_RDONLY|O_BINARY))`

```
void
wsect(uint sec, void *buf)
{
  if(lseek(fsfd, sec * BSIZE, 0) != sec * BSIZE)
    die("lseek");
  if(write(fsfd, buf, BSIZE) != BSIZE)
    die("write");
}
```
修改为
```
void
wsect(uint sec, void *buf)
{
  off_t off = (off_t)sec * BSIZE;
  if(lseek(fsfd, off, 0) != off) {
    fprintf(stderr, "wsect: lseek failed sec=%u off=%lld errno=%d %s\n", sec, (long long)off, errno, strerror(errno));
    die("lseek");
  }
  ssize_t wr = write(fsfd, buf, BSIZE);
  if(wr != BSIZE) {
    fprintf(stderr, "wsect: write failed sec=%u off=%lld wrote=%zd errno=%d %s\n", sec, (long long)off, wr, errno, strerror(errno));
    die("write");
  }
}
```

```
void
rsect(uint sec, void *buf)
{
  if(lseek(fsfd, sec * BSIZE, 0) != sec * BSIZE)
    die("lseek");
  if(read(fsfd, buf, BSIZE) != BSIZE)
    die("read");
}
```
修改为
```
void
rsect(uint sec, void *buf)
{
  off_t off = (off_t)sec * BSIZE;
  if(lseek(fsfd, off, 0) != off) {
    fprintf(stderr, "rsect: lseek failed sec=%u off=%lld errno=%d %s\n", sec, (long long)off, errno, strerror(errno));
    die("lseek");
  }
  ssize_t rc = read(fsfd, buf, BSIZE);
  if(rc != BSIZE) {
    fprintf(stderr, "rsect: read failed sec=%u off=%lld read=%zd errno=%d %s\n", sec, (long long)off, rc, errno, strerror(errno));
    die("read");
  }
}

```

`bcopy(p, buf + off - (fbn * BSIZE), n1);`更改为`memmove(buf + off - (fbn * BSIZE), p, n1);`

~~因为有些是BSD，在Windows上不兼容~~

然后`make qemu`
开始写作业吧烧酒
![image](https://linux.do/uploads/default/optimized/4X/7/4/4/744b620d312cf023be66be495e543d50a9374b6d_2_1035x735.png)