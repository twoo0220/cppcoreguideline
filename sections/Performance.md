
# <a name="S-performance"></a>Per: 성능

??? should this section be in the main guide???

이 장에서는 높은 성능 또는 지연이 거의 없어야 하는 사람들을 위한 규칙을 포함하고 있다. 
다시 말해 예측 가능한 짧은 시간 안에 작업을 완료하기 위해 가능한 적은 시간과 리소스를 어떻게 사용하는지와 관련된 규칙이다.

규칙들은 (대부분의) 어플리케이션에 필요한 것보다 더 제한적이고 어떻게 보면 거슬릴 수도 있는 것이다.
일반적인 코드에 이러한 것들을 맹목적으로 따라 하지 않기 바란다:
지연을 없애는 목적을 달성하기 위해서는 굉장한 노력이 필요하다.

성능 규칙 요약:

* [Per.1: Don't optimize without reason](#Rper-reason)
* [Per.2: Don't optimize prematurely](#Rper-Knuth)
* [Per.3: Don't optimize something that's not performance critical](#Rper-critical)
* [Per.4: Don't assume that complicated code is necessarily faster than simple code](#Rper-simple)
* [Per.5: Don't assume that low-level code is necessarily faster than high-level code](#Rper-low)
* [Per.6: Don't make claims about performance without measurements](#Rper-measure)
* [Per.7: Design to enable optimization](#Rper-efficiency)
* [Per.10: Rely on the static type system](#Rper-type)
* [Per.11: Move computation from run time to compile time](#Rper-Comp)
* [Per.12: Eliminate redundant aliases](#Rper-alias)
* [Per.13: Eliminate redundant indirections](#Rper-indirect)
* [Per.14: Minimize the number of allocations and deallocations](#Rper-alloc)
* [Per.15: Do not allocate on a critical branch](#Rper-alloc0)
* [Per.16: Use compact data structures](#Rper-compact)
* [Per.17: Declare the most used member of a time-critical struct first](#Rper-struct)
* [Per.18: Space is time](#Rper-space)
* [Per.19: Access memory predictably](#Rper-access)
* [Per.30: Avoid context switches on the critical path](#Rper-context)

### <a name="Rper-reason"></a>Per.1: 이유없이 최적화하지 마라

##### Reason

최적화가 필요하지 않은 경우, 최적화에 쏟은 노력의 주된 결과는 더 많은 오류와 더 높은 유지 관리 비용으로 이어집니다.

##### Note

일부는 습관적으로 또는재밌어서 최적화를 하기도 합니다.

???

### <a name="Rper-Knuth"></a>Per.2: 성급하게 최적화하지 마라

##### Reason

정교하게 최적화된 코드는 일반적으로 최적화되지 않은 코드보다 규모가 크고 변경하기가 더 어렵습니다.

???

### <a name="Rper-critical"></a>Per.3: 성능 크리티컬하지 않는 곳에는 최적화를 하지 마라

##### Reason

프로그램의 성능에 중요하지 않은 부분을 최적화해도 시스템 성능에는 영향을 미치지 않습니다.

##### Note

프로그램이 대부분의 시간을 웹이나 사람을 기다리는 데 소비한다면 내부 메모리 연산 최적화는 쓸모없을 수도 있습니다.

다시 말해: 만약 프로그램이 처리 시간의 4%를 계산 A를 수행하고 40%의 시간을 계산 B에 사용하는 경우, A를 50% 개선하는 것은 B를 5% 개선하는 것만큼만 영향을 미칩니다(A 또는 B에 얼마나 많은 시간을 소비하는 지조차 모른다면, <a href="#Rper-reason">Per.1</a> 와 <a
href="#Rper-Knuth">Per.2</a>를 참조하세요).

### <a name="Rper-simple"></a>Per.4: 복잡한 코드가 간단한 코드보다 빠르다고 추측하지 마라

##### Reason

간단한 코드는 매우 빠를 수 있습니다. 가끔 최적화는 간단한 코드로 놀라운 일을 해냅니다.

##### Example, good

```c++
    // 명확한 의도 표현, 빠른 실행

    vector<uint8_t> v(100000);

    for (auto& c : v)
        c = ~c;
```

##### Example, bad

```c++
    // 더 빠르게 하려고 의도했지만, 실제로는 느립니다.

    vector<uint8_t> v(100000);

    for (size_t i = 0; i < v.size(); i += sizeof(uint64_t))
    {
        uint64_t& quad_word = *reinterpret_cast<uint64_t*>(&v[i]);
        quad_word = ~quad_word;
    }
```

##### Note

???

???

### <a name="Rper-low"></a>Per.5: 저수준 코드가 고수준 코드보다 필연적으로 빠를 것이라고 추측하지 마라

##### Reason

저수준(low-level) 코드는 때때로 최적화를 방해합니다. 최적화는 가끔 고수준(high-level) 코드로 놀라운 일을 해냅니다.

##### Note

???

???

### <a name="Rper-measure"></a>Per.6: 측정없이 성능 개선을 주장하지 마라

##### Reason

성능 분야는 미신과 가짜 속설(bogus folklore)로 가득 차 있습니다. 최신 하드웨어와 최적화 도구는 단순한 가정들을(naive assumptions) 뒤엎고 있으며 전문가들조차 종종 놀라움을 금지 못합니다.

##### Note

정확한 성능 측정은 어렵고 전문적인 도구가 필요할 수 있습니다.

##### Note

Unix `time`이나 표준 라이브러리 `<chrono>`를 사용한 몇 가지 간단한 마이크로 벤치마크는 가장 확실한 오해를 불식시키는 데 도움이 될 수 있습니다. 전체 시스템을 정확하게 측정할 수 없다면 최소한 몇 가지 주요 작업과 알고리듬을 측정해 보세요. 프로파일러(profiler)는 시스템의 어느 부분이 성능에 중요한지 알려줄 수 있습니다. 종종, 놀랄 것입니다.

???

### <a name="Rper-efficiency"></a>Per.7: 최적화가 가능하게 설계하라

##### Reason

초기 디자인을 최적화해야 하는 경우가 많기 때문입니다.  
나중에 개선할 가능성을 무시한 디자인은 변경하기 어렵기 때문입니다.

##### Example

C (와 C++) 표준에서:

```c++
    void qsort (void* base, size_t num, size_t size, int (*compar)(const void*, const void*));
```

언제 메모리를 정렬하고 싶어하십니까?
실제로, 우리는 일반적으로 컨테이너에 저장된 요소들의 순서를 정렬합니다. `qsort` 함수를 호출하면 많은 유용한 정보(예: 요소 유형)가 버리고, 사용자가 이미 알고 있는 정보(예: 요소 크기)를 반복해야 하며, 추가 코드(예: `double`을 비교하는 함수)를 작성하도록 강요합니다.
이것은 프로그래머의 추가 작업을 의미하고 오류가 발생하기 쉬우며 컴파일러에서 최적화에 필요한 정보를 빼앗아갑니다.

```c++
    double data[100];

    // compare_doubles 함수로 정의된 순서를 사용하여 주소 데이터에서
    // 시작하는 sizeof(double)의 메모리 덩어리(chunk) 100개 채우기
    qsort(data, 100, sizeof(double), compare_doubles);
```

인터페이스 디자인의 관점에서 볼 때 `qsort`는 유용한 정보를 버린다는 것입니다.

우리는 더 좋게할 수 있습니다 (C++ 98에서)
We can do better (in C++98)

```c++
    template<typename Iter>
        void sort(Iter b, Iter e);  // sort [b:e)

    sort(data, data + 100);
```

여기서는 배열의 크기, 요소의 유형, `double`을 비교하는 방법에 대한 컴파일러의 지식을 사용합니다.

C++ 11과 [개념(concepts)](#SS-concepts)을 사용하면 더 좋게 만들 수 있습니다.

```c++
    // Sortable 자료형은 < 와 비슷한 요소의 무작위 접근 순서여야 한다고 지정합니다.
    void sort(Sortable& c);

    sort(c);
```

핵심은 좋은 구현이 선택될 수 있도록 충분한 정보를 전달하는 것입니다. 여기서 여기 표시된 `sort` 인터페이스는 여전히 약점이 있습니다: (`<`) 보다 작은 요소 유형이 정의된 것에 암시적으로 의존한다는 점입니다. 인터페이스를 완성하려면 비교 기준을 허용하는 두 번째 버전이 필요합니다:

```c++
    // p를사용하여 C의 요소와 비교하기  
    void sort(Sortable& c, Predicate<Value_type<Sortable>> p);
```

표준 라이브러리 사양인 `sort`는 이 두 가지 버전을 제공합니다. 하지만 의미론(semantics)은 개념을 사용하는 코드가 아닌 영어로 표현됩니다.

##### Note

이른 최적화는 [모든 악의 근원](#Rper-Knuth)이라고 하지만, 그렇다고해서 성능을 경멸할 이유는 없습니다.  디자인을 개선할 수 있는 요소를 고려하는 것은 절대 이르지 않으며, 성능 개선은 일반적으로 바람직한 개선입니다.기본적으로 효율적이고, 유지 관리가 용이하며, 최적화 가능한 코드를 작성하는 일련의 습관을 기르는 것을 목표로 하세요. 특히 일회성 구현 세부 사항이 아닌 함수를 작성할 때는 다음 사항을 고려하세요.

* 정보 전달:
나중에 구현을 개선할 수 있도록 충분한 정보를 전달하는 깔끔한 [인터페이스](#S-interfaces)를 선호하세요. 정보는 우리가 제공하는 인터페이스를 통해 구현으로 들어오고 나간다는 점을 유의하세요.
* 데이터 압축: 기본적으로, `std::vector`와 같은 [압축 데이터 사용](#Rper-compact)과 [체계적인 방식으로 접근](#Rper-access)하는 것을 권장합니다. 연결된 구조체가 필요하다면, 이 구조체가 사용자에게 보이지 않도록 인터페이스를 공들여 만드세요.
* 함수 인자 전달 및 반환: 수정 가능한 데이터와 수정 불가능한 데이터를 구분하세요. 사용자에게 리소스 관리 부담을 주지 마세요. 사용자에게 잘못된 런타임 간접 참조를 강요하지 마세요. 인터페이스를 통해 정보를 전달하는 [기존 방식](#Rf-conventional)을 사용하세요; 기존 방식이 아니거나 "최적화된" 데이터 전달 방식은 나중에 재구현을 심각할 정도로 복잡하게 만들 수 있습니다.
* 추상화: 지나치게 일반화하지 마세요. 가능한 모든 사용(및 오용)을 수용하려 하고 모든 디자인 결정을 나중으로(컴파일 타임 또는 런타임 간접 참조를 사용하여) 미루는 디자인은 일반적으로 복잡하고 규모가 커지며 이해하기 어렵도록 엉망진창이 됩니다. 구체적인 사례를 통해 일반화하되, 일반화할 때 성능을 유지합니다. 미래의 필요성에 대한 단순한 추측을 바탕으로 일반화하지 마세요. 가장 이상적인 것은 제로 오버헤드(zero-overhead) 일반화입니다.
* 라이브러리: 인터페이스가 좋은 라이브러리를 사용하세요. 사용할 수 있는 라이브러리가 없다면 직접 라이브러리를 만들고 좋은 라이브러리의 인터페이스 스타일을 모방하세요. [표준 라이브러리](#S-stdlib)는 가장 먼저 영감을 얻을 수 있는 좋은 곳입니다.
* 분리(isolation): 원하는 인터페이스를 제공하여 지저분하거나 오래된 스타일의 코드로부터 분리하세요. 이것은 유용하고 필요하지만 지저분한 코드를 위한 "래퍼(wrapper) 제공"이라고도 불립니다. 나쁜 디자인이 코드에 "스며들지" 않도록 하세요.

##### Example

Consider:

```c++
    template <class ForwardIterator, class T>
    bool binary_search(ForwardIterator first, ForwardIterator last, const T& val);
```

`binary_search(begin(c), end(c), 7)` 는 `7`이 `c`에 있는지 여부를 알려줍니다. 그러나 그 `7`이 어디에 있는지 아니면 `7`이 두 개 이상 있는지 여부를 알려주지 않습니다.

때로는 최소한의 정보량(여기서는 `true` 또는 `false`)만 전달해도 충분하지만, 좋은 인터페이스 필요한 정보를 호출자에게 다시 전달합니다. 그러니 표준 라이브러리는 다음과 같은 기능도 제공합니다.

```c++
    template <class ForwardIterator, class T>
    ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T& val);
```

`lower_bound` 는 일치하는 요소가 있으면 첫 번째 요소로, 일치하지 않으면 `val`보다 큰 첫 번째 요소로, 일치하는 요소가 없으면 `last`로 반복자를 반환합니다.

그러나 `lower_bound`는 여전히 모든 용도에 대해 충분한 정보를 반환하지 않으므로 표준 라이브러리에서도 다음을 제공합니다

```c++
    template <class ForwardIterator, class T>
    pair<ForwardIterator, ForwardIterator>
    equal_range(ForwardIterator first, ForwardIterator last, const T& val);
```

`equal_range`는 첫 번째와 마지막 일치 이후를 지정하는 반복자의 `pair`를 반환합니다.

```c++
    auto r = equal_range(begin(c), end(c), 7);
    for (auto p = r.first(); p != r.second(), ++p)
        cout << *p << '\n';
```

물론 이 세 가지 인터페이스는 동일한 기본 코드로 구현됩니다. 가장 간단한 방법("간단한 것을 간단하게!")부터 완전한 정보를 반환하지만 항상 필요한 것은 아닌 정보("유용한 정보를 숨기기 마세요")에 이르기까지 기본적인 이진 탐색 알고리듬을 사용자에게 제시하는 세 가지 방법일 뿐입니다. 당연히 이러한 인터페이스를 제작하려면 경험과 도메인 지식이 필요합니다.

##### Note

단순히 첫 번째 구현과 가장 먼저 생각나는 사용 사례에 맞춰 인터페이스를 만들지 마세요. 첫 번째 초기 구현이 완료되면 이를 검토하세요. 일단 배포되면 실수를 수정하기 어렵습니다.

##### Note

효율성의 필요성은 [저수준 코드](#Rper-low)의 필요성을 의미하지는 않습니다. 고수준 코드는 느리거나 비대하다는 뜻이 아닙니다.

##### Note

모든 일에는 비용이 듭니다. 비용에 대해 지나치게 걱정할 필요는 없지만(최신 컴퓨터는 정말 빠릅니다), 자신이 사용하는 것의 비용 규모를 대략적으로 파악하세요.
예를 들어 메모리 접근, 함수 호출, 문자열 비교, 시스템 호출, 디스크 접근, 네트워크를 통한 메시지의 비용에 대해 대략적으로 파악하세요.

##### Note

한 가지 구현만 떠올릴 수 있다면, 아마도 안정적인 인터페이스를 고안할 수 있는 무언가가 없는 것입니다.
모든 코드에 안정적인 인터페이스가 필요한 것은 아니므로 구현 세부 사항일 수도 있지만 잠시 멈춰고 생각해 보세요.
"이 작업을 멀티 스레드를 사용하여 구현해야만 한다면 어떤 인터페이스가 필요할까? 벡터화할까?"라는 질문이 유용할 수 있습니다.

##### Note

이 규칙은 [성급하게 최적화하지 마세요](#Rper-Knuth) 규칙과 모순되지 않습니다. 이 규칙은 개발자가 필요한 경우 적절하고 시기상조가 아닌 최적화를 나중에 활성화하도록 권장함으로써 이를 보완합니다.

##### Enforcement

까다롭습니다.
아마도 `void*` 함수 인자를 찾아보면 나중에 최적화를 방해하는 인터페이스의 예시를 찾을 수 있을 것입니다.

### <a name="Rper-type"></a>Per.10: 정적 타입 시스템에 의지하라

##### Reason

타입 위반, 약한 타입(예: `void*`), 저수준 코드(예: 개별 바이트 단위로 순서 조작)는 최적화 도구의 작업을 훨씬 더 어렵게 만듭니다. 보통 간단한 코드가 수작업으로 만든 복잡한 코드보다 더 잘 최적화되는 경우가 많습니다.

???

### <a name="Rper-Comp"></a>Per.11: 계산을 실행 시간에서 컴파일 시간으로 옮겨라

##### Reason

코드 크기와 런타임을 줄이기 위해.
상수를 사용하여 데이터 경합을 피하기 위해.
컴파일 시 오류를 포착하기 위해(오류 처리 코드의 필요성을 제거합니다.)

##### Example

```c++
    double square(double d) { return d*d; }
    static double s2 = square(2);    // 옛날 방식 : 동적 초기화

    constexpr double ntimes(double d, int n)   // assume 0 <= n
    {
            double m = 1;
            while (n--) m *= d;
            return m;
    }
    constexpr double s3 {ntimes(2, 3)};  // 최신 방식 : 컴파일 시 초기화
```

`s2`의 초기화와 같은 코드는 드물지 않으며, 특히 `square()`보다 조금 더 복잡한 초기화의 경우 더욱 그렇습니다. 그러나 `s3`의 초기화와 비교하면 두 가지 문제가 있습니다:

* 런타임에 함수 호출로 인한 오버헤드를 겪게 됩니다.
* 초기화가 일어나기 전에 다른 스레드에서 `s2`에 접근할 수 있습니다.

Note: 상수를 두고 데이터 경합을 할 수 없습니다.

##### Example

작은 객체는 핸들 자체에, 큰 객체는 힙에 저장할 수 있는 핸들을 제공하는 인기 있는 기법을 고려해 보세요.

```c++
    constexpr int on_stack_max = 20;

    template<typename T>
    struct Scoped {     // Scoped 안에 T 저장
            // ...
        T obj;
    };

    template<typename T>
    struct On_heap {    // 힙에(free store) T 저장
            // ...
            T* objp;
    };

    template<typename T>
    using Handle = typename std::conditional<(sizeof(T) <= on_stack_max),
                        Scoped<T>,      // 첫 번째 대안
                        On_heap<T>      // 두 번쨰 대안
                   >::type;

    void f()
    {
        Handle<double> v1;                   // double은 스택에 저장됩니다.
        Handle<std::array<double, 200>> v2;  // array는 힙에(free store) 저장됩니다.
        // ...
    }
```

`Scoped`와 `On_heap`이 호환 가능한 사용자 인터페이스를 제공한다고 가정합니다. 여기서는 컴파일 시 사용할 최적의 타입을 계산합니다. 호출할 최적의 함수를 선택하는 데는 비슷한 기법들이 있습니다.

##### Note

모든 것을 컴파일 타임에 실행하는 것이 이상적인 것은 {아닙니다}. 물론 대부분의 계산은 입력에 의존하기 때문에 컴파일 타임으로 옮길 수 없습니다. 하지만 이러한 논리적 제약을 넘어 복잡한 컴파일 타임 계산은 컴파일 시간을 심각하게 증가시킬 수 있고 디버깅을 복잡하게 만들 수 있습니다. 심지어 컴파일 시간 계산으로 인해 코드 속도가 느려질 수도 있습니다. 물론 이런 경우는 드물지만 일반적인 계산을 별도의 최적 하위 계산으로 분리하면 명령어 캐시의 효율성을 떨어뜨릴 수 있습니다.

##### Enforcement

* constexpr 일 수 있지만 그렇지 않은 간단한 함수를 찾습니다.
* 모든 상수 표현식 인수로 호출되는 함수를 찾습니다.
* constexpr 이 될 수 있는 매크로를 찾습니다.

### <a name="Rper-alias"></a>Per.12: 불필요한 별칭을 제거하라

???

### <a name="Rper-indirect"></a>Per.13: 불필요한 간접 접근(indirection)을 제거하라

???

### <a name="Rper-alloc"></a>Per.14: 할당과 해제의 수를 최소화하라

???

### <a name="Rper-alloc0"></a>Per.15: 중요 분기에서는 할당을 하지 마라

???

### <a name="Rper-compact"></a>Per.16: 압축적인 자료 구조를 사용하라

##### Reason

보통 성능은 메모리 접근시간에 반비례한다.

???

### <a name="Rper-struct"></a>Per.17: 빠른 처리를 요구하는 구조체에서 가장 많이 쓰이는 멤버를 먼저 선언해라

???

### <a name="Rper-space"></a>Per.18: 공간은 시간이다

##### Reason

보통 성능은 메모리 접근시간에 반비례한다.

???

### <a name="Rper-access"></a>Per.19: 예측할 수 있는 메모리 접근을 해라

##### Reason

성능은 캐시 성능에 매우 민감하며 캐시 알고리듬은 인전한 데이터에 대한 간단한(일반적으로 선형) 접근을 선호합니다.

##### Example

```c++
    int matrix[rows][cols];

    // bad
    for (int c = 0; c < cols; ++c)
        for (int r = 0; r < rows; ++r)
            sum += matrix[r][c];

    // good
    for (int r = 0; r < rows; ++r)
        for (int c = 0; c < cols; ++c)
            sum += matrix[r][c];
```

### <a name="Rper-context"></a>Per.30: 중요 경로에서 문맥전환을 피하라

???
