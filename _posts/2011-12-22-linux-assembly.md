---
layout: post
title: 리눅스 어셈블리 분석 기초
date: '2011-12-22T17:51:00.000+09:00'
tags:
    - 어셈블리
modified_time: '2015-11-23T11:46:18.369+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6872448404844464965
blogger_orig_url: https://jeremyko.blogspot.com/2011/12/blog-post.html
---

어셈블리 초보자의 입장에서 간단한 c소스를 역어셈블하고 각각의 의미를 알아본다.

다음은 테스트할 간단한 소스이다.

```cpp
void Swap(int *parm_a, int *parm_b)
{
    int temp = 0;
    temp = *parm_a;
    *parm_a = *parm_b;
    *parm_b = temp;
}
    void main()
{
    int a = 5, b = 7;
    Swap(&a, &b);
}
```

컴파일을 한다음, objdump나 gdb를 이용해서 역 어셈블 한다.

    grcom502</export/GenHomeRNC/jeremyko/TestingBed>: objdump -f testSwap

    grcom502</export/GenHomeRNC/jeremyko/TestingBed>: gdb testSwap
    ...
    Reading symbols from /export/GenHomeRNC/jeremyko/TestingBed/testSwap...done.
    (gdb) disassemble main
    Dump of assembler code for function main:
    0x0804837d <main+0>:    push   %ebp
    ...
    End of assembler dump.

다음이 역어셈블 된 내용이다.

---

## main함수

---

    0x0804837d <main+0>:    lea    0x4(%esp),%ecx
    0x08048381 <main+4>:    and    $0xfffffff0,%esp
    0x08048384 <main+7>:    pushl  -0x4(%ecx)
    0x08048387 <main+10>:   push   %ebp
    0x08048388 <main+11>:   mov    %esp,%ebp
    0x0804838a <main+13>:   push   %ecx
    0x0804838b <main+14>:   sub    $0x18,%esp
    0x0804838e <main+17>:   movl   $0x5,-0x8(%ebp)
    0x08048395 <main+24>:   movl   $0x7,-0xc(%ebp)
    0x0804839c <main+31>:   lea    -0xc(%ebp),%eax
    0x0804839f <main+34>:   mov    %eax,0x4(%esp)
    0x080483a3 <main+38>:   lea    -0x8(%ebp),%eax
    0x080483a6 <main+41>:   mov    %eax,(%esp)
    0x080483a9 <main+44>:   call   0x8048354 <Swap>
    0x080483ae <main+49>:   mov    $0x0,%eax
    0x080483b3 <main+54>:   add    $0x18,%esp
    0x080483b6 <main+57>:   pop    %ecx
    0x080483b7 <main+58>:   pop    %ebp
    0x080483b8 <main+59>:   lea    -0x4(%ecx),%esp
    0x080483bb <main+62>:   ret

---

## Main 해석:

---

    <main+0> , <main+4> , <main+5>

이 부분은 SSE(Streaming SIMD Extension) data type을 위한 stack realign 이다. 컴파일 시 -mpreferred-stack-boundary=2 옵션(4바이트정렬)을 추가하면 이 부분은 없어진다.
http://www.iamroot.org/xe/4610 내용 참조.

    0x0804837d <main+0>: lea 0x4(%esp),%ecx

%esp + 4 주소를 %ecx 레지스터에 넣는다.
main(이 역시 함수)을 호출할때 리턴주소를 스택에 push했을것이므로 현재의
%esp는 main 종료 시 돌아갈 주소를 가르키고 있는 상태이다.  
 (\_\_libc_start_main() 함수의 return address를 말한다. )
=> 그렇다면 왜 esp가 가르키던 복귀주소를 직접 ecx에 저장 하지 않나? 뭐하러 offset을 이용하지?
그건 보안과 관련되어 있다. 아래 계속 되는 내용을 참조.

이 부분의 역활은, esp를 16바이트 단위로 정렬하기 위해(즉 16의 배수 위치로 esp를 변경한다)
and 명령을 실행하는 과정에서 원래 esp가 변경될 수있으므로 함수 리턴 주소를 저장하려는 목적.

    0x08048381 <main+4>: and $0xfffffff0,%esp

%esp 의 위치를 and 연산을 통해 변경한다(성능향상을 위한 16바이트 정렬).  
 => 정수 N의 16의 배수는 N이 왼쪽으로 네번 쉬프트 된 결과이고, 하위 4비트가 0이다.
결과적으로 esp는 이전보다 동일하거나, 작은 값을 갖게 된다. (스택 증가).

    0x08048384 <main+7>: pushl -0x4(%ecx)

ecx 레지스터의 값에서 -4 offset한 값을 현재 stack에 저장한다. (return address를 저장).
=> ecx에는 변경이전의 esp(복귀주소를 가르키던)에 4 offset한 값(주소)이 저장되어 있다.
이값에서 -4 는 복귀주소값이다.

    0x08048387 <main+10>: push %ebp
    0x08048388 <main+11>: mov %esp,%ebp

일반적인 main() 함수 프롤로그 과정이다.
즉, 수행중이던 함수의 시작 위치 주소값인 ebp를 스택에 삽입,
삽입된 ebp를 가리키는 스택포인터의 위치를 베이스포인터에 대입.
(ebp(목적지) 에 esp(소스)를 복사! 즉 ebp = esp 대입. 왜? => 지금 push된 곳의 위치를 base로
삼기 위함이다. 함수호출시 베이스 기준으로 움직이기 때문에 새로운 base을 만들어야하기 때문.)

    0x0804838a <main+13>: push %ecx

이것은 보안과 연관있다. %ecx 레지스터를 현재 stack에 저장하여 canary 역할을 하도록 만듬.

    0x0804838b <main+14>: sub $0x18,%esp

스택을 24바이트 예약한다 . 이공간은 지역변수를 위해 사용된다.

    0x0804838e <main+17>: movl $0x5,-0x8(%ebp)

베이스포인터 위 -8에 , 변수 a에 5 저장(ecx가 있어서 -4 아니고 -8)

    0x08048395 <main+24>: movl $0x7,-0xc(%ebp)

베이스포인터 위 -12, 변수 b에 7 저장

이시점에서의 스택의 모습은 다음과 같다.

    ------------------------- ESP
    ...
    -------------------------
    지역변수 7
    -------------------------
    지역변수 5
    -------------------------
    ecx => 이값은 %esp + 4의 위치 주소이다
    ------------------------- EBP
    Old EBP
    -------------------------
    Main 복귀주소
    -------------------------

주소는 아래로 증가한다.

    0x0804839c <main+31>: lea -0xc(%ebp),%eax
    0x0804839f <main+34>: mov %eax,0x4(%esp)
    0x080483a3 <main+38>: lea -0x8(%ebp),%eax
    0x080483a6 <main+41>: mov %eax,(%esp)
    0x080483a9 <main+44>: call 0x8048354 <Swap>
    0x080483ae <main+49>: mov $0x0,%eax

함수 결과는 eax 레지스터에 저장한다.

    0x080483b3 <main+54>: add $0x18,%esp

스택 포인터를 24 바이트만큼 증가시킨다 (즉 스택 감소).
위의 sub $0x18,%esp를 취소하는것이다.

    0x080483b6 <main+57>: pop %ecx

stack에서 %ecx 레지스터를 꺼낸다. (%esp 4byte 증가).

    0x080483b7 <main+58>: pop %ebp

stack에서 %ebp(이전 base frame pointer) 레지스터를 꺼낸다. (%esp 4byte 증가)
예전 ebp를 복구한다.

    0x080483b8 <main+59>: lea -0x4(%ecx),%esp

%ecx - 4 위치 주소를 %esp에 넣는다.
ecx 레지스터에는 %esp + 4의 위치 주소가 있다.  
결국, \_\_libc_start_main() 함수의 원본 return address 위치로 %esp 레지스터를 이동시킨다.

    0x080483bb <main+62>: ret

---

## Swap 함수 해석

---

    08048354 <Swap>:
    8048354: 55 push %ebp
    8048355: 89 e5 mov %esp,%ebp

이부분은 위에서 이미 설명되었다.

    8048357: 83 ec 10 sub $0x10,%esp

16 바이트만큼 스택 예약

    804835a: c7 45 fc 00 00 00 00 movl $0x0,0xfffffffc(%ebp)

0을 copy to-> ebp주소+0xfffffffc(ebp주소+(-4))
뒤에 'l'은 Long.
스택에 할당받은 공간에 지역변수의 데이터 값을 저장.
temp 변수의 "0" 값을 베이스포인터의 상위 4바이트 공간에 저장.

    8048361: 8b 45 08 mov 0x8(%ebp),%eax

ebp주소+8 위치의 주소를 (parm_a) eax에 저장.

    8048364: 8b 00 mov (%eax),%eax

eax주소가 가르키는 값를 eax에 저장.

    8048366: 89 45 fc mov %eax,0xfffffffc(%ebp)

ebp -4 주소에 eax를 저장.
즉 temp 에 parm_a 변수값을 저장;

    8048369: 8b 45 0c mov 0xc(%ebp),%eax

ebp +12 위치의 주소를 (parm_b) eax에 저장.
*parm_a = *parm_b;

    804836c: 8b 10 mov (%eax),%edx

eax주소가 가르키는 데이터를 edx에 저장.

    804836e: 8b 45 08 mov 0x8(%ebp),%eax

ebp +8를 eax에 저장

    8048371: 89 10 mov %edx,(%eax)

edx가 eax가 가르키는 값을 포인트하게함.

    8048373: 8b 55 0c mov 0xc(%ebp),%edx

ebp +12가 edx를 포인트하게함.

    8048376: 8b 45 fc mov 0xfffffffc(%ebp),%eax

ebp -4를 eax에 저장

    8048379: 89 02 mov %eax,(%edx)

eax가 edx가 가르키는 값을 포인트

    804837b: c9 leave
    804837c: c3 ret

## 참조 자료

[http://x82.inetcop.org/h0me/papers/FC_exploit/FC5_main_function.txt](http://x82.inetcop.org/h0me/papers/FC_exploit/FC5_main_function.txt)
[http://www.tipssoft.com/bulletin/board.php?bo_table=FAQ&wr_id=618](http://www.tipssoft.com/bulletin/board.php?bo_table=FAQ&wr_id=618)
[http://www.tipssoft.com/bulletin/board.php?bo_table=FAQ&wr_id=619](http://www.tipssoft.com/bulletin/board.php?bo_table=FAQ&wr_id=619)
