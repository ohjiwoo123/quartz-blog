## 작전명 ASC Full-Parsing

AudioSpecificConfig 라는 raw-data를 풀 파싱해야할 이유가 생겼다.
그 이유는 즉슨 AAC라는 오디오 코덱도 더 세분화 하면 여러가지로 나뉜다.  
profile에 따라서 AAC-LC, AAC-HE, AAC-HEV2 이렇게...
이것을 어떻게 구분하는가 ? 
방법은 크게 두가지가 있다.

아래의 문서를 참고하자.  
`ISO_IEC_14996-3`
`1.6.5.2 Implicit and explicit signaling of SBR`


1. implicit signaling (암시적=간접적 신호)
```
implicit signaling: If EXT_SBR_DATA or EXT_SBR_DATA_CRC extension_payload() elements are
detected in the bitstream payload, this implicitly signals the presence of SBR data. The ability to
detect and decode implicitly signaled SBR is mandatory for all High Efficiency AAC Profile (HE AAC
Profile) decoders.
```
- EXT_SBR_DATA EXT_SBR_DATA_CRC extension_payload()가 비트스트림에서 발견된다면, 이것은 간접적으로 SBR 이 있다는 것을 의미한다.  

2. explicit signaling (명시적=직접적 신호)  
```
2. explicit signaling: The presence of SBR data is signaled explicitly by means of the SBR Audio!

Object Type in the AudioSpecificConfig(). When explicit signaling is used, implicit signaling shall not
occur. Two different types of explicit signaling are available:
(오디오 오브젝트 타입이 명시되어 있다면 implicit signaling은 발생하지 않는다.)
2.A. hierarchical signaling: If the first audioObjectType (AOT) signaled is the SBR AOT, a second audio
object type is signaled which indicates the underlying audio object type. This signaling method is not
backward compatible. This method may be needed in systems that do not convey the length of the
AudioSpecificConfig(), such as LATM with audioMuxVersion==0, and content authors are encouraged
to use it only when thus needed.
(계층적 시그널링으로, 첫번째 AOT가 SBR인 경우 두번째 AOT가 보인다. 이방법은 ASC의 길이를 전달하지 않는 경우 필요할 수 있다고 한다. 예를들면, LATM (audioMuxVersion이 0인 경우) 그리고 컨텐츠 사용자가 사용하도록 권장하는 경우)  

2.B. backward compatible signaling: The extensionAudioObjectType is signaled at the end of the
AudioSpecificConfig(). This method shall only be used in systems that convey the length of the
AudioSpecificConfig(). Hence, it shall not be used for LATM with audioMuxVersion==0.
(이전버전과 호환되는 시그널링 : 확장 오디오타입이 ASC 끝부분에 표기된다. 이 방법은 ASC의 길이를 알 수 있을 때만 실행된다. 
LATM (audioMuxVersion=0)인 경우 사용 되지 않는다.)
```

아래 표는 
SBR 시그널링과 그에 따른 디코더의 행동양식이다.  
![](https://velog.velcdn.com/images/ohjiwoo123/post/a393ab4e-c428-42a6-941c-7cdad9096489/image.png)

a. Implicit signaling은 sampling-frequency 값을 구하기 위해, 또는 SBR 데이터의 존재를 확인하기 위해서 payload를 확인한다. ASC()의 samplingFrequncy 값의 곱하기 2가 output으로 사용된다.  
b. Explicitly signals SBR데이터가 없는 경우, ASC()의 samplingFrequency 값을 그대로 사용.  
c. extensionSamplingFrequency in AudioSpecificConfig(). 값을 사용한다.  

위의 표를 해석하자면 다음과 같다.  

1. HE AAC profile decoder behavior in case of implicit signaling  
- 이전버전과 호환되는 SBR 데이터가 존재하는 경우,  `extensionAudioObjectType` 은 SBR AOT가 아니며, `sbrPresentFlag` 값은 -1로 설정된다. 이것은 implicit signaling 일 것이라는 신호이다.  

- HE AAC Profile 디코더는 듀얼 레이트 시스템이므로, SBR 도구는 샘플링 속도의 두 배로 작동합니다.  
디코더는 다음 두가지 방법으로 sample rate를 결정한다.  
a) bitstream payload를 디코딩 하기 이전에 SBR data가 있는지 확인한다. SBR data가 없다면 그대로 사용하고 있다면 sample rate는 곱하기 2 로 사용하여야한다.
b) SBR data가 가능하고, 샘플레이트가 2배로 계산된다고 가정했을때, the SBR Tool can be used for upsampling only, as described in subclause 4.6.18.5.  

2. HE AAC Profile decoder behavior in case of explicit signaling  
2.B (이전 버전과 호환 됨)의 경우 `extensionAudioObjectType` = SBR AOT 이다.  
sbrPresentFlag가 0이면 생략하고 1인 경우에만 SBR data가 있다고 가정하면 된다. (SBR Tool 동작)  

2.A (이전 버전과 호환 되지 않음)의 경우,   
extensionAudioObjectType = SBR AOT이고, sbrPresentFlag = 1로 설정된다.  
이 경우, implicit signaling은 존재할 수 없다.  
이러한 계층적 explicit signaling의 경우에는, SBR data는 항상 존재하며, 
HE AAC Profile decoder shall operate the SBR Tool.  


## HE-AAC란?
High-Efficiency Advanced Audio Coding 의 약자이다.
고효율 고급 오디오 부호화(High-Efficiency Advanced Audio Coding, HE-AAC)는 디지털 오디오에서 쓰이는 손실 데이터 압축 방식이다. 복잡성이 낮은 고급 오디오 부호화(AAC LC)이며, 스트리밍 오디오와 같은 낮은 비트레이트 애플리케이션에 최적화되었다.

고효율 고급 오디오 부호화 버전1(HE-AAC v1)은 스펙트럼 대역 복제(Spectral band replication, SBR)를 사용하여 주파수 영역(frequency domain)에서 압축 효율을 향상시킨다. 고효율 고급 오디오 부호화 버전2(HE-AAC v2)는 스펙트럼 대역 복제와 파라메트릭 스테레오(PS)가 한 쌍을 이루어 스테레오 신호의 압축 효율을 향상시킨다. 이 기법들을 사용하여 상당한 압축률의 향상을 이끌어 냈으나,박수와 같은 압축하기 어렵다고 알려진 음원에서는 음질의 열화가 발생한다고 알려져 있다.
## SBR (Spectral Band Replication)이란?
SBR은 한글로, 스펙트럼 대역 복제라는 의미이다.  
HE-AAC-V1 = AAC-LC + SBR 이다.

## Signaling of Parametric Stereo (PS)
PS는 한글로, 파라메트릭 스테레오의 신호 전달
HE-AAC-V2 = AAC-LC + SBR + PS 이다.