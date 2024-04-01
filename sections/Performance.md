
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

* Information passing:
Prefer clean [interfaces](#S-interfaces) carrying sufficient information for later improvement of implementation.
Note that information flows into and out of an implementation through the interfaces we provide.
* Compact data: By default, [use compact data](#Rper-compact), such as `std::vector` and [access it in a systematic fashion](#Rper-access).
If you think you need a linked structure, try to craft the interface so that this structure isn't seen by users.
* Function argument passing and return:
Distinguish between mutable and non-mutable data.
Don't impose a resource management burden on your users.
Don't impose spurious run-time indirections on your users.
Use [conventional ways](#Rf-conventional) of passing information through an interface;
unconventional and/or "optimized" ways of passing data can seriously complicate later reimplementation.
* Abstraction:
Don't overgeneralize; a design that tries to cater for every possible use (and misuse) and defers every design decision for later
(using compile-time or run-time indirections) is usually a complicated, bloated, hard-to-understand mess.
Generalize from concrete examples, preserving performance as we generalize.
Do not generalize based on mere speculation about future needs.
The ideal is zero-overhead generalization.
* Libraries:
Use libraries with good interfaces.
If no library is available build one yourself and imitate the interface style from a good library.
The [standard library](#S-stdlib) is a good first place to look for inspiration.
* Isolation:
Isolate your code from messy and/or old-style code by providing an interface of your choosing to it.
This is sometimes called "providing a wrapper" for the useful/necessary but messy code.
Don't let bad designs "bleed into" your code.

##### Example

Consider:

```c++
    template <class ForwardIterator, class T>
    bool binary_search(ForwardIterator first, ForwardIterator last, const T& val);
```

`binary_search(begin(c), end(c), 7)` will tell you whether `7` is in `c` or not.
However, it will not tell you where that `7` is or whether there are more than one `7`.

Sometimes, just passing the minimal amount of information back (here, `true` or `false`) is sufficient, but a good interface passes
needed information back to the caller. Therefore, the standard library also offers

```c++
    template <class ForwardIterator, class T>
    ForwardIterator lower_bound(ForwardIterator first, ForwardIterator last, const T& val);
```

`lower_bound` returns an iterator to the first match if any, otherwise to the first element greater than `val`, or `last` if no such element is found.

However, `lower_bound` still doesn't return enough information for all uses, so the standard library also offers

```c++
    template <class ForwardIterator, class T>
    pair<ForwardIterator, ForwardIterator>
    equal_range(ForwardIterator first, ForwardIterator last, const T& val);
```

`equal_range` returns a `pair` of iterators specifying the first and one beyond last match.

```c++
    auto r = equal_range(begin(c), end(c), 7);
    for (auto p = r.first(); p != r.second(), ++p)
        cout << *p << '\n';
```

Obviously, these three interfaces are implemented by the same basic code.
They are simply three ways of presenting the basic binary search algorithm to users,
ranging from the simplest ("make simple things simple!")
to returning complete, but not always needed, information ("don't hide useful information").
Naturally, crafting such a set of interfaces requires experience and domain knowledge.

##### Note

Do not simply craft the interface to match the first implementation and the first use case you think of.
Once your first initial implementation is complete, review it; once you deploy it, mistakes will be hard to remedy.

##### Note

A need for efficiency does not imply a need for [low-level code](#Rper-low).
High-level code does not imply slow or bloated.

##### Note

Things have costs.
Don't be paranoid about costs (modern computers really are very fast),
but have a rough idea of the order of magnitude of cost of what you use.
For example, have a rough idea of the cost of
a memory access,
a function call,
a string comparison,
a system call,
a disk access,
and a message through a network.

##### Note

If you can only think of one implementation, you probably don't have something for which you can devise a stable interface.
Maybe, it is just an implementation detail - not every piece of code needs a stable interface - but pause and consider.
One question that can be useful is
"what interface would be needed if this operation should be implemented using multiple threads? be vectorized?"

##### Note

This rule does not contradict the [Don't optimize prematurely](#Rper-Knuth) rule.
It complements it encouraging developers enable later - appropriate and non-premature - optimization, if and where needed.

##### Enforcement

Tricky.
Maybe looking for `void*` function arguments will find examples of interfaces that hinder later optimization.

### <a name="Rper-type"></a>Per.10: 정적 타입 시스템에 의지하라

##### Reason

Type violations, weak types (e.g. `void*`s), and low-level code (e.g., manipulation of sequences as individual bytes) make the job of the optimizer much harder. Simple code often optimizes better than hand-crafted complex code.

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
