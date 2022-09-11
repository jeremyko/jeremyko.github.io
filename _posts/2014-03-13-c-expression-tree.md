---
layout: post
title: 'c++ expression tree '
date: '2014-03-13T19:20:00.003+09:00'
tags:
    - c++
    - 수식트리
modified_time: '2022-03-26T21:01:33.930+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-3156913092191625085
blogger_orig_url: https://jeremyko.blogspot.com/2014/03/c-expression-tree.html
---

[https://github.com/jeremyko/expression-tree-with-user-functions](https://github.com/jeremyko/expression-tree-with-user-functions)

<h3> <span style="color:{{site.span_h3_color}}"> 
C++ 로 작성된, 사용자가 정의한 함수를 사용할수 있게한 수식트리.
</span> </h3>

간단한 사칙연산 수식의 경우엔 이미 공개된 소스가 많지만, 좀 더 복잡한 경우에 적용 불가하므로 한번 작성해보았다.

사용자가 입력하는 infix 수식을 postfix로 변환 후, 이를 가지고 수식트리를 생성하게 된다.

다음과 같은 간단한 사칙연산의 경우엔

    'A'+'B'+'C'='ABC'

다음처럼 수식트리가 구성될것이다.

![blog-image](/assets/img/20140313-expression_tree_plain.png)

그런데 좀더 복잡한 경우, 예를 들어서

    '1' + StrCat3('A','B','C') = '1ABC'

라는 수식이 있다고 가정해보자. 여기서 `StrCat3` 은 사용자가 임의로 작성한 함수이다.  
이 함수는 어떤식으로 트리에 표현하면 될까..?

![blog-image](/assets/img/20140313-expression_tree_question.png)

이 코드에서는 약간 변형된 이진트리를 사용했는데, 피 연산되는 오른쪽 노드가 list를 가지고, 동적으로 함수 인자를 할당, 추가하는 방식으로 구현됬다.
위의 수식 `'1' + StrCat3('A','B','C') = '1ABC'` 경우엔 다음처럼 트리가 구성된다.

![blog-image](/assets/img/20140313-expression_tree_func1.png)

함수내에 함수들이 중첩되는 좀더 복잡한 경우를 생각해보자.

    '1' + StrCat3('A', 'B', StrCat3('C','D','E')) = '1ABCDE'

이경우에 구성되는 트리는 다음과 같다.

![blog-image](/assets/img/20140313-expression_tree_func2.png)

만약 어떤 함수가 인자를 하나도 갖지 않는 경우엔 어떻게 될까.

    100 = GetInt100()

이 경우에 구성되는 트리는 다음과 같다.

![blog-image](/assets/img/20140313-expression_tree_func3.png)

이진 트리를 만들기 위해 `NULL` 자식을 추가했다.

트리를 평가해서 결과를 구하기 위해서는 스택이 넘치는것을 방지 하기 위해서 비 재귀적인 방법으로 처리를 했다.

그런데 함수 인자를 위해 추가된 노드를 위해서는, 일반적 후위순회 처리에 추가해서, 동적으로 할당된 함수의 인자 노드까지 모두 순회하게 처리해줘야 한다.

이 부분에서는 재귀적인 방법으로 처리를 하고 있는데,먼저 해당 노드의 `right->rightSiblingForMore2funcArgs` 을 처리하고, 이후 `rightSiblingForMore2funcArgs` 을 처리하면 된다.

<h3> <span style="color:{{site.span_h3_color}}"> 
Usage
</span> </h3>

<span style="color:{{site.span_emphasis_color}}">
general arithmetic
</span>

```cpp
ExpressionTree expTree;
expression_result* pExpressionRslt;
bool bRslt = false;

bRslt = expTree.SetInfixExpression("1 * -2 ");
EXPECT_TRUE(bRslt);
if (bRslt)
{
   bRslt = expTree.EvaluateExpression();
   EXPECT_TRUE(bRslt);
   pExpressionRslt = expTree.GetResult();
   cout << "expTree.EvaluateExpression returns :"
        << pExpressionRslt->nResultLong << "\n";
   EXPECT_EQ(-2, pExpressionRslt->nResultLong);
}
```

<span style="color:{{site.span_emphasis_color}}">
custom user functions
</span>

```cpp
//see user_functions.h, user_functions.cpp
//-----------------------------------------------------
bRslt = expTree.SetInfixExpression("SumInt4(SumInt4(1,1+1,1+1,2), 1+1, 1+1, 1+1)");
EXPECT_TRUE(bRslt);
bRslt = expTree.EvaluateExpression();
EXPECT_TRUE(bRslt);
pExpressionRslt = expTree.GetResult();
cout << "expTree.EvaluateExpression returns :"
     << pExpressionRslt->nResultLong << "\n";
EXPECT_EQ(13, pExpressionRslt->nResultLong);

//-----------------------------------------------------
bRslt = expTree.SetInfixExpression("GetBool('1'='1')");
EXPECT_TRUE(bRslt);
bRslt = expTree.EvaluateExpression();
EXPECT_TRUE(bRslt);

//-----------------------------------------------------
bRslt = expTree.SetInfixExpression("StrCat('ab','cd')=StrCat('a','b')+StrCat('c','d') ");
EXPECT_TRUE(bRslt);
bRslt = expTree.EvaluateExpression();
EXPECT_TRUE(bRslt);

//-----------------------------------------------------
bRslt = expTree.SetInfixExpression("StrCat3('1','2',StrCat3('a','b',StrCat3('X','Y','Z')) )");
EXPECT_TRUE(bRslt);
bRslt = expTree.EvaluateExpression();
EXPECT_TRUE(bRslt);
EXPECT_EQ(4, expTree.GetDepth());
pExpressionRslt = expTree.GetResult();
cout << "expTree.EvaluateExpression returns :" << pExpressionRslt->strResult << "\n";
EXPECT_STREQ("12abXYZ", pExpressionRslt->strResult);
```

<span style="color:{{site.span_emphasis_color}}">
using placeholder
</span>

```cpp
// placeholder identifier is :$ph1, :$ph2, .... :$ph50 (max)
bRslt = expTree.SetInfixExpression ( ":$ph1+:$ph2" );
EXPECT_TRUE ( bRslt );
if ( bRslt )
{
    expTree.SetNumberLongValueOfPlaceHolder ( 1, 1 );
    expTree.SetNumberLongValueOfPlaceHolder ( 2, 2 );
    bRslt = expTree.EvaluateExpression ( );
    EXPECT_TRUE ( bRslt );
    pExpressionRslt = expTree.GetResult ( );
    cout << "expTree.EvaluateExpression returns :" << pExpressionRslt->nResultLong << "\n";
    EXPECT_EQ ( 3, pExpressionRslt->nResultLong );

    //-----------------------------------------------------
    //there's no rebuild tree
    expTree.SetNumberFloatValueOfPlaceHolder ( 1, 2.0f );
    expTree.SetNumberFloatValueOfPlaceHolder ( 2, 3.0f );
    bRslt = expTree.EvaluateExpression ( );
    EXPECT_TRUE ( bRslt );
    pExpressionRslt = expTree.GetResult ( );
    cout << "expTree.EvaluateExpression returns :" << pExpressionRslt->nResultFloat << "\n";
    EXPECT_FLOAT_EQ ( 5.0f, pExpressionRslt->nResultFloat );

    //-----------------------------------------------------
    expTree.SetStringValueOfPlaceHolder ( 1, "A" );
    expTree.SetStringValueOfPlaceHolder ( 2, "B" );
    bRslt = expTree.EvaluateExpression ( );
    EXPECT_TRUE ( bRslt );
    pExpressionRslt = expTree.GetResult ( );
    cout << "expTree.EvaluateExpression returns :" << pExpressionRslt->strResult << "\n";
    EXPECT_STREQ ( "AB", pExpressionRslt->strResult );
}
```
