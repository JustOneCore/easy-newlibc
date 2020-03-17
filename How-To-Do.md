# How To Do

发布版本：1.0

作者邮箱：703632667@qq.com

日期：2020.03

文件密级：公开资料

------

**前言**

**概述**

如何移植newlib。

**读者对象**

本文档（本指南）主要适用于以下工程师：

技术支持工程师

软件开发工程师

**产品版本**

**修订记录**

| **日期**   | **版本** | **作者**  | **修改说明** |
| ---------- | -------- | --------- | ------------ |
| 2020-03-17 | V1.0.0   | Jason Zhu | 初始版本     |

------

[TOC]

------

## 1 引用参考

## 2 术语

## 3 简介

## 4 如何做

参考freedom-e-sdk\bsp\libwrap
在链接脚本添加的链接选项添加-Wl,--start-group -Wl,--end-group -Wl,--wrap=malloc -Wl,--wrap=free -Wl,--wrap=open -Wl,--wrap=lseek -Wl,--wrap=read -Wl,--wrap=write -Wl,--wrap=fstat -Wl,--wrap=stat -Wl,--wrap=close -Wl,--wrap=link -Wl,--wrap=unlink -Wl,--wrap=execve -Wl,--wrap=fork -Wl,--wrap=getpid -Wl,--wrap=kill -Wl,--wrap=wait -Wl,--wrap=isatty -Wl,--wrap=times -Wl,--wrap=sbrk -Wl,--wrap=_exit -Wl,--wrap=puts -Wl,--wrap=_malloc -Wl,--wrap=_free -Wl,--wrap=_open -Wl,--wrap=_lseek -Wl,--wrap=_read -Wl,--wrap=_write -Wl,--wrap=_fstat -Wl,--wrap=_stat -Wl,--wrap=_close -Wl,--wrap=_link -Wl,--wrap=_unlink -Wl,--wrap=_execve -Wl,--wrap=_fork -Wl,--wrap=_getpid -Wl,--wrap=_kill -Wl,--wrap=_wait -Wl,--wrap=_isatty -Wl,--wrap=_times -Wl,--wrap=_sbrk -Wl,--wrap=__exit -Wl,--wrap=_puts


misc\write_hex.c
/* See LICENSE of license details. */

#include <stdint.h>
#include <unistd.h>
#include "platform.h"

void write_hex(int fd, unsigned long int hex)
{
  uint8_t ii;
  uint8_t jj;
  char towrite;
  write(fd , "0x", 2);
  for (ii = sizeof(unsigned long int) * 2 ; ii > 0; ii--) {
    jj = ii - 1;
    uint8_t digit = ((hex & (0xF << (jj*4))) >> (jj*4));
    towrite = digit < 0xA ? ('0' + digit) : ('A' +  (digit - 0xA));
    write(fd, &towrite, 1);
  }
}


stdlib\malloc.c
/* See LICENSE for license details. */

/* These functions are intended for embedded RV32 systems and are
   obviously incorrect in general. */

void* __wrap_malloc(unsigned long sz)
{
  extern void* sbrk(long);
  void* res = sbrk(sz);
  if ((long)res == -1)
    return 0;
  return res;
}

void __wrap_free(void* ptr)
{
}


sys\_exit.c
/* See LICENSE of license details. */

#include <unistd.h>
#include "platform.h"
#include "weak_under_alias.h"

void __wrap_exit(int code)
{
  const char message[] = "\nProgam has exited with code:";

  write(STDERR_FILENO, message, sizeof(message) - 1);
  write_hex(STDERR_FILENO, code);
  write(STDERR_FILENO, "\n", 1);

  for (;;);
}
weak_under_alias(exit);


sys\close.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_close(int fd)
{
  return _stub(EBADF);
}
weak_under_alias(close);


sys\execve.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_execve(const char* name, char* const argv[], char* const env[])
{
  return _stub(ENOMEM);
}
weak_under_alias(execve);


sys\fork.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"

int fork(void)
{
  return _stub(EAGAIN);
}


sys\fstst.c
/* See LICENSE of license details. */

#include <errno.h>
#include <unistd.h>
#include <sys/stat.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_fstat(int fd, struct stat* st)
{
  if (isatty(fd)) {
    st->st_mode = S_IFCHR;
    return 0;
  }

  return _stub(EBADF);
}
weak_under_alias(fstat);


sys/getpid.c
/* See LICENSE of license details. */
#include "weak_under_alias.h"

int __wrap_getpid(void)
{
  return 1;
}
weak_under_alias(getpid);


sys\isatty.c
/* See LICENSE of license details. */

#include <unistd.h>
#include "weak_under_alias.h"

int __wrap_isatty(int fd)
{
  if (fd == STDOUT_FILENO || fd == STDERR_FILENO)
    return 1;

  return 0;
}
weak_under_alias(isatty);


sys\kill.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_kill(int pid, int sig)
{
  return _stub(EINVAL);
}
weak_under_alias(kill);


sys\link.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_link(const char *old_name, const char *new_name)
{
  return _stub(EMLINK);
}
weak_under_alias(link);


sys\lseek.c
/* See LICENSE of license details. */

#include <errno.h>
#include <unistd.h>
#include <sys/types.h>
#include "stub.h"
#include "weak_under_alias.h"

off_t __wrap_lseek(int fd, off_t ptr, int dir)
{
  if (isatty(fd))
    return 0;

  return _stub(EBADF);
}
weak_under_alias(lseek);


sys\open.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_open(const char* name, int flags, int mode)
{
  return _stub(ENOENT);
}
weak_under_alias(open);


sys\openat.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_openat(int dirfd, const char* name, int flags, int mode)
{
  return _stub(ENOENT);
}
weak_under_alias(openat);


sys/puts.c
/* See LICENSE of license details. */

#include <stdint.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>

#include "platform.h"
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_puts(const char *s)
{
  while (*s != '\0') {
    while (UART0_REG(UART_REG_TXFIFO) & 0x80000000) ;
    UART0_REG(UART_REG_TXFIFO) = *s;

    if (*s == '\n') {
      while (UART0_REG(UART_REG_TXFIFO) & 0x80000000) ;
      UART0_REG(UART_REG_TXFIFO) = '\r';
    }
    
    ++s;
  }

  return 0;
}
weak_under_alias(puts);


sys\read.c
/* See LICENSE of license details. */

#include <stdint.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>

#include "platform.h"
#include "stub.h"
#include "weak_under_alias.h"

ssize_t __wrap_read(int fd, void* ptr, size_t len)
{
  uint8_t * current = (uint8_t *)ptr;
  volatile uint32_t * uart_rx = (uint32_t *)(UART0_CTRL_ADDR + UART_REG_RXFIFO);
  volatile uint8_t * uart_rx_cnt = (uint8_t *)(UART0_CTRL_ADDR + UART_REG_RXCTRL + 2);

  ssize_t result = 0;

  if (isatty(fd)) {
    for (current = (uint8_t *)ptr;
        (current < ((uint8_t *)ptr) + len) && (*uart_rx_cnt > 0);
        current ++) {
      *current = *uart_rx;
      result++;
    }
    return result;
  }

  return _stub(EBADF);
}
weak_under_alias(read);


sys\sbrk.c
/* See LICENSE of license details. */

#include <stddef.h>
#include "weak_under_alias.h"

void *__wrap_sbrk(ptrdiff_t incr)
{
  extern char _end[];
  extern char _heap_end[];
  static char *curbrk = _end;

  if ((curbrk + incr < _end) || (curbrk + incr > _heap_end))
    return NULL - 1;

  curbrk += incr;
  return curbrk - incr;
}
weak_under_alias(sbrk);


sys/stat.c
/* See LICENSE of license details. */

#include <errno.h>
#include <sys/stat.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_stat(const char* file, struct stat* st)
{
  return _stub(EACCES);
}
weak_under_alias(stat);


sys\stub.h
/* See LICENSE of license details. */
#ifndef _SIFIVE_SYS_STUB_H
#define _SIFIVE_SYS_STUB_H

static inline int _stub(int err)
{
  return -1;
}

#endif /* _SIFIVE_SYS_STUB_H */


sys/times.c
/* See LICENSE of license details. */

#include <errno.h>
#include <sys/times.h>
#include "stub.h"
#include "weak_under_alias.h"

clock_t __wrap_times(struct tms* buf)
{
  return _stub(EACCES);
}
weak_under_alias(times);


sys\unlink.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"
#include "weak_under_alias.h"

int __wrap_unlink(const char* name)
{
  return _stub(ENOENT);
}
weak_under_alias(unlink);


sys\wait.c
/* See LICENSE of license details. */

#include <errno.h>
#include "stub.h"

int wait(int* status)
{
  return _stub(ECHILD);
}


sys\weak_under_alias.h
#ifndef _BSP_LIBWRAP_WEAK_UNDER_ALIAS_H
#define _BSP_LIBWRAP_WEAK_UNDER_ALIAS_H

#define weak_under_alias(name) \
  extern __typeof (__wrap_##name) __wrap__##name __attribute__ ((weak, alias ("__wrap_"#name)))

#endif


sys\write.c
/* See LICENSE of license details. */

#include <stdint.h>
#include <errno.h>
#include <unistd.h>
#include <sys/types.h>

#include "platform.h"
#include "stub.h"
#include "weak_under_alias.h"

ssize_t __wrap_write(int fd, const void* ptr, size_t len)
{
  const uint8_t * current = (const char *)ptr;

  if (isatty(fd)) {
    for (size_t jj = 0; jj < len; jj++) {
      while (UART0_REG(UART_REG_TXFIFO) & 0x80000000) ;
      UART0_REG(UART_REG_TXFIFO) = current[jj];

      if (current[jj] == '\n') {
        while (UART0_REG(UART_REG_TXFIFO) & 0x80000000) ;
        UART0_REG(UART_REG_TXFIFO) = '\r';
      }
    }
    return len;
  }

  return _stub(EBADF);
}
weak_under_alias(write);