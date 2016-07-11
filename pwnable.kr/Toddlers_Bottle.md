
## fd
---
首先 ssh 到 `fd@pwnable.kr -p2222` (password:guest)，发现目录下有三个文件：

不可读的 flag 文件，一个 c 文件，以及它的二进制文件。查看 c 文件：

```c
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
char buf[32];
int main(int argc, char* argv[], char* envp[]){
        if(argc<2){
                printf("pass argv[1] a number\n");
                return 0;
        }
        int fd = atoi( argv[1] ) - 0x1234;
        int len = 0;
        len = read(fd, buf, 32);
        if(!strcmp("LETMEWIN\n", buf)){
                printf("good job :)\n");
                system("/bin/cat flag");
                exit(0);
        }
        printf("learn about Linux file IO\n");
        return 0;

}
```

首先程序需要传一个整数作为参数，然后把其减去 0x1234 的值赋值给 fd，接着 [read 函数](http://linux.die.net/man/2/read) 从 fd 中读取 32 个字节到 buf 的字符数组中。如果 buf 中的字符串与 “LETMEWIN\n” 相同，则可以获取到 flag，解题结束。

### 解法

传入参数 4660，使得 fd 为 0, 然后从命令行中输入字符串 “LETMEWIN\n”，即将其读取到 buf 中，得到答案。代码如下：

```
fd@ubuntu:~$ ./fd 4660  (or ./fd `python -c "print(0x1234)"`)
LETMEWIN
good job :)
mommy! I think I know what a file descriptor is!!
```

### 知识点

Linux 下的 [文件描述符概念](https://zh.wikipedia.org/wiki/%E6%96%87%E4%BB%B6%E6%8F%8F%E8%BF%B0%E7%AC%A6) (File descriptor)

由于 Linux 的设计思想是把一切设备都视作文件。因此，文件描述符为在该平台上进行设备相关的编程实际上提供了一个统一的方法。文件描述符在形式上是一个非负整数。实际上，它是一个索引值，指向内核为每一个进程所维护的该进程打开文件的记录表。当程序打开一个现有文件或者创建一个新文件时，内核向进程返回一个文件描述符。

**最常用的三个文件描述符即，控制台 (Console) 的标准输入，标准输出，和标准错误输出，分别对应 0, 1, 2。**


## collision
---
首先 ssh 到 `col@pwnable.kr -p2222` (password:guest)，发现目录下有三个文件：

不可读的 flag 文件，一个 c 文件，以及它的二进制文件。查看 c 文件：

```c
#include <stdio.h>
#include <string.h>
unsigned long hashcode = 0x21DD09EC;
unsigned long check_password(const char* p){
        int* ip = (int*)p;
        int i;
        int res=0;
        for(i=0; i<5; i++){
                res += ip[i];
        }
        return res;
}

int main(int argc, char* argv[]){
        if(argc<2){
                printf("usage : %s [passcode]\n", argv[0]);
                return 0;
        }
        if(strlen(argv[1]) != 20){
                printf("passcode length should be 20 bytes\n");
                return 0;
        }

        if(hashcode == check_password( argv[1] )){
                system("/bin/cat flag");
                return 0;
        }
        else
                printf("wrong passcode.\n");
        return 0;
}
```

首先程序需要传入一个长度为 20 的字符串，然后它经过函数处理后得到的返回值需要与 0x21DD09EC 相等。  
对于这个函数，传入的参数为 char 型指针，其指向的字符串长度为 20 字节。将这 20 个字节分为 5 部分，每一部分 4 个字节，强转为 int 类型。这 5 个 int 值相加的和即为返回值。  

### 解法

将 0x21DD09EC 拆分为 5 部分，如四个 0x01010101 加一个 0x1dd905e8 (0x21DD09EC = 0x01010101 * 4 + 0x1dd905e8) 然后将其拼成要求的字符串。注意使用小端序！代码如下：

```
col@ubuntu:~$ ./col `python -c "print '\x01' * 16 + '\xe8\x05\xd9\x1d'"`
daddy! I just managed to create a hash collision :)
```

### 知识点

字符串和整型在内存中的存取方法。（想了好久才想明白，底层知识不扎实）  
0x00 没有对应的字符  
**小端序**和**大端序**的问题，放两个链接 [1](http://www.cnblogs.com/graphics/archive/2011/04/22/2010662.html) [2](http://www.cnblogs.com/flysnail/archive/2011/10/25/2223721.html)  



