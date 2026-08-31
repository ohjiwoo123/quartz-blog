
내가 확인하려 했던 라이브러리는 golang aacgoparser이다.   
https://github.com/Comcast/gaad/tree/6c3900593fd0ebd258253e61a4777f68bfc97aa7  

우선 방법은 두가지가 있는데,

1. 원격 저장소를 활용하여, git clone 없이 바로 import 하는 방법  
```
# 1) 새로운 폴더를 만든다

# 2) main.go에 코드 작성 및 사용할 라이브러리 import 
# 코드는 아래와 같음  
var []byte buf
buf = <ADTS+AAC data>

// Parsing the buffer
adts, err := gaad.ParseADTS(buf)

// Looping through top level elements and accessing sub-elements
var sbr bool
if adts.Fill_elements != nil {
	for _, e := range adts.Fill_elements {
		if e.Extension_payload != nil &&
			e.Extension_payload.Extension_type == gaad.EXT_SBR_DATA {
			sbr = true
		}
	}
}


# 3) go mod init test
# 4) go mod init test
# 5) go build 후 테스트 파일 실행 또는 go run . 
```

2. 로컬 저장소로 git clone 하여, 내가 라이브러리 즉각 수정하면서 import 하는 방법 
```
# 1) git clone https://github.com/Comcast/gaad
# 2) cmd 폴더를 만들어준다. (여기서 main.go 및 go mod init test, go mod tidy 까지 실행)  
# 3) go build 또는 go run 으로 실행  
```
