+++
title = "C에서의 자료구조 라이브러리 구현 알아보기"
description = "element와 data structure 메모리 관리, 비연속적 자료구조"
date = 2026-04-07

[extra]
page_style = "post"
+++

항상 궁금했던 게, C에서는 data structures(자료구조들)를 메모리에서 어떻게 관리하는지임.

크게 2가지인데, element(원소)의 관리와 data structure 자체의 관리임.

암튼 그걸 위해서 C의 참고용으로 쓸 만한 자료구조 라이브러리들을 찾아봤는데, 일단은 이것들로 찾아볼 듯? (+ 그리고 이해를 돕기 위한 Rust와의 비교)

- [glib](https://github.com/gnome/glib) - [docs](https://docs.gtk.org/glib/data-structures.html)
- [c-algorithms](https://github.com/fragglet/c-algorithms)
- [Collections-C](https://github.com/srdja/Collections-C)

## element와 data structure의 메모리 관리

### element의 메모리 관리

#### 문제

Rust와 비교하면 이해하기 좋을 거 같은데, Rust의 소유권을 대충이나마 알고 나서 C의 메모리 관리를 어떤 식으로 하는 게 (일반적으로) 좋은지 좀 더 이해하게 되긴 했는데

그래도 여전히 이해가 안 되는 게 있다면, element의 관리임. Rust야 소유권이 넘어가지만, C는 언어 차원에서 소유권 개념이 없으니 호출자와 data structure 자체 중 누가 관리하는지 정해야 하는데, 이게 어떻게 되는 건지?

예로 들어 특정 함수 내부에서 배열을 받아서 특정 인덱스의 element를 Rust처럼 “빌린다/넘긴다”로 구분해서 처리하는 게 아니라, C에서는 그냥 값이나 포인터를 다루는 것뿐이라 그 포인터가 가리키는 대상의 수명과 해제 책임이 코드만 보고는 바로 드러나지 않는다. 그래서 이게 빌리는 건지, 넘기는 건지, 지우는 건지도 명시가 없으면 되게 애매해지는 거 같음.

#### 조사

AI와 실제 공식문서, 코드 약간 참고(전부는 아님)

**C 범용 자료구조 라이브러리의 container 특징**
1. **pointer container(포인터 저장형 컨테이너), 기본 non-owning**
   * 포인터만 저장
   * pointee(포인터가 가리키는 대상) 자체의 해제는 보통 호출자가 직접 관리
2. **pointer container + destroy callback**
   * 여전히 Rust처럼 언어 차원의 ownership은 아님
   * 일부 라이브러리/data structure는 element 제거 또는 data structure 파괴 시 호출할 **destroy callback**을 지원
   * 개별 element에 연결된 정리 동작을 data structure 동작에 연결하는 방식
3. **pointer container + free-all 계열 API**
   * 일부 라이브러리/data structure는 callback 대신
   * `destroy_free`, `remove_all_free` 같은 식으로 **전체 정리 시 포인터가 가리키는 대상까지 한꺼번에 free하는 API**를 제공
   * 보통 개별 element 제거 시마다 자동으로 동작하는 방식과는 구분됨 (callback과 free-all 둘 다 제공할 수도 있음)
4. **value-copy container(값 복사 저장형 컨테이너)**
   * 값을 복사해서 저장
   * 이 경우 container는 “복사해 둔 value(값)”의 storage(저장소)는 관리하지만, 그 값 안에 포인터가 있으면 그 내부까지 자동으로 관리하는 건 아님
   * 따라서 포인터 대상의 수명까지 포함한 deep copy/deep free 여부를 별도로 봐야 함

**실제 라이브러리 기준으로**
1. **GLib**
   * `GArray`: 값 저장형
   * `GPtrArray`: 포인터 저장형, destroy callback을 붙여서 element 제거/파괴 시 정리 동작을 연결 가능
   * `GList` / `GQueue`: 기본적으로 data 포인터 저장형
   * `GHashTable`: 기본 삽입은 복사 아님, 수명 관리를 사용자가 의식해야 함. 대신 key/value destroy function을 지정할 수 있음
2. **c-algorithms**
   * [공식문서](https://fragglet.github.io/c-algorithms/doc/)에서 모두 `void *` 기반 포인터 저장형 container이라고 말함.
   * 일부 data structure는 free callback 등록으로 정리 동작을 연결 가능
3. **Collections-C**
   * pointer container와 value-copy 성격의 sized container가 분리
   * pointer container 쪽은 `destroy_free`, `remove_all_free` 같은 free-all 계열 API 제공

#### 정리

정리해보자면 value-copy container의 경우 container가 자기 내부에 복사해 둔 value의 storage를 관리한다. 

다만 이걸 Rust 식으로 “소유권이 container에게 있다”라고 그대로 말하면 약간 오해의 소지가 있는데, C에는 그런 언어 차원의 개념이 없고, 그냥 container가 복사본을 보관하고 그 복사본의 수명을 관리한다고 보는 편이 더 정확함.   
그래서 이런 container는 포인터가 아닌 값 자체를 저장할 때 쓰는 경우가 자연스럽다.   
따라서 container에서 element를 제거할 때는 그 복사되어 저장된 value가 들어 있던 slot(슬롯)이 자연스럽게 정리된다.   

C에서 stored value가 pointer인 경우, 처리가 더 까다로운데,
Rust와 다르게 함수/타입 시스템 차원에서 ownership에 따라 다르게 처리해주지 않으므로, 필요에 따라 직접 free를 호출해야 함.
그렇지 않으면 메모리 누수가 발생할 수 있음.

Rust와 달리 ownership 개념이 없으므로, element를 가져와 data structure에선 삭제하고 free하지 않고 남기는 경우도 있을 거라서 그럼.      
따라서 이걸 포인터가 가리키는 대상의 메모리까지 해제하는 건지 아니면 그냥 빼오기만 하는 건지는 API 문서나 함수 의미로 직접 정해져 있어야 하고, 사용자가 그걸 의식하고 써야 함.

대신 remove나 replace가 많이 발생하는 data structure의 경우 (정확히는 data structure 조작이 일어날 때, 그 element가 가리키는 대상의 메모리도 같이 정리되어야 하는 상황)에서,
사용자의 실수 방지나 편의성 처리를 위해 destroy callback을 지원한다.
대표적으로 GLib의 `GPtrArray`와 `GHashTable`이 있다.

### data structure의 메모리 관리

#### 문제

이제 element의 메모리 관리는 대충 이해했는데, data structure의 메모리 관리는 여전히 잘 모르겠음.

contiguous memory(연속 메모리)를 사용하는 배열 기반 data structure는 비교적 이해하기 쉽다.  
보통 `size`와 `capacity`를 기준으로 현재 사용 중인 element 수와 확보된 buffer(버퍼) 크기를 구분하면 되기 때문이다.

그런데 트리나 hash table처럼, 내부적으로 하나의 연속 buffer만으로 표현되지 않는 data structure는 이야기가 조금 다르다.  
이런 구조는 node(노드), bucket(버킷), 보조 배열 같은 여러 단위의 메모리를 따로 관리해야 할 것 같은데, 실제로는 어떤 식으로 할당과 해제가 이루어지는지 잘 감이 오지 않았다.

매 삽입/삭제마다 node를 위한 메모리의 할당과 해제를 반복하는 건가?  
아니면 일정한 관리 단위가 따로 있는 건가? 뭐 근데 생각해보면 같은 역할이어도 구현체마다 다를 수 있을거 같긴함. 

GC가 있으면 그냥 unreachable memory면 상관없는데, C는 포인터 연산에 주소 이동까지 가능해서 rc 기반의 커스텀 GC를 만들어서 추가할수도 없고, (제한된 범용 사용을 강제하면 될수도?)

그래서 이게 어떻게 관리되는가...

#### 조사

AI와 실제 공식문서, 코드 약간 참고(전부는 아님)

**C 범용 자료구조 라이브러리의 data structure 내부 메모리 관리 방식**

1. **contiguous buffer 기반**
   * 내부에 연속 메모리 buffer를 하나 두고 관리
   * 보통 `size`, `capacity` 개념이 있음
   * 배열 기반 data structure가 대표적임 
2. **node 기반**
   * element 하나당 node 하나를 두는 방식
   * 삽입 시 node 단위로 할당, 삭제 시 node 단위로 해제
   * linked list, tree 같은 구조가 여기에 해당
3. **buffer + node 혼합형**
   * 내부에 bucket/slot 배열 같은 연속 메모리를 두면서도
   * 구현 방식에 따라 별도 entry(엔트리)/node를 함께 관리할 수 있음
   * hash table이 대표적임
4. **custom allocator / memory pool 사용 가능**
   * 무조건 일반 `malloc/free`만 쓰는 건 아님
   * 라이브러리에 따라 allocator를 교체하거나 pool을 붙일 수 있음
   * Collections-C는 memory pool을 별도 기능으로 제공하고, list 같은 data structure의 allocator를 pool allocator로 바꾸는 예제도 README에 있다.

**자료구조별로 보면**
1. **배열 기반 data structure**
   * 비교적 단순함
   * 내부 buffer를 들고 있다가 capacity가 부족하면 재할당
   * value-copy / pointer container 둘 다 제공하는 경우 많음
2. **linked list**
   * 비연속 메모리 기반이지만 구조 자체는 비교적 단순한 편
   * list 본체와 각 node가 분리되어 있고
   * 삽입/삭제 시 node 단위 할당/해제가 일어남
   * 다만 저장되는 payload 자체는 보통 포인터라서, 메모리 배치 기준으로는 node 기반이고, element 저장 방식 기준으로는 pointer container인 경우가 많음
3. **tree**
   * 이것도 node 기반 (node 단위 할당/해제)
   * linked list보다 삭제/재연결 쪽이 더 복잡함
   * 저장 방식 기준으로는 key/value를 `void *`나 포인터로 다루는 pointer container 계열이 많음
4. **hash table**
   * 구현에 따라 차이가 큼
   * open addressing이면 slot 배열 같은 연속 메모리 중심으로 관리할 수 있음
   * chaining이면 bucket 배열 + 체인용 node 구조가 섞일 수 있음
   * 따라서 내부 메모리 관리 기준으로는 contiguous buffer 성격과 node 성격이 섞일 수 있고, element 저장 방식 기준으로는 보통 key/value 포인터를 저장하는 pointer container에 가깝다

**실제 non-contiguous / node 기반 자료구조를 라이브러리별로 보면**
1. **GLib**
   * `GSList` / `GList` / `GQueue`: list 계열이고, data 포인터를 저장하므로 **node 기반 + pointer container**
   * `GTree`: balanced tree 기반의 key/value container라서 **node 기반 + pointer container**
   * `GHashTable`: key/value를 복사하지 않으므로 API 기준으로는 **pointer container**. 다만 내부 구현까지 단순 node 기반이라고 하긴 애매함
2. **c-algorithms**
   * 라이브러리 전반이 `void *` 기반이라 모두 **pointer container**
   * `list.h` / `slist.h`: **node 기반 + pointer container**
   * `avl-tree.h`: **node 기반 + pointer container**
   * `hash-table.h`: key/value가 `void *`이고 free function 등록도 가능해서, **pointer container + free callback 연결 가능**
3. **Collections-C**
   * README에서 pointer container와 sized container를 분리함
   * `CC_List`, `CC_SList`, `CC_HashTable`, `CC_TreeTable` 등은 **pointer container**
   * 이 중 리스트/트리 계열은 자연스럽게 **node 기반 + pointer container**로 볼 수 있음

**대충 보면** - (사실 상 다 pointer container라고 봄.)
1. **non-contiguous / node 기반 자료구조는 대부분 pointer container**
   * GLib의 `GList`, `GSList`, `GTree`
   * c-algorithms의 linked list, AVL tree
   * Collections-C의 `CC_List`, `CC_SList`, `CC_TreeTable`
2. **value-copy container는 주로 contiguous array 쪽에 몰려 있음**
   * GLib의 `GArray`
   * Collections-C의 `CC_ArraySized`
3. **hash table은 내부 메모리 관리 방식은 구현 따라 달라도, 외부 API 기준으로는 대체로 pointer container**
   * key/value를 복사하지 않고 포인터나 `void *`를 저장하는 경우가 많음
   * 대신 destroy/free callback이나 free-all 계열 API를 붙여서 element 정리를 연결하는 방식이 흔함.

#### 정리

대충 보면 contiguous 쪽은 value-copy / pointer container 둘 다 만들기 쉬운 반면, non-contiguous / node 기반 자료구조는 범용 라이브러리에서 대부분 pointer container로 제공된다.

이게 불가능해서라기보다는, generic C 라이브러리에서 value-copy 방식으로 만들었을 때 애매한 점이 많기 때문인 듯함.

삽입/삭제 자체만 보면 node 기반도 value-copy가 불가능한 건 아니다.     
node마다 값을 직접 넣고, 삽입 시 복사하고, 삭제 시 그 node를 해제하면 되니까 구현 자체는 가능함.    
그리고 한 번 들어간 뒤의 재연결이나 이동은 보통 node 포인터만 바꾸면 되므로, 그 과정에서 값 복사가 계속 일어나는 것도 아님.

근데 조회와 반환 쪽에서 문제가 있다.

contiguous 쪽은 내부가 연속된 buffer라서 특정 위치만 알면 곧바로 접근할 수 있다.     
즉 value-copy를 하더라도 “연속된 저장소에 값을 복사해 넣고, offset으로 다시 접근한다”는 모델이 비교적 단순하다.

반면 non-contiguous / node 기반은 node 안에 값을 직접 저장한다고 해도, 그 값을 사용자에게 어떻게 돌려줄지가 애매하다.

1. **포인터로 넘겨주기**
   * node 내부 값의 주소를 넘겨주면 되긴 함
   * 근데 그러면 결국 사용자 입장에서는 포인터를 받아 쓰게 되므로, pointer container와 체감상 큰 차이가 줄어듦
2. **값을 복사해서 넘겨주기**
   * shallow copy면 값 안에 포인터가 있을 때 문제가 그대로 남음
   * deep copy면 복사 비용, 버퍼 관리, 타입별 정책 문제가 생김

즉 node 기반 value-copy container의 핵심 문제는 “삽입이 어렵다”가 아니라, **generic한 조회/반환 API를 깔끔하게 만들기 어렵다**는 쪽에 더 가까움.

게다가 non-contiguous / node 기반 자료구조는 원래도 node마다 포인터 필드 같은 오버헤드가 붙고, allocation/free가 자주 일어나고, cache locality도 좋지 않은 편이다.         
그런데 여기에 value-copy 기반 API까지 얹으면 복사/반환 규칙까지 더 복잡해진다. (반대로 `GArray`는 반환에서 memcpy같은 복사가 일어나지만, capacity가 충분하면 추가에서 할당이 일어나지 않고 매모리와 캐시 효율이 좋다.)     
반면 `void *`를 저장하는 pointer container는 구현도 단순하고, list / tree / hash table 전반에 공통적으로 적용하기도 쉽다.  

그래서 node 기반 data structure에서도 value-copy 방식 자체는 가능하지만, 적어도 범용 C 라이브러리에서는 이런 특별한 사용 예시를 지원하지 않는 것 같다.   
아마 그런 이유로 GLib, c-algorithms, Collections-C 같은 라이브러리들도 non-contiguous / node 기반 자료구조는 다 pointer container 쪽으로 제공하는 것 같음.

## custom allocator / memory pool 을 사용하는 이유

기본적으로 C 자료구조 구현은 내부 메모리를 malloc/free로 직접 관리하는 경우가 많다.   
하지만 자료구조의 특성에 따라 allocation 패턴이 매우 자주 반복되거나, 작은 크기의 메모리를 대량으로 다루게 되면 일반 힙 할당만으로는 비효율적일 수 있다.   
이럴 때 custom allocator나 memory pool을 붙여 내부 메모리 관리 방식을 바꾸기도 한다.    

대표적으로 linked list, tree, chaining 기반 hash table처럼 node 단위 할당/해제가 자주 발생하는 자료구조에서 의미가 크다.

memory pool은 이런 node들을 위해 미리 큰 메모리 덩어리를 확보해 두고, 그 안에서 같은 크기의 node를 재사용하는 방식으로 동작할 수 있다.   
이 경우 매번 운영체제/일반 힙에 직접 요청하지 않고 pool 내부에서 빠르게 꺼내고 반납할 수 있다.   

**간단한 예시**

```c
// 이렇게 pool을 미리 할당해서 node 추가/삭제 시 할당이 발생하지 않게 한다.
#define POOL_COUNT 1024

typedef struct Node {
    void *data;
    struct Node *next;
} Node;

typedef struct NodePool {
    Node storage[POOL_COUNT];
    Node *free_list;
} NodePool;
```

- Node 할당
  - 평소: malloc(sizeof(Node))
  - pool 사용: pool_alloc(pool) → pool 안에서 Node 하나 꺼냄
- free
  - 평소: free(node)
  - pool 사용: pool_free(pool, node) → OS에 반납 안 하고 pool의 빈 칸 목록으로 되돌림

**사용 시 장점**
1. 작은 객체의 반복 할당/해제 비용 감소
   - node 기반 자료구조에서는 작은 크기의 allocation이 매우 자주 일어날 수 있다.
   - pool을 쓰면 이런 비용을 줄이기 쉽다.
2. fragmentation 완화 가능
   - 같은 크기의 node를 반복적으로 관리하면 일반 힙보다 메모리 배치가 단순해질 수 있다.
3. allocation 패턴 제어
   - 자료구조 내부 메모리를 어떤 방식으로 확보하고 반납할지 구현자가 통제하기 쉬워진다.
4. 같은 수명의 메모리 묶음 관리
   - arena/pool 계열 allocator를 쓰면 특정 자료구조 인스턴스와 함께 내부 메모리를 한꺼번에 정리하는 전략도 가능하다.
**사용 시 단점**
1. pool 고갈 처리 필요
2. variable-size payload에는 사용 불가능
3. 요소의 free 순서/소유권 문제는 여전함 (메모리풀은 node의 메모리 관리 문제만 보완해줌.)
4. 디버깅이 더 까다로울 수 있음 (실제로는 사용하는 상태라 use-after-free 문제를 OS나 분석 툴이 잡기 어려움)

## 대표적인 C 자료구조 라이브러리 코드 분석하기

편의를 위해 GLib는 Gitlab에서 관리되지만, Github에 있는 미러 레포지토리로 봄. 

### GLib - `GArray`

[`garray.h`](https://github.com/GNOME/glib/blob/2.88.0/glib/garray.h#L38-L59)는 배열 관련 타입을 선언하는데, 헤더에선 최소한으로 노출하고,

실제 구현하는 구조체는 `GRealArray`, `GRealPtrArray` 같은 구조체를 쓴다. 여기에는 더 많은 필드가 내부에 들어있음.

[예시](https://github.com/GNOME/glib/blob/2.88.0/glib/garray.c#L52-L73)
```c
// garray.h - https://github.com/GNOME/glib/blob/2.88.0/glib/garray.h#L38-L59
struct _GArray
{
  gchar *data;
  guint len;
};

// garray.c - https://github.com/GNOME/glib/blob/2.88.0/glib/garray.c#L52-L73
struct _GRealArray
{
  guint8 *data;
  guint   len;
  guint   elt_capacity;
  guint   elt_size;
  guint   zero_terminated : 1;
  guint   clear : 1;
  gatomicrefcount ref_count;
  GDestroyNotify clear_func;
};
```

[인덱스 접근인 `g_array_index(a,t,i)`](https://github.com/GNOME/glib/blob/2.88.0/glib/garray.h#L69)의 경우 매크로로, 내부적으론 일반적인 배열의 접근과 동일하다. 따라서 해당 키워드로 수정과 조회가 다 가능하다.

포인터가 아니므로 변수에 받는 순간 값 복사가 일어난다. 따라서 그 복사본을 수정해도 배열 원본에는 반영되지 않는다.

```c
// 정의
#define g_array_index(a,t,i)      (((t*) (void *) (a)->data) [(i)])

// 사용 예
Foo x = g_array_index(arr, Foo, 3);
g_array_index(arr, Foo, 3) = new_value;
```

2가지 삭제 연산을 제공하는데,
일반적으론 `memmove()`로 메모리를 이동함.  (예: `g_array_remove_index()`). `g_array_remove_index_fast()`는 빠른 삭제로 마지막 원소를 삭제 위치에 `memcpy()`로 덮어씀. 빠르지만 순서가 바뀜.

그 외에는 append, insert, update 류 연산을 제공하는데, 알고있는것과 크게 다르지 않음.

### [GLib - `GPtrArray`](https://docs.gtk.org/glib/struct.PtrArray.html)

`GArray`와 유사하지만, 포인터를 저장하기 위한 배열. 포인터를 위한 여러 편의 기능을 제공한다.

말했던 destroy callback 함수 중 하나인 [`g_ptr_array_set_free_func()`](https://docs.gtk.org/glib/type_func.PtrArray.set_free_func.html)   
`g_ptr_array_steal()`는 노드를 삭제하지만 free callback를 호출하지 않고 호출자에게 포인터의 소유권을 넘긴다.    
`g_ptr_array_new_null_terminated()`는ㄴ len과 별개로 끝에 NULL 센티넬을 유지하게 하는 생성자다.    

실제로 쓰기 위한 설명은 아니니 이정도로만 찾아봄.

### GLib - `GList` or `GSList`

`GList`는 내부 구조를 숨기지 않고 그대로 사용한다.

```c
// glist.h - https://github.com/GNOME/glib/blob/2.88.0/glib/glist.h#L39-L46
struct _GList
{
  gpointer data;
  GList *next;
  GList *prev;
};
```

특이한게 `len`이나 별도 정보를 저장하는 컨테이너가 없고, 노드 하나가 리스트의 일부인 형태다.  

다음과 같은 주의사항을 공식문서에서도 명시한다.

> The doubly-linked list does not keep track of the number of items and does not keep track of both the start and end of the list. If you want fast access to both the start and the end of the list, and/or the number of items in the list, use a GQueue instead.   
> 이중 연결 리스트는 항목의 개수를 저장하지 않으며, 리스트의 시작과 끝 위치도 저장하지 않습니다. 리스트의 시작과 끝 위치, 그리고 항목 개수에 빠르게 접근해야 한다면 GQueue 를 사용하는 것이 좋습니다.
> 
> Note that most of the GList functions expect to be passed a pointer to the first element in the list. The functions which insert elements return the new start of the list, which may have changed.   
> GList 함수 대부분은 리스트의 첫 번째 요소에 대한 포인터를 인수로 받는다는 점에 유의하십시오. 요소를 삽입하는 함수는 리스트의 새로운 시작 위치를 반환하는데, 이 위치는 변경될 수 있습니다.
> 
> [doubly-linked-lists - 공식문서](https://docs.gtk.org/glib/data-structures.html#doubly-linked-lists)

따라서 빈 리스트인 경우 `NULL`이고, 연산의 결과로 head가 바뀔 수 있다. (리스트 구조가 바뀔 수 있는 연산은 대체로 `GList *`를 반환하고, 그 반환값은 보통 처리 결과 리스트의 head라고 함.)  

제거는 2종류인데, `g_list_remove_link()`는 연결만 끊고 노드는 살려두고, `g_list_remove()`는 노드까지 해제한다.    
단, 둘 다 각 요소 data까지 free하지는 않으므로 직접 처리해야 한다.

노드의 추가/삭제에 따라서 노드의 할당과 해제가 발생하는 걸 볼 수 있다.    
[`g_list_append()`](https://github.com/GNOME/glib/blob/2.88.0/glib/glist.c#L166-L221)는 매크로나 여러 흐름을 타고 들어가면 최종적으로 [`g_slice_alloc()`](https://github.com/GNOME/glib/blob/2.88.0/glib/gslice.c#L171-L197)를 호출한다.   
[`g_list_remove()`](https://github.com/GNOME/glib/blob/2.88.0/glib/glist.c#L494-L525)는 free 하는 함수를 호출한다.


`GSList`는 Singly Linked List로, 그 외의 특징은 동일하므로 생략.

### GLib - `GHashTable`

TODO

### Collections-C - `sized array`

TODO

### Collections-C - `list` / `hash table`

TODO

### c-algorithms - `list` / `avl` / `hash`

TODO
