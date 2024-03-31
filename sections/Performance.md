
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
    // ... fill a ...

    // 100 chunks of memory of sizeof(double) starting at
    // address data using the order defined by compare_doubles
    qsort(data, 100, sizeof(double), compare_doubles);
```

From the point of view of interface design is that `qsort` throws away useful information.

We can do better (in C++98)

```c++
    template<typename Iter>
        void sort(Iter b, Iter e);  // sort [b:e)

    sort(data, data + 100);
```

Here, we use the compiler's knowledge about the size of the array, the type of elements, and how to compare `double`s.

With C++11 plus [concepts](#SS-concepts), we can do better still

```c++
    // Sortable specifies that c must be a
    // random-access sequence of elements comparable with <
    void sort(Sortable& c);

    sort(c);
```

The key is to pass sufficient information for a good implementation to be chosen.
In this, the `sort` interfaces shown here still have a weakness:
They implicitly rely on the element type having less-than (`<`) defined.
To complete the interface, we need a second version that accepts a comparison criteria:

```c++
    // compare elements of c using p
    void sort(Sortable& c, Predicate<Value_type<Sortable>> p);
```

The standard-library specification of `sort` offers those two versions,
but the semantics is expressed in English rather than code using concepts.

##### Note

Premature optimization is said to be [the root of all evil](#Rper-Knuth), but that's not a reason to despise performance.
It is never premature to consider what makes a design amenable to improvement, and improved performance is a commonly desired improvement.
Aim to build a set of habits that by default results in efficient, maintainable, and optimizable code.
In particular, when you write a function that is not a one-off implementation detail, consider

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

To decrease code size and run time.
To avoid data races by using constants.
To catch errors at compile time (and thus eliminate the need for error-handling code).

##### Example

```c++
    double square(double d) { return d*d; }
    static double s2 = square(2);    // old-style: dynamic initialization

    constexpr double ntimes(double d, int n)   // assume 0 <= n
    {
            double m = 1;
            while (n--) m *= d;
            return m;
    }
    constexpr double s3 {ntimes(2, 3)};  // modern-style: compile-time initialization
```

Code like the initialization of `s2` isn't uncommon, especially for initialization that's a bit more complicated than `square()`.
However, compared to the initialization of `s3` there are two problems:

* we suffer the overhead of a function call at run time
* `s2` just might be accessed by another thread before the initialization happens.

Note: you can't have a data race on a constant.

##### Example

Consider a popular technique for providing a handle for storing small objects in the handle itself and larger ones on the heap.

```c++
    constexpr int on_stack_max = 20;

    template<typename T>
    struct Scoped {     // store a T in Scoped
            // ...
        T obj;
    };

    template<typename T>
    struct On_heap {    // store a T on the free store
            // ...
            T* objp;
    };

    template<typename T>
    using Handle = typename std::conditional<(sizeof(T) <= on_stack_max),
                        Scoped<T>,      // first alternative
                        On_heap<T>      // second alternative
                   >::type;

    void f()
    {
        Handle<double> v1;                   // the double goes on the stack
        Handle<std::array<double, 200>> v2;  // the array goes on the free store
        // ...
    }
```

Assume that `Scoped` and `On_heap` provide compatible user interfaces.
Here we compute the optimal type to use at compile time.
There are similar techniques for selecting the optimal function to call.

##### Note

The ideal is {not} to try execute everything at compile time.
Obviously, most computations depend on inputs so they can't be moved to compile time,
but beyond that logical constraint is the fact that complex compile-time computation can seriously increase compile times
and complicate debugging.
It is even possible to slow down code by compile-time computation.
This is admittedly rare, but by factoring out a general computation into separate optimal sub-calculations it is possible to render the instruction cache less effective.

##### Enforcement

* Look for simple functions that might be constexpr (but are not).
* Look for functions called with all constant-expression arguments.
* Look for macros that could be constexpr.

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

Performance is very sensitive to cache performance and cache algorithms favor simple (usually linear) access to adjacent data.

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
