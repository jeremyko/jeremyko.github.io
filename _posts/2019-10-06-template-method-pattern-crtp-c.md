---
layout: post
title: template method pattern 에  CRTP 사용하기 (c++)
date: '2019-10-06T16:52:00.001+09:00'
tags:
    - cpp
    - template method pattern
    - CRTP
modified_time: '2022-03-26T20:56:32.844+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-597786629018288037
blogger_orig_url: https://jeremyko.blogspot.com/2019/10/template-method-pattern-crtp-c.html
---

그동안 개발하면서 template method pattern을 즐겨 사용하고 있는데,  
virtual 함수를 이용하여 구현했었다.  
그런데 CRTP 를 이용해도 비슷한(동일한건 아님) 처리가 가능하다.

예를 들어 다음처럼 template method pattern 구현을 한경우,

```cpp
#include <iostream>
#include <memory>
#include <vector>

template<typename T, typename... Args>
std::unique_ptr<T> make_unique(Args&&... args)
{
    return std::unique_ptr<T>(new T(std::forward<Args>(args)...));
}

class Base {
    public:
        virtual ~Base() {}

        void DoWork() {
            DoBasicWork();
            DoSpecialWork();
        }
        void DoBasicWork(){
            std::cout << "Do Basic work..\n";
        }
        virtual void DoSpecialWork() = 0;
};

class Derived1 : public Base
{
    public:
        void DoSpecialWork() override {
            std::cout << "Do derived 1's special work..\n";
        }
};

class Derived2 : public Base
{
    public:
        void DoSpecialWork() override {
            std::cout << "Do derived 2's special work..\n";
        }
};

int main()
{
    std::vector<std::unique_ptr<Base>> vec_base;
    vec_base.push_back(make_unique<Derived1>());
    vec_base.push_back(make_unique<Derived2>());
    for(auto it = vec_base.begin(); it != vec_base.end(); ++it){
        (*it)->DoWork();
    }
    return 0;
}
```

CRTP 를 이용해서 (비슷하게) 다시 구현하면 다음과 같다.  
동적 다형성(dynamic polymorphism)을 희생한 경우이다.

```cpp
//.... 생략 ....
template<typename Derived>
class Base
{
    public:
        virtual ~Base() {}
        void DoWork() {
            DoBasicWork();
            static_cast<Derived*>(this)->DoSpecialWork();
        }
        void DoBasicWork(){
            std::cout << "Do Basic work..\n";
        }
};

class Derived1 : public Base<Derived1>
{
    public:
        void DoSpecialWork(){
            std::cout << "Do derived 1's special work..\n";
        }
};

class Derived2 : public Base<Derived2>
{
    public:
        void DoSpecialWork(){
            std::cout << "Do derived 2's special work..\n";
        }
};

int main()
{
    Derived1 child1;
    Derived2 child2;
    child1.DoWork();
    child2.DoWork();
    return 0;
}
```

처음 예제와 같이 STL Container 에 저장하는 등의 동적 다형성이 필요하다면,  
가상함수를 사용할수 밖에 없지만, 두번째 예제와 같은식으로 사용하는것으로  
충분하다면 CRTP 를 이용한 방식도 고려해볼만 하겠다.
