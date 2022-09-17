---
layout: post
title: sizeWithFont오류
date: '2011-05-30T17:54:00.000+09:00'
tags:
    - sizeWithFont
    - iphone
modified_time: '2012-04-04T17:54:56.261+09:00'
blogger_id: tag:blogger.com,1999:blog-7360229670252766698.post-1791814448166401290
blogger_orig_url: https://jeremyko.blogspot.com/2011/05/sizewithfont.html
---

UITextView 기존내용을 사용자가 터치한 경우, 편집상태로 만들면서
커서를 보이면서 화면을 보이게끔 스크롤이 필요하다.

처음부터 터치된 곳까지의 문자열을 계산해서, 그 문장이 차지하는 높이를 구하면
UIScrollView의 scrollRectToVisible 를 이용해서 커서위치를 보이게 만들어주면 된다.

그런데 이 위치, 즉 전체화면에서 현재 터치한 부분의 좌표값을 얻어오는데 문제가 있었다.

**첫번째 방법**

`NSString` 의 `sizeWithFont:constrainedToSize:lineBreakMode` 사용이다.

대부분의 경우에 이것을 사용하면 되지만, 이상하게도 이것은 아주 긴 내용이거나,
폰트를 변경함에 따라서 정확한 값을 얻을수 없다.

아래의 방법을 이용해서 사용하는 예는 많이 찾아 볼수있었지만 직접 테스트 해보면 정확한 값을 가져오지 못해서 커서가
화면 아래나 위로 숨어버리는 문제가 발생했다.

    Data provided by Pastebin.com - Download Raw
    CGSize maximumTextViewSize = CGSizeMake(320- 30 ,FLT_MAX);
    // 게다가 좌우 여백을 줘야 그나마 비슷하게 구해진다!

    CGSize expectedLabelSize =[strToCursor  sizeWithFont:self.currentTextView.font
                                                            constrainedToSize:maximumTextViewSize
                                                            lineBreakMode:UILineBreakModeWordWrap];

    DLog(@"expectedLabelSize height [%f]", expectedLabelSize.height); //sometimes wrong height !!

**두번째는...**

이건 뭔 API 를 써야 될찌 감도 안와서 궁리하다가 적용한 원시적인 방법으로(방법이라고 할 수도 없을듯...),
UITextView 의 `contentSize.height` 가 실제로 텍스트를 가지고 있는 UITextView가 차지하는
높이이므로, 이 높이를 가지고 처리하는 방법이다.

그럴려면 먼저 임시 UITextView를 만들고,내용을 채우고, apple 내부에서 무슨 논리로
처리했는지는 모르겠지만 암튼 텍스트에 맞게 적절히 설정된 높이를 얻어와서 그 좌표를 가지고
스크롤 시켜주면, 커서 위치가 키보드 출력이후에도 사용자에게 보이게끔 잘 설정된다.

    Data provided by Pastebin.com - Download Raw
    // kinda primitive way - - ;;;
    // 전역으로 UITextView변수를 만들고
    // gTextViewforHeightCalculate에 텍스트를 붙여넣고, 실제 높이를 구한다.
    // API이용해서 정확하게 가져올수만 있다면 이짓을 할필요는 없을건데...

    CGRect viewFrame = [gTextViewforHeightCalculate frame];
    viewFrame.size.height = CGFLOAT_MAX  ;
    gTextViewforHeightCalculate.frame = viewFrame;
    gTextViewforHeightCalculate.minNumberOfLines = 1;
    gTextViewforHeightCalculate.returnKeyType = UIReturnKeyDefault;
    gTextViewforHeightCalculate.font =[UIFont fontWithName:self.cur_font size: [self.cur_font_size intValue]];

    //커서 이전까지의 문자열
    NSRange tmpRange;
    tmpRange.location= 0;
    tmpRange.length =  self.currentTextView.selectedRange.location ;
    NSString* strToCursor =  [self.currentTextView.text substringWithRange: tmpRange] ;
    gTextViewforHeightCalculate.text = strToCursor ;

    float nHeight = gTextViewforHeightCalculate.contentSize.height;
    DLog(@"UITextView height [%f]", nHeight);

    // 커서 위치만큼 scroll
    CGRect cgView;
    cgView.origin.x = 0.0;
    cgView.origin.y = self.currentTextView.frame.origin.y + nHeight;
    cgView.size.width = 320;
    cgView.size.height = 40;

    [self.scrollView scrollRectToVisible:cgView animated:TRUE];
