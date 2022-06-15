# 201544098_tcp  // tcp/ip 네트워크 프로그래밍 
5주차 과제


![201544098 장종수](https://user-images.githubusercontent.com/90183114/162001241-4d48c86b-c304-4075-9d05-8f1668a44c82.PNG)

6주차 과제


![캡처](https://user-images.githubusercontent.com/90183114/163393594-5e558d8a-8bd4-446f-b94c-16861d6463a1.PNG)

오류가 발생했습니다...


![UDP](https://user-images.githubusercontent.com/90183114/163393662-b6e641ee-5770-4e6a-af0a-507a3c749dc9.png)


기말고사 code 분석
chat_ser.c
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pthread.h>

#define BUF_SIZE 100
#define MAX_CLNT 256

void * handle_clnt(void * arg);
//클라이언트로 부터 받은 메세지 처리
void send_msg(char * msg, int len);
//클라이언트로 부터 메시지를 받음
coid error_handling(char *msg);

int clnt_cnt = 0;
int clnt_socks[MAX_CLNT];
pthread_mutex_t mutx;
//mutex : 쓰레드 동시접근 해결

int main(int argc, char *argv[])
{
	int serv_sock, clnt_sock;
	struct sockaddr_in serv_adr, clnt_adr;
	int clnt_adr_sz;
pthread_t t_id;
if(argc!=2) { //prot 번호 받기
    printf("Usage : %s <port>\n", arg[0]);
    exit(1);
}

pthread_mutex_init(&mutx, NULL); //mutex 초기화
serv_sock = socket(PF_INET, SOCK_STREAM, 0);
//서버 소켓 연결

memset(&serv_adr, 0 , sizeof(serv_adr));
serv_adr.sin_family=AF_INET;
serv_adr.sin_addr.s_addr=htonl(INADDR_ANY);
serv_adr.sin_port=htons(atoi(argv[1]));

if(bind(serv_sock, (struct sockaddr*) &serv_adr, sizeof(serv_adr))==-1)
    error_handling("bind() error");
if(listen(serv_sock, 5)==-1)
    error_handling("listen() error");

    while(1) // 값 가져오기
{
    clnt_adr_sz=sizeof(clnt_adr);
    // 크기 할당
    clnt_sock=accept(serv_sock, (struct sockaddr*)&clnt_adr, &clnt_adr_sz);
    //소켓 할당
    pthread_mutex_lock(&mutx);
    clnt_socks[clnt_cnt++] = clnt_sock;
    // 클라이언트에서 소켓 받아오기
    pthread_mutex_unlock(&mutx);
    // 쓰레드 놓아줌

    pthread_create(&t_id, NULL, handle_clnt, (void*)&clnt_sock);
    //쓰레드 생성 및 handle_clnt 함수 실행
    pthread_detach(t_id);
    //pthread_detach 함수를 호출해서 쓰레드 소멸을 도움
    printf("Connected client IP: %s \n", inet_ntoa(clnt_adr.sin_addr));

}

close(serv_sock);
return 0;

}
void * handle_clnt(void * arg) 
//클라이언트가 보낸 메세지 처리
{
    int clnt_sock = *((int*)arg); 
    // 클라이언트 소켓 할당
    int str_len=0, i;
    char msg[BUF_SIZE];

    while((str_len=read(clnt_sock, msg, sizeof(msg)))!=0)
    //클라이언트가 보낸 메세지를 읽음
        send_msg(msg, str_len);

        
    pthread_mutex_lock(&mutx);
    //소켓 정보를 참조하는 동안 소켓의 추가 및 삭제를 막음(보호)
    for(i=0; i<clnt_cnt; i++)
    {
        if(clnt_sock==clnt_socks[i])
        {
            while(i++<clnt_cnt-1)
                clnt_socks[i]=clnt_socks[i+1];
            break;
        }
    }
    clnt_cnt--;
    pthread_mutex_unlock(&mutx);
    //쓰레드 보호 해제
    close(clnt_sock);
    return NULL;        
}

void send_msg(char * msg, int len)
//mutex로 잡고있는 모든 내용을 보냄
{
    int i;
    pthread_mutex_lock(&mutx);
    for(i=0; i<clnt_cnt; i++)
        write(clnt_socks[i], msg, len);
    pthread_mutex_unlock(&mutx);
}
void error_handling(char * msg)
//에러 핸들링 코드
{
        fputs(msg, stderr);
        fputc('\n', stderr);
        exit(1);
}
--------------------------------------------------------------------------------------chat_client.c
                         
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <string.h>
#include <arpa/inet.h>
#include <sys/socket.h>
#include <netinet/in.h>
#include <pthread.h>

#define BUF_SIZE 100
#define NAME_SIZE 20

void * send_msg(void * arg);
//메시지 전송
void * recv_msg(void * arg);
//메시지 receive
void error_handling(char * msg);
//error 처리
char name[NAME_SIZE] = "[DEFAULT]";
//이름 선언 및 초기화
char msg[BUF_SIZE];
//메세지 선언

int main(int argc, char *argv[])
{
    int sock;
    struct sockaddr_in serv_addr;
    pthread_t snd_thread, rcv_thread;
    // 쓰레드 전송 및 receive 쓰레드 선언
    void * thread_return;
    if(argc!=4) {
        printf("Usage : %s <IP> <port> <name> \n", argv[0]);
        exit(1);
    }

    sprintf(name, "[%s]", argv[3]);
    sock=socket(PF_INET, SOCK_STREAM, 0);
    //서버 소켓 연결

    memset(&serv_addr, 0, sizeof(serv_addr));
    //초기화
    serv_addr.sin_family = AF_INET;
    serv_addr.sin_addr.s_addr=inet_addr(argv[1]);
    serv_addr.sin_port=htons(atoi(argv[2]));

    if(connect(sock, (struct sockaddr*)&serv_addr, sizeof(serv_addr))==-1)
        error_handling("connect() error");

    pthread_create(&snd_thread, NULL, send_msg, (void*)&sock);
    //메세지 전송
    pthread_create(&rcv_thread, NULL, recv_msg, (void*)&sock);
    //메세지 receive
    pthread_join(snd_thread, &thread_return);
    //pthread - 전송 쓰레드 소멸
    pthread_join(rcv_thread, &thread_return);
    //pthread - receive 쓰레드 소멸
    close(sock);
    return 0;

}

void * send_msg(void * arg) 
// 전송 쓰레드 처리
{
    int sock = *((int*)arg);
    char name_msg[NAME_SIZE+BUF_SIZE];
    while(1)
    {
        fgets(msg, BUF_SIZE, stdin);
        if(!strcmp(msg,"q\n")||!strcmp(msg, "Q\n"))
        {
            close(sock);
            exit(0);
        }
        sprintf(name_msg, "%s %s", name, msg);
        write(sock, name_msg, strlen(name_msg));
    }
    return NULL;
}

void * recv_msg(void * arg)
// receive 쓰레드 처리
{
    int sock=*((int*)arg);
    char name_msg[NAME_SIZE+BUF_SIZE];
    int str_len;
    while(1)
    {
        str_len=read(sock, name_msg, NAME_SIZE+BUF_SIZE-1);
        if(str_len==-1)
            return(void*)-1;
        name_msg[str_len]=0;
        fputs(name_msg, stdout);
    }
    return NULL;

}

void error_handling(char *msg)
{
    fputs(msg, stderr);
    fputc('\n', stderr);
    exit(1);
}

