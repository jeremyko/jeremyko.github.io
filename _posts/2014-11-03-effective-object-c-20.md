---
layout: post
title: Effective Object-C 2.0
date: '2014-11-03T01:16:00.002+09:00'
tags:
    - 번역
    - 책
modified_time: '2015-08-24T10:36:17.280+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-6037007967933541393
blogger_orig_url: https://jeremyko.blogspot.com/2014/11/effective-object-c-20.html
---

며칠전 강컴으로 구입한 책이다. 책을 사서 처음 부터 읽지 않고 관심있는 부분부터 보는 습관인데, 지금까지 2개의 아이템만을 읽었을 뿐인데 번역의 질이 실망스럽다. 처음부터 완전 엉망은 아니지만, 100% 번역을 믿고 읽을 수준은 아니라서, 이런책이 원서보다 더 읽기 힘든책이다. 그리고 단지 2개의 아이템을 읽었을 뿐인데 편집상의 심각한 오류도 보인다. 도저히 무슨 내용인지 몰라 고민하다가 인터넷으로 원서내용을 볼수 있는 사이트를 알게돼서 직접 비교해보았다.

<h3> <span style="color:{{site.span_h3_color}}"> 
item 6
</span> </h3>

<span style="color:{{site.span_emphasis_color}}">
41 페이지
</span>

    '보통은 원자성은 필요없다. 더높은 레벨의 쓰레드 안정성을 보장하지 않기 때문이다'

    원문 :
    Usually, atomicity is not required anyway, since it does not ensure thread safety, which usually requires a deeper level of locking.
    --> 원자성이 스레드 안전을 보장 하는것은 아니기에 이를 사용할 일은 별로 없다.
    일반적으로, 쓰레드 안전을 위해서는 몇 단계의 잠금이 요구되기 마련이다.
    ('더 높은' 이라는 말은 아주 이해하기 애매함)

<h3> <span style="color:{{site.span_h3_color}}"> 
item 7
</span> </h3>

<span style="color:{{site.span_emphasis_color}}">
44 페이지 10~11줄
</span>

    '...다음과 같이 재정의할 수있다'

    원문 :
    This subclass might override the setter for lastName like so:
    --> 다음처럼 재정의 했을 것이다.
    (가능성을 말하는것이지 그렇게 해야한다는게 아님! 이런 미묘한 차이가 이해하는데 매우 중요함)

<span style="color:{{site.span_emphasis_color}}">
44 페이지 19~20줄
</span>

    '...빈 문자열로 설정할 수 있다. 인스턴스변수를 세터를 통해...'

    원문 :
    The base class EOCPerson might set the last name to the empty string in its default initializer. If it did this through the setter, the subclass’s setter would be called and throw an exception.
    -->...빈 문자열로 설정했을 수 있다. 이것을 세터를 통해...
    (설정 할 수있다..라고 해석을 해서 이해가 안됨. 가능성을 말하는것이지 그렇게 해야한다는게 아님!)

<h3> <span style="color:{{site.span_h3_color}}"> 
item 22
</span> </h3>

<span style="color:{{site.span_emphasis_color}}">
141 페이지 하단
</span>

    '그러나 지정 초기화 메서드가 금방 재정의되기 때문에 정의할 필요가 없는 복잡한 구조의
    인스턴스 변수를 초기화하는 것 같은 부작용이 있다면 사용하고 싶지 않을 것이다.'

    원문 :
    But you wouldn’t want to do this when the designated initializer has a
    side effect that you don’t want to happen to the copy, such as setting up
    complex internal data structures that you are immediately going to overwrite.
    --> 그러나 초기화 이후 즉시 덮어써질 복사본의 복잡한 내부 데이터 구조들에 대해서,
    지정된 초기화 코드가 설정을 수행하는 등의 원하지 않는 부작용을
    발생 시킨다면 사용하고 싶지 않을 것이다.

<span style="color:{{site.span_emphasis_color}}">
142 페이지 9줄부터
</span>

    '이 패턴을 사용할 때 가변 클래스에서 가변 복사본을 반환하는 copywithZone을 재정의 해서는 안된다'

    원문 :
    When using this pattern, you should not override copyWithZone:
    in your mutable class to return a mutable copy.
    --> 이 패턴을 사용할 때, 가변 클래스에서 가변 복사본을 반환하기 위해
    copywithZone을 재정의 해서는 안된다.
    ( 반환하는 --> 반환하기 위해 )

<span style="color:{{site.span_emphasis_color}}">
142 페이지 16줄
</span>

    '...주의해야 한다'

    원문 :
    Note the subtlety...
    --> 미묘한 차이를 알아야 한다. (절대 동의어가 아님).

<span style="color:{{site.span_emphasis_color}}">
142 페이지 18줄
</span>

    '동일한 효과를 내는 또다른 방법은 copy, immutableCopy, mutableCopy 메서드가
    모두 동일한 클래스를 반환하게 하는 것이다. copy는 항상 같은 클래스를 반환 하지만
    다른 두 메서드는 상황에 따라 다른 것을 반환한다'

    원문 :
    Another way this could have been achieved is to have three methods:
    copy, immutableCopy, and mutableCopy, where copy always returns the same class,
    but the other two return specific variants.
    --> 이를 달성하는 또다른 방법은 copy, immutableCopy, mutableCopy 세가지 메서드를 갖게 하고,
    copy는 항상 동일한 클래스를 리턴하지만,
    다른 둘은 각자의 특정 클래스(immutable, mutable)를 리턴 하게 하는 것이다.
    (완전 오역 임)

<span style="color:{{site.span_emphasis_color}}">
142 페이지 마지막에서 3번째 줄
</span>

    '얕은 복사를 한 복사본의 내용물들은 원본과 같은 객체를 가르킨다.
    깊은 복사를 한 복사본의 내용물들은 원본 내용물의 목사본을 가르킨다'

    --> 이건 그림 3.2 에 대한 설명임. 전혀 관련없는 문맥이 등장하니..편집에 실수.

<span style="color:{{site.span_emphasis_color}}">
142 페이지 1줄
</span>

    '복사를 하는 코드가 복잡해질 것이다'

    원문 :
    but that would add complexity everywhere a copy is taken.
    --> 복사가 발생되는 모든 곳이 복잡해질 것이다.
    (곰곰 읽어보면, 절대 같은 의미가 될수 없다. 복사가 아니고 복사가 일어나는
    모든곳이다. 그래서 책 내용이 이해가 안되는 이유다.)

<span style="color:{{site.span_emphasis_color}}">
142 페이지 12 줄
</span>

    '또 일반적으로 모든 내부 객체를 복사하길 원하지 않는다'

    원문 :
    also, it’s usually not desirable to copy every object.
    --> ... 복사하는 것은 바람직하지 않다'

<span style="color:{{site.span_emphasis_color}}">
142 페이지 끝에서 2번째부터
</span>

    '일반적으로 클래스를 새로 만들 때 시스템 프레임워크가 구현한 방식인 얕은
    복사를 하는 copyWithZone: 과 같은 방식으로 클래스의 복사 메서드를 작성 하길 원할 것이다.'

    원문 :
    Usually, you will want your own classes to follow the same pattern as used
    in the system frameworks, with copyWithZone: performing a shallow copy.
    --> 일반적으로, 당신이 작성한 클래스는 시스템 프레임워크에서 사용되는
    방식처럼 얕은 복사를 수행하는 copyWithZone 을 이용하는 동일한 패턴을 따르기를 원할 것이다.

<span style="color:{{site.span_emphasis_color}}">
페이지 144 마지막줄
</span>

    '자신이 만든 객체가 깊은 목사가 필요하다면 깊은 복사 메서드를 추가하라'

    원문 :
    Consider adding a deep-copy method if deep copies of your object will be useful.
    --> 자신이 만든 객체에 깊은 복사가 유용한 경우에 깊은 복사 메서드 추가를 고려하라.
    (곰곰 읽어보면, 절대 같은 의미가 될수 없다)

아무래도 그 원서 사이트 옆에 띄우고 같이 책을 봐야 할듯함..
