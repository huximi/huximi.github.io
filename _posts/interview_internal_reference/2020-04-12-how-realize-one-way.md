---
layout: post
title: "如何实现一个高效的单向链表逆序输出"
subtitle: '如何实现一个高效的单向链表逆序输出'
author: "GuYang"
header-img: "img/post-bg-2018.jpg"
tags:    
    - 面试题
    - 单向链表  
---
##### **问题**：如何实现一个高效的单向链表逆序输出？ 

#
下面是其中一种写法，也可以有不同的写法，比如递归等。供参考。


```
typedef struct node{
    int           data;
    struct node*  next;
    node(int d):data(d), next(NULL){}
}node;

void reverse(node* head)
{
    if(NULL == head || NULL == head->next){
        return;
    }
    
    node* prev=NULL;
    node* pcur=head->next;
    node* next;
    
    while(pcur!=NULL){
        if(pcur->next==NULL){
            pcur->next=prev;
            break;
        }
        next=pcur->next;
        pcur->next=prev;
        prev=pcur;
        pcur=next;
    }
    
    head->next=pcur;
    node*tmp=head->next;
    while(tmp!=NULL){
        cout<<tmp->data<<"\t";
        tmp=tmp->next;
    }
}

```
