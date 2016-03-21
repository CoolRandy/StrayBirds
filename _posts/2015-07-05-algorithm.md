---
layout: post
title: 各类算法题解
category: 技术
comments: true
---

该篇blog部分迁移自我的博客园博文[CoolRandy](http://www.cnblogs.com/CoolRandy/p/4575267.html)

##字符串压缩
利用字符重复出现的次数，编写一个方法，实现基本的字符串压缩功能


```c
#include<stdio.h>
#include<stdlib.h>
#include<conio.h>
#include <string.h>

/*
重复字符压缩
*/
void RepeatCharReduce(char *str, int n, char *s){

//    char s[20];
    char tmp = str[0];
    s[0] = tmp;
    int j = 1;
    int count = 1;
    for(int i = 1; i <n; i++){
        if(str[i] == tmp){
            count++;
        }
        else{
            s[j] = count;
            j++;
            tmp = str[i];
            s[j] = str[i];
            count = 1;
            j++;
        }
    }
    s[j] = count;

}

int main(){
    char str[] = "aabcccccaaa";
    char s[20] = "0";
    RepeatCharReduce(str, 11, s);
    for(int j = 0; j < 11; j++){
        printf("%c", str[j]);
    }
    printf("\n");
    for(int i = 0; i < 11; i += 2){
        printf("%c%d", s[i], s[i+1]);
    }
    printf("\n");
    getch();
    return 0;
}
```
该代码实现结果压缩字符串aabcccccaaa为a2b1c5a3

##字符串移动
问题描述：给定一个字符串，要求把字符串前面的若干个字符移动到字符串的尾部，
如把字符串“abcdef”前面的2个字符'a'和'b'移动到字符串的尾部，使得原字符串变成字符串“cdefab”。
请写一个函数完成此功能，要求对长度为n的字符串操作的时间复杂度为 O(n)，空间复杂度为 O(1)

```c
  #include<stdio.h>
#include<stdlib.h>
#include<conio.h>
#include<math.h>
#include<time.h>
#include <string.h>

void ReverseString(char *s, int from, int to)
{
    while(from < to){
    char t = s[from];
    s[from++] = s[to];
    s[to--] = t;
    }
}

void LeftRotateString(char *s, int n, int m)
{
    m %= n;
    ReverseString(s, 0, m-1);
    ReverseString(s, m, n-1);
    ReverseString(s, 0, n-1);
}

int main(){
    char s[] = "abcdefgh";
    for(int i = 0; i < 8; i++){
        printf("%c ", s[i]);
    }
    printf("\n");
    LeftRotateString(s, 8, 3);
    for(int i = 0; i < 8; i++){
        printf("%c ", s[i]);
    }
    printf("\n");
    getch();
    return 0;
}
```
运行结果输出：  abcdefgh
                defghabc

#链表翻转
问题描述：给出一个链表和一个数k，比如，链表为1→2→3→4→5→6，k=2，则翻转后2→1→6→5→4→3，若k=3，翻转后3→2→1→6→5→4，
若k=4，翻转后4→3→2→1→6→5，用程序实现

```c
  #include<stdio.h>
#include<stdlib.h>
#include<conio.h>
#include<math.h>
#include<time.h>
#include <string.h>

#define OK 1
#define ERROR 0
#define TRUE 1
#define FALSE 0
 
#define MAXSIZE 20 /* 存储空间初始分配量 */
 
typedef int Status;/* Status是函数的类型,其值是函数结果状态代码，如OK等 */
typedef int ElemType;/* ElemType类型根据实际情况而定，这里假设为int */

 typedef struct Node{
     int data;
     struct Node *next;
}Node, *LinkList;

/* 初始化顺序线性表 */
Status InitList(LinkList *L)
{
    *L=(LinkList)malloc(sizeof(Node)); /* 产生头结点,并使L指向此头结点 */
    if(!(*L)) /* 存储分配失败 */
    {
        return ERROR;
    }
    (*L)->next=NULL; /* 指针域为空 */
    return OK;
}


/*  随机产生n个元素的值，建立带表头结点的单链线性表L（头插法） */
void CreateListHead(LinkList *L, int n)
{
    LinkList p;
    int i;
    srand(time(0));                         /* 初始化随机数种子 */
    *L = (LinkList)malloc(sizeof(Node));
    (*L)->next = NULL;                      /*  先建立一个带头结点的单链表 */
    for (i=0; i < n; i++)
    {
        p = (LinkList)malloc(sizeof(Node)); /*  生成新结点 */
        p->data = rand()%100+1; /*  随机生成100以内的数字 */
        printf("%d ", p->data);
        p->next = (*L)->next;
        (*L)->next = p;                      /*  插入到表头 */
    }
}
/* 初始条件：顺序线性表L已存在。操作结果：返回L中数据元素个数 */
int ListLength(LinkList L)
{
    int i=0;
    LinkList p=L->next; /* p指向第一个结点 */
    while(p)
    {
        i++;
        p=p->next;
    }
    return i;
}
Status visit(ElemType c)
{
    printf("-> %d ",c);
    return OK;
}
/* 初始条件：顺序线性表L已存在 */
/* 操作结果：依次对L的每个数据元素输出 */
Status ListTraverse(LinkList L)
{
    LinkList p=L->next;
    while(p)
    {
        visit(p->data);
        p=p->next;
    }
    printf("\n");
    return OK;
}

LinkList ListReverse2(LinkList L)
{
    LinkList current, p;
    if (L == NULL)
    {
        return NULL;
    }
    current = L->next;
    while (current->next != NULL)
    {
        p = current->next;
        current->next = p->next;
        p->next = L->next;
        L->next = p;
    }
    ListTraverse(L);
    return L;
}
/*
1、链表翻转。给出一个链表和一个数k，
比如，链表为1→2→3→4→5→6，k=2，则翻转后2→1→6→5→4→3，
若k=3，翻转后3→2→1→6→5→4，
若k=4，翻转后4→3→2→1→6→5，用程序实现。

对于链表而言只是对指针的指向调换，所以不会耗费额外存储空间，空间复杂度O(1)
时间复杂度此处看来也是线性的
*/

LinkList ReverseSpecArea(LinkList L, int k){
    LinkList current, p, q;
    LinkList temp;
    int i = 1;
    if (L == NULL)
    {
        return NULL;
    }
    current = L->next;

    while (current->next != NULL)
    {
        p = current->next;
        current->next = p->next;
        p->next = L->next;
        L->next = p;
        if(++i >= k){
            break;
        }
    }//current始终指向起先除去头结点的第一个元素
    temp = current;
    current = current->next;
    while(current->next != NULL){
        p = current->next;
        current->next = p->next;
        p->next = temp->next;
        temp->next = p;
    }
    ListTraverse(L);

    return L;
}

 
void ReverseString(char *s, int from, int to)
{
    while(from < to){
    char t = s[from];
    s[from++] = s[to];
    s[to--] = t;
    }
}

void LeftRotateString(char *s, int n, int k)
{
    k %= n;
    ReverseString(s, 0, k-1);
    ReverseString(s, k, n-1);
}

int main(){
    LinkList L;
    LinkList h;
    Status i;
    int j,k,pos,value;
    int length;
    char opp;
    ElemType e;
    i=InitList(&L);
    printf("%d\n", i);
    
    CreateListHead(&L,10);
    printf("\n");
    length = ListLength(L);
    printf("%d\n", length);
    printf("整体创建L的元素(头插法)：\n");
    ListTraverse(L);
    printf("\n");
    h = L->next;
    while(h){
        printf("%d ", h->data);
        h = h->next;
    }
    printf("\n");
    ListReverse2(L);
//    printf("反转指定位置3的元素\n");
//    ReverseSpecArea(L, 3);
    printf("反转指定位置5的元素\n");
    ReverseSpecArea(L, 5);

    getch();
    return 0;
}
```

