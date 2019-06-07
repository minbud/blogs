---
title: linux-pipe-fifo-read
date: 2019-04-11 15:59:18
tags: linux
---

### 问题  
　　最近在写UNP书上关于pipe和fifo关于进程间通信的例子的时候，遇到了一些问题。  
　　书上是一个父进程fork出一个子进程，然后父进程执行client(), 子进程执行server(),最后父进程调用waitpid()等待子进程的结束。  
　　为什么不是父进程执行server(),子进程client()呢？关键点在于client、server由哪个进程执行和是否执行waitpid会影响程序的正常运行。在调换client、server的情况下，子进程无法顺利结束，而且父进程也会因此阻塞在waitpid中。  
<!--more-->
### 原因
　　server()的作用是等待client()发送来的文件名，打开相应文件并发文件内容发给client()，由client()打印出文件内容。从逻辑上看，server()会先于client()结束。  
　　另外client()调用read读取server()发送过来的文件内容，read()系统调用会在读到EOF时返回0，而pipe、fifo是利用写引用计数来判断当前写端数量的，并且会在写引用计数为0时，使得read读到EOF，从而返回０，结束等待。
　　使用fork创建进程时，pipe打开的读写端会同时在父子进程可用，即会增加引用计数，因此在fork后在父子进程上都需要先关闭不需要的读写端口。
    
　　综上，当父进程执行server，子进程执行client时,父进程先于子进程执行完毕，然后阻塞在waitpid等待子进程结束；而子进程则调用read将文件读取打印后阻塞等待read返回0，即等待父进程关闭写端口；此时陷入死锁，两个进程都无法结束。  
　　而调换顺序后，父进程执行client，子进程执行server，子进程先执行完毕，终止后系统自动关闭子进程打开的读写pipe、fifo端口，并发送EOF符号，此时父进程读取并打印完后，read读到EOF返回0，父进程结束client，执行waitpid等待子进程终止，并顺利退出并结束。  
    
### 解决方法
　　1. 调换server、client  
　　2. 执行server进程，退出server后，主动关闭写端口  
　　3. [参考－](https://bbs.csdn.net/topics/391870404) [参考二](https://stackoverflow.com/questions/5425938/using-eof-for-signaling-on-unnamed-pipes)　
    
```cpp
#include <stdio.h>
#include <unistd.h>
#include <string.h>
#include <fcntl.h>
#include <sys/stat.h>
#include <wait.h>

void client_process(int pRead, int pWrite);
void server_process(int pRead, int pWrite);
int pipe1[2], pipe2[2];
int main(int argc, char **argv) {

    if(pipe(pipe1)==-1 || pipe(pipe2)==-1)
    {
        printf("pipe error!\n");
        return 0;
    }

    int pid;
    if((pid = fork()) == 0)
    {
        close(pipe1[1]);
        close(pipe2[0]);
        client_process(pipe1[0], pipe2[1]);
        printf("client out\n");
        return 0;
    }

    close(pipe1[0]);
    close(pipe2[1]);
    server_process(pipe2[0], pipe1[1]);
    printf("server out\n");
    waitpid(pid, NULL, 0);
    return 0;
}
/*
#define FIFO1 "./fifoTest1"
#define FIFO2 "./fifoTest2"
#define FILEMODE (S_IRUSR | S_IWUSR | S_IRGRP | S_IROTH)

int main(int argc, char **argv) {
    int readfd, writefd;
    if(mkfifo(FIFO1, FILEMODE) < 0) {
        printf("mkfifo fifo1 failed\n");
        unlink(FIFO1);
        return 0;
    }
    else if(mkfifo(FIFO2, FILEMODE) < 0){
        printf("mkfifo fifo2 failed\n");
        unlink(FIFO2);
        return 0;
    }
    int pid;

    if((pid = fork()) == 0)
    {
        readfd = open(FIFO1, O_RDONLY, 0);
        writefd = open(FIFO2, O_WRONLY, 0);
        if(readfd<0 || writefd<0){
            printf("open failed\n");
            return 0;
        }
        //client_process(readfd, writefd);		// 调换顺序
        server_process(readfd, writefd);
        return 0;
    }

    writefd = open(FIFO1, O_WRONLY, 0);
    readfd = open(FIFO2, O_RDONLY, 0);
    if(readfd<0 || writefd<0){
        printf("open failed\n");
        return 0;
    }
    //server_process(readfd, writefd);
    client_process(readfd, writefd);

    waitpid(pid, NULL, 0);
    close(readfd);
    close(writefd);

    unlink(FIFO1);
    unlink(FIFO2);
    return 0;
}*/


#define MAXLINE 256

void client_process(int pRead, int pWrite)
{
    printf("client running\n");
    size_t len;
    ssize_t n;
    char buff[MAXLINE];

    if(fgets(buff, MAXLINE, stdin) < 0){
        printf("fgets error!\n");
        return;
    }

    len = strlen(buff);
    if(write(pWrite, buff, len) < 0){
        printf("write error!\n");
        return;
    }
    
    while((n = read(pRead, buff, MAXLINE)) != 0){
        write(STDOUT_FILENO, buff, n);
    }
    printf("client out\n");
    return;
}
void server_process(int pRead, int pWrite){
    printf("server running\n");
    int fd;
    ssize_t n;
    char buff[MAXLINE+1];

    if((n = read(pRead, buff, MAXLINE)) == 0){
        printf("read error!\n");
        return;
    }
    else
        printf("reading file: %s\n", buff);

    if(buff[n-1] == '\n')
        --n;
    buff[n]='\0';
    if((fd = open(buff, O_RDONLY)) < 0){
        sprintf(buff, "cannot open file!\n");
        write(pWrite, buff, strlen(buff));
        printf("server out\n");
        return;
    }
    else {
        while ((n = read(fd, buff, MAXLINE)) != 0)
            write(pWrite, buff, n);
        close(pWrite);				　　//写完之后主动关闭写端口
        close(fd);
    }
    printf("server out\n");
}
```
