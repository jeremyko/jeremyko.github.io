---
layout: post
title: Linux Asynchronous I/O 예제
date: '2012-10-26T12:02:00.001+09:00'
tags:
    - c++
    - linux
    - aio
modified_time: '2022-03-26T21:03:13.726+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6639121789432227087
blogger_orig_url: https://jeremyko.blogspot.com/2012/10/linux-asynchronous-io.html
---

지금 프로젝트에서 작성한 프로그램의 개선점을 찾다보니, 비동기 작업 처리를 위해 aio를 이용하면 좋을듯 하다.

아래 사이트 내용을 참조하여 완전한 예제로 구성해보았다.

[http://www.ibm.com/developerworks/linux/library/l-async/](http://www.ibm.com/developerworks/linux/library/l-async/)

컴파일은 gcc -lrt -o aioTest aioTest.c 으로 수행.

file.txt 내용은 다음과 같다.

    123456789abcdefghijklmnopqrstuvwxyz

---

```cpp
//aioTest.c
#include <aio.h>
#include <stdio.h>
#include <stdlib.h>
#include <errno.h>
#include <string.h>

// gcc -lrt -o aioTest aioTest.c
// librt" stands for "real time library".

//////////////////////////////////////////////////////////////////
// busy-wait until the status changes
void testBusyWait()
{
    printf("\n\n==================================================\n");
    int fd, ret;
    //int BUFSIZE = 10240;
    int BUFSIZE = 5;
    struct aiocb my_aiocb;

    fd = open( "file.txt", O_RDONLY );
    if (fd < 0) perror("open");

    /* Zero out the aiocb structure (recommended) */
    bzero( (char *)&my_aiocb, sizeof(struct aiocb) );

    /* Allocate a data buffer for the aiocb request */
    my_aiocb.aio_buf = malloc(BUFSIZE+1);

    if (!my_aiocb.aio_buf) perror("malloc");

    /* Initialize the necessary fields in the aiocb */
    my_aiocb.aio_fildes = fd;
    my_aiocb.aio_nbytes = BUFSIZE;
    my_aiocb.aio_offset = 0; //the first offset in the file

    //below block aio /////////////////////////////////
    int MAX_LIST = 1;
    struct aioct *cblist[MAX_LIST];
    /* Clear the list. */
    bzero( (char *)cblist, sizeof(cblist) );
    /* Load one or more references into the list */
    cblist[0] = &my_aiocb;
    //below block aio /////////////////////////////////
    ret = aio_read( &my_aiocb );

    //below block aio /////////////////////////////////
    ret = aio_suspend( cblist, MAX_LIST, NULL );
    //below block aio /////////////////////////////////

    if (ret < 0) perror("aio_read");

    while ( aio_error( &my_aiocb ) == EINPROGRESS ) {
        printf("busy waiting...\n");
    }

    if ((ret = aio_return( &my_aiocb )) > 0) {
        /* got ret bytes on the read */
        printf("ret [%d]\n", ret);
        //print buffer
        printf("buff[%s]\n",my_aiocb.aio_buf);
    } else {
        /* read failed, consult errno */
        printf("read failed\n");
    }
}

//////////////////////////////////////////////////////////////////
// lio_listio , Initiate a list of I/O operations
// with busy waiting
void testListio()
{
    printf("\n\n==================================================\n");
    int fd, ret;
    //int BUFSIZE = 10240;
    int BUFSIZE = 5;
    int MAX_LIST = 2;
    struct aiocb aiocb1, aiocb2;
    struct aiocb *list[MAX_LIST];

    fd = open( "file.txt", O_RDONLY );
    if (fd < 0) perror("open");

    /* Zero out the aiocb structure (recommended) */
    bzero( (char *)&aiocb1, sizeof(struct aiocb) );
    bzero( (char *)&aiocb2, sizeof(struct aiocb) );

    //aiocb1
    aiocb1.aio_fildes = fd;
    aiocb1.aio_buf = malloc(BUFSIZE+1);
    aiocb1.aio_nbytes = BUFSIZE;
    aiocb1.aio_offset = 0; //the first offset in the file
    aiocb1.aio_lio_opcode = LIO_READ;
    //aiocb2
    aiocb2.aio_fildes = fd;
    aiocb2.aio_buf = malloc(BUFSIZE+1);
    aiocb2.aio_nbytes = BUFSIZE;
    aiocb2.aio_offset = BUFSIZE; //!!!
    aiocb2.aio_lio_opcode = LIO_READ;

    bzero( (char *)list, sizeof(list) );
    list[0] = &aiocb1;
    list[1] = &aiocb2;

    //WAIT
    //ret = lio_listio( LIO_WAIT, list, MAX_LIST, NULL );

    // NO_WAIT
    ret = lio_listio( LIO_NOWAIT, list, MAX_LIST, NULL );
    while ( aio_error( &aiocb1 ) == EINPROGRESS ||
            aio_error( &aiocb2 ) == EINPROGRESS )
    {
        printf("busy waiting...\n");
    }
    // NO_WAIT

    //aiocb1
    if ((ret = aio_return( &aiocb1 )) > 0) {
        printf("\n******\n");
        printf("aiocb1 ret [%d]\n", ret);
        printf("aiocb1 buff[%s]\n",aiocb1.aio_buf);
    } else {
        /* read failed, consult errno */
        printf("aiocb1 read failed\n");
    }

    //aiocb2
    if ((ret = aio_return( &aiocb2 )) > 0)
    {
        printf("\n******\n");
        printf("aiocb2 ret [%d]\n", ret);
        printf("aiocb2 buff[%s]\n",aiocb2.aio_buf);
    } else {
        /* read failed, consult errno */
        printf("aiocb2 read failed\n");
    }
}

//////////////////////////////////////////////////////////////////
//aio with callback
//also available : Asynchronous notification with signals

struct aiocb my_aiocb;

void aio_completion_handler( sigval_t sigval )
{
    printf("aio_completion_handler\n");
    struct aiocb *req;

    req = (struct aiocb *)sigval.sival_ptr;

    /* Did the request complete? */
    if (aio_error( req ) == 0) {

        /* Request completed successfully, get the return status */
        int ret = aio_return( req );
        printf("ret [%d]\n", ret);
        printf("buff[%s]\n",req->aio_buf);
    }
    return;
}


void testCallback()
{
    printf("\n\n==================================================\n");
    int fd, ret;
    //int BUFSIZE = 10240;
    int BUFSIZE = 20;

    fd = open( "file.txt", O_RDONLY );
    if (fd < 0) perror("open");

    /* Set up the AIO request */
    bzero( (char *)&my_aiocb, sizeof(struct aiocb) );
    my_aiocb.aio_fildes = fd;
    my_aiocb.aio_buf = malloc(BUFSIZE+1);
    my_aiocb.aio_nbytes = BUFSIZE;
    my_aiocb.aio_offset = 0;

    /* Link the AIO request with a thread callback */
    my_aiocb.aio_sigevent.sigev_notify = SIGEV_THREAD;
    my_aiocb.aio_sigevent.sigev_notify_function = aio_completion_handler;
    my_aiocb.aio_sigevent.sigev_notify_attributes = NULL;
    my_aiocb.aio_sigevent.sigev_value.sival_ptr = &my_aiocb;

    ret = aio_read( &my_aiocb );
}


//////////////////////////////////////////////////////////////////
int main(int argc,char** argv)
{
    testBusyWait();
    testListio();
    testCallback();

    while(1) {
        sleep(1);
    }
}
```

수행 결과는 다음과 같다.

    ==============================================================
    ret [5]
    buff[12345]

    ==============================================================
    aiocb1 busy waiting...
    aiocb1 busy waiting...
    .....
    aiocb1 busy waiting...

    ******
    aiocb1 ret [5]
    aiocb1 buff[12345]

    ******
    aiocb2 ret [5]
    aiocb2 buff[6789a]

    ==============================================================
    aio_completion_handler
    ret [20]
    buff[123456789abcdefghijk]
