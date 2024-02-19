linux作为一个系统，有自己的专属函数，称作系统函数。系统函数主要作为内核函数与用户之间的桥梁，使用户在内核上完成一些操作。系统内核的作用是调动硬件完成计算等功能。  

图1 用户-系统函数-内核

linux将所有硬件设置看作为文件，文件在系统中执行时，内核里面的进程控制块（PCB）会产生一个维护进程的文件描述表（结构体file），每个文件描述表里会存储该进程中打开、创建文件的文件描述符。而用户在对系统中的相关文件进行读写操作时，需要通过系统函数获取内核中存储的文件的文件描述符，用户则基于文件描述符对系统文件进行操作。

（1）文件描述符的获取——open打开文件
①对于本地已经存在的文件，若想对文件执行读写命令：

#include <unistd.h>
int fd = open("a.txt", O_RDWR); //fd 表示获取的可读写的文件描述符
O_RDWR表示以读写方式打开文件（O_RDONLY: 以只读方式打开文件，O_WRONLY: 以只写方式打开文件）。通过open命令可以让内核给文件分配文件描述符，如果想要释放文件描述符，可以通过系统函数close。

②创建一个新的文件，要添加 O_CREAT 属性, 并且给新文件指定操作权限：

    // 创建新文件
    int fd = open("./new.txt", O_CREAT|O_RDWR, 0664);
0664表示mode，属于八进制数，实际权限=（mode & ~umask掩码（0002））。

③在创建新文件的时候我们还可以通过 O_EXCL进行文件的检测, 如下:

    // 创建新文件之前, 先检测是否存在
    // 文件存在创建失败, 返回-1, 文件不存在创建成功, 返回分配的文件描述符
    int fd = open("./new.txt", O_CREAT|O_EXCL|O_RDWR);
（2）文件描述符的释放——close关闭文件
close(fd);// fd表示获取到的文件的文件描述符
（3）基于文件描述符的读写操作——read/write
read：输入为（文件描述符，读取内容存储指针，存储指针字节大小）

        输出为（读取内容的长度）；

write：输入为（文件描述符，内容存储指针，写入的长度）

        无输出。

        下面我们通过一个实例来讲述read与write的用法：

#include<stdio.h>
#include<stdlib.h>
#include<unistd.h>
#include<string.h>
#include<fcntl.h>
/*
功能：实现从read文件中读取，并将内容写入write文件
作者：ljl
*/
 
int main()
{
    //read.txt为当前目录下已创建的文件，fd1表示获取的对应的文件描述符
    int fd1=open("./read.txt", O_RDWR); 
    //创建写入的文件
    int fd2=open("./write.txt", O_RDWR|O_CREAT); 
    
    char buf[4096];//设置存储读取内容的字节大小
    int len;
    while ( (len=read(fd1,buf,sizeof(buf)))>0)
    {
        //将从fd1读到的数据buf通过fd2写入到write文件中
        write(fd2,buf,len); 
    }
    close(fd1);
    close(fd2);
    return 0;
}

（4）基于文件描述符的移动文件指针——lseek
lseek：输入为（文件描述符，偏移量，指定指针位置）

        输出为（变换后指针与头部的偏移量）。

        一般文件中指针的位置在文件头部（若不设置），我们可以通过lseek函数移动文件指针或进行文件大小的拓展。

        下面我们将介绍lseek的使用方式：

①将指针移动到文件头部（若指针不在头部）

lseek(fd, 0, SEEK_SET);
②返回当前指针的位置（返回值为当前指针与头部的偏移量）

lseek(fd, 0, SEEK_CUR); 
③移动指针到文件尾部

lseek(fd, 0, SEEK_END);
④文件的拓展，可以从文件尾部向后偏移x个字节

 lseek(fd, 1000, SEEK_END);
（5）系统函数异常输入——perror
        一般系统函数调用出错时，返回值一般为-1，并且错误所对应的错误号会被记录在全局变量errno中，通过调用perror函数可以打印错误号所对应的表述信息。使用方法如下：
