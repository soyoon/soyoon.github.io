---
layout: post
title:  "[java] 깊은 복사와 얕은 복사"
date:   2016-03-18 22:00:00
author: Soyoon
categories: Etc
tags: DeepCopy ShallowCopy
---

# 2016.03.18

## [java] 깊은 복사와 얕은 복사

int a[] = {1, 2, 3};
int b[] = a;

대입연산자로 객체를 복사하면 같은 메모리를 가리키면서 a[] 값을 변경하게 되면 b[] 값도 바뀌게 된다. 일를 막기 위해 Arrays.copyOf() 함수를 사용하면 안바뀜


### Deep Copy
참조 복사라고도 하며 새로운 객체를 생성해서 복사한 객체의 값이 변해도 안바뀜
int a[] = {1, 2, 3};
int b[] = Arrays.copyOf(a, 3);
a[0] = 7;
System.out.println(b[0]);

출력 : 1


### Shallow Copy
값복사 라고도 하며 복사한 객체의 값을 변경하면 새로운 객체에도 값이 변경됨
동일한   값을 가리키고 있기 때문

int a[] = {1, 2, 3};
int b[] = a;
a[0] = 7;
System.out.println(b[0]);

출력 : 7


### 추가로 찾아보기
이차원 배열의 경우에는 Arrays.copyOf() 를 해도 값이 변경된다.

포인터 값이 들어가게 되는 객체의 경우에는 깊은복사가 안되는 듯 한데. 원인 찾아보기
