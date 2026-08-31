공식 홈페이지 : https://flatbuffers.dev/
C++ 벤치마크 홈페이지 : https://flatbuffers.dev/benchmarks/
C++ 랭귀지 가이드 : https://flatbuffers.dev/languages/cpp/

flatc = 컴파일러
fbs = 스키마 (프로토콜)

1. .fbs 파일을 완성한다.
2. flac를 이용하여 스키마를 컴파일 한다.
3. 라이브러리를 참조하여 너의 프로그램에 코드를 생성한다.
4. flatbuffer를 사용하여 직렬화한다.
5. flatbuffer를 사용하여 

```
# --grpc : grpc 코드 생성 
# --gen-objectt-api : C++ Object API 생성
flatc --cpp --grpc --gen-object-api -o "${OUT_DIR}/cpp" ${SCHEMAS};
```


### Serialize Data 
```
#include "flatbuffers.h"
#include "monster_generated.h"

int main() {
  // Used to build the flatbuffer
  FlatBufferBuilder builder;

  // Auto-generated function emitted from `flatc` and the input
  // `monster.fbs` schema.
  auto monster = CreateMonsterDirect(builder, "Abominable Snowman", 100);

  // Finalize the buffer.
  builder.Finish(monster);
}
```

### 직렬화 데이터 전송 및 저장 
```
// Get a pointer to the flatbuffer.
const uint8_t* flatbuffer = builder.GetBufferPointer();

```

### 데이터 읽기 
```
// Get a view of the root monster from the flatbuffer.
const Monster snowman = GetMonster(flatbuffer);

// Access the monster's fields directly.
ASSERT_EQ(snowman->name(), "Abominable Snowman");
ASSERT_EQ(snowman->health(), 100);
```


