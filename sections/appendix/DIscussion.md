
# <a name="S-discussion"></a>Appendix C: Discussion

이 절은 규칙들과 규칙들의 집합에 대한 추가적인 내용을 담고 있다.  
특히, 여기서 추가적인 이유, 더 긴 예제, 대안책들에 대한 토론들을 표현하고자 한다.

### <a name="Sd-order"></a>토론: 멤버 변수들을 멤버 선언 순서에 따라 정의하고 초기화하라

멤버 변수들은 항상 클래스의 정의부에서 선언된 순서에 따라 초기화되므로, 생성자의 초기화리스트에 그 순서대로 작성해야 한다. 다른 순서로 작성하는 것은 멤버 변수들이 눈에 보이는 순서대로 실행되지 않아 순서에 종속적인 버그를 찾기 어렵게 만들며 코드를 헷갈리게할 뿐이다.

```c++
    class Employee {
        string email, first, last;
    public:
        Employee(const char* firstName, const char* lastName);
        // ...
    };

    Employee::Employee(const char* firstName, const char* lastName)
      : first(firstName),
        last(lastName),
        // BAD: first 와 last 가 어직 생성되지 않음
        email(first + "." + last + "@acme.com")
    {}
```

이 예제에서 `email`은 맨 처음에 선언되었기 때문에 `first`와 `last` 보다 먼저 생성된다. 그 말은 `first`와 `last`가 원하는 값으로 설정되기 전이 아니라 아예 생성되기도 전에 `email`의 생성자가 사용하려 한다는 뜻이다.

클래스의 정의부와 생성자 본문이 서로 다른 파일에 있다면, 멤버 변수의 선언 순서가 생성자의 정확성에 미치는 장거리 영향력을 파악하기 훨씬 더 어려워진다.

**References**:

* [\[Cline99\]](../Bibliography.md) §22.03-11
* [\[Dewhurst03\]](../Bibliography.md) §52-53
* [\[Koenig97\]](../Bibliography.md) §4
* [\[Lakos96\]](../Bibliography.md) §10.3.5
* [\[Meyers97\]](../Bibliography.md) §13
* [\[Murray93\]](../Bibliography.md) §2.1.3
* [\[Sutter00\]](../Bibliography.md) §47

### <a name="Sd-init"></a>토론: 초기화로 `=`, `{}`, `()` 를 사용하라 

???

### <a name="Sd-factory"></a>토론: 초기화 중에 "가상 동작(virtual behavior)"이 필요한 경우 팩토리(factory) 함수를 사용하라

`f`와 `g` 같은 함수에 대한 기반 클래스 생성자 또는 소멸자로부터 파생 클래스로 가상 디스패치를 원하는 경우, 다른 기법이 필요하다. 예를 들어 포스트 생성자 기법은 호출자가 초기화를 완료하기 위해 호출해야 하는 별도의 멤버 함수로, 멤버 함수에서는 가상 호출이 정상적으로 작동하므로 `f`와 `g`를 안전하게 호출할 수 있다. 이외에도 몇 가지 기법이 참고 문헌에 나와있다. 다음은 전체가 아닌 옵션 목록이다:

* *책임 전가(Pass the buck):* 사용자 코드가 객체를 생성한 직후에 초기화이 후 함수를 호출해야 한다는 점을 문서화하라
* *느린 초기화 후(Post-initialize lazily):* 멤버 함수를 처음 호출하는 동안 수행하라. 기반 클래스의 부울 플래그를 통해 객체 생성 후 초기화가 이루어지지 않았는지 여부를 확인하라
* *가상 기반 클래스 시멘틱 사용(Use virtual base class semantics):* 언어 규칙에 따라 가장 많이 파생된 클래스가 어떤 기본 생성자를 호출할지 결정하므로 이를 유리하게 사용할 수 있다. ([\[Taligent94\]](../Bibliography.md) 참고)
* *팩토리 함수를 사용하라:* 이렇게 하면 생성자 이후 함수의 필수 호출을 쉽게 강제할 수 있다.

다음은 마지막 옵션의 예시 입니다:

```c++
    class B {
    public:
        B()
        {
            /* ... */
            f(); // BAD: C.82: 생성자 및 소멸자에서 가상 함수를 호출하지 마라
            /* ... */
        }

        virtual void f() = 0;
    };

    class B {
    protected:
        class Token {};

    public:
        // 생성자는 make_shared가 접근할 수 있도록 public이어야 한다.
        // protected 접근 수준은 Token을 요구하여 얻을 수 있다.
        explicit B(Token) { /* ... */ } // 불완전하게 초기화된 객체 생성
        virtual void f() = 0;

        template<class T>
        static shared_ptr<T> create()    // 객체 생성을 위한 인터페이스
        {
            auto p = make_shared<T>(typename T::Token{});
            p->post_initialize();
            return p;
        }

    protected:
        virtual void post_initialize()    // 생성 직후 호출
            { /* ... */ f(); /* ... */ }   // GOOD: 가상 디스패치는 안전합니다
    };


    class D : public B {                 // 일부 파생 클래스
    protected:
        class Token {};

    public:
        // 생성자는 make_shared가 접근할 수 있도록 public이어야 한다.
        // protected 접근 수준은 Token을 요구하여 얻을 수 있다.
        explicit D(Token) : B{ B::Token{} } {}
        void f() override { /* ...  */ };

    protected:
        template<class T>
        friend shared_ptr<T> B::create();
    };

    shared_ptr<D> p = D::create<D>();    // D 객체 생성
```

이 설계에는 다음과 같은 원칙이 필요하다:

* `D`와 같은 파생 클래스는 공개적으로 호출 가능한 생성자를 노출해서는 안된다. 그렇지 않으면 `D`의 사용자가 `초기화 후 함수(PostInitialize)`를 호출하지 않은 `D`객체를 생성할 수 있다.
* 할당은 `operator new`로 제한된다. 그러나 `B`는 `new`를 재정의할 수 있다([SuttAlex05](../Bibliography.md)의 Items 45 and 46 참조).
* `D`는 `B`가 선택한 것과 동일한 매개변수를 가진 생성자를 정의해야만 한다. 그러나 `create`의 여러 오버로드를 정의하면서 이 문제를 완화할 수 있으며, 오버로드는 인수 유형에 템플릿을 지정할 수도 있다.  

위의 요구사항이 충족되면, 설계는 완전히 생성된 모든 `B` 파생 객체에 대해 `PostInitialize`가 호출되었음을 보장한다. `PostInitialize`는 가상일 필요는 없지만 가상 함수를 자유롭게 호출할 수 있어야 한다.

요약하면, 완벽한 생성 후 기법은 존재하지 않는다. 최악의 기법은 호출자에게 생성자 이후 수동으로 호출하도록 요청함으로써 모든 문제를 회피하는 것이다. 가장 좋은 방법도 객체를 구성하는 다른 구문(컴파일 때 쉽게 확인 가능)이나 파생 클래스 작성자의 협조(컴파일 때 확인 불가능)가 필요하다.

**References**:

* [\[Alexandrescu01\]](../Bibliography.md) §3
* [\[Boost\]](../Bibliography.md)
* [\[Dewhurst03\]](../Bibliography.md) §75
* [\[Meyers97\]](../Bibliography.md) §46
* [\[Stroustrup00\]](../Bibliography.md) §15.4.3
* [\[Taligent94\]](../Bibliography.md)

### <a name="Sd-dtor"></a>토론: 기반 클래스 소멸자를 공개, 가상 또는 보호 및 비가상으로 설정하라

소멸이 가상으로 작동해야만 하는가? 즉, `base` 클래스에 대한 포인터를 통한 소멸이 허용되어야 하는가?  
위 질문에 대한 대답이 '그렇다' 라면, `base` 의 소멸자는 호출 가능해야 하고 그렇지 않으면 가상으로 호출하면 정의되지 않은 동작이 발생한다.  
대답이 '아니오' 라면, 파생 클래스만 자체 소멸자에서 호출할 수 있도록 보호해야 하며, 가상으로 동작할 필요가 없으므로 비가상이어야 한다.

##### 예제

기반 클래스는 보통 공개적으로 파생된 클래스를 갖기 위한 것이므로 코드를 호출할 때 `shared_ptr<base>`와 같은 것을 사용해야 한다:

```c++
    class Base {
    public:
        ~Base();                   // BAD, 비가상
        virtual ~Base();           // GOOD
        // ...
    };

    class Derived : public Base { /* ... */ };

    {
        unique_ptr<Base> pb = make_unique<Derived>();
        // ...
    } // ~Base 가 가상인 경우에만 ~pb가 올바른 소멸자를 호출
```

단위 전략(policy) 클래스와 같이 드문 경우에 다형성 동작이 아닌 편의상 기반 클래스로 사용된다. 이러한 소멸자는 protected와 비가상으로 만드는 것이 좋다:

```c++
    class My_policy {
    public:
        virtual ~My_policy();      // BAD, public 과 virtual
    protected:
        ~My_policy();              // GOOD
        // ...
    };

    template<class Policy>
    class customizable : Policy { /* ... */ }; // note: private 상속
```

##### Note

이 간단한 가이드라인은 미묘한 문제를 설명하며 상속 및 객체 지향 설계 원칙의 현대적인 사용을 반영한다.

기반 클래스 `Base`의 경우, `unique_ptr<Base>`를 사용할 때와 같이 호출 코드가 `Base`에 대한 포인터를 통해 파생 객체를 삭제하려 할 수 있다. `Base`의 소멸자가 public이고 비가상인 경우(기본값), 실제로 파생 객체를 가리키는 포인터에서 실수로 호출될 수 있으며, 이 경우 삭제 시도의 동작이 정의되지 않는다. 이러한 상황으로 인해 이전 코딩 표준에서는 모든 기반 클래스 소멸자가 가상이어야 한다는 포괄적인 요건을 부과했다. 이것은 (일반적인 경우라고 해도) 지나친 요구다. 대신 기반 클래스 소멸자가 public인 경우에만 가상으로 만드는 것으로 규칙을 만들어야 한다.

기반 클래스를 만드는 것은 추상화를 정의하는 것이다(Items 35 ~ 37 참조). 해당 추상화에 참여하는 각 멤버 함수에 대해 결정해야 한다는 점을 상기하라:

* 가상으로 동작해야 하는지 여부.
* `Base`에 대한 포인터를 사용하여 모든 호출자가 공개적으로 사용할 수 있어야 하는지 아니면 숨겨진 내부 구현 세부 사항이어야 하는지 여부

Item 39에 설명된 대로, 일반 멤버 함수의 경우, `Base`에 대한 포인터를 통해 호출할 수 있도록 허용할지, 가상 함수를 호출하는 경우 가상 동작을 포함할지(NVI 또는 템플릿 메서드 패턴과 같이 가상 함수를 호출하는 경우), 가상으로 호출할지 또는 전혀 호출하지 않을지 선택할 수 있다. NVI 패턴은 public 가상 함수를 피하기 위한 기법이다.(NVI : Non-virtual-interface)

소멸은 비가상 호출을 위험하거나 잘못되게 만드는 특별한 시맨틱(semantics)이 있긴 하지만, 또 다른 연산으로 볼 수 있다. 그러므로 기반 클래스 소멸자의 경우, `Base`에 대한 포인터를 통해 가상으로 호출할 수 있는지 아니면 전혀 호출할 수 없는지를 선택해야 하며, "비가상"은 옵션이 아니다. 이런 이유로 기반 클래스 소멸자는 호출할 수 있는 경우(즉, public인 경우) 가상이고, 그렇지 않은 경우 비가상이다.

생성자와 소멸자는 심층 가상 호출을 할 수 없으므로 소멸자에는 NVI 패턴을 적용할 수 없다.(Items 39 및 55 참조.)

결론: 기반 클래스를 작성할 때는 항상 소멸자를 명시적으로 작성하라. 암시적으로 생성된 소멸자는 public이고 비가상이기 떄문이다. 기본 형태가 괜찮고 적절한 가시성과 가상성을 제공하는 함수를 작성하는 경우 언제든지 구현을 `=default`로 설정할 수 있다.

##### 예외

일부 컴포넌트 아키텍처(예: COM 및 CORBA)는 표준 삭제 메커니즘을 사용하지 않으며, 객체 처리를 위해 다른 프로토콜을 장려한다. 로컬 패턴(local pattern)과 관용구를 따르고 이 가이드라인을 적절히 적용하라.

또한 다음과 같이 드문 경우들도 고려하라:

* `B`는 기반 클래스인 동시에 그 자체로 인스턴스화할 수 있는 구체적인 클래스이므로 `B` 객체를 생성하고 소멸하려면 소멸자가 public이어야 한다.

소멸자가 public이 되어야 하더라도, 첫 번째 가상 함수로서 추가 기능이 필요하지 않은데도 모든 런타임 유형 오버헤드가 발생하기 때문에 가상으로 만들지 않는 것이 큰 부담이 될 수 있다.

드문 경우지만, 소멸자를 public이고 비가상으로 만들되, 추가 파생 객체를 `B`처럼 다형성으로 사용해서는 안 된다는 점을 명확하게 문서화할 수 있다. 이것이 `std::unary_function`으로 수행한 작업이다.

그러나 일반적으로 구체적인 기반 클래스는 피하라(Item 35 참조). 예를 들어, `unary_function`은 독립적으로 인스턴스화되도록 의도되지 않은 타입 정의 번들이다. 이 항목의 조언을 따르고 protected, 비가상 소멸자를 제공하는 것이 더 나은 설계이다.

**References**:

* [\[C++CS\]](../Bibliography.md) Item 50
* [\[Cargill92\]](../Bibliography.md) pp. 77-79, 207
* [\[Cline99\]](../Bibliography.md) §21.06, 21.12-13
* [\[Henricson97\]](../Bibliography.md) pp. 110-114
* [\[Koenig97\]](../Bibliography.md) Chapters 4, 11
* [\[Meyers97\]](../Bibliography.md) §14
* [\[Stroustrup00\]](../Bibliography.md) §12.4.2
* [\[Sutter02\]](../Bibliography.md) §27
* [\[Sutter04\]](../Bibliography.md) §18

### <a name="Sd-noexcept"></a>토론: noexcept 사용

???

### <a name="Sd-never-fail"></a>토론: 소멸자, 할당 해제, 스왑(swap)은 절대 실패해서는 안 된다

소멸자, 리소스 할당 해제 함수(예: `delete 연산자`) 또는 `throw`를 사용하는 `swap` 함수에서 오류가 보고되는 것을 허용하지 마라. 이러한 연산이 실패할 수 있는 경우 유용한 코드를 작성하는 것은 거의 불가능하며, 문제가 발생하더라도 재시도하는 것은 거의 의미가 없다. 특히 소멸자가 예외를 던질 수 있는 타입은 C++ 표준 라이브러리에서 사용이 전면적으로 금지되어 있다. 현재 대부분의 소멸자는 암시적으로 기본값이 `noexcept`이다.

##### 예제
- `Nefarious` : 비난의 뜻이 강한 말로, 법이나 전통의 위반을 암시하고, 보통 극도로 사악한 것을 의미

```c++
    class Nefarious {
    public:
        Nefarious()  { /* 예외를 던질 수 있는 코드 */ }   // ok
        ~Nefarious() { /* 예외를 던질 수 있는 코드 */ }   // BAD, 예외를 던져서는 안된다
        // ...
    };
```

1. `Nefarious` 객체는 지역 변수로도 안전하게 사용하기 어렵다:

```c++
        void test(string& s)
        {
            Nefarious n;          // 점점 더 심각해지는 문제(trouble brewing)
            string copy = s;      // 문자열 복사
        } // copy 파과한 다음 n
```

여기, `s`를 복사하면 예외가 발생하고, `n`의 소멸자도 예외가 발생하면 두 예외가 동시에 전파될 수 없으므로 `std::terminate`를 통해 프로그램이 종료된다.  

2. `Nefarious` 멤버나 기반으로 한 클래스 역시 소멸자가 반드시 `Nefarious`의 소멸자를 호출해야 하며, 마찬가지로 동작이 좋지 않아 안전하게 사용하기 어렵다:  

```c++
    class Innocent_bystander {
        Nefarious member;     // 이런, 클래스의 소멸자를 둘러싼 독과 같습니다
        // ...
    };

    void test(string& s)
    {
        Innocent_bystander i;   // 훨씬 더 심각해지는 문제(more trouble brewing)
        string copy2 = s;       // 문자열 복사
    } // copy 파괴한 다음 i
```

여기서 `copy`를 생성하면 `i`의 소멸자도 던질 수 있기 때문에 같은 문제가 발생하고, 그렇다면 `std::terminate`를 호출하게 된다.  

3. 전역 또는 정적 `Nefarious` 객체도 안정적으로 생성할 수 없다:

```c++
    static Nefarious n;       // 이런, 어떤 소멸자 예외도 잡을 수 없습니다
```

4. `Nefarious` 배열을 안정적으로 생성할 수 없다:

```c++
    void test()
    {
        std::array<Nefarious, 10> arr; // 이 줄에서 std::terminate(!)가 가능합니다
    }
```

배열의 동작은 소멸자가 존재할 때 정의되지 않는데, 그 이유는 고안할 수 있는 합리적인 롤백 동작이 없기 때문이다. 생각해보자: 4번째 객체의 생성자가 예외를 던지면 코드가 포기하고 정리 모드에서 이미 생성된 객체의 소멸자를 호출하려고 시도하고 그 소멸자 중 하나 이상이 예외를 던지는 `arr`를 생성하기 위해 컴파일러가 생성할 수 있는 코드는 무엇인가? 여기에 만족할만한 정답은 없다.

5. 표준 컨테이너에서는 `Nefarious` 객체를 사용할 수 없다:

```c++
    std::vector<Nefarious> vec(10);   // 이 줄에서 std::terminate()가 가능합니다
```

표준 라이브러리에서는 함께 사용되는 모든 소멸자가 예외를 던지는 것을 금지하고 있다. 표준 컨테이너에 `Nefarious` 객체를 저장하거나 표준 라이브러리의 다른 부분과 함께 사용할 수 없다.

##### Note

이는 트랜잭션 프로그래밍에서 처리 중 문제가 발생하면 작업을 철회하고 문제가 발생하지 않으면 작업을 커밋하는 두 가지 주요 작업에 필요하기 때문에 실패해서는 안 되는 핵심 기능이다. 실패없는 연산을 이용하여 안전하게 작업을 철회할 수 있는 방법이 없다면 무장애 롤백(no-fail rollback)을 구현할 수 없다. 장애없는 작업을 사용하여 상태 변경을 안전하게 커밋할 방법이 없는 경우(특히, `swap`을 포함하되 이에 국한되지 않음) 장애없는 커밋을 구현할 수 없다.

C++ 표준에 나와 있는 다음 조언과 요구 사항을 고려하라:

> 스택 해제 중에 호출된 소멸자가 예외와 함께 종료되면 terminate가 호출된다 (15.5.1). 따라서 소멸자는 일반적으로 예외를 포착하고 예외가 소멸자 밖으로 전파되지 않도록 해야한다. --[\[C++03\]](#Cplusplus03) §15.2(3)
>
> No destructor operation defined in the C++ Standard Library (including the destructor of any type that is used to instantiate a standard-library template) will throw an exception. --[\[C++03\]](#Cplusplus03) §17.4.4.8(3)

Deallocation functions, including specifically overloaded `operator delete` and `operator delete[]`, fall into the same category, because they too are used during cleanup in general, and during exception handling in particular, to back out of partial work that needs to be undone.
Besides destructors and deallocation functions, common error-safety techniques rely also on `swap` operations never failing -- in this case, not because they are used to implement a guaranteed rollback, but because they are used to implement a guaranteed commit. For example, here is an idiomatic implementation of `operator=` for a type `T` that performs copy construction followed by a call to a no-fail `swap`:

```c++
    T& T::operator=(const T& other)
    {
        auto temp = other;
        swap(temp);
        return *this;
    }
```

(See also Item 56. ???)

Fortunately, when releasing a resource, the scope for failure is definitely smaller. If using exceptions as the error reporting mechanism, make sure such functions handle all exceptions and other errors that their internal processing might generate. (For exceptions, simply wrap everything sensitive that your destructor does in a `try/catch(...)` block.) This is particularly important because a destructor might be called in a crisis situation, such as failure to allocate a system resource (e.g., memory, files, locks, ports, windows, or other system objects).

When using exceptions as your error handling mechanism, always document this behavior by declaring these functions `noexcept`. (See Item 75.)

**References**: [\[C++CS\]](#CplusplusCS) Item 51; [\[C++03\]](#Cplusplus03) §15.2(3), §17.4.4.8(3), [\[Meyers96\]](#Meyers96) §11, [\[Stroustrup00\]](#Stroustrup00) §14.4.7, §E.2-4, [\[Sutter00\]](#Sutter00) §8, §16, [\[Sutter02\]](#Sutter02) §18-19

## <a name="Sd-consistent"></a>Define Copy, move, and destroy consistently

##### Reason

 ???

##### Note

If you define a copy constructor, you must also define a copy assignment operator.

##### Note

If you define a move constructor, you must also define a move assignment operator.

##### Example

```c++
    class X {
        // ...
    public:
        X(const X&) { /* stuff */ }

        // BAD: failed to also define a copy assignment operator

        X(x&&) noexcept { /* stuff */ }

        // BAD: failed to also define a move assignment operator
    };

    X x1;
    X x2 = x1; // ok
    x2 = x1;   // pitfall: either fails to compile, or does something suspicious
```

If you define a destructor, you should not use the compiler-generated copy or move operation; you probably need to define or suppress copy and/or move.

```c++
    class X {
        HANDLE hnd;
        // ...
    public:
        ~X() { /* custom stuff, such as closing hnd */ }
        // suspicious: no mention of copying or moving -- what happens to hnd?
    };

    X x1;
    X x2 = x1; // pitfall: either fails to compile, or does something suspicious
    x2 = x1;   // pitfall: either fails to compile, or does something suspicious
```

If you define copying, and any base or member has a type that defines a move operation, you should also define a move operation.

```c++
    class X {
        string s; // defines more efficient move operations
        // ... other data members ...
    public:
        X(const X&) { /* stuff */ }
        X& operator=(const X&) { /* stuff */ }

        // BAD: failed to also define a move construction and move assignment
        // (why wasn't the custom "stuff" repeated here?)
    };

    X test()
    {
        X local;
        // ...
        return local;  // pitfall: will be inefficient and/or do the wrong thing
    }
```

If you define any of the copy constructor, copy assignment operator, or destructor, you probably should define the others.

##### Note

If you need to define any of these five functions, it means you need it to do more than its default behavior -- and the five are asymmetrically interrelated. Here's how:

* If you write/disable either of the copy constructor or the copy assignment operator, you probably need to do the same for the other: If one does "special" work, probably so should the other because the two functions should have similar effects. (See Item 53, which expands on this point in isolation.)
* If you explicitly write the copying functions, you probably need to write the destructor: If the "special" work in the copy constructor is to allocate or duplicate some resource (e.g., memory, file, socket), you need to deallocate it in the destructor.
* If you explicitly write the destructor, you probably need to explicitly write or disable copying: If you have to write a non-trivial destructor, it's often because you need to manually release a resource that the object held. If so, it is likely that those resources require careful duplication, and then you need to pay attention to the way objects are copied and assigned, or disable copying completely.

In many cases, holding properly encapsulated resources using RAII "owning" objects can eliminate the need to write these operations yourself. (See Item 13.)

Prefer compiler-generated (including `=default`) special members; only these can be classified as "trivial", and at least one major standard library vendor heavily optimizes for classes having trivial special members. This is likely to become common practice.

**Exceptions**: When any of the special functions are declared only to make them nonpublic or virtual, but without special semantics, it doesn't imply that the others are needed.
In rare cases, classes that have members of strange types (such as reference members) are an exception because they have peculiar copy semantics.
In a class holding a reference, you likely need to write the copy constructor and the assignment operator, but the default destructor already does the right thing. (Note that using a reference member is almost always wrong.)

**References**: [\[C++CS\]](#CplusplusCS) Item 52; [\[Cline99\]](#Cline99) §30.01-14, [\[Koenig97\]](#Koenig97) §4, [\[Stroustrup00\]](#Stroustrup00) §5.5, §10.4, [\[SuttHysl04b\]](#SuttHysl04b)

Resource management rule summary:

* [Provide strong resource safety; that is, never leak anything that you think of as a resource](#Cr-safety)
* [Never throw while holding a resource not owned by a handle](#Cr-never)
* [A "raw" pointer or reference is never a resource handle](#Cr-raw)
* [Never let a pointer outlive the object it points to](#Cr-outlive)
* [Use templates to express containers (and other resource handles)](#Cr-templates)
* [Return containers by value (relying on move or copy elision for efficiency)](#Cr-value-return)
* [If a class is a resource handle, it needs a constructor, a destructor, and copy and/or move operations](#Cr-handle)
* [If a class is a container, give it an initializer-list constructor](#Cr-list)

### <a name="Cr-safety"></a>Discussion: Provide strong resource safety; that is, never leak anything that you think of as a resource

##### Reason

Prevent leaks. Leaks can lead to performance degradation, mysterious error, system crashes, and security violations.

**Alternative Formulation**:  
Have every resource represented as an object of some class managing its lifetime.

##### Example

```c++
    template<class T>
    class Vector {
    // ...
    private:
        T* elem;   // sz elements on the free store, owned by the class object
        int sz;
    };
```

This class is a resource handle. It manages the lifetime of the `T`s. To do so, `Vector` must define or delete [the set of special operations](???) (constructors, a destructor, etc.).

##### Example

    ??? "odd" non-memory resource ???

##### Enforcement

The basic technique for preventing leaks is to have every resource owned by a resource handle with a suitable destructor. A checker can find "naked `new`s". Given a list of C-style allocation functions (e.g., `fopen()`), a checker can also find uses that are not managed by a resource handle. In general, "naked pointers" can be viewed with suspicion, flagged, and/or analyzed. A complete list of resources cannot be generated without human input (the definition of "a resource" is necessarily too general), but a tool can be "parameterized" with a resource list.

### <a name="Cr-never"></a>Discussion: Never return or throw while holding a resource not owned by a handle

##### Reason

That would be a leak.

##### Example

```c++
    void f(int i)
    {
        FILE* f = fopen("a file", "r");
        ifstream is { "another file" };
        // ...
        if (i == 0) return;
        // ...
        fclose(f);
    }
```

If `i == 0` the file handle for `a file` is leaked. On the other hand, the `ifstream` for `another file` will correctly close its file (upon destruction). If you must use an explicit pointer, rather than a resource handle with specific semantics, use a `unique_ptr` or a `shared_ptr` with a custom deleter:

```c++
    void f(int i)
    {
        unique_ptr<FILE, int(*)(FILE*)> f(fopen("a file", "r"), fclose);
        // ...
        if (i == 0) return;
        // ...
    }
```

Better:

```c++
    void f(int i)
    {
        ifstream input {"a file"};
        // ...
        if (i == 0) return;
        // ...
    }
```

##### Enforcement

A checker must consider all "naked pointers" suspicious.
A checker probably must rely on a human-provided list of resources.
For starters, we know about the standard-library containers, `string`, and smart pointers.
The use of `span` and `string_span` should help a lot (they are not resource handles).

### <a name="Cr-raw"></a>Discussion: A "raw" pointer or reference is never a resource handle

##### Reason

To be able to distinguish owners from views.

##### Note

This is independent of how you "spell" pointer: `T*`, `T&`, `Ptr<T>` and `Range<T>` are not owners.

### <a name="Cr-outlive"></a>Discussion: Never let a pointer outlive the object it points to

##### Reason

To avoid extremely hard-to-find errors. Dereferencing such a pointer is undefined behavior and could lead to violations of the type system.

##### Example

```c++
    string* bad()   // really bad
    {
        vector<string> v = { "This", "will", "cause", "trouble", "!" };
        // leaking a pointer into a destroyed member of a destroyed object (v)
        return &v[0];
    }

    void use()
    {
        string* p = bad();
        vector<int> xx = {7, 8, 9};
        // undefined behavior: x may not be the string "This"
        string x = *p;
        // undefined behavior: we don't know what (if anything) is allocated a location p
        *p = "Evil!";
    }
```

The `string`s of `v` are destroyed upon exit from `bad()` and so is `v` itself. The returned pointer points to unallocated memory on the free store. This memory (pointed into by `p`) may have been reallocated by the time `*p` is executed. There may be no `string` to read and a write through `p` could easily corrupt objects of unrelated types.

##### Enforcement

Most compilers already warn about simple cases and has the information to do more. Consider any pointer returned from a function suspect. Use containers, resource handles, and views (e.g., `span` known not to be resource handles) to lower the number of cases to be examined. For starters, consider every class with a destructor as resource handle.

### <a name="Cr-templates"></a>Discussion: Use templates to express containers (and other resource handles)

##### Reason

To provide statically type-safe manipulation of elements.

##### Example

```c++
    template<typename T> class Vector {
        // ...
        T* elem;   // point to sz elements of type T
        int sz;
    };
```

### <a name="Cr-value-return"></a>Discussion: Return containers by value (relying on move or copy elision for efficiency)

##### Reason

To simplify code and eliminate a need for explicit memory management. To bring an object into a surrounding scope, thereby extending its lifetime.

**See also**: [F.20, the general item about "out" output values](#Rf-out)

##### Example

```c++
    vector<int> get_large_vector()
    {
        return ...;
    }

    auto v = get_large_vector(); //  return by value is ok, most modern compilers will do copy elision
```

##### Exception

See the Exceptions in [F.20](#Rf-out).

##### Enforcement

Check for pointers and references returned from functions and see if they are assigned to resource handles (e.g., to a `unique_ptr`).

### <a name="Cr-handle"></a>Discussion: If a class is a resource handle, it needs a constructor, a destructor, and copy and/or move operations

##### Reason

To provide complete control of the lifetime of the resource. To provide a coherent set of operations on the resource.

##### Example

    ??? Messing with pointers

##### Note

If all members are resource handles, rely on the default special operations where possible.

```c++
    template<typename T> struct Named {
        string name;
        T value;
    };
```

Now `Named` has a default constructor, a destructor, and efficient copy and move operations, provided `T` has.

##### Enforcement

In general, a tool cannot know if a class is a resource handle. However, if a class has some of [the default operations](#SS-ctor), it should have all, and if a class has a member that is a resource handle, it should be considered as resource handle.

### <a name="Cr-list"></a>Discussion: If a class is a container, give it an initializer-list constructor

##### Reason

It is common to need an initial set of elements.

##### Example

```c++
    template<typename T> class Vector {
    public:
        Vector(std::initializer_list<T>);
        // ...
    };

    Vector<string> vs { "Nygaard", "Ritchie" };
```

##### Enforcement

When is a class a container? ???
