# OS Practical – CS-305-MJ-P (SPPU T.Y.B.Sc. C.S. NEP-2020, Sem V)

Operating System-I · Slipwise C codes. Har code block ke top-right mein GitHub ka copy button aata hai.

**Run:** `gcc file.c -o file && ./file < file.in`  (`nice` ke liye `sudo`)

## Slips

[1](#slip-1) [2](#slip-2) [3](#slip-3) [4](#slip-4) [5](#slip-5) [6](#slip-6) [7](#slip-7) [8](#slip-8) [9](#slip-9) [10](#slip-10) [11](#slip-11) [12](#slip-12) [13](#slip-13) [14](#slip-14) [15](#slip-15) [16](#slip-16) [17](#slip-17) [18](#slip-18) [19](#slip-19) [20](#slip-20) [21](#slip-21) [22](#slip-22) [23](#slip-23) [24](#slip-24) [25](#slip-25) [26](#slip-26) [27](#slip-27) [28](#slip-28) [29](#slip-29) [30](#slip-30)

**Notes:** Slips 1, 4, 11 ka Banker data inconsistent hai (Need negative aata hai), examiner se puchho. SCAN/C-SCAN me disk size 200 maana hai. `cscan` 199→0 jump count karta hai (378; bina jump 179). `count`/`search` ke liye `sample.txt` chahiye (neeche last me diya hai).

---

## Slip 1

### Slip 1 – Q.1: Banker menu: Accept Available, Display Allocation/Max, Need  (`bankmenu.c`)

```c
/*
Q.1  Write a C program to simulate Banker's algorithm for the purpose of deadlock avoidance.
Implement following functionality (menu driven).
a. Accept Available
b. Display Allocation, Max
c. Display the contents of need matrix
d. Display Available
Process Allocation Max Available
 A B C A B C A B C
P0 2 3 2 9 7 5 3 3 2
P1 4 0 0 5 2 2
P2 5 0 4 1 0 4
P3 4 3 3 4 4 4
P4 2 2 4 6 5 5
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_alloc_max()
{
int i,j;
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
}
void accept_avail()
{
int j;
printf("Enter Available resources: ");
for(j=0;j<m;j++)
scanf("%d",&avail[j]);
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int main()
{
int ch;
do
{
printf("\n1.Accept Allocation and Max\n2.Accept Available\n3.Display Allocation and Max\n4.Display Need matrix\n5.Display Available\n0.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: accept_alloc_max(); break;
case 2: accept_avail(); break;
case 3: show_matrix("Allocation",alloc); show_matrix("Max",max); break;
case 4: calc_need(); show_matrix("Need",need); break;
case 5: show_avail(); break;
}
}
while(ch!=0);
return 0;
}
```

### Slip 1 – Q.2: shell with count command  (`count.c`)

```c
/*
Q.2  Write a C program that behaves like a shell which displays the command prompt '$'. It accepts the command, tokenize the command line and execute it by creating the child process. Also implement the additional command 'count' as
a. $ count c filename : It will display the number of characters in given file
b. $ count w filename : It will display the number of words in given file
c. $ count l filename : It will display the number of lines in given file
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
void count(char *opt,char *fname)
{
FILE *fp=fopen(fname,"r");
int ch,c=0,w=0,l=0,inword=0;
if(!fp)
{
printf("Cannot open %s\n",fname);
return;
}
while((ch=fgetc(fp))!=EOF)
{
c++;
if(ch=='\n')
l++;
if(ch==' ' || ch=='\n' || ch=='\t')
inword=0;
else if(!inword)
{
inword=1;
w++;
}
}
fclose(fp);
if(opt[0]=='c') printf("Characters = %d\n",c);
else if(opt[0]=='w') printf("Words = %d\n",w);
else if(opt[0]=='l') printf("Lines = %d\n",l);
else printf("Use c, w or l\n");
}
int main()
{
char cmd[200],*arg[32];
int i;
while(1)
{
printf("$ ");
fflush(stdout);
if(!fgets(cmd,200,stdin))
break;
for(i=0;i<30 && (arg[i]=strtok(i?NULL:cmd," \t\n"));i++);
arg[i]=NULL;
if(i==0)
continue;
if(strcmp(arg[0],"exit")==0)
break;
if(strcmp(arg[0],"count")==0 && i==3)
count(arg[1],arg[2]);
else if(fork()==0)
{
execvp(arg[0],arg);
perror("command");
exit(1);
}
else
wait(NULL);
}
return 0;
}
```

Input (`count.in`):

```text
count c sample.txt
count w sample.txt
count l sample.txt
exit
```

---

## Slip 2

### Slip 2 – Q.1: Linked file allocation  (`linked.c`)

```c
/*
Q.1  Write a program to simulate Linked file allocation method. Assume disk with n number of blocks. Give value of n as input. Randomly mark some block as allocated and accordingly maintain the list of free blocks. Write menu driven program with menu options as mentioned below and implement each option.
a. Show Bit Vector
b. Create New File
c. Show Directory
d. Exit
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
int n,cnt=0,bit[100];
struct File { char name[20]; int len,blk[50]; } f[20];
void show_bits()
{
int i;
printf("Bit vector: ");
for(i=0;i<n;i++)
printf("%d",bit[i]);
printf("\n");
}
int pick()
{
int i;
for(i=0;i<n;i++)
if(!bit[i])
{
bit[i]=1;
return i;
}
return -1;
}
int free_count()
{
int i,c=0;
for(i=0;i<n;i++)
if(!bit[i])
c++;
return c;
}
void create()
{
struct File x;
int k;
if(cnt==20)
{
printf("Directory full\n");
return;
}
printf("Enter file name and size (blocks): ");
scanf("%s %d",x.name,&x.len);
if(x.len<1 || x.len>49 || free_count()<x.len)
{
printf("Cannot create file\n");
return;
}
for(k=0;k<x.len;k++)
x.blk[k]=pick();
f[cnt++]=x;
printf("File created\n");
}
void show_dir()
{
int i,k;
if(cnt==0)
printf("Directory is empty\n");
for(i=0;i<cnt;i++)
{
printf("%s : start = %d : ",f[i].name,f[i].blk[0]);
for(k=0;k<f[i].len;k++)
printf("%d -> ",f[i].blk[k]);
printf("NULL\n");
}
}
int main()
{
int i,ch;
srand(time(0));
printf("Enter number of disk blocks: ");
scanf("%d",&n);
for(i=0;i<n;i++)
bit[i]=(rand()%100<25);
do
{
printf("\n1.Show Bit Vector\n2.Create New File\n3.Show Directory\n4.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: show_bits(); break;
case 2: create(); break;
case 3: show_dir(); break;
}
}
while(ch!=4);
return 0;
}
```

Input (`linked.in`):

```text
20
2
f1 3
2
f2 2
3
1
4
```

### Slip 2 – Q.2: FCFS disk scheduling  (`fcfsdisk.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using FCFS algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
55, 58, 39, 18, 90, 160, 150, 38, 184
Start Head Position: 50
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50];
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
pos=head;
printf("Order served: ");
for(i=0;i<n;i++)
go(req[i]);
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`fcfsdisk.in`):

```text
200
9
55 58 39 18 90 160 150 38 184
50
```

---

## Slip 3

### Slip 3 – Q.1: Banker: need matrix + safe state / safe sequence  (`banksafe.c`)

```c
/*
Q.1  Write a C program to simulate Banker's algorithm for the purpose of deadlock avoidance. Consider the following snapshot of system, A, B, C and D is the resource type.
Process Allocation Max Available
 A B C D A B C D A B C D
P0 0 0 1 2 0 0 1 2 1 5 2 0
P1 1 0 0 0 1 7 5 0
P2 1 3 5 4 2 3 5 6
P3 0 6 3 2 0 6 5 2
P4 0 0 1 4 0 6 5 6
a) Calculate and display the content of need matrix?
b) Is the system in safe state? If so display the safe sequence.
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_alloc_max()
{
int i,j;
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
}
void accept_avail()
{
int j;
printf("Enter Available resources: ");
for(j=0;j<m;j++)
scanf("%d",&avail[j]);
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int safe()
{
int work[10],fin[10]={0},seq[10],cnt=0,i,j,found,ok;
calc_need();
for(j=0;j<m;j++)
work[j]=avail[j];
while(cnt<n)
{
found=0;
for(i=0;i<n;i++)
{
if(fin[i]) continue;
ok=1;
for(j=0;j<m;j++)
if(need[i][j]>work[j])
{
ok=0;
break;
}
if(ok)
{
for(j=0;j<m;j++)
work[j]+=alloc[i][j];
fin[i]=1;
seq[cnt++]=i;
found=1;
}
}
if(!found)
{
printf("System is NOT in safe state\n");
return 0;
}
}
printf("System is in SAFE state\nSafe sequence: ");
for(i=0;i<n;i++)
printf("P%d ",seq[i]);
printf("\n");
return 1;
}
int main()
{
accept_alloc_max();
accept_avail();
calc_need();
show_matrix("Need",need);
safe();
return 0;
}
```

Input (`banksafe.in`):

```text
5 4
0 0 1 2 1 0 0 0 1 3 5 4 0 6 3 2 0 0 1 4
0 0 1 2 1 7 5 0 2 3 5 6 0 6 5 2 0 6 5 6
1 5 2 0
```

### Slip 3 – Q.2: SSTF disk scheduling  (`sstf.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using SSTF algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
98, 183, 37, 122, 14, 124, 65
Start Head Position: 53
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],k,best,done[50]={0};
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
pos=head;
printf("Order served: ");
for(k=0;k<n;k++)
{
best=-1;
for(i=0;i<n;i++)
if(!done[i] && (best==-1 || abs(req[i]-pos)<abs(req[best]-pos)))
best=i;
done[best]=1;
go(req[best]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`sstf.in`):

```text
200
7
98 183 37 122 14 124 65
53
```

---

## Slip 4

### Slip 4 – Q.1: Banker menu: Accept Available, Display Allocation/Max, Need  (`bankmenu.c`)

```c
/*
Q.1  Write a C program to simulate Banker's algorithm for the purpose of deadlock avoidance.
Implement following functionality (menu driven).
a. Accept Available
b. Display Allocation, Max
c. Display the contents of need matrix
d. Display Available
Process Allocation Max Available
 A B C A B C A B C
P0 2 3 2 9 7 5 3 3 2
P1 4 0 0 5 2 2
P2 5 0 4 1 0 4
P3 4 3 3 4 4 4
P4 2 2 4 6 5 5
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_alloc_max()
{
int i,j;
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
}
void accept_avail()
{
int j;
printf("Enter Available resources: ");
for(j=0;j<m;j++)
scanf("%d",&avail[j]);
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int main()
{
int ch;
do
{
printf("\n1.Accept Allocation and Max\n2.Accept Available\n3.Display Allocation and Max\n4.Display Need matrix\n5.Display Available\n0.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: accept_alloc_max(); break;
case 2: accept_avail(); break;
case 3: show_matrix("Allocation",alloc); show_matrix("Max",max); break;
case 4: calc_need(); show_matrix("Need",need); break;
case 5: show_avail(); break;
}
}
while(ch!=0);
return 0;
}
```

### Slip 4 – Q.2: SCAN disk scheduling  (`scan.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using SCAN algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
82, 170, 43, 140, 24, 16, 190, 65
Start Head Position: 50
Direction: Left
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k,dir;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
printf("Enter direction (1=Right, 0=Left): ");
scanf("%d",&dir);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
if(dir==1)
{
for(k=i;k<n;k++)
go(req[k]);
go(size-1);
for(k=i-1;k>=0;k--)
go(req[k]);
}
else
{
for(k=i-1;k>=0;k--)
go(req[k]);
go(0);
for(k=i;k<n;k++)
go(req[k]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`scan.in`):

```text
200
8
82 170 43 140 24 16 190 65
50
0
```

---

## Slip 5

### Slip 5 – Q.1: FIFO page replacement  (`fifo.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the FIFO page replacement algorithm. Assume the memory of n frames.
Reference String: 3,4,5,6,3,4,7,3,4,5,6,7,2,4,6
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],load[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit==-1)
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(load[i]<load[v]) v=i;
}
fr[v]=ref[t];
load[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`fifo.in`):

```text
3
15
3 4 5 6 3 4 7 3 4 5 6 7 2 4 6
```

### Slip 5 – Q.2: Banker: resource request granted or not  (`bankreq.c`)

```c
/*
Q.2  Consider a system with 'n' processes and 'm' resource types. Accept number of instances for every resource type. For each process accept the allocation and maximum requirement matrices. Write a program to display the contents of need matrix and to check if the given request of a process can be granted immediately or not. (Use resource request algorithm)
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_total()
{
int i,j,total[10];
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter total instances of each resource type: ");
for(j=0;j<m;j++)
scanf("%d",&total[j]);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
for(j=0;j<m;j++)
{
avail[j]=total[j];
for(i=0;i<n;i++)
avail[j]-=alloc[i][j];
}
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int safe()
{
int work[10],fin[10]={0},seq[10],cnt=0,i,j,found,ok;
calc_need();
for(j=0;j<m;j++)
work[j]=avail[j];
while(cnt<n)
{
found=0;
for(i=0;i<n;i++)
{
if(fin[i]) continue;
ok=1;
for(j=0;j<m;j++)
if(need[i][j]>work[j])
{
ok=0;
break;
}
if(ok)
{
for(j=0;j<m;j++)
work[j]+=alloc[i][j];
fin[i]=1;
seq[cnt++]=i;
found=1;
}
}
if(!found)
{
printf("System is NOT in safe state\n");
return 0;
}
}
printf("System is in SAFE state\nSafe sequence: ");
for(i=0;i<n;i++)
printf("P%d ",seq[i]);
printf("\n");
return 1;
}
void request()
{
int p,j,req[10];
calc_need();
printf("Enter process number: ");
scanf("%d",&p);
printf("Enter request: ");
for(j=0;j<m;j++)
scanf("%d",&req[j]);
for(j=0;j<m;j++)
{
if(req[j]>need[p][j])
{
printf("Error: request exceeds maximum claim\n");
return;
}
if(req[j]>avail[j])
{
printf("Process must wait: resources not available\n");
return;
}
}
for(j=0;j<m;j++)
{
avail[j]-=req[j];
alloc[p][j]+=req[j];
}
if(safe())
printf("Request can be granted immediately\n");
else
{
for(j=0;j<m;j++)
{
avail[j]+=req[j];
alloc[p][j]-=req[j];
}
printf("Request cannot be granted immediately\n");
}
}
int main()
{
accept_total();
calc_need();
show_matrix("Need",need);
show_avail();
request();
return 0;
}
```

Input (`bankreq.in`):

```text
5 3
10 5 7
0 1 0 2 0 0 3 0 2 2 1 1 0 0 2
7 5 3 3 2 2 9 0 2 2 2 2 4 3 3
1
1 0 2
```

---

## Slip 6

### Slip 6 – Q.1: Sequential (contiguous) file allocation  (`seq.c`)

```c
/*
Q.1  Write a program to simulate Sequential file allocation method. Assume disk with n number of blocks. Give value of n as input. Randomly mark some block as allocated and accordingly maintain the list of free blocks. Write menu driven program with menu options as mentioned below and implement each option.
a. Show Bit Vector
b. Create New File
c. Show Directory
d. Exit
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
int n,cnt=0,bit[100];
struct File { char name[20]; int start,len; } f[20];
void show_bits()
{
int i;
printf("Bit vector: ");
for(i=0;i<n;i++)
printf("%d",bit[i]);
printf("\n");
}
void create()
{
struct File x;
int i,k;
if(cnt==20)
{
printf("Directory full\n");
return;
}
printf("Enter file name and size (blocks): ");
scanf("%s %d",x.name,&x.len);
if(x.len<1)
{
printf("Invalid size\n");
return;
}
for(i=0;i+x.len<=n;i++)
{
for(k=i;k<i+x.len && !bit[k];k++);
if(k==i+x.len)
break;
}
if(i+x.len>n)
{
printf("Not enough contiguous free blocks\n");
return;
}
x.start=i;
for(k=i;k<i+x.len;k++)
bit[k]=1;
f[cnt++]=x;
printf("File created\n");
}
void show_dir()
{
int i;
if(cnt==0)
printf("Directory is empty\n");
for(i=0;i<cnt;i++)
printf("%s : start = %d, length = %d\n",f[i].name,f[i].start,f[i].len);
}
int main()
{
int i,ch;
srand(time(0));
printf("Enter number of disk blocks: ");
scanf("%d",&n);
for(i=0;i<n;i++)
bit[i]=(rand()%100<25);
do
{
printf("\n1.Show Bit Vector\n2.Create New File\n3.Show Directory\n4.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: show_bits(); break;
case 2: create(); break;
case 3: show_dir(); break;
}
}
while(ch!=4);
return 0;
}
```

Input (`seq.in`):

```text
20
2
f1 3
2
f2 2
3
1
4
```

### Slip 6 – Q.2: C-SCAN (circular scan)  (`cscan.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using C-SCAN algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head movements. (Assume disk size=200)
15, 45, 75, 105, 135, 165, 195, 80
Start Head Position: 100
Direction: Right
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
for(k=i;k<n;k++)
go(req[k]);
go(size-1);
go(0);
for(k=0;k<i;k++)
go(req[k]);
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`cscan.in`):

```text
200
8
15 45 75 105 135 165 195 80
100
```

---

## Slip 7

### Slip 7 – Q.1: Banker: need matrix + safe state / safe sequence  (`banksafe.c`)

```c
/*
Q.1  Write a C program to simulate Banker's algorithm for the purpose of deadlock avoidance. Consider the following snapshot of system, A, B, C and D is the resource type.
Process Allocation Max Available
 A B C D A B C D A B C D
P0 0 0 1 2 0 0 1 2 1 5 2 0
P1 1 0 0 0 1 7 5 0
P2 1 3 5 4 2 3 5 6
P3 0 6 3 2 0 6 5 2
P4 0 0 1 4 0 6 5 6
a) Calculate and display the content of need matrix?
b) Is the system in safe state? If so display the safe sequence.
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_alloc_max()
{
int i,j;
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
}
void accept_avail()
{
int j;
printf("Enter Available resources: ");
for(j=0;j<m;j++)
scanf("%d",&avail[j]);
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int safe()
{
int work[10],fin[10]={0},seq[10],cnt=0,i,j,found,ok;
calc_need();
for(j=0;j<m;j++)
work[j]=avail[j];
while(cnt<n)
{
found=0;
for(i=0;i<n;i++)
{
if(fin[i]) continue;
ok=1;
for(j=0;j<m;j++)
if(need[i][j]>work[j])
{
ok=0;
break;
}
if(ok)
{
for(j=0;j<m;j++)
work[j]+=alloc[i][j];
fin[i]=1;
seq[cnt++]=i;
found=1;
}
}
if(!found)
{
printf("System is NOT in safe state\n");
return 0;
}
}
printf("System is in SAFE state\nSafe sequence: ");
for(i=0;i<n;i++)
printf("P%d ",seq[i]);
printf("\n");
return 1;
}
int main()
{
accept_alloc_max();
accept_avail();
calc_need();
show_matrix("Need",need);
safe();
return 0;
}
```

Input (`banksafe.in`):

```text
5 4
0 0 1 2 1 0 0 0 1 3 5 4 0 6 3 2 0 0 1 4
0 0 1 2 1 7 5 0 2 3 5 6 0 6 5 2 0 6 5 6
1 5 2 0
```

### Slip 7 – Q.2: nice() system call  (`nice.c`)

```c
/*
Q.2  Write a program that demonstrates the use of nice() system call. After a child process is started using fork(), assign higher priority to the child using nice() system call.
*/
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int pid=fork();
if(pid==0)
{
printf("Child : nice value before = %d\n",nice(0));
errno=0;
if(nice(-5)==-1 && errno!=0)
perror("nice (run with sudo)");
printf("Child : nice value after = %d\n",nice(0));
}
else
{
printf("Parent: nice value = %d\n",nice(0));
wait(NULL);
}
return 0;
}
```

---

## Slip 8

### Slip 8 – Q.1: Index file allocation  (`index.c`)

```c
/*
Q.1  Write a program to simulate Index file allocation method. Assume disk with n number of blocks. Give value of n as input. Randomly mark some block as allocated and accordingly maintain the list of free blocks. Write menu driven program with menu options as mentioned below and implement each option.
a. Show Bit Vector
b. Create New File
c. Show Directory
d. Exit
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
int n,cnt=0,bit[100];
struct File { char name[20]; int len,idx,blk[50]; } f[20];
void show_bits()
{
int i;
printf("Bit vector: ");
for(i=0;i<n;i++)
printf("%d",bit[i]);
printf("\n");
}
int pick()
{
int i;
for(i=0;i<n;i++)
if(!bit[i])
{
bit[i]=1;
return i;
}
return -1;
}
int free_count()
{
int i,c=0;
for(i=0;i<n;i++)
if(!bit[i])
c++;
return c;
}
void create()
{
struct File x;
int k;
if(cnt==20)
{
printf("Directory full\n");
return;
}
printf("Enter file name and size (blocks): ");
scanf("%s %d",x.name,&x.len);
if(x.len<1 || x.len>49 || free_count()<x.len+1)
{
printf("Cannot create file\n");
return;
}
x.idx=pick();
for(k=0;k<x.len;k++)
x.blk[k]=pick();
f[cnt++]=x;
printf("File created\n");
}
void show_dir()
{
int i,k;
if(cnt==0)
printf("Directory is empty\n");
for(i=0;i<cnt;i++)
{
printf("%s : index block = %d -> ",f[i].name,f[i].idx);
for(k=0;k<f[i].len;k++)
printf("%d ",f[i].blk[k]);
printf("\n");
}
}
int main()
{
int i,ch;
srand(time(0));
printf("Enter number of disk blocks: ");
scanf("%d",&n);
for(i=0;i<n;i++)
bit[i]=(rand()%100<25);
do
{
printf("\n1.Show Bit Vector\n2.Create New File\n3.Show Directory\n4.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: show_bits(); break;
case 2: create(); break;
case 3: show_dir(); break;
}
}
while(ch!=4);
return 0;
}
```

Input (`index.in`):

```text
20
2
f1 3
2
f2 2
3
1
4
```

### Slip 8 – Q.2: nice() system call  (`nice.c`)

```c
/*
Q.2  Write a program that demonstrates the use of nice() system call. After a child process is started using fork(), assign higher priority to the child using nice() system call.
*/
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int pid=fork();
if(pid==0)
{
printf("Child : nice value before = %d\n",nice(0));
errno=0;
if(nice(-5)==-1 && errno!=0)
perror("nice (run with sudo)");
printf("Child : nice value after = %d\n",nice(0));
}
else
{
printf("Parent: nice value = %d\n",nice(0));
wait(NULL);
}
return 0;
}
```

---

## Slip 9

### Slip 9 – Q.1: Optimal page replacement  (`opt.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the optimal page replacement algorithm. Assume the memory of n frames.
Reference String: 8, 5, 7, 8, 5, 7, 2, 3, 7, 3, 5, 9, 4, 6, 2
*/
#include <stdio.h>
int n,ref[50];
int nextuse(int page,int from)
{
int k;
for(k=from;k<n;k++)
if(ref[k]==page)
return k;
return 1000;
}
int main()
{
int f,i,t,v,hit,faults=0,fr[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit==-1)
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(nextuse(fr[i],t+1)>nextuse(fr[v],t+1)) v=i;
}
fr[v]=ref[t];
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`opt.in`):

```text
3
15
8 5 7 8 5 7 2 3 7 3 5 9 4 6 2
```

### Slip 9 – Q.2: Non-preemptive Shortest Job First  (`sjfnp.c`)

```c
/*
Q.2  Write the program to simulate Non-preemptive Shortest Job First (SJF) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || bt[i]<bt[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`sjfnp.in`):

```text
4
0 7
2 4
4 1
5 4
```

---

## Slip 10

### Slip 10 – Q.1: Round Robin  (`rr.c`)

```c
/*
Q.1  Write the program to simulate Round Robin (RR) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. Also give the time quantum as input. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,q,t=0,done=0,p,run,s=0,head=0,tail=0,at[20],bt[20],rem[20],ct[20],inq[20]={0},queue[500],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
printf("Enter time quantum: ");
scanf("%d",&q);
while(done<n)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(head==tail)
{
gantt[t++]=-1;
continue;
}
p=queue[head++];
run=rem[p]<q?rem[p]:q;
while(run--)
{
gantt[t++]=p;
rem[p]--;
}
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(rem[p]>0)
queue[tail++]=p;
else
{
ct[p]=t;
done++;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`rr.in`):

```text
3
0 5
1 3
2 1
2
```

### Slip 10 – Q.2: LOOK disk scheduling  (`look.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using LOOK algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
55, 58, 39, 18, 90, 160, 150, 38
Start Head Position: 100
Direction: Left
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k,dir;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
printf("Enter direction (1=Right, 0=Left): ");
scanf("%d",&dir);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
if(dir==1)
{
for(k=i;k<n;k++)
go(req[k]);
for(k=i-1;k>=0;k--)
go(req[k]);
}
else
{
for(k=i-1;k>=0;k--)
go(req[k]);
for(k=i;k<n;k++)
go(req[k]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`look.in`):

```text
200
8
55 58 39 18 90 160 150 38
100
0
```

---

## Slip 11

### Slip 11 – Q.1: Banker menu: Accept Available, Display Allocation/Max, Need  (`bankmenu.c`)

```c
/*
Q.1  Write a C program to simulate Banker's algorithm for the purpose of deadlock avoidance.
Implement following functionality (menu driven).
a. Accept Available
b. Display Allocation, Max
c. Display the contents of need matrix
d. Display Available
Process Allocation Max Available
 A B C A B C A B C
P0 2 3 2 9 7 5 3 3 2
P1 4 0 0 5 2 2
P2 5 0 4 1 0 4
P3 4 3 3 4 4 4
P4 2 2 4 6 5 5
*/
#include <stdio.h>
int n,m,alloc[10][10],max[10][10],need[10][10],avail[10];
void accept_alloc_max()
{
int i,j;
printf("Enter number of processes and resource types: ");
scanf("%d %d",&n,&m);
printf("Enter Allocation matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&alloc[i][j]);
printf("Enter Max matrix:\n");
for(i=0;i<n;i++)
for(j=0;j<m;j++)
scanf("%d",&max[i][j]);
}
void accept_avail()
{
int j;
printf("Enter Available resources: ");
for(j=0;j<m;j++)
scanf("%d",&avail[j]);
}
void calc_need()
{
int i,j;
for(i=0;i<n;i++)
for(j=0;j<m;j++)
need[i][j]=max[i][j]-alloc[i][j];
}
void show_matrix(char *title,int a[10][10])
{
int i,j;
printf("%s:\n",title);
for(i=0;i<n;i++)
{
printf("P%d: ",i);
for(j=0;j<m;j++)
printf("%d ",a[i][j]);
printf("\n");
}
}
void show_avail()
{
int j;
printf("Available: ");
for(j=0;j<m;j++)
printf("%d ",avail[j]);
printf("\n");
}
int main()
{
int ch;
do
{
printf("\n1.Accept Allocation and Max\n2.Accept Available\n3.Display Allocation and Max\n4.Display Need matrix\n5.Display Available\n0.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: accept_alloc_max(); break;
case 2: accept_avail(); break;
case 3: show_matrix("Allocation",alloc); show_matrix("Max",max); break;
case 4: calc_need(); show_matrix("Need",need); break;
case 5: show_avail(); break;
}
}
while(ch!=0);
return 0;
}
```

### Slip 11 – Q.2: Optimal page replacement  (`opt.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the optimal page replacement algorithm. Assume the memory of n frames.
Reference String: 8, 5, 7, 8, 5, 7, 2, 3, 7, 3, 5, 9, 4, 6, 2
*/
#include <stdio.h>
int n,ref[50];
int nextuse(int page,int from)
{
int k;
for(k=from;k<n;k++)
if(ref[k]==page)
return k;
return 1000;
}
int main()
{
int f,i,t,v,hit,faults=0,fr[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit==-1)
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(nextuse(fr[i],t+1)>nextuse(fr[v],t+1)) v=i;
}
fr[v]=ref[t];
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`opt.in`):

```text
3
15
8 5 7 8 5 7 2 3 7 3 5 9 4 6 2
```

---

## Slip 12

### Slip 12 – Q.1: FCFS CPU scheduling  (`fcfscpu.c`)

```c
/*
Q.1  Write the program to simulate FCFS CPU scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || at[i]<at[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`fcfscpu.in`):

```text
3
0 5
1 3
2 1
```

### Slip 12 – Q.2: bubble sort (parent) + insertion sort (child)  (`sort.c`)

```c
/*
Q.2  Implement the C program to accept n integers to be sorted. Main function creates child process using fork system call. Parent process sorts the integers using bubble sort and waits for child process using wait system call. Child process sorts the integers using insertion sort.
*/
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int a[20],n,i,j,t,key;
printf("Enter number of integers: ");
scanf("%d",&n);
printf("Enter integers: ");
for(i=0;i<n;i++)
scanf("%d",&a[i]);
fflush(stdout);
if(fork()==0)
{
for(i=1;i<n;i++)
{
key=a[i];
for(j=i-1;j>=0 && a[j]>key;j--)
a[j+1]=a[j];
a[j+1]=key;
}
printf("Child (insertion sort): ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
}
else
{
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(a[j]>a[j+1])
{
t=a[j];
a[j]=a[j+1];
a[j+1]=t;
}
printf("Parent (bubble sort): ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
wait(NULL);
}
return 0;
}
```

Input (`sort.in`):

```text
5
9 3 7 1 5
```

---

## Slip 13

### Slip 13 – Q.1: Non-preemptive Shortest Job First  (`sjfnp.c`)

```c
/*
Q.2  Write the program to simulate Non-preemptive Shortest Job First (SJF) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || bt[i]<bt[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`sjfnp.in`):

```text
4
0 7
2 4
4 1
5 4
```

### Slip 13 – Q.2: LOOK disk scheduling  (`look.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using LOOK algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
55, 58, 39, 18, 90, 160, 150, 38
Start Head Position: 100
Direction: Left
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k,dir;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
printf("Enter direction (1=Right, 0=Left): ");
scanf("%d",&dir);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
if(dir==1)
{
for(k=i;k<n;k++)
go(req[k]);
for(k=i-1;k>=0;k--)
go(req[k]);
}
else
{
for(k=i-1;k>=0;k--)
go(req[k]);
for(k=i;k<n;k++)
go(req[k]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`look.in`):

```text
200
8
55 58 39 18 90 160 150 38
100
0
```

---

## Slip 14

### Slip 14 – Q.1: execve() + binary search  (`execve.c`)

```c
/*
Q.1  Implement the C program that accepts an integer array. Main function forks child process. Parent process sorts an integer array and passes the sorted array to child process through the command line arguments of execve() system call. The child process uses execve() system call to load new program that uses this sorted array for performing the binary search the particular item in the array.
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc,char *argv[])
{
int a[20],n,i,j,t,key,lo,hi,mid;
char buf[22][12],*args[24];
if(argc>1)
{
key=atoi(argv[1]);
n=argc-2;
for(i=0;i<n;i++)
a[i]=atoi(argv[i+2]);
printf("New program received sorted array: ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
lo=0;
hi=n-1;
while(lo<=hi)
{
mid=(lo+hi)/2;
if(a[mid]==key)
{
printf("%d found at position %d\n",key,mid+1);
return 0;
}
if(a[mid]<key)
lo=mid+1;
else
hi=mid-1;
}
printf("%d not found\n",key);
return 0;
}
printf("Enter number of integers: ");
scanf("%d",&n);
printf("Enter integers: ");
for(i=0;i<n;i++)
scanf("%d",&a[i]);
printf("Enter item to search: ");
scanf("%d",&key);
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(a[j]>a[j+1])
{
t=a[j];
a[j]=a[j+1];
a[j+1]=t;
}
args[0]=argv[0];
sprintf(buf[0],"%d",key);
args[1]=buf[0];
for(i=0;i<n;i++)
{
sprintf(buf[i+1],"%d",a[i]);
args[i+2]=buf[i+1];
}
args[n+2]=NULL;
fflush(stdout);
if(fork()==0)
{
execve("/proc/self/exe",args,NULL);
perror("execve");
exit(1);
}
wait(NULL);
return 0;
}
```

Input (`execve.in`):

```text
5
40 10 30 20 50
30
```

### Slip 14 – Q.2: FIFO page replacement  (`fifo.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the FIFO page replacement algorithm. Assume the memory of n frames.
Reference String: 3,4,5,6,3,4,7,3,4,5,6,7,2,4,6
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],load[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit==-1)
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(load[i]<load[v]) v=i;
}
fr[v]=ref[t];
load[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`fifo.in`):

```text
3
15
3 4 5 6 3 4 7 3 4 5 6 7 2 4 6
```

---

## Slip 15

### Slip 15 – Q.1: orphan process  (`orphan.c`)

```c
/*
Q.1  Write a C program to illustrate the concept of orphan process. Parent process creates a child and terminates before child has finished its task. So child process becomes orphan process. (Use fork(), sleep(), getpid(), getppid())
*/
#include <stdio.h>
#include <unistd.h>
int main()
{
int pid=fork();
if(pid>0)
{
printf("Parent: PID = %d, child PID = %d\n",getpid(),pid);
printf("Parent terminating\n");
sleep(1);
}
else
{
printf("Child : PID = %d, parent PID = %d\n",getpid(),getppid());
sleep(3);
printf("Child : PID = %d, new parent PID = %d\n",getpid(),getppid());
}
return 0;
}
```

### Slip 15 – Q.2: Preemptive Priority scheduling  (`prip.c`)

```c
/*
Q.2  Write the program to simulate Preemptive Priority scheduling. The arrival time and first CPU-burst and priority for different n number of processes should be input to the algorithm. The next CPU-burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],pr[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time, burst time and priority of P%d: ",i+1);
scanf("%d %d %d",&at[i],&bt[i],&pr[i]);
rem[i]=bt[i];
}
while(done<n)
{
cur=-1;
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || pr[i]<pr[cur]))
cur=i;
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`prip.in`):

```text
3
0 4 2
1 3 1
2 1 3
```

---

## Slip 16

### Slip 16 – Q.1: LRU page replacement (counter)  (`lru.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the LRU (using counter method) page replacement algorithm. Assume the memory of n frames.
Reference String: 3,5,7,2,5,1,2,3,1,3,5,3,1,6,2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],last[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
last[hit]=t;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(last[i]<last[v]) v=i;
}
fr[v]=ref[t];
last[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`lru.in`):

```text
3
15
3 5 7 2 5 1 2 3 1 3 5 3 1 6 2
```

### Slip 16 – Q.2: shell with count command  (`count.c`)

```c
/*
Q.2  Write a C program that behaves like a shell which displays the command prompt '$'. It accepts the command, tokenize the command line and execute it by creating the child process. Also implement the additional command 'count' as
a. $ count c filename : It will display the number of characters in given file
b. $ count w filename : It will display the number of words in given file
c. $ count l filename : It will display the number of lines in given file
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
void count(char *opt,char *fname)
{
FILE *fp=fopen(fname,"r");
int ch,c=0,w=0,l=0,inword=0;
if(!fp)
{
printf("Cannot open %s\n",fname);
return;
}
while((ch=fgetc(fp))!=EOF)
{
c++;
if(ch=='\n')
l++;
if(ch==' ' || ch=='\n' || ch=='\t')
inword=0;
else if(!inword)
{
inword=1;
w++;
}
}
fclose(fp);
if(opt[0]=='c') printf("Characters = %d\n",c);
else if(opt[0]=='w') printf("Words = %d\n",w);
else if(opt[0]=='l') printf("Lines = %d\n",l);
else printf("Use c, w or l\n");
}
int main()
{
char cmd[200],*arg[32];
int i;
while(1)
{
printf("$ ");
fflush(stdout);
if(!fgets(cmd,200,stdin))
break;
for(i=0;i<30 && (arg[i]=strtok(i?NULL:cmd," \t\n"));i++);
arg[i]=NULL;
if(i==0)
continue;
if(strcmp(arg[0],"exit")==0)
break;
if(strcmp(arg[0],"count")==0 && i==3)
count(arg[1],arg[2]);
else if(fork()==0)
{
execvp(arg[0],arg);
perror("command");
exit(1);
}
else
wait(NULL);
}
return 0;
}
```

Input (`count.in`):

```text
count c sample.txt
count w sample.txt
count l sample.txt
exit
```

---

## Slip 17

### Slip 17 – Q.1: Non-preemptive Shortest Job First  (`sjfnp.c`)

```c
/*
Q.2  Write the program to simulate Non-preemptive Shortest Job First (SJF) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || bt[i]<bt[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`sjfnp.in`):

```text
4
0 7
2 4
4 1
5 4
```

### Slip 17 – Q.2: Sequential file allocation with Delete File  (`seqdel.c`)

```c
/*
Q.2  Write a program to simulate Sequential (Contiguous) file allocation method. Assume disk with n number of blocks. Give value of n as input. Randomly mark some block as allocated and accordingly maintain the list of free blocks. Write menu driven program with menu options as mentioned below and implement each option.
a. Show Bit Vector
b. Create New File
c. Show Directory
d. Delete File
e. Exit
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
int n,cnt=0,bit[100];
struct File { char name[20]; int start,len; } f[20];
void show_bits()
{
int i;
printf("Bit vector: ");
for(i=0;i<n;i++)
printf("%d",bit[i]);
printf("\n");
}
void create()
{
struct File x;
int i,k;
if(cnt==20)
{
printf("Directory full\n");
return;
}
printf("Enter file name and size (blocks): ");
scanf("%s %d",x.name,&x.len);
if(x.len<1)
{
printf("Invalid size\n");
return;
}
for(i=0;i+x.len<=n;i++)
{
for(k=i;k<i+x.len && !bit[k];k++);
if(k==i+x.len)
break;
}
if(i+x.len>n)
{
printf("Not enough contiguous free blocks\n");
return;
}
x.start=i;
for(k=i;k<i+x.len;k++)
bit[k]=1;
f[cnt++]=x;
printf("File created\n");
}
void show_dir()
{
int i;
if(cnt==0)
printf("Directory is empty\n");
for(i=0;i<cnt;i++)
printf("%s : start = %d, length = %d\n",f[i].name,f[i].start,f[i].len);
}
void delete_file()
{
char nm[20];
int i,k;
printf("Enter file name to delete: ");
scanf("%s",nm);
for(i=0;i<cnt;i++)
if(strcmp(f[i].name,nm)==0)
{
for(k=f[i].start;k<f[i].start+f[i].len;k++)
bit[k]=0;
f[i]=f[--cnt];
printf("File deleted\n");
return;
}
printf("File not found\n");
}
int main()
{
int i,ch;
srand(time(0));
printf("Enter number of disk blocks: ");
scanf("%d",&n);
for(i=0;i<n;i++)
bit[i]=(rand()%100<25);
do
{
printf("\n1.Show Bit Vector\n2.Create New File\n3.Show Directory\n4.Delete File\n5.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: show_bits(); break;
case 2: create(); break;
case 3: show_dir(); break;
case 4: delete_file(); break;
}
}
while(ch!=5);
return 0;
}
```

Input (`seqdel.in`):

```text
20
2
f1 3
2
f2 2
3
4
f1
3
5
```

---

## Slip 18

### Slip 18 – Q.1: Preemptive SJF  (`sjfp.c`)

```c
/*
Q.1  Write the program to simulate Preemptive Shortest Job First (SJF) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
cur=-1;
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || rem[i]<rem[cur]))
cur=i;
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`sjfp.in`):

```text
4
0 8
1 4
2 9
3 5
```

### Slip 18 – Q.2: orphan process  (`orphan.c`)

```c
/*
Q.1  Write a C program to illustrate the concept of orphan process. Parent process creates a child and terminates before child has finished its task. So child process becomes orphan process. (Use fork(), sleep(), getpid(), getppid())
*/
#include <stdio.h>
#include <unistd.h>
int main()
{
int pid=fork();
if(pid>0)
{
printf("Parent: PID = %d, child PID = %d\n",getpid(),pid);
printf("Parent terminating\n");
sleep(1);
}
else
{
printf("Child : PID = %d, parent PID = %d\n",getpid(),getppid());
sleep(3);
printf("Child : PID = %d, new parent PID = %d\n",getpid(),getppid());
}
return 0;
}
```

---

## Slip 19

### Slip 19 – Q.1: Preemptive Priority scheduling  (`prip.c`)

```c
/*
Q.2  Write the program to simulate Preemptive Priority scheduling. The arrival time and first CPU-burst and priority for different n number of processes should be input to the algorithm. The next CPU-burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],pr[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time, burst time and priority of P%d: ",i+1);
scanf("%d %d %d",&at[i],&bt[i],&pr[i]);
rem[i]=bt[i];
}
while(done<n)
{
cur=-1;
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || pr[i]<pr[cur]))
cur=i;
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`prip.in`):

```text
3
0 4 2
1 3 1
2 1 3
```

### Slip 19 – Q.2: nice() system call  (`nice.c`)

```c
/*
Q.2  Write a program that demonstrates the use of nice() system call. After a child process is started using fork(), assign higher priority to the child using nice() system call.
*/
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int pid=fork();
if(pid==0)
{
printf("Child : nice value before = %d\n",nice(0));
errno=0;
if(nice(-5)==-1 && errno!=0)
perror("nice (run with sudo)");
printf("Child : nice value after = %d\n",nice(0));
}
else
{
printf("Parent: nice value = %d\n",nice(0));
wait(NULL);
}
return 0;
}
```

---

## Slip 20

### Slip 20 – Q.1: LOOK disk scheduling  (`look.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using LOOK algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
55, 58, 39, 18, 90, 160, 150, 38
Start Head Position: 100
Direction: Left
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k,dir;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
printf("Enter direction (1=Right, 0=Left): ");
scanf("%d",&dir);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
if(dir==1)
{
for(k=i;k<n;k++)
go(req[k]);
for(k=i-1;k>=0;k--)
go(req[k]);
}
else
{
for(k=i-1;k>=0;k--)
go(req[k]);
for(k=i;k<n;k++)
go(req[k]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`look.in`):

```text
200
8
55 58 39 18 90 160 150 38
100
0
```

### Slip 20 – Q.2: LRU page replacement (counter)  (`lru.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the LRU (using counter method) page replacement algorithm. Assume the memory of n frames.
Reference String: 3,5,7,2,5,1,2,3,1,3,5,3,1,6,2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],last[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
last[hit]=t;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(last[i]<last[v]) v=i;
}
fr[v]=ref[t];
last[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`lru.in`):

```text
3
15
3 5 7 2 5 1 2 3 1 3 5 3 1 6 2
```

---

## Slip 21

### Slip 21 – Q.1: MFU page replacement  (`mfu.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the MFU page replacement algorithm. Assume the memory of n frames.
Reference String: 8, 5, 7, 8, 5, 7, 2, 3, 7, 3, 5, 9, 4, 6, 2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],cnt[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
cnt[hit]++;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(cnt[i]>cnt[v]) v=i;
}
fr[v]=ref[t];
cnt[v]=1;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`mfu.in`):

```text
3
15
8 5 7 8 5 7 2 3 7 3 5 9 4 6 2
```

### Slip 21 – Q.2: nice() system call  (`nice.c`)

```c
/*
Q.2  Write a program that demonstrates the use of nice() system call. After a child process is started using fork(), assign higher priority to the child using nice() system call.
*/
#include <stdio.h>
#include <errno.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int pid=fork();
if(pid==0)
{
printf("Child : nice value before = %d\n",nice(0));
errno=0;
if(nice(-5)==-1 && errno!=0)
perror("nice (run with sudo)");
printf("Child : nice value after = %d\n",nice(0));
}
else
{
printf("Parent: nice value = %d\n",nice(0));
wait(NULL);
}
return 0;
}
```

---

## Slip 22

### Slip 22 – Q.1: Linked file allocation with Delete File  (`linkdel.c`)

```c
/*
Q.1  Write a program to simulate Linked file allocation method. Assume disk with n number of blocks. Give value of n as input. Randomly mark some block as allocated and accordingly maintain the list of free blocks. Write menu driven program with menu options as mentioned below and implement each option.
a. Show Bit Vector
b. Create New File
c. Show Directory
d. Delete File
e. Exit
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <time.h>
int n,cnt=0,bit[100];
struct File { char name[20]; int len,blk[50]; } f[20];
void show_bits()
{
int i;
printf("Bit vector: ");
for(i=0;i<n;i++)
printf("%d",bit[i]);
printf("\n");
}
int pick()
{
int i;
for(i=0;i<n;i++)
if(!bit[i])
{
bit[i]=1;
return i;
}
return -1;
}
int free_count()
{
int i,c=0;
for(i=0;i<n;i++)
if(!bit[i])
c++;
return c;
}
void create()
{
struct File x;
int k;
if(cnt==20)
{
printf("Directory full\n");
return;
}
printf("Enter file name and size (blocks): ");
scanf("%s %d",x.name,&x.len);
if(x.len<1 || x.len>49 || free_count()<x.len)
{
printf("Cannot create file\n");
return;
}
for(k=0;k<x.len;k++)
x.blk[k]=pick();
f[cnt++]=x;
printf("File created\n");
}
void show_dir()
{
int i,k;
if(cnt==0)
printf("Directory is empty\n");
for(i=0;i<cnt;i++)
{
printf("%s : start = %d : ",f[i].name,f[i].blk[0]);
for(k=0;k<f[i].len;k++)
printf("%d -> ",f[i].blk[k]);
printf("NULL\n");
}
}
void delete_file()
{
char nm[20];
int i,k;
printf("Enter file name to delete: ");
scanf("%s",nm);
for(i=0;i<cnt;i++)
if(strcmp(f[i].name,nm)==0)
{
for(k=0;k<f[i].len;k++)
bit[f[i].blk[k]]=0;
f[i]=f[--cnt];
printf("File deleted\n");
return;
}
printf("File not found\n");
}
int main()
{
int i,ch;
srand(time(0));
printf("Enter number of disk blocks: ");
scanf("%d",&n);
for(i=0;i<n;i++)
bit[i]=(rand()%100<25);
do
{
printf("\n1.Show Bit Vector\n2.Create New File\n3.Show Directory\n4.Delete File\n5.Exit\nEnter choice: ");
scanf("%d",&ch);
switch(ch)
{
case 1: show_bits(); break;
case 2: create(); break;
case 3: show_dir(); break;
case 4: delete_file(); break;
}
}
while(ch!=5);
return 0;
}
```

Input (`linkdel.in`):

```text
20
2
f1 3
2
f2 2
3
4
f1
3
5
```

### Slip 22 – Q.2: Non-preemptive Priority scheduling  (`prinp.c`)

```c
/*
Q.2  Write the program to simulate Non-preemptive Priority scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],pr[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time, burst time and priority of P%d: ",i+1);
scanf("%d %d %d",&at[i],&bt[i],&pr[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || pr[i]<pr[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`prinp.in`):

```text
3
0 5 3
1 3 1
2 1 2
```

---

## Slip 23

### Slip 23 – Q.1: bubble sort (parent) + insertion sort (child)  (`sort.c`)

```c
/*
Q.2  Implement the C program to accept n integers to be sorted. Main function creates child process using fork system call. Parent process sorts the integers using bubble sort and waits for child process using wait system call. Child process sorts the integers using insertion sort.
*/
#include <stdio.h>
#include <unistd.h>
#include <sys/wait.h>
int main()
{
int a[20],n,i,j,t,key;
printf("Enter number of integers: ");
scanf("%d",&n);
printf("Enter integers: ");
for(i=0;i<n;i++)
scanf("%d",&a[i]);
fflush(stdout);
if(fork()==0)
{
for(i=1;i<n;i++)
{
key=a[i];
for(j=i-1;j>=0 && a[j]>key;j--)
a[j+1]=a[j];
a[j+1]=key;
}
printf("Child (insertion sort): ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
}
else
{
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(a[j]>a[j+1])
{
t=a[j];
a[j]=a[j+1];
a[j+1]=t;
}
printf("Parent (bubble sort): ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
wait(NULL);
}
return 0;
}
```

Input (`sort.in`):

```text
5
9 3 7 1 5
```

### Slip 23 – Q.2: Non-preemptive Shortest Job First  (`sjfnp.c`)

```c
/*
Q.2  Write the program to simulate Non-preemptive Shortest Job First (SJF) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || bt[i]<bt[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`sjfnp.in`):

```text
4
0 7
2 4
4 1
5 4
```

---

## Slip 24

### Slip 24 – Q.1: Round Robin  (`rr.c`)

```c
/*
Q.1  Write the program to simulate Round Robin (RR) scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. Also give the time quantum as input. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,q,t=0,done=0,p,run,s=0,head=0,tail=0,at[20],bt[20],rem[20],ct[20],inq[20]={0},queue[500],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
printf("Enter time quantum: ");
scanf("%d",&q);
while(done<n)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(head==tail)
{
gantt[t++]=-1;
continue;
}
p=queue[head++];
run=rem[p]<q?rem[p]:q;
while(run--)
{
gantt[t++]=p;
rem[p]--;
}
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(rem[p]>0)
queue[tail++]=p;
else
{
ct[p]=t;
done++;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`rr.in`):

```text
3
0 5
1 3
2 1
2
```

### Slip 24 – Q.2: execve() + binary search  (`execve.c`)

```c
/*
Q.1  Implement the C program that accepts an integer array. Main function forks child process. Parent process sorts an integer array and passes the sorted array to child process through the command line arguments of execve() system call. The child process uses execve() system call to load new program that uses this sorted array for performing the binary search the particular item in the array.
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc,char *argv[])
{
int a[20],n,i,j,t,key,lo,hi,mid;
char buf[22][12],*args[24];
if(argc>1)
{
key=atoi(argv[1]);
n=argc-2;
for(i=0;i<n;i++)
a[i]=atoi(argv[i+2]);
printf("New program received sorted array: ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
lo=0;
hi=n-1;
while(lo<=hi)
{
mid=(lo+hi)/2;
if(a[mid]==key)
{
printf("%d found at position %d\n",key,mid+1);
return 0;
}
if(a[mid]<key)
lo=mid+1;
else
hi=mid-1;
}
printf("%d not found\n",key);
return 0;
}
printf("Enter number of integers: ");
scanf("%d",&n);
printf("Enter integers: ");
for(i=0;i<n;i++)
scanf("%d",&a[i]);
printf("Enter item to search: ");
scanf("%d",&key);
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(a[j]>a[j+1])
{
t=a[j];
a[j]=a[j+1];
a[j+1]=t;
}
args[0]=argv[0];
sprintf(buf[0],"%d",key);
args[1]=buf[0];
for(i=0;i<n;i++)
{
sprintf(buf[i+1],"%d",a[i]);
args[i+2]=buf[i+1];
}
args[n+2]=NULL;
fflush(stdout);
if(fork()==0)
{
execve("/proc/self/exe",args,NULL);
perror("execve");
exit(1);
}
wait(NULL);
return 0;
}
```

Input (`execve.in`):

```text
5
40 10 30 20 50
30
```

---

## Slip 25

### Slip 25 – Q.1: orphan process  (`orphan.c`)

```c
/*
Q.1  Write a C program to illustrate the concept of orphan process. Parent process creates a child and terminates before child has finished its task. So child process becomes orphan process. (Use fork(), sleep(), getpid(), getppid())
*/
#include <stdio.h>
#include <unistd.h>
int main()
{
int pid=fork();
if(pid>0)
{
printf("Parent: PID = %d, child PID = %d\n",getpid(),pid);
printf("Parent terminating\n");
sleep(1);
}
else
{
printf("Child : PID = %d, parent PID = %d\n",getpid(),getppid());
sleep(3);
printf("Child : PID = %d, new parent PID = %d\n",getpid(),getppid());
}
return 0;
}
```

### Slip 25 – Q.2: shell with search command  (`search.c`)

```c
/*
Q.2  Write a C program that behaves like a shell which displays the command prompt '$'. It accepts the command, tokenize the command line and execute it by creating the child process. Also implement the additional command 'search' as
a. $ search f filename pattern : It will search the first occurrence of pattern in the given file
b. $ search a filename pattern : It will search all the occurrence of pattern in the given file
c. $ search c filename pattern : It will count the number of occurrence of pattern in the given file
*/
#include <stdio.h>
#include <stdlib.h>
#include <string.h>
#include <unistd.h>
#include <sys/wait.h>
void search(char *opt,char *fname,char *pat)
{
FILE *fp=fopen(fname,"r");
char line[256],*p;
int ln=0,total=0;
if(!fp)
{
printf("Cannot open %s\n",fname);
return;
}
while(fgets(line,256,fp))
{
ln++;
p=line;
while((p=strstr(p,pat))!=NULL)
{
total++;
if(opt[0]=='f')
{
printf("First occurrence: line %d: %s",ln,line);
fclose(fp);
return;
}
if(opt[0]=='a')
printf("Line %d, column %d\n",ln,(int)(p-line)+1);
p+=strlen(pat);
}
}
fclose(fp);
if(opt[0]=='c')
printf("Occurrences = %d\n",total);
else if(total==0)
printf("Pattern not found\n");
}
int main()
{
char cmd[200],*arg[32];
int i;
while(1)
{
printf("$ ");
fflush(stdout);
if(!fgets(cmd,200,stdin))
break;
for(i=0;i<30 && (arg[i]=strtok(i?NULL:cmd," \t\n"));i++);
arg[i]=NULL;
if(i==0)
continue;
if(strcmp(arg[0],"exit")==0)
break;
if(strcmp(arg[0],"search")==0 && i==4)
search(arg[1],arg[2],arg[3]);
else if(fork()==0)
{
execvp(arg[0],arg);
perror("command");
exit(1);
}
else
wait(NULL);
}
return 0;
}
```

Input (`search.in`):

```text
search f sample.txt is
search a sample.txt is
search c sample.txt is
exit
```

---

## Slip 26

### Slip 26 – Q.1: Preemptive Priority scheduling  (`prip.c`)

```c
/*
Q.2  Write the program to simulate Preemptive Priority scheduling. The arrival time and first CPU-burst and priority for different n number of processes should be input to the algorithm. The next CPU-burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],pr[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time, burst time and priority of P%d: ",i+1);
scanf("%d %d %d",&at[i],&bt[i],&pr[i]);
rem[i]=bt[i];
}
while(done<n)
{
cur=-1;
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || pr[i]<pr[cur]))
cur=i;
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`prip.in`):

```text
3
0 4 2
1 3 1
2 1 3
```

### Slip 26 – Q.2: FIFO page replacement  (`fifo.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the FIFO page replacement algorithm. Assume the memory of n frames.
Reference String: 3,4,5,6,3,4,7,3,4,5,6,7,2,4,6
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],load[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit==-1)
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(load[i]<load[v]) v=i;
}
fr[v]=ref[t];
load[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`fifo.in`):

```text
3
15
3 4 5 6 3 4 7 3 4 5 6 7 2 4 6
```

---

## Slip 27

### Slip 27 – Q.1: SSTF disk scheduling  (`sstf.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using SSTF algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
98, 183, 37, 122, 14, 124, 65
Start Head Position: 53
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],k,best,done[50]={0};
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
pos=head;
printf("Order served: ");
for(k=0;k<n;k++)
{
best=-1;
for(i=0;i<n;i++)
if(!done[i] && (best==-1 || abs(req[i]-pos)<abs(req[best]-pos)))
best=i;
done[best]=1;
go(req[best]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`sstf.in`):

```text
200
7
98 183 37 122 14 124 65
53
```

### Slip 27 – Q.2: LRU page replacement (counter)  (`lru.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the LRU (using counter method) page replacement algorithm. Assume the memory of n frames.
Reference String: 3,5,7,2,5,1,2,3,1,3,5,3,1,6,2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],last[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
last[hit]=t;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(last[i]<last[v]) v=i;
}
fr[v]=ref[t];
last[v]=t;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`lru.in`):

```text
3
15
3 5 7 2 5 1 2 3 1 3 5 3 1 6 2
```

---

## Slip 28

### Slip 28 – Q.1: SCAN disk scheduling  (`scan.c`)

```c
/*
Q.2  Write a simulation program for disk scheduling using SCAN algorithm.
Accept total number of disk blocks, disk request string, and current head position from the user.
Display the list of requests in the order in which it is served.
Also display the total number of head moments.
82, 170, 43, 140, 24, 16, 190, 65
Start Head Position: 50
Direction: Left
*/
#include <stdio.h>
#include <stdlib.h>
int pos,total;
void go(int x)
{
printf("%d ",x);
total+=abs(x-pos);
pos=x;
}
int main()
{
int n,i,size,head,req[50],j,t,k,dir;
printf("Enter total number of disk blocks: ");
scanf("%d",&size);
printf("Enter number of requests: ");
scanf("%d",&n);
printf("Enter request string: ");
for(i=0;i<n;i++)
scanf("%d",&req[i]);
printf("Enter current head position: ");
scanf("%d",&head);
printf("Enter direction (1=Right, 0=Left): ");
scanf("%d",&dir);
pos=head;
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(req[j]>req[j+1])
{
t=req[j];
req[j]=req[j+1];
req[j+1]=t;
}
for(i=0;i<n && req[i]<head;i++);
printf("Order served: ");
if(dir==1)
{
for(k=i;k<n;k++)
go(req[k]);
go(size-1);
for(k=i-1;k>=0;k--)
go(req[k]);
}
else
{
for(k=i-1;k>=0;k--)
go(req[k]);
go(0);
for(k=i;k<n;k++)
go(req[k]);
}
printf("\nTotal head movements = %d\n",total);
return 0;
}
```

Input (`scan.in`):

```text
200
8
82 170 43 140 24 16 190 65
50
0
```

### Slip 28 – Q.2: MFU page replacement  (`mfu.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the MFU page replacement algorithm. Assume the memory of n frames.
Reference String: 8, 5, 7, 8, 5, 7, 2, 3, 7, 3, 5, 9, 4, 6, 2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],cnt[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
cnt[hit]++;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(cnt[i]>cnt[v]) v=i;
}
fr[v]=ref[t];
cnt[v]=1;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`mfu.in`):

```text
3
15
8 5 7 8 5 7 2 3 7 3 5 9 4 6 2
```

---

## Slip 29

### Slip 29 – Q.1: execve() + binary search  (`execve.c`)

```c
/*
Q.1  Implement the C program that accepts an integer array. Main function forks child process. Parent process sorts an integer array and passes the sorted array to child process through the command line arguments of execve() system call. The child process uses execve() system call to load new program that uses this sorted array for performing the binary search the particular item in the array.
*/
#include <stdio.h>
#include <stdlib.h>
#include <unistd.h>
#include <sys/wait.h>
int main(int argc,char *argv[])
{
int a[20],n,i,j,t,key,lo,hi,mid;
char buf[22][12],*args[24];
if(argc>1)
{
key=atoi(argv[1]);
n=argc-2;
for(i=0;i<n;i++)
a[i]=atoi(argv[i+2]);
printf("New program received sorted array: ");
for(i=0;i<n;i++)
printf("%d ",a[i]);
printf("\n");
lo=0;
hi=n-1;
while(lo<=hi)
{
mid=(lo+hi)/2;
if(a[mid]==key)
{
printf("%d found at position %d\n",key,mid+1);
return 0;
}
if(a[mid]<key)
lo=mid+1;
else
hi=mid-1;
}
printf("%d not found\n",key);
return 0;
}
printf("Enter number of integers: ");
scanf("%d",&n);
printf("Enter integers: ");
for(i=0;i<n;i++)
scanf("%d",&a[i]);
printf("Enter item to search: ");
scanf("%d",&key);
for(i=0;i<n-1;i++)
for(j=0;j<n-i-1;j++)
if(a[j]>a[j+1])
{
t=a[j];
a[j]=a[j+1];
a[j+1]=t;
}
args[0]=argv[0];
sprintf(buf[0],"%d",key);
args[1]=buf[0];
for(i=0;i<n;i++)
{
sprintf(buf[i+1],"%d",a[i]);
args[i+2]=buf[i+1];
}
args[n+2]=NULL;
fflush(stdout);
if(fork()==0)
{
execve("/proc/self/exe",args,NULL);
perror("execve");
exit(1);
}
wait(NULL);
return 0;
}
```

Input (`execve.in`):

```text
5
40 10 30 20 50
30
```

### Slip 29 – Q.2: FCFS CPU scheduling  (`fcfscpu.c`)

```c
/*
Q.1  Write the program to simulate FCFS CPU scheduling. The arrival time and first CPU burst for different n number of processes should be input to the algorithm. The next CPU burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
int main()
{
int n,i,t=0,done=0,cur=-1,s=0,at[20],bt[20],rem[20],ct[20],gantt[500];
float tw=0,tt=0;
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and burst time of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
rem[i]=bt[i];
}
while(done<n)
{
if(cur==-1)
{
for(i=0;i<n;i++)
if(at[i]<=t && rem[i]>0 && (cur==-1 || at[i]<at[cur]))
cur=i;
}
gantt[t++]=cur;
if(cur!=-1 && --rem[cur]==0)
{
ct[cur]=t;
done++;
cur=-1;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d\n",i+1,at[i],bt[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i];
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`fcfscpu.in`):

```text
3
0 5
1 3
2 1
```

---

## Slip 30

### Slip 30 – Q.1: Round Robin with fixed IO waiting time (2 units)  (`rrio.c`)

```c
/*
Q.1  Write the program to simulate Round Robin (RR) CPU-scheduling. The arrival time and first CPU-burst for different n number of processes should be input to the algorithm. Also give the time quantum as input. Assume the fixed IO waiting time (2 units). The next CPU-burst should be generated randomly. The output should give Gantt chart, turnaround time and waiting time for each process. Also find the average waiting time and turnaround time.
*/
#include <stdio.h>
#include <stdlib.h>
#include <time.h>
int main()
{
int n,i,q,t=0,done=0,p,run,s=0,head=0,tail=0,at[20],bt[20],bt2[20],rem[20],ct[20],ready[20],stage[20],inq[20]={0},queue[500],gantt[500];
float tw=0,tt=0;
srand(time(0));
printf("Enter number of processes: ");
scanf("%d",&n);
for(i=0;i<n;i++)
{
printf("Enter arrival time and first CPU burst of P%d: ",i+1);
scanf("%d %d",&at[i],&bt[i]);
bt2[i]=rand()%5+1;
printf("Next CPU burst (random) = %d\n",bt2[i]);
rem[i]=bt[i];
ready[i]=at[i];
stage[i]=0;
}
printf("Enter time quantum: ");
scanf("%d",&q);
while(done<n)
{
for(i=0;i<n;i++)
if(ready[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(head==tail)
{
gantt[t++]=-1;
continue;
}
p=queue[head++];
run=rem[p]<q?rem[p]:q;
while(run--)
{
gantt[t++]=p;
rem[p]--;
}
for(i=0;i<n;i++)
if(ready[i]<=t && rem[i]>0 && !inq[i])
{
queue[tail++]=i;
inq[i]=1;
}
if(rem[p]>0)
queue[tail++]=p;
else if(stage[p]==0)
{
stage[p]=1;
rem[p]=bt2[p];
ready[p]=t+2;
inq[p]=0;
}
else
{
ct[p]=t;
done++;
}
}
printf("\nGantt chart:\n");
for(i=1;i<=t;i++)
if(i==t || gantt[i]!=gantt[s])
{
if(gantt[s]==-1) printf("| Idle (%d-%d) ",s,i);
else printf("| P%d (%d-%d) ",gantt[s]+1,s,i);
s=i;
}
printf("|\n\nProcess AT BT1 BT2 CT TAT WT\n");
for(i=0;i<n;i++)
{
printf("P%d %d %d %d %d %d %d\n",i+1,at[i],bt[i],bt2[i],ct[i],ct[i]-at[i],ct[i]-at[i]-bt[i]-bt2[i]-2);
tt+=ct[i]-at[i];
tw+=ct[i]-at[i]-bt[i]-bt2[i]-2;
}
printf("Average turnaround time = %.2f\nAverage waiting time = %.2f\n",tt/n,tw/n);
return 0;
}
```

Input (`rrio.in`):

```text
3
0 5
1 3
2 1
2
```

### Slip 30 – Q.2: MFU page replacement  (`mfu.c`)

```c
/*
Q.1  Write the simulation program for demand paging and show the page scheduling and total number of page faults according the MFU page replacement algorithm. Assume the memory of n frames.
Reference String: 8, 5, 7, 8, 5, 7, 2, 3, 7, 3, 5, 9, 4, 6, 2
*/
#include <stdio.h>
int n,ref[50];
int main()
{
int f,i,t,v,hit,faults=0,fr[10],cnt[10];
printf("Enter number of frames: ");
scanf("%d",&f);
printf("Enter length of reference string: ");
scanf("%d",&n);
printf("Enter reference string: ");
for(i=0;i<n;i++)
scanf("%d",&ref[i]);
for(i=0;i<f;i++)
fr[i]=-1;
for(t=0;t<n;t++)
{
hit=-1;
for(i=0;i<f;i++)
if(fr[i]==ref[t])
hit=i;
printf("%d : ",ref[t]);
if(hit>=0)
{
cnt[hit]++;
}
else
{
faults++;
v=-1;
for(i=0;i<f;i++)
if(fr[i]==-1)
{
v=i;
break;
}
if(v==-1)
{
v=0;
for(i=1;i<f;i++)
if(cnt[i]>cnt[v]) v=i;
}
fr[v]=ref[t];
cnt[v]=1;
}
for(i=0;i<f;i++)
if(fr[i]==-1) printf("- ");
else printf("%d ",fr[i]);
if(hit>=0) printf("  Hit\n");
else printf("  Fault\n");
}
printf("Total page faults = %d\n",faults);
return 0;
}
```

Input (`mfu.in`):

```text
3
15
8 5 7 8 5 7 2 3 7 3 5 9 4 6 2
```

---

## sample.txt

```text
hello world
this is a test
```
