---
layout: post
title: RtlpNotOwnerCriticalSection
date: '2009-04-28T23:05:00.000+09:00'
tags:
    - c++
    - TryEnterCriticalSection
    - EnterCriticalSection
    - LeaveCriticalSection
    - RtlpNotOwnerCriticalSection
    - 소유하고 있지 않은 리소스를 해제
modified_time: '2016-09-09T17:51:22.435+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-4276059339158765637
blogger_orig_url: https://jeremyko.blogspot.com/2009/04/rtlpnotownercriticalsection.html
---

Exception code: C0000264 {응용 프로그램 오류}
응용 프로그램이 소유하고 있지 않은 리소스를 해제하려고 했습니다. 응용 프로그램을 마치려면 [확인]을 클릭하십시오.

    Call stack:
    Address Frame
    7C99E837 03EDE7B0 RtlpNotOwnerCriticalSection+AC
    7C98B1D0 03EDE7C8 RtlIpv4StringToAddressExW+9852
    6400C566 03EDE824 plugware::CCriticalSectionLocalScope::unlock+16
    6400C516 03EDE880 plugware::CAutoLock::~CAutoLock+16
    6401532C 03EDE9F0 CThreadSockAsyncServer::doWork_STATUS_ONLINE+39C

local scope lock 을 사용하면서 critical section 소유권 문제였다.

```cpp
inline CCriticalSectionLocalScope::CCriticalSectionLocalScope()
{
    //::InitializeCriticalSection(&m_h_cs);
    ::InitializeCriticalSectionAndSpinCount( &m_h_cs, CSPINCOUNT );
    // spin lock 사용설정
}

inline CCriticalSectionLocalScope::~CCriticalSectionLocalScope()
{
    ::DeleteCriticalSection(&m_h_cs);
}

inline const bool CCriticalSectionLocalScope::acquire_lock()
{
    //  이 부분이 문제 !!~~
    #if(_WIN32_WINNT >= 0x0400)
        return 1L & TryEnterCriticalSection(&m_h_cs);
    #else
        EnterCriticalSection(&m_h_cs);
        return true;
    #endif //_WIN32_WINNT >= 0x0400
}

inline void CCriticalSectionLocalScope::unlock()
{
    ::LeaveCriticalSection(&m_h_cs);
}

// CAutoLock 구현
inline CAutoLock::CAutoLock(CCriticalSectionLocalScope& c_lock, char* szUsage )
: m_c_lock(c_lock)
{
    memset(m_szUsage, 0x00, sizeof(m_szUsage));
    strncpy(m_szUsage, szUsage, sizeof(m_szUsage));

    // spin lock을 사용하므로 이부분이 불필요하다고 생각되어 막고
    //while (false == m_c_lock.acquire_lock()) {}
    // 아래처럼 사용한것이 문제다..
    m_c_lock.acquire_lock() ;

    TRACE("CAutoLock 생성 [%s]\n", m_szUsage);
}

inline CAutoLock::~CAutoLock()
{
    m_c_lock.unlock();
    TRACE("CAutoLock 제거 [%s]\n", m_szUsage );
}
```

CAutoLock 이라는것의 객체를 사용해서 지역범위의 lock을 구현하는것인데, TryEnterCriticalSection 이 API는 소유권(Ownership) 획득을 보장하지 않는다.

EnterCriticalSection 은 블록킹을 하면서 소유권 획득을 기다리고, TryEnterCriticalSection 는 즉시 리턴하면서 ,대신 결과값으로 이미 소유권을 가지고 있고나 획득한 경우에는 0 이 아닌 값을 리턴한다.

위처럼 잘못 사용하면, 소유권을 갖지 못한 상태에서 LeaveCriticalSection 을 호출시에 커널 내부의 에러가 발생한다.
(만약, 예외처리가 없다면 다른 thread의 EnterCriticalSection 호출이 무한 대기상태 즉, 프로그램 hang... 상태로 빠질것이다, )

그리고 덧붙이자면 TryEnterCriticalSection 이함수는 컴퓨터의 CPU가 2개 이상인 경우는 사용할일이 거의 없다고 보인다. 참조한 소스의 원목적은 SPIN LOCK을 구현하기 위한거 같은데 멀티코어라면

```cpp
::InitializeCriticalSectionAndSpinCount( &m_h_cs, CSPINCOUNT );
```

이렇게 설정해주고 EnterCriticalSection 호출하면 알아서 SPIN LOCK으로 동작하기 떄문이다.

반면, 싱글코어인 경우에서는 이런 설정이 효과가 없으므로, 동기화의 효율을 좀더 높이기 위한 방편으로 TryEnterCriticalSection 와 loop로 대기하는 방식을 위처럼 직접 구현해서 사용할수 있다.
