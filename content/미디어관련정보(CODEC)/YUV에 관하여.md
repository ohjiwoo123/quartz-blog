RGB와 YUV에 관하여 정말 좋은 글이 있었다.


아래 글을 참고하여 나의 글을 정리해본다.
https://blog.naver.com/wndrlf2003/220253497246  

일단 YUV란 
RGB 보다 용량을 적게 표현하는 것이다. 
Y : Luminance (휘도 = 밝기)
luminance or "brightness" (greyscale) component (or the picture in black and white).
U : Blue luminance (파랑 계열의 밝기)
V : Red luminance (빨강 계열의 밝기)


| - | RGB 방식 (R, G, B) | YUV 방식(Y, U, V)
| :- | - | :-: | 
| 흰색 | 255, 255, 255 | 235, 0, 0 |
| 회색 | 128, 0, 0 | 128, 0, 0 |
| 검정색 | 0, 0, 0 | 16, 0, 0 | 


## RGB - YUV 변환 

Y = 0.257R + 0.504G + 0.095B + 16
U = -0.148R -0.291G + 0.499B + 128
V = 0.439R - 0.368G - 0.071B + 128


## 서브샘플링
동영상을 크기를 줄이기 위해 사용된다.  
사람 눈은 휘도에 더 민감하기 때문에 샘플링에도 적용하여 압축을 한다.
Y성분을 CbCr보다 많이 할당하게 되면 감소한 데이터에 비해 시각적 차이는 거의 없다.  
숫자는 바이트를 의미한다. 
1 YCbCr 4:4:4 format (YUV 444)
2 YCbCr 4:2:2 format (YUV 422)
3 YCbCr 4:2:0 format (YUV 420)
4 YCbCr 4:1:1 format (YUV 411)
