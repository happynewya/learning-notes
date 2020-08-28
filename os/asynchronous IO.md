### IO Multiplexing

Everything in Linux is a file. Linux describes files, sockets, devices with a **table of file descriptor**. To handle multiple messages from different files, sockets or devices, a mechanism to multiplexing I/O is pivotal in Linux system.

A simple solution is forking several threads. Each thread is responsible for reading messages from one specific source from a file id. Though this thought is intuitive, the performance will deteriorate when the number of threads scale up.

The solution is to use a kernel mechanism for polling over a set of file descriptors. 

#### Select

A call to select( ) will **block** until the given file descriptors are ready to perform I/O, or until an optionally specified timeout has elapsed

```c
int select(int nfds, fd_set *readfds, fd_set *writefds, fd_set *exceptfds, struct timeval *timeout);
```

1. 从用户空间拷贝 fd_set 到内核空间
2. 注册回调函数 __pollwait
3. 遍历所有 fd， 对全部指定设备做一次 poll（这里的 poll 是一个文件操作，它有两个参数，一个是文件 fd 本身，一个是当设备尚未就绪时调用的回调函数 __pollwait，这个函数把设备自己特有的等待队列传给内核，让内核把当前的进程挂载到其中）
4. 当设备就绪时，设备就会唤醒在自己特有等待队列中的所有节点，于是当前进程就获取到了完成的信号，poll 文件操作返回的是一组标准的掩码，其中的各个位指示当前的不同的就绪状态（全0为没有任何事件触发），根据 mask 可对 fd_set 赋值
5. 如何所有设备的返回的掩码都没有显示任何的事件触发，就去掉回调函数的函数指针，进入有限时的睡眠状态，再回复和不断做 poll，直到其中一个设备有事件触发为止
6. 只要有事件触发，系统调用返回，将 fd_set 从内核空间拷贝到用户空间，回到用户态，用户可以对相关的 fd 作进一步的度或者写操作

一个案例

```c
#include <stdio.h>
#include <sys/types.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <wait.h>
#include <signal.h>
#include <errno.h>
#include <sys/select.h>
#include <sys/time.h>
#include <unistd.h>
 
#define MAXBUF 256
 
void child_process(void)
{
  sleep(2);
  char msg[MAXBUF];
  struct sockaddr_in addr = {0};
  int n, sockfd,num=1;
  srandom(getpid());
  /* Create socket and connect to server */
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = inet_addr("127.0.0.1");
 
  connect(sockfd, (struct sockaddr*)&addr, sizeof(addr));
 
  printf("child {%d} connected \n", getpid());
  while(1){
        int sl = (random() % 10 ) +  1;
        num++;
     	sleep(sl);
  	sprintf (msg, "Test message %d from client %d", num, getpid());
  	n = write(sockfd, msg, strlen(msg));	/* Send message */
  }
 
}
 
int main()
{
  char buffer[MAXBUF];
  int fds[5];
  struct sockaddr_in addr;
  struct sockaddr_in client;
  int addrlen, n,i,max=0;;
  int sockfd, commfd;
  fd_set rset;
  for(i=0;i<5;i++)
  {
  	if(fork() == 0)
  	{
  		child_process();
  		exit(0);
  	}
  }
 
  sockfd = socket(AF_INET, SOCK_STREAM, 0);
  memset(&addr, 0, sizeof (addr));
  addr.sin_family = AF_INET;
  addr.sin_port = htons(2000);
  addr.sin_addr.s_addr = INADDR_ANY;
  bind(sockfd,(struct sockaddr*)&addr ,sizeof(addr));
  listen (sockfd, 5); 
 
  for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    fds[i] = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    if(fds[i] > max)
    	max = fds[i];
  }
  
  while(1){
	FD_ZERO(&rset);
  	for (i = 0; i< 5; i++ ) {
  		FD_SET(fds[i],&rset);
  	}
 
   	puts("round again");
	select(max+1, &rset, NULL, NULL, NULL);
 
	for(i=0;i<5;i++) {
		if (FD_ISSET(fds[i], &rset)){
			memset(buffer,0,MAXBUF);
			read(fds[i], buffer, MAXBUF);
			puts(buffer);
		}
	}	
  }
  return 0;
}
```



#### Poll

Poll 将select的可监听的Socket数上限1024改为无限制

```c
int poll (struct pollfd *fds, unsigned int nfds, int timeout);
struct pollfd {
      int fd;
      short events; 
      short revents;
};
```

#### Epoll

主要涉及到三个函数

```c
 // Create a function epollfd
int epoll_create(int size);

 // to listen for changes to increase the file descriptor is deleted
int epoll_ctl(int epfd, int op, int fd, struct epoll_event *event);
 
 // start blocking
int epoll_wait(int epfd, struct epoll_event *events, int maxevents, 

```

```c
 for (i=0;i<5;i++) 
  {
    memset(&client, 0, sizeof (client));
    addrlen = sizeof(client);
    pollfds[i].fd = accept(sockfd,(struct sockaddr*)&client, &addrlen);
    pollfds[i].events = POLLIN;
  }
  sleep(1);
  while(1){
  	puts("round again");
	poll(pollfds, 5, 50000);
 
	for(i=0;i<5;i++) {
		if (pollfds[i].revents & POLLIN){
			pollfds[i].revents = 0;
			memset(buffer,0,MAXBUF);
			read(pollfds[i].fd, buffer, MAXBUF);
			puts(buffer);
		}
	}
  }
```

