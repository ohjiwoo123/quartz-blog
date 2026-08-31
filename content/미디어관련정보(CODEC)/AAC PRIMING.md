AAC PRIMING에 관해서...

이 글을 보고 처음 AAC PRIMING이 무엇인지 접하게 되었다.  
[https://stackoverflow.com/questions/59173435/aac-packet-size/59332275#59332275](https://stackoverflow.com/questions/59173435/aac-packet-size/59332275#59332275)

프레임 개수가  
44100 Hz 일 때,  
The audio duration = 5246.2585 ms

`duration * sampling rate / frame size = 5246.2585 * 44.1/1024 = 225.9375 frames`

이어야 하는데,  
ffprobe에서

`nb_frames=228` 을 던진다는 내용이다.

항상 2.0625 프레임이 차이가 난다고 이야기한다.  
There is always a 2.0625 difference between my calculations and FFprobe output. Any ideas what I am doing wrong here? How can I accurately calculate the number of frames?

2112 samples is exactly 2.0625 packets.  
정확하게 2.0625 패킷에 해당한다는 내용이다.

AAC PRIMING에 대해 보려면  
애플 개발자 홈페이지에서 제공하는  
AAC encoding background 부분을 참고하면 좋다.

AAC requires data beyond the source PCM audio samples in order to correctly encode and decode audio samples due to the nature of the encoding algorithm. AAC encoding uses a transform over consecutive sets of 2048 audio samples, applied every 1024 audio samples (overlapped). For correct audio to be decoded, both transforms for any period of 1024 audio samples are needed. For this reason, encoders add at least 1024 samples of silence before the first ‘true’ audio sample, and often add more. This is called variously “priming”, “priming samples”, or “encoder delay”. A couple of definitions for use in this discussion:

AAC 인코딩은 매번 1024 오디오 샘플들 마다 연속적인 2048 오디오 샘플들로 변환이 적용된다.  
이런 이유로 최소한 1024 샘플은 첫번째 오디오 샘플이나 그이상으로  
프라이밍 샘플 또는 인코더 지연이라고 부르는 것을 사용한다.

아직 실무에선 일어나진 않았었으나, 당시에 공부하던 기록을 남겨둠.

[https://developer.apple.com/documentation/quicktime-file-format/appendix_g_audio_priming_handling_encoder_delay_in_aac](https://developer.apple.com/documentation/quicktime-file-format/appendix_g_audio_priming_handling_encoder_delay_in_aac)