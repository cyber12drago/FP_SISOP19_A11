# FP_SISOP19_A11
Buatlah program C yang menyerupai crontab menggunakan daemon dan thread. Ada sebuah file crontab.data untuk menyimpan config dari crontab. Setiap ada perubahan file tersebut maka secara otomatis program menjalankan config yang sesuai dengan perubahan tersebut tanpa perlu diberhentikan. Config hanya sebatas * dan 0-9 (tidak perlu /,- dan yang lainnya)
```
#include <sys/types.h>
#include <sys/stat.h>
#include <stdio.h>
#include <stdlib.h>
#include <fcntl.h>
#include <errno.h>
#include <unistd.h>
#include <syslog.h>
#include <string.h>
#include <sys/time.h>
#include <time.h>
#include <pthread.h> 

pthread_t tid[1000];

struct cront
{
    char minutes[105];
    char hours[105];
    char day_month[105];
    char month[105];
    char day_week[105];
    char s1[1005];
    char s2[1005];
};

int cek_thread=0;
void* cek_cron(void *cektime)
{
    FILE* fp;
    fp=fopen("/home/diondevara/FP/crontab.data","r");
    time_t cur_time=time(NULL);
    struct tm tm=*localtime(&cur_time);
    struct cront* waktu=cektime;
     

    if(tm.tm_min==atoi(waktu->minutes)|| strcmp(waktu->minutes,"*")==0){
      if(tm.tm_hour==atoi(waktu->hours) || strcmp(waktu->hours,"*")==0){
        if(tm.tm_mday==atoi(waktu->day_month)|| strcmp(waktu->day_month,"*")==0){
          if(tm.tm_mon+1==atoi(waktu->month)|| strcmp(waktu->month,"*")==0){
          if(tm.tm_wday==atoi(waktu->day_week) || strcmp(waktu->day_week,"*")==0){
            pid_t cron;
            cron=fork();
            if(cron==0)
            {
              char *argv[]={waktu->s1,waktu->s2,NULL};
              execv(waktu->s1,argv);
                sleep(5);
            }
          }
          }
        }
      } 
  }
}



int main() {
  pid_t pid, sid;

  pid = fork();

  if (pid < 0) {
    exit(EXIT_FAILURE);
  }

  if (pid > 0) {
    exit(EXIT_SUCCESS);
  }

  umask(0);

  sid = setsid();

  if (sid < 0) {
    exit(EXIT_FAILURE);
  }

  if ((chdir("/home/diondevara/FP/")) < 0) {
    exit(EXIT_FAILURE);
  }

  close(STDIN_FILENO);
  close(STDOUT_FILENO);
  close(STDERR_FILENO);


  int err,i,j,z,v=0;
  char data[105];
  pthread_t tid[100];
  while(1) {
      printf("assa");
      struct stat st;
      strcpy(data,"crontab.data");
      if(stat(data,&st)==0){
        //times = st.st_mtime;
        FILE *isi_file = fopen("crontab.data", "r");
        char temp[1000][1000]={'\0'},c;
        i=0;j=0;
        do{
          c=fgetc(isi_file);  
          if(c!='\n'){
            temp[i][j] = c;j++;  
          }
          else
          {
            i++;j=0;
          }
        }while(c!=EOF);

        cek_thread=0;
        int x=0,y;
        i=0,j=0;
        while(temp[i][j]!='\0'){
          struct cront* cek_time;
          int cek=1;
          x++;
          while(temp[i][j]!='\0'){
            y=0;
            while(temp[i][j]!=' '&&temp[i][j]!='\0'){
               if(cek==1){
                cek_time->minutes[y]=temp[i][j];
                y++;
              }
              else if(cek==2){
                cek_time->hours[y]=temp[i][j];
                y++;
              }
              else if(cek==3){
                cek_time->day_month[y]=temp[i][j];
                y++;
      
              }
              else if(cek==4){
                cek_time->month[y]=temp[i][j];
                y++;
              }
              else if(cek==5){
                cek_time->day_week[y]=temp[i][j];
                y++;
              }
              else if(cek==6){
                cek_time->s1[y]=temp[i][j];
                y++;
              }
              else if(cek==7){
                cek_time->s2[y]=temp[i][j];
                y++;
              }
              j++;
            }
            cek++;
            j++;
          }
          j=1;i++;cek=1;
          //printf("%s %s %s %s %s %s %s",cek_time.minutes,cek_time.hours,cek_time.day_month,cek_time.month,cek_time.day_week,cek_time.s1,cek_time.s2);
          pthread_create(&(tid[z]), NULL,&cek_cron, (void*)cek_time);
          z++;
        }
        fclose(isi_file);
        if(v!=z){
            for(i=v;i<z;i++){
              pthread_join(tid[i], NULL);
            }
            v=z;
        }
      }
    }
  
  exit(EXIT_SUCCESS);
}
```
## Langkah-langkah
1. Open file crontab.data
2. Baca input config cron dari file
3. Cek apakah config seharusnya dijalankan atau tidak pada waktu ini
4. Jika iya maka eksekusi script config
