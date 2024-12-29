# 1. Composable함수들(Composable functions)

## Composable 함수의 의미 (The meaning of Composable functions)
- Composable 함수 하나는 트리의 하나의 노드이다
- @Composable () -> Unit 는 일반적인 함수 리턴이 아니라 트리에 노드를 하나 삽입하는 것이다(이를 emit 이라고 표현함)
- Composable 함수를 실행하는 것은 메모리에서 그려주기위해 트리에서 해당 노드를 최신 상태로 업뎃 하는것이다

## Composable 함수의 속성 (Properties of Composable functions)
- 함수에 @Composable 를 붙이면 추가적인 속성이나 코드들들 붙여준다. 누가? compose compiler 가
- Compose Runtime
  - composable 함수로 트리를 만든다
  - 상태관리를 하면서 변경이 일어나면 recomposition 을 수행한다
  - 위 작업을 위해 여러가지 최적화 기법을 사용한다(우선순위에 따른 임의의 Composition 정렬, 스마트 recomposition, 또는 위치 기억법(positional memoization)
  - 최적화 기법은 코드에 '확실성'(certainties) 이 있어야 가능하다.

### 호출 컨텍스트 (Calling context) : composer
- Composable 함수의 속성은 대부분 Compose Compiler에 의해 활성화됩니다.(만들어 진다)
- 모든 Composable 함수는 Intermediate Representation 단계에서 마지막 파라매터에 composer 가 붙는다
- composer 를 통해 트리를 구성, 변경한다.

### 멱등성 (Idempotent)
- Composable 함수는 생성하는 노드 트리에 대해 멱등성을 가져야 합니다
- 입력값이 변경 됬을때만 Composable 함수를 다시 호출함 - recomposition

#### 통제가 안되는 사이드 이펙트를 없애도록 -> 통제가 되는 사이드 이펙트는 가능
- Composable 함수는 Compose Compiler에 의해 어떤 순서로든, 심지어 병렬로도 실행될 수 있습니다 즉 Composable 함수들이 순서대로 호출되야 되는 로직을 구성하면 안됨
- 이상적으로 Composable 함수 내에서는 사이드 이펙트나 상태 보존이 없도록 만들어야함
- 사이드이펙트를 통제하기 위해 effect handler 를 제공한다

### 재시작 가능 (Restartable)
- 입력값이 바뀌면 또 호출함

### 빠른 실행 (Fast execution)
- 여러번 호출될수 있으므로 Composable 함수는 빨리 실행하고 끝낼수 있도록 만들어야한다
- 오래 걸리는 작업은 코루틴으로 처리

### 위치 기억법 (Positional memoization)
- Composable 함수는 호출 위치가 바뀌면(이전에 호출된 Composable 함수가 없어지거나 뭔가 새로생기거나) recomposition 한다
(예를들어 list 를 이용해 loop 문내에서 Composable 을 호출할때 list 중간에 삽입/삭제가 되는 경우)
  - 이때 key(id){} 를 사용해서 고유값과 {} 의 Composable 함수를 기억하게 만들수 있고 이를 통해 recomposition 을 방지 함
  - list loop 일때만 해당함. 일반적인 if 로 Composable 함수 호출 여부(visible 처리)를 결정하는 경우 이후에 호출되는 Composable 함수는 key(id) {} 를 사용해도 recomposition 됨
- 시간이 좀 오래걸리는 작업이면서 recomposition 이 일어 났을때 해당 작업을 다시 할필요가 없는 경우 remember 를 사용한다

### Composable 함수의 색깔 (The color of Composable functions)
- Composable 함수와 일반함수는 이를 가지고 해결하려는 문제점, 사용 목적, 방법이 다르다.
  - 이를 함수의 색깔이 다르다 라고 표한함
  - 동기 - 비동기 함수와 비슷함(일반 함수 - suspend 함수)
- 이 각기 다른 2가지 종류의 함수는 서로 잘 섞이지는 않지만 그래도 연결 지점(다른말로 integration point)이 필요하다.
  - 코루틴에서는 CoroutineScope.launch 가 해당(일반 함수 에서 suspend 호출)
  - Composable 에서는 Composition.setContent 가 해당(일반함수 에서 Composable 호출)
- ref: https://journal.stuffwithstuff.com/2015/02/01/what-color-is-your-function/
- 일반함수에는 없는 Composable 함수 고유의 기능: restartable, skippable, reactive

### Composable 함수 타입 (Composable function types)
- 함수이던 람다이던 @Composable 어노테이션을 붙이면 Composable 함수가 됩

# 2. Compose 컴파일러 (The Compose compiler)
- compose runtime, compiler 가 컴포즈 아키텍쳐이고 compose UI 는 이를 활용하는 클라이언트임
- source - compiler - runtime - ui 요런 구조

## Compose 컴파일러 (The Compose compiler)
- compose compiler 는 kotlin 의 컴파일러 플러그인(APT 랑은 다름). APT 는 컴파일 전에 실행하지만 컴파일러 플러그인은 컴파일 과정에 동작
- APT 는 코드에 새로운 코드를 추가하기만하는데 컴파일러 플러그인은 맘대로 수정(표현식을 변경한다거나)이 가능함
  - APT 는 그외에도 정적분석, 이를 통한 검증이 가능함
- KSP 도 컴파일러 플러그인으로 만들어짐

## Compose 어노테이션들 (Compose annotations)
- 모든 compose 어노테이션은 compose runtime 에서 제공한다.

### @Composable
- compose compiler 가 부여하는 능력 또는 속성
  - remember 호출가능, composer, 슬롯테이블 활용
  - effect 들의 라이프사이클 제공
  - recomposition 등을 위한 고유 id 를 받음
  - 트리내의 위치를 지정당함
  - CompositionLocals 사용
- 최종 트리에 적재되는 노드로서 emit 함
- Composable 함수는 UI 노드 일수도 있지만 runtime 을 사용하는 라이브러리(위에서 말하는 클라이언트)에 따라 UI 가 아닐수도 있음

- 슬롯테이블?: 컴포지션 데이터가 저장되는 자료구조(컨테이너) gap buffer 와 비슷. 밑에 챕터에 설명 나옴


### @ComposableCompilerApi
- compiler 에 의해서만 사용되는거임. 

### @InternalComposeApi
- stable 이지만 변경될 여지가 있음. kotlin internal 키워드 한정자 보다 더 넓은 범위(어노테이션을 이런식으로 활용?)

### @DisallowComposableCalls
- Composable 함수를 호출 못하도록 강제함
- 주로 인라인 람다에서 호출이 안되도록 제약을 걸때 사용(ex: remember) - 일반 인라인 람다는 당연하게도 Composable 함수 호출 가능함
- 사용할일은 별로 없지만, @DisallowComposableCalls 선언된 람다 내부에서 다른 인라인 람다를 사용한다면 동일하게 선언해서 제약을 걸어줘야됨

### @ReadOnlyComposable
- composition 을 할때 인메모리에 저장 및 수정하기위한 슬롯 생성하는 과정을 안하도록
  - compose compiler 는 Composable 함수를 그룹화 하여 runtime 에 emit 함
  - restartable group, movable group 등이 있음
  - @ReadOnlyComposable 를 사용하면 위에 짓을 안함
- 중첩된 호출에도 동일하게 적용해야 함
- CompositionLocal 이 그 예(.current)

### @NonRestartableComposable
- 재시작 불가능한 Composable 함수로 만들어 버림
- recomposition 에 필요한 추가 코드를 생성하지 않음
- 새로운사실: 인라인 된 Composable 함수나 리턴 타입이 Unit이 아닌 Composable 함수는 재시작할 수 없다.
- 내부에서 그냥 바로 다른 Composable 함수를 호출하는 정도의 간단한 상황에서 사용

### @StableMarker
- 타입 안정성을 제공하기 위한 어노테이션에 사용(@Immutable, @Stable)
- 타입이 안정적(stable) 하다는 의미?
  - equals 함수를 호출했을때 동일한 인스턴스는 항상 동일해야한다
  - public 프로퍼티가 변경되면 composition 에 알린다
  - public 프로퍼티 또한 안정적(stable) 해야한다
  - 추가적으로 primitive type, 람다, 문자열도 stable 로 취급
- 컴파일러에 제공하는 약속. 컴파일단계에서 이 요건에 대한 유효성 검사는 안한다
- 대부분의 경우 컴파일러가 알아서 타입 안정성(stable) 을 추론하지만 아래 2가지 못하는 경우가 있다
  - 대상이 interface 또는 abstract 인경우
  - 구현체가 mutable 이지만 개발자가 안정적인 타입으로 처리하고 싶은경우

### @Immutable
- 인스턴스화가 되면 그이후로는 값이 절대 변경되지 않는거에 사용가능
- 모든 class 멤버가 val 사용 (그리고 custom getter 사용 안하는거)

### @Stable
- @Immutable 보다는 좀더 약한 느낌
- class 멤버중에 private property 가 가변인경우 or 가변 상태를 딜리게이트해서 겉보기에만 불변인경우(custom getter 라던가)

- @Stable, @Immutable, primitive type 으로 된 Composable 함수는 이전호출과 인풋값이 동일하면 recomposition 을 생략(skip) 함(이거시 스마트 recomposition)
- 위 조건에 안맞는데 대충쓰면 런타임 오류 발생함

## 컴파일러 확장 등록 (Registering Compiler extensions)
- Compose Compiler 플러그인의 작동 방식과 해당 어노테이션을 사용하는 방법
- Kotlin 컴파일러가 제공하는 메커니즘인 ComponentRegistrar를 사용하여 Kotlin 컴파일러 파이프라인에 자신을 등록
  - ComposeComponentRegistrar는 다양한 목적을 위해 일련의 컴파일러 익스텐션(라이브 리터럴같은)을 등록

### Kotlin 컴파일러 버전 (Kotlin Compiler version)
- Kotlin 이랑 Compose 랑 둘이 버전을 맞춰야함

### 정적 분석 (Static analysis)
- IDEA 플러그인이랑 연계하여 타이핑중에 알려줌

#### 정적 검사기 (Static Checkers)
- 위에랑 비슷, 안맞으면 IDE 로 바로 알려줌

#### 호출 검사 (Call checks)
- Composable 함수(람다 포함) 가 호출하는 상황이 적절한지 검증
- @DisallowComposableCalls, @ReadOnlyComposable 가 붙어있는거

#### 타입 검사 (Type checks)
- @Composable 가 적절하게 붙어 있는지

#### 선언 검사 (Declaration checks)
- 이런것들 검사: @Composable 이랑 suspend 랑 같이 사용 못함, main 함수에 @Composable 사용 못함 backing 필드에 사용 못함

#### 진단 제지기 (Diagnostic suppression)
- 일부 언어적인 제한사항을 우회하기 위해 진단을 off 함
- 사례1 인라인 함수의 람다 파라매터에 @Composable 어노테이션을 호출자쪽에서 붙일수있게 -> kotlin 2.0 에서 에러
- 사례2 @Composable 람다에 named arguement 가 사용가능 -> kotlin 2.0 에서 에러

### 런타임 버전 검사 (Runtime version check)
- compose runtime, compiler, kotlin 의 버전이 맞는지 확인

### 코드 생성 (Code generation)
- 이제 코드 생성의 단계
- 아니.. 이게 지금 컴포즈 컴파일 동작 순서대로 하는 거였어? 왜 그걸 인제 설명을 하는겨

### 코틀린 IR (The Kotlin IR)
- IR 을 통해서 composer 를 껴넣고, 플랫폼 의존성이 없도록 만들수 있음
- 코틀린 컴파일러 플러그인 만들기(Writing Your Second Kotlin Compiler Plugin) 링크: https://blog.bnorm.dev/writing-your-second-compiler-plugin-part-1

### 낮추기 (Lowering)
- 고급 프로그래밍에서 좀 더 로우하게 동작하도록 변경하는 과정을 표현함
- 정규화의 한 형태

#### 클래스 안정성 추론 (Inferring class stability)
- Composable 함수의 입력값이 stable 한지 추론함
- 뭐가 stable 한 타입인지는 위에 참고
- 모든 클래스를 대상으로 안정성 추론 검사를 하지 않는다. public class, data class 외에 것들은 안함
  - interface, -enum-, annotaiton, 익명 객체, inline, compainon, expect(멀티플랫폼용 기능) 
  - 그럼 private class 는??
- 제네릭은 클래스 매개변수가 stable 하냐에 따라 안정성이 결정됨. 근데 런타임때 이게 결정됨으로 compose 에서 이를 위해 별도 처리함
  - 모든 제네릭이 그런건 아님
- 클래스에 private var 가 있어도 stable 하지 않다고 간주함
- val 에 get() 쓰는거도 일단 불안정으로 추론: @Stable 붙여줘야함
- 상황에 따라 stable 한지 확인가능한 unit test 코드 링크: https://cs.android.com/androidx/platform/frameworks/support/+/androidx-main:compose/compiler/compiler-hosted/integration-tests/src/jvmTest/kotlin/androidx/compose/compiler/plugins/kotlin/ClassStabilityTransformTests.kt

### 라이브 리터럴 활성화 (Enabling live literals)
- IDE 를 이용해 타이핑중에 알려줄수 있는거

### Compose 람다식 기억법 (Compose lambda memoization)
- Composable 함수에 전달된 람다식의 실행을 최적화하는 방법
- 슬롯테이블에 저장했다가 재호출시 read 해서 사용

#### Composable이 아닌 람다식(Non‐composable lambdas)
- 람다가 다른 값을 캡쳐하는게 없으면 기본그대로 사용
- 다른 값을 캡쳐하면 remember<람다의 티런타입>(캡쳐값1, 캡쳐값2, ..., 람다) 요렇게 내부적으로 저장함

#### Composable 람다식 (Composable lambdas)
- composableLambda($composer ..) 를 사용
- Composable 람다식 내부 시작과 끝지점에 replaceable group 을 추가한다
  - 그래서 도넛홀 스킵핑 가능
  - 도넛 홀 생략하기(donut‐hole skipping) 참고: https://medium.com/sungbinland/optimizing-recomposition-in-jetpack-compose-donut-hole-skipping-6baf22f059bb

### Composer 주입하기 (Injecting the Composer)
- Compose Compiler가 모든 Composable 함수에 Composer 라는 합성 매개변수를 함수 맨뒤에다가 추가한다.

### 비교 전파 (Comparison propagation)
- composer 외에도 추가되는 매개변수
- Composable 함수의 입력값들이 변경됫는지 여부의 검사를 할건지 말건지를 비트 단위로 저장하기 위해 사용함
  - 입력 파라매터 갯수가 많으면 몇개 더 붙는듯
- $changed: Int, 비트마스크로서 사용함

### 디폴트 매개변수 (Default parameters)
- composer 외에도 추가되는 매개변수
- Composable 함수는 kotlin 의 디폴트 매개변수 기능을 사용못함. 이를 해결하기 위해 디폴트값을 가진 매개변수 하나마다 $default 를 추가함
- 비트마스킹을 통해 전달된 값을 쓸건지 디폴트값을 쓸건지 결정

### 컨트롤 플로우 그룹 생성 (Control flow group generation)
- 로직 컨트롤에 따라 Composable 함수에 그룹을 생성한다
- Composable 함수의 본문(시작, 끝)에 아래 3가지 그룹중 하나를 삽입해서 런타임에 호출되어 생성함
```
fun ComposableFunc($composer) {
  $composer.startReplaceableGroup(key)  // 추가됨
  // 함수 본문 내용
  $composer.endReplaceableGroup(key)  // 추가됨
}
```
- 그룹은 Composable 함수의 호출 위치에 대한 정보를 가지고 있다.

#### 교체 가능한 그룹 (Replaceable groups)
- if (condition) A() else B() 같이 조건에 따라 호출 위치가 바뀔수 있는것들
- Composable 람다, @NonRestartableComposable 붙은거

#### 이동 가능한 그룹 (Movable groups)
- 루프문 for(...) { key() { A() } } 같이 고유한 정체성이 보장되고 이동 가능한 그룹
- 정체성을 해치지 않으면서 호출 순서를 변경할 수 있도록 한다.

#### 재시작 가능한 그룹 (Restartable groups)
- 재시작 가능한 그룹은 상태(state)를 읽는 모든 Composable 함수에 대해 생성되는 유형의 그룹
  - 의존성(입력 파라매터, remember 등) 이 변경되면 recomposition
  - 거의 restartable 이고 그게 아닌거는 위에 두가지랑 readonly
  - skippable 은 restartable 안에 포함됨


- 블럭이 정확히 1번만 실행하면 그룹이 필요없다(파라매터가 아예없는거)
- 조건부 로직은 replaceable
- 루프문 + key() 는 Movable

### Klib과 미끼 생성 (Klib and decoy generation)
- klib(멀티플랫폼) 및 Kotlin/JS 지원을 추가적으로 동작 하는것들이 있다

# 3. 컴포즈 런타임 (The Compose runtime)

## 슬롯 테이블 및 변경 목록 (The slot table and the list of changes)
- 리니어한 자료구조
- 최초 composition 에서 데이터(호출 위치, 매개변수, 기억해야될거, CompositionalLocal 등)가 저장되고 recomposition 마다 업뎃 됨
- 슬롯테이블은 상태를 기록하고, 변경 점들을 적용하는것은 Applier가 한다
- recomposer 는 언제 어떤 스레드에서 변경사항을 적용할지 결정한다

### 슬롯 테이블 심층 분석 (The slot table in depth)
- 갭 버퍼(텍스트 에디터에서 주로 사용하는 동적 배열 자료구조. 커서-같은위치 기준으로 삽입/삭제가 자주 일어나는거에 효율 적임) 기반
- 그룹, 슬롯 2가지 배열에 저장
  - 그룹 배열: 그룹 필드를 저장(Int). 부모-자식-그자식 요런구조, 선형탐색을 위해 앵커라고 일종의 포인터 개념의 접근방식을 사용
  - 슬롯 배열: 해당 그룹과 관련된 데이터 저장(Any?). 
- 슬록 테이블 리더는 여러개일수 잇으나 라이터는 한개임. 서로 동기화 안정성 보장됨
  - 리더: 그룹과 슬롯 배열의 데이터 접근
  - 라이터: 갭(이동이 가능하고 크기 조절이 가능한 범위성 포인터)에 의존하여 시작,끝,갭길이를 추적하고 어딜에 쓸건지 결정. 
    - 그룹, 슬롯등을 이동가능해서 리더랑 같은 동작이 가능함.
    - 빠른 접근을 위해 앵커 목록도 추적함
- 참고: https://medium.com/androiddevelopers/under-the-hood-of-jetpack-compose-part-2-of-2-37b2c20c6cdd

### 변경 목록 (The list of changes)
- emit: 슬롯테이블을 업뎃. 최종적으로 변경할 사항들을 만듬(그리고 나중에 lazy 처리)
- 즉 emit 할때마다 변경할 목록을 만듬
- Recomposer 는 어떤스레드에서 변경사항을 적용할지 결정
  - 사이드 이펙트, LauchedEffect 랑 변경사항 적용이랑 같은 스레드를 쓸때도 있음

## Composer

### Composer 키우기 (Feeding the Composer)
- Layout 을 살펴보면 내부에 ReusableComposeNode 를 통해 emit 하고, ReusableComposeNode 은 composer 에 위임하는 부분이 많다
```
Layout(...) {
  ReusableComposeNode(...) {
    currentComposer.startReusableNode()
    currentComposer.createNode(factory)
    // ...
    Updater<T>(currentComposer).update() // initialization
    // ...
    currentComposer.startReplaceableGroup(0x7ab4aae9)
    content()
    currentComposer.endReplaceableGroup()
    currentComposer.endNode()
  }

}
```
- remember 도 전달 받은 값을 composer 를 통해 캐싱한다.
```
remember(...) {
  currentComposer.cache(...)
}
```

### 변경 사항 모델링 (Modeling the Changes)
- currentComposer 에 위임된 작업들은 Change( (Applier<*>, SlotWriter, RememberManager) ‐> Unit) 를 통해 내부의 변경 목록에 추가된다.
   - 위 타입은 지금은 다른거(ChangeList) 로 변경됨
- 일련의 Composable 함수들이 다 호출되고 모든 변경 사항이 만들어지면(즉 emit 후에) 이후에 Applier 에 의해 일괄 적용됨: 즉 변경사항이 만들어 지자 마자 바로 슬롯테이블에 반영이 되는게 아님

### 작성 시기 최적화 (Optimizing when to write)
- composition 이 완료되면 composition.applyChanges() 을 호출 슬롯테이블에 변경사항을 기록함
  - 모든 항목은 그룹 형태로 저장됨
- composer 가 그룹 동작을 시작할때 상황에 따라 보류중인 작업 존재 여부, 재사용 가능여부등에 따라 다르게 동작한다.

### 값 기억하기 (Remembering values)
- composition 때 remember 를 통한 값변경됫는지는 바로 확인하지만 composer 가 삽입과정에 있는 상태가 아니면 Change 로 기록해둔다.
- RememberObserver 인경우에도 상황에 따라 Change 로 기록(https://developer.android.com/reference/kotlin/androidx/compose/runtime/RememberObserver)

### 재구성 범위 (Recompose scopes)
- restartable group 이 생성될때마다 RecomposeScope 를 만들고 currentRecomposeScope로 설정한다
- 재구성이 요청되면(invalidate 호출) composer 는 재구성을 위해 슬롯테이블을 해당 그룹 시작위치에 배치하고 재구성 블럭을 호출함
- composer 는 재구성할 것들(보류중이며 다음 recomposition)을 스택으로 유지
- Composable 함수 내 상태 스냅샷에서 읽기 관련 작업이 있을때만 RecomposeScope 이 동작함

### Composer와 사이드 이펙트 (SideEffects in the Composer)
- composer 가 side effect 로 기록해 두엇다가 composition 이후(변경사항 적용완료 시점)에 실행한다.
- side effect 는 슬롯테이블에 저장되지 않는다

### CompositionLocals 저장 (Storing CompositionLocals)
- 역시 슬롯테이블에 그룹 형태로 저장됨

### 소스 정보 저장 (Storing source information)
- Compose의 툴에서 활용할만 정보들 저장

### CompositionContext를 이용한 Composition 연결 (Linking Compositions via CompositionContext)
- CompositionContext 은 composition 과 subcomposition 를 연결하기 위한 용도
- subcomposition 은 단독의 invalidation 을 가지면서 상위 composition 과 트리관계를 가지도록함
- AndroidView 같은거

### 현재 상태 스냅샷에 접근 (Accessing the current State snapshot)
- composer 는 현재 스냅샷의 참조를 가지고 있음

### 노드 탐색 (Navigating the nodes)
- 노드 탐색 동작은 Applier 가 하지만 어떤걸 탐색할지 읽기/쓰기는 composer 가 하나봄
- 배열에 저장

### 구독자와 작성자의 동기화 유지 (Keeping reader and writer in sync)
- reader 랑 writer 변경 사항이 적용될 때까지 일시적으로 차이가 있을수 있음
- 이 둘을 같은 위치를 유지하기 위해 둘의 거리를 추적하고 맞춤

### 변경 사항 적용하기 (Applying the changes)
- Applier 가 한다 (materializing - 구체화 라고도 부름) 결과적으로 슬롯 테이블을 업뎃함(https://developer.android.com/reference/kotlin/androidx/compose/runtime/Applier)
- Applier 는 interface 고 실제 구현체는 클라에서 함
- Applier는 모든 노드를 방문하고 적용하면서 전체 트리를 순회한다

### 노드 트리 구축 시 성능 (Performance when building the node tree)
- 트리 만들때 상향, 하향에 따라 차이가 있음
- Applier의 구현체에 따라 성능차가 있을수 있다.
- 새로운 항목이 삽입됬을때 알림 받아야될 노드의 수가 많을수록 느린거임
- https://developer.android.com/reference/kotlin/androidx/compose/runtime/Applier#insertTopDown(kotlin.Int,kotlin.Any)
  - 위 링크(or 책 내용) 에 있는 방법으로는 하향식이 더 느리다. 
  - 하향: 밑에 노드가 생길때마다 그위 모든 부모에게 돌아가서 알림을 줘야함
  - 상향: 밑에 부터 시작해서 부모가 생겨나는 식이라 위에 부모 한번만 알림을 줘도됨

### 변경 사항이 적용되는 방식 (How changes are applied)
- 구현체 UiApplier 코드 참고: compose/ui/ui/src/androidMain/kotlin/androidx/compose/ui/node/UiApplier.android.kt
- Android 는 상향식만 사용함

### 노드 연결 및 그리기 (Attaching and drawing the nodes)
- 노드가 스스로 연결하고 그리기를 수행함
- 가장 위에는 Owner 가 있고(즉 트리 루트임) 하위로 LayoutNode 들이 구성됨
- Owner 는 AndroidComposeView 에 의해 구현된다

## Composition
- composer 가 Composition 의 참조를 가지고 있지만 Composition 을 composer 가 생성하는게 아님
- 클라가 compose runtime 으로 접근하는 진입점은 두가지
  - composable 함수 작성
  - setContent. 이걸 호출해야 composition 이 생성됨

### Composition 생성하기 (Creating a Composition)
- Composition 생성예
  - setContent 함수 내용에 Composition(UiApplier(owner.root), parent) 이 있음
  - rememberVectorPainter  Composition(VectorApplier(this.vector.root), compositionContext) 
  - SubcomposeLayout?? 이건 안쓰는거 같은데.?
- Composition 생성시 CompositionContext 를 사용
  - CompositionContext: Composition 을 새로 생성시 기존 CompositionContext 가 있으면 그거랑 연결하는 용도
- 마지막 파라매터에 CoroutineContext 도 전달가능, 이거는 recomposition 때 이걸로 사용(지정 안하면 메인 Ui 디스패쳐 사용)
- 다 사용하면 composition.dispose() 호출 해야됨

### 초기 Composition 과정 (The initial Composition process)
- composeInitial 를 호출함. 이거는 parent 거를 호출하면서 루트꺼를 호출
  - 현재 state 를 스냅샷으로 가져옴(스냅샷은 스레드 세이프함)
  - 스냅샷을 가져올때 읽기/쓰기에 대한 관찰자 를 전할하고 이를 통해 알림받아 recomposition 수행
  - 실제 composition 은 composeContent 에서 일어남
  - composition 절차는 composer 에게 위임함
- 그러면 위임받은 composer 가 composition 하는거는
  - 이미 composition 을 진행중이면 새로운 composition 은 폐기함
  - startRoot 을 호출해서 슬롯테이블의 루트 그룹을 시작함
  - startGroup 을 호출 
  - content 를 호출해서 모든 변경사항을 emit 
  - endGroup
  - entRoot

### 초기 Composition 후 변경 사항 적용 (Applying changes after initial Composition)
- 초기 composition 후 Applier 는 composition.applyChanges() 통해 모든 변경 사항을 적용
  - applier.onBeginChanges()
  - applier.onEndChanges()
 - RememberedObserver 를 통해 composition 이 시작될때 끝날때를 알림 받음
   - 사이드이펙트(LaunchedEffect, DisposableEffect) 들이 RememberedObserver 를 통해 구현되어 컴포지션 이후에 호출되도록 알림 받음

### Composition에 대한 추가 정보 (Additional information about the Composition)
- Composition 은 현재 보류중인 invalidation, isComposing 을 알고 있음
- 런타임을 ControlledComposition 라고 외부에서 제어가능한 몇가지 기능이 있는걸 사용
- Composition 은 본인이 관찰해야 되는 객체를 알림 받을수 있는 방법을 제공함 이를 통해 recomposition 을 수행
- Composition 중 오류가 나면 중단함

## Recomposer
- Recomposer는 ControlledComposition을 제어하고, 필요 할 때 recomposition 을 트리거

### Recomposer 생성 (Spawning the Recomposer)
- Composition을 생성할 때에는 해당 composition 에 대한 부모를 함께 제공해야 합니다. 
- 루트 composition의 부모가 Recomposer가 되어야 하는 경우, Recomposer를 생성해야 하는 상황이 생기기도 합니다.
- 클라(안드로이드의 경우 Compose UI) 에서 composition 과 그 부모가 될 Recomposer 를 생성함
- 안드로이드의 경우 ViewGroup.setContent 호출 통해 수행, 윈도우루트뷰를 전달해서 뷰의 lifecycle 을 recomposer 에 연결함
  - WindowRecomposerFactory 사용
- Recomposer 생성시 CoroutineContext 가 필요함
  - AndroidUiDispatcher + PausableMonotonicFrameClock(특적 구간동안 프레임이 생성안되도록 중단)
  - 이 컨텍스트로 Job 을 만들어 Android의 Window가 파괴되거나 연결되지 않을 때 취소시킴
  - LaunchedEffect 실행에 사용하는 컨텍스트임(메인 스레드)
- 이걸로 scope 을 만들어서 invalidation을 기다리고 그에 따라 recomposition을 트리거하는 recomposition 하는데 사용
- WindowRecomposerFactory 는 Android 라이프사이클에 따라 상황에 맞는 Recomposer 메소드를 호출함
  - Lifecycle.Event.ON_CREATE 시점에 recomposer.runRecomposeAndApplyChanges() 호출

### Recomposition 프로세스 (Recomposition process)
- recomposer.runRecomposeAndApplyChanges() 호출 하면 invalidation 을 기다리기 시작함
- 가장 먼저 하는 일은 변경사항 전파에 필요한 프로세스에 관찰자를 등록함. 이를 통해 상태변경시 recomposition 
- 모든 Composition 에 invalidation 을 수행함(모든게 변경되었다고 가정)
- 모노리틱 클럭으로 다음 프레임을 기다림
- 보류중인 모든 스냅샷을 가져와 invalidation을 수행함
- 적용할 변경 사항이 있는 모든 Composition들을 검토하고, 해당 Composition에 대해 composition.applyChanges()를 호출

### Recomposition의 동시성 (Concurrent recomposition)
- runRecomposeConcurrentlyAndApplyChanges 함수를 통해 CoroutineContext 를 받아서 동시 수행 처리가 가능

### Recomposer의 상태 (Recomposer states)
- ShutDown: Recomposer가 취소되고 정리 작업이 완료되었습니다. 더 이상 사용할 수 없습니다.
- ShuttingDown: Recomposer가 취소되었지만 여전히 정리 작업 중입니다. 더 이상 사용할 수 없습니다.
- Inactive: Recomposer 는 Composer 의 invalidation 을 무시하고, 이에따라 recomposition을 트리거하지 않습니다. Recomposition 관찰을 시작하려면 runRecomposeAndApplyChanges를 호출해야 합니다. 이는 생성 후 Recomposer의 초기 상태입니다.
- InactivePendingWork: Recomposer가 비활성 상태이지만 이미 프레임을 기다리는 보류 중인 이펙트가 있을 가능성이 있습니다. 프레임은 Recomposer가 실행을 시작하자마자 생성됩니다.
- Idle: Recomposer가 composition 및 스냅샷 invalidation을 추적하고 있지만 현재 수행할 작업이 없습니다.
- PendingWork: Recomposer가 보류 중인 작업에 대해 알림을 받았으며, 이미 작업을 수행 중 이거나 수행할 기회를 기다리고 있습니다.

# 4. Compose UI
- 클라 라이브러리로서 Compose UI 외에도 Mosaic 이라고도 있음

## Compose UI와 런타임의 통합 (Integrating UI with the Compose runtime)
- 위 두가지 목표는 레이아웃 트리를 구축하는것
- 클라이언트 노드 타입은 다르게 사용 가능함(Compose UI용, 웹용 등)
- 초기 composition -> 변경이 일어남(recomposition 필요) -> Composable 함수 호출 -> 노드 crud 하면서 변경사항 예약 -> Applier 가 실제 변경사항을 적용

### 예약된 변경 목록을 실제 트리의 변경 목록으로 매핑 (Mapping scheduled changes to actual changes to the tree)
- Composable 함수 호출시 자신의 변경사항을 emit
- 이때 side table 이란걸 사용(변경 목록 -> 실제 노드 매핑에 필요한 정보를 가지고 있다)
- composition 은 여러개 일수 있다(노드 수만큼도 가능)

### Compose UI 관점에서의 Composition (Composition from the point of view of Compose UI)
- setContent 를 호출하면 루트 composition 생성
- 안드의 경우 ComposeView 를 여러개 쓰면 그만큼 composition 도 여러개 가능
- 안드의 경우 Composition 으로 LayoutNode 를 emit
- 안드의 UI 컴포넌트들은 Layout 으로 만들어짐
  - Layout 은 ReusableComposeNode 를 사용하고 ReusableComposeNode 는 key 가 변경됬을대 새로만들도록 최적화
    - 파라매터 update 람다는 값이 변경됬을때만 호출됨
  - AndroidView 의 경우는 ReusableComposeNode 를 안쓰고 표준 ComposeNode 를 사용
- LayoutNode 외에도 다른종류의 Composition 과 노드 타입들이 있음

### Compose UI 관점에서의 Subcomposition (Subcomposition from the point of view of Compose UI)
- 루트에서 만든 composition 을 최대한 재사용하지만 이런 이유들로 별도 subcomposition 을 생성한다
  - 초기 composition 이후 특정 정보를 알아 올때까지 지연처리 하기 위해
    - 예: SubcomposeLayout, BoxWithConstraints
    - https://stackoverflow.com/questions/69663194/jetpack-compose-how-does-subcomposelayout-work/69665926#69665926
  - 하위 트리에서 만드는 노드 타입을 변경할때
    - 예: rememberVectorPainter, VectorPainter - VNode 를 emit 한다


## UI에 변경사항 반영하기 (Reflecting changes in the UI)
- 구체화(materialization)

### 다양한 타입의 Applier들 (Different types of Appliers)
- Applier 의 구현체는 하나의 노드타입만 대응하도록 되있음
  - AbstractApplier: 공통 로직(스택자료구조, up, down 같은 탐색 동작)
  - UiApplier<LayoutNode>: 대부분의 안드로이드 UI, 상향식
  - VectorApplier<VNode>: 벡터 그래픽 렌더링용, 하향식(딱히 다른노드에 알림이 필요없으므로 성능저하는 없)

### 새로운 LayoutNode를 구체화 하기(Materializing a new LayoutNode)
- UiApplier 구현들을 보면 insert/remove/move 등이 노드의 것을 호출하는 위임방식으로 구현되있음. 이를 통해 UI 노드 를 LayoutNode 로 모델링해서 플랫폼 독립성을 얻어냄(즉 여러 플랫폼에서 사용가능)
- 노드 트리의 최초는 owner 이며 이는 플랫폼마다 구현이 다르다(안드로이드는 View). 그 하위로 LayoutNode 가 연결됨
- LayoutNode.insertAt, LayoutNode.attach 설명..
  - zIndex 에 따른 정렬, 자식들 무효화등 처리
  - LayoutNodeWrapper(NodeCoordinator 로 변경됨)를 이용한 measuring 처리
  - owner 설정
  - 접근성 처리(필요하면 owner 에게 알림)

#### 전체 과정의 마무리 (Closing the circle)
- 순서대로: setContent 호출 -> AndroidComposeView 생성 -> owner 가 view 계층에 연결됨 -> 루트 LayoutNode 연결
- 위상태에서 노드가 하나 새로 만들어진다고 한다면: 부모로 재측정(onRequestMeasure 호출) -> 부모가 owner 이면 view 에서 measure, layout 이 호출됨 -> 부모 노드에서 부터 draw
  - 측정(혹은 측정 예정) 중인데 또 재측정 요청이 오면 무시함

### 노드 제거를 위한 변경 사항 구체화 (Materializing a change to remove nodes)
- UiApplier 가 current.removeAt(index, count)를 호출
  - 제거
  - zIndex 처리
  - owner 를 null 로
  - 부모에게 재측정 요청
  - 접근성 처리(마찬가지로 owner 에게 알림)

### 노드 이동을 위한 변경 사항 구체화 (Materializing a change to move nodes)
- UiApplier 가 current.move(from, to, count)를 호출

### 모든 노드를 지우는 변경 사항 구체화 (Materializing a change to clear all the nodes)
- 지우는거랑 동일

## Compose UI에서의 측정 (Measuring in Compose UI)
- 재측정(remeasure) -> 재배치(relayout) -> 지연 측정 처리
- 측정은 LayoutNodeWrapper 에 위임함
  - LayoutNodeWrapper 측정에 필요한 정보(상태)를 가지는 래퍼
- 부모 노드 -> 외부 래퍼 -> 내 노드의 modifier 체인 래퍼 -> 내부 래퍼 -> zIndex 순서대로 자식들(자식입장에서는 외부래퍼) 요런순서로 측정
  - 재측정이 요청될때마다 위 과정을 거침
- 측정하면서 측정정책(람다) MeasurePolicy 에서 오는 변경가능한 상태를 읽음
- 측정결과가 이전과 달라지면 부모에게도 재측정 요청

### 측정 정책 (Measuring policies)
- 측정 정책이 변경될때마다 재측정 요청함
- 커스텀 Layout: https://developer.android.com/develop/ui/compose/layouts/custom
  - Modifier.layout()
  - Layout()
- Box 의 측정 정책 예시

### 고유 크기 측정 (Intrinsic measurements)
- 측정 단계에서 자식 composable 측정은 딱 한번만 가능함(성능이유로 강제함)
- 자식의 추정 크기를 알아야 할 때 유용
- Modifier.width(IntrinsicSize.Max) 의 경우 자식중에서 가장 넓은 크기에 맞춰짐
  - 자식중에 fillMaxWidth 를 제외한..(wrapContentWidth 는 이게 안됨)
- https://developer.android.com/jetpack/compose/layouts/intrinsic-measurements

### 레이아웃 제약 조건 (Layout Constraints)
- 자식은 Constraints 에 min, max 사이에 적절한 크기로 사용하는게 일반적임
- Constraints 에 max 가 Constraints.Infinity 로 왔다면 자식크기를 알아서 잘 계산해줘야함
  - LazyColumn 예시. 
    - 내부적으로 LazyList 를 사용, 아이템은 SubcomposeLayout 을 사용한다
    - 자식 아이템이 fillMaxHeight 여도 꽉채우지 않고 알아서 조절해 버림
  - LazyVerticalGrid 예시
    - 정확한 크기를 위해 min == max 를 강제함

## LookaheadLayout
- LookaheadScope 으로 바뀜
- 동일한 컴포저블이 새로운 위치로 변경되는 예시가 있다고 치면,
- 일단 movableContentOf 를 사용하면 유용: https://developer.android.com/reference/kotlin/androidx/compose/runtime/package-summary#movableContentOf(kotlin.Function0)
  - 리컴포지션 때 이 영역내에서 호출된 상태와 노드를 기억해두었다가 새로운 위치에서 재사용한다
- 새로 호출될 곳의 크기와 위치의 정보를 먼저 제공해주는게 LookaheadLayout
  - 화면 전환 애니메이션 같은거 연출할때 사용
  - 관찰자(observe) 를 통해 점진적으로 제공함. 즉 딱 애니메이션용

### 레이아웃을 사전 계산하는 방법으로서의 LookaheadLayout (Yet another way of pre‐calculating layouts)
- SubcomposeLayout, intrinsics, LookaheadLayout 비교
- SubcomposeLayout: 측정때 까지 composition(과 측정) 을 지연시킴. 특정 자식을 subcompose 해서 측정값을 다른곳에 쓴다던가 특정 자식은 나중에 compose 한다던가(조건부 composition). 미리 배치될곳의 값을 알아오는 용도만으로는 상대적으로 비용이 크다
- Intrinsics: Subcomposition 보단 효율적. 근데 정확히 우리가 원하는 기능이 아님
- LookaheadLayout: 자동 애니메이션을 지원하기 위한게 그냥 목적. 캐싱을 좀더 많이함. 측정+배치도함. 다만 constraint 조작은 못함

### LookaheadLayout의 동작 원리 (How it works)
- 사전단계(look ahead)에 미리 연산된 값을 사용 가능
- 코드를 보면 설명 내용 현재랑 안맞음
  - Modifier.intermediateLayout 는 LookaheadScope 아니어도 사용가능
  - IntermediateMeasureScope 멤버 lookaheadSize 사용 가능
  - LookaheadScope 멤버 Placeable.PlacementScope.lookaheadScopeCoordinates
  - LookaheadScope 멤버 LayoutCoordinates.toLookaheadCoordinates()

### Lookaheadlayout의 내부 동작 (Internals of Lookaheadlayout)
- 측정: 기존 노드 돌듯이 LayoutNodeWrapper 의 LayoutNodeWrapper 통해
- 배치: 측정때랑 비슷한데 다만 placeAt 의 목적이 실제 배치가 아니라 계산을 위한것

- 전반적으로 샘플이랑 꽤 다르고 샘플코드도 동작안함.. 

## Modifier 체인 모델링 (Modeling modifier chains)
- Modifier는 아래 3가지를 제공하는 추상화
  - then: 다른 Modifier 를 결합. 많이 사용하는 height padding 등이 코드를 보면 then 을 사용하고 ModifierNodeElement : Modifier.Element : Modifier 타입을 파라매터로
  - 함수 체인: fold
  - any, all 조건 체크
- 링크드 리스트 형태. then 내부를 보면 CombinedModifier(a, CombinedModifier(b, CombinedModifier(c ...))) 이런식임
  - CombinedModifier(in: Modifier, out: Modifier)

### LayoutNode에 modifier 설정 (Setting modifiers to the LayoutNode)
- Layout 내부에 ReusableComposeNode 를 만들때 
  - update 때 set(materialized, { this.modifier = it }) 이런 코드가 있음
  - val materialized = currentComposer.materialize(modifier) 이렇게 구체화된 modifier 라는걸 사용
- 구체화된 Modifier 란? 2가지 타입이 있음
  - standard(or normal): 이전 설명에 나왓던 상태가 없어서 래핑되는 방식으로 사용햐는거
  - composed: 상태가 있는거. 내부에서 remember 나 CompositionLocal 를 사용하는경우 Composition 의 컨텍스트에서 사용되야함
- ComposedModifier 멤버데 @Composable factory 람다가 있어서 이걸로 Composition 컨텍스트 처리가 가능함
  - 예시로 .clickable 을 보면 composed 를 사용하고 람다내에서 remember 등을 사용해서 구현함(현재는 node 를 이용한 다른방식으로 구현함)
  - focusable, scroll, draggable 등등 Modifier.composed()
  - Modifier.composed() 는 성능이 좋지 않다. @composable 함수인데 리턴이 Modifier 라 내부적으로 메모이제이션이 안되고 매번 호출됨. 그래서 Modifier.Node 를 사용 하도록 권장(clickable 변경한거 참고)
- Owner(AndroidComposeView)의 루트(root) LayoutNode 을 만드는 특수한 경우 직접 modifier 를 설정하기도 함

### LayoutNode가 새로운 modifier를 받아 들이는 방법 (How LayoutNode ingests new modifiers)
- LayoutNode에 modifier 가 설정되면 이미 있던 modifier 는 캐시 영영으로 들어감
- 노드는 새로운 modifier 를 초기화해서 헤드 부터 foldIn(h->t) 으로 체인하면서 캐싱되엇던거 중에 동일한거는 재사용 함
- 외부 LayoutNodeWrapper 를 재구성하면서 자신의 modifier 체인을 만들면서 래핑함 이때 꼬리부터 foldOut(t->h) 으로 체인하면서 캐싱했던거 재사용(가능한거만)
- 헤드까지 오면 부모듸 내부 LayoutNodeWrapper 로 할당됨
- 이후 캐싱된거 버리고 attach 호출, 필요하면 부모 무효화

## 노드 트리 그리기 (Drawing the node tree)
- LayoutNodeWrapper 체인 따라서 그림(부모-나-자식)
- 안드로이드 뷰는 측정 - 배치 - 그리기
- compose 는 루트노드 부터 dirty 체크 된거 측정 - 배치 - 그리기
  - compose 는 멀티플랫폼 지향이라 그리기자체는 추상화된 canvas 에 위임함
  - 안드로이드의 경우는 Canvas 를 사용하면서 암시적으로 paint 를 재사용함
- LayoutNodeWrapper 는 자체적인 그리기 레이어를 가지고 있을수 있음(없을수도 있음)
  - RenderNodeLayer: compose UI 기본으로 사용 하드웨어를 이용
  - ViewLayer: RenderNodeLayer 를 직접적으로 사용못하는경우 대체수단. 안드로이드 View 에 의존하고 RenderNodeLayer 보다 기능이 많음
- LayoutNodeWrapper 에 그리기 레이어가 없으면 modifier 그리기를 시도. 이후 다음 래퍼꺼 그리기
- 모든 래퍼가 다 그려지면 다시 그려질 노드 display list 를 업데이트함
- 스냅샷 상태가 변경될때마다 그걸 관찰해서 그리기 레이어를 무효화함

## Jetpack Compose에서의 Semantics (Semantics in Jetpack Compose)
- semantic: 접근성이나 테스팅을 위한 별도의 메카니즘
- 루트 노드 modifier 에 semanticsModifier 라는걸 지정함
- 각 노드에 id 를 부여함

### semantics 변화에 대한 알림 (Notifying about semantic changes)
- 안드로이드의 경우 owner 가 AccessibilityDelegateCompat 를 이용해 접근성 동작을 수행함
- 구조적 변화가 생기면 conflated channel 을 이용해 0.1 초마다 접근성 이벤트를 보냄
- 그리고 semantic 트리의 노드 업뎃함

### 병합된/병합되지 않은 semantic 트리 (Merged and unmerged semantic trees)
- 병합된거를 구현고 싶은경우 mergeDescendants 속성을 사용함

# 5. 상태 스냅샷 시스템 (State snapshot system)
- 상태를 표현하고 상태 변경 사항을 전파
- 반응형 모델

## 스냅샷 상태란 (What snapshot state is)
- 변경 사항을 기억하고 관찰할 수 있는 분리된 상태
- mutableStateOf, mutableStateListOf, mutableStateMapOf, derivedStateOf, produceState, collectAsState
- State 타입
  - 얘도 @Stable 임. 그러니 이걸 쓰는 타입도 @Stable 이 되야함
- compose compiler 에 의해 Composable 함수내에서 선언된 스냅샷들은 래핑되어 자동으로 읽기 자동 추적을 한다
  - 상태가 변경되면 무효화 시킴(RecomposeScope)
- 동시성 시나리오를 보장함(스레드 세이프)

### 동시성 제어 시스템 (Concurrency control systems)
- 동시성 제어 문제 개념 설명...
- 동시성 제어를 다루는 3가지 관점
  - 낙관적: 커밋할때 몇가지 규칙이 안맞을때만 중단하고 재실행 -> jetpack compose 는 이걸 사용
  - 비관적: 경쟁관계 풀릴때까지 차단
  - 반낙관적: 특정상황만 차단
- 더불어 다중 버전 동시성 제어(MVCC) 를 사용함
  - 데이터베이스 객체가 쓰여질 때마다 새로운 버전을 생성함으로써 동시성과 성능을 향상시킵니다. 또한 객체의 여러 최신 관련 버전을 읽을 수 있도록 합니다

### 다중 버전 동시성 제어 (MCC or MVCC) (Multiversion concurrency control (MCC or MVCC))
- Non-lock concurrency control
- composable 함수는 병렬로 호출이 가능함
- 이때 각 순간에 스냅샷을 한벌 복사해서 작업후 각 다른 버전의 스냅샷이 생김
 - 이렇게 생성된 다른 버전은 모든 로컬 변경 사항이 완료될때까지 다른 스레드에서 접근이 안됨
- 원본 데이터를 직접 수정하는것이 아니라 복사본에 수정된 결과를 가지고 이를 변경 히스토리로 만들어 '상태 기록' 이라고 함
- 각 스냅샷은 고유 id 를 가지고 있고 이걸 가지고 작업하기 때문에 별도 lock 은 필요 없

## 스냅샷 (The Snapshot)
- 해당 시점에 모든 State 를 사용 하는 객체들의 복사본
- 각 스레드는 다른 스냅샷으로 작업한다
- compose runtime 에 Snapshot 이라는 클래스가 있음
  - Snapshot.takeSnapshot() 스냅샷 생성 - 이때 스냅샷에대한 고유 id 를 만듬
  - snapshot.dispose() 버림 - 스냅샷을 다 썻으면 폐기해야함(즉 라이프사이클이 있다)
- 스냅샷 설명글: https://dev.to/zachklipp/introduction-to-the-compose-snapshot-system-19cn
- Snapshot.takeSnapshot() 로 만들어진 스냅샷은 기본적으로 읽기만 가능함
- 스냅샷 클래스
  - ReadonlySnapshot: 읽기만 가능
  - MutableSnapshot: 수정도 가능
  - NestedReadonlySnapshot/NestedMutableSnapshot: 스냅샷 트리에서 하위 스냅샷에 대한 읽기/쓰기
  - GlobalSnapshot: 전역 스냅샷. 루트 스냅샷
  - TransparentObserverMutableSnapshot: 특수타입. 읽기/쓰기가 일어났을때 옵저버에게 알림 용도 id 가 상위 스냅샷과 동일함

### 스냅샷 트리 (The snapshot tree)
- GlobalSnapshot 가 루트로 트리를 형성한다
- Nested 스냅샷은 독립적으로 삭제가능한 스냅샷의 복사본
- 모든 스냅샷 타입은 Nested 스냅샷을 만들수 있고 여러개 가질수도 있다.
- subcomposition 이 필요한경우 Nested 스냅샷을 생성한다.
- 부모 스냅샷을 유지하면서 본인걸 삭제 가능
- 모든 스냅샷은 readonlay 스냅샷을 만들수 있으나 mutable 스냅샷은 mutable 스냅샷 에서만 생성 가능

## 스냅샷과 쓰레딩 (Snapshots and threading)
- 각 스냅샷은 병렬로(여러 스레드에서) 처리될수 있다
- Snapshot.current: 현 스레드에서 active 한 스냅샷 리턴. 없으면 global 스냅샷 리턴

## 읽고 쓰기 관찰하기 (Observing reads and writes)
- 상태가 바뀌면 observe 가능(read/write)
```
val ss = Snapshot.current.takeNestedSnapshot {  }  // 여기 블럭내에서 변경이 일어나면
ss.enter {  }  // 여기 블럭으로 콜백을 받음
```
- 위 두가지 조합해서 만든게 snapshotFlow
  - snapshotFlow 블럭 내에 State 가 변경될때마다 flow 로 받음
- Nested 스냅샷에 read 옵저버를 걸었을 경우 부모까지 알림이 간다

- write 옵저버는 mutable 스냅샷에대해서만 가능함
  - Snapshot.takeMutableSnapshot 사용

- read/write 옵저버 사용 예시로 recomposer 의 composing 함수가 있음
- derivedStateOf 는 Snapshot.observe(readObserver, writeObserver, block) 로 구현함

## 가변적인 스냅샷 (MutableSnapshots)
- mutable 스냅샷은 recomposition 을 트리거 하기위해 가변 상태를 다룰때 사용
- State 는 처음 스냅샷 생성했을때를 유지하다가 변경 사항이 생기면 트리의 아래에서 위로 전파함
  - NestedMutableSnapshot.apply 혹은 MutableSnapshot.apply 를 호출
  - 만약 apply 호출이 실패하면 해당 스냅샷과 변경사항은 버리고 새로운 composition 을 예약함
- 모든 하위 트리 스냅샷이 적용되고 마지막 루트까지 오면 전역(global)에 적용한 상태가 됨
- mutable 스냅샷은 생성후 apply 나 dispose 를 통해 끝남(생명주기)
- apply 호출은 atomic 함
- 변경 사항 적용(apply)을 안하고 dispose 를 하면 변경사항도 같이 버림
- snapshot.enter 와 snapshot.apply 를 합친 Snapshot.withMutableSnapshot 라는 것도 있음

## 글로벌 스냅샷과 중첩된 스냅샷 (GlobalSnapshot and nested snapshots)
- GlobalSnapshot: mutable, 전역에 하나만 존재, apply 가 없음, dispose 도 없음, nested 안됨
  - 변경사항을 적용하려면 Snapshot.advanceGlobalSnapshot 통해 수행함(그러면 이전 GlobalSnapshot 을 버림)
- GlobalSnapshot 스냅샷 시스템을 초기화 할때 생성함
  - 이후 각 composition 마다 자체적으로 nested mutable 스냅샷을 생성하고 트리로 구성함
  - 이후 subcomposition 마다 nested 스냅샷을 생성하고 트리로 구성함
- GlobalSnapshotManager.ensureStarted 를 통해 read/write 옵저빙을 시작함

## 상태 객체 및 상태 기록 (StateObjects and StateRecords)
- 다중 버전 동시성 제어는 상태가 기록될 때마다 새 버전이 생성되고 성능상 이점이 있음
  - 스냅샷 생성: O(1) 상태수, 스냅샷 커밋: O(n) 변경된 상태수
  - 스냅샷 시스템에 관련 없이 가비지 컬렉팅에 자유로움
- 스냅샷 상태는 내부적으로 StateObject 로 모델링함. 복사된 버전은 StateRecord 로 모델링
  - 스냅샷 생성시 id 가 가장 큰값이 최신의 유효한 버전
- 어떤 스냅샷이 유효한거고 유효하지 않은건지?
  - StateRecord id(스냅샷 id) 가 큰값 이고 invalid set 에 포함이 안된거 -> valid
  - 1번 스냅샷 생성 이후 2번 스냅샷 생성되었다고 하면, 1번에 변경 기록이 생긴거는 -> invalid
  - 적용 전에 폐기되는 스냅샷 안에 있는 기록들 -> invalid
- StateObject 
  - StateRecord 를 가지고 있고 링크드 리스트 형태임(클래스 멤버 val firstStateRecord 가 헤드)
  - mergeRecords 함수를 가짐: 충돌 병합용
- StateRecord
  - 현재 스냅샷 id 를 참조해서 가지고 잇음
  - 링크드 리스트의 노드 형태
- mutableStateOf 예시
  - StateObject 의 구현체 SnapshotMutableState(밸류 의 래퍼임) 를 리턴
  - StateRecord 의 구현체 StateStateRecord 사용
  - 읽기 동작시 StateRecord 를 뒤져서 가장 최근 vaild 한 값을 리턴함
- mutableStateListOf 예시
  - StateObject 의 구현체 SnapshotStateList 를 리턴
  - StateRecord 의 구현체 StateListStateRecord 사용

### 읽기와 쓰기 상태 (Reading and writing state)
- 읽기 동작: StateRecord 목록을 순회하여 가장 최근 에 유효한 것(가장 높은 스냅샷 ID를 가진)을 찾는다
  - mutableStateOf 예시(SnapshotMutableStateImpl)
    - readable 호출. 위에 유효 조건 확인. 읽기 옵저버들에게 알림. 스냅샷을 현재 스레드 스냅샷으로 엮음(복사된 스냅샷이 아니면 global 스냅샷을 엮음)
- 쓰기 동작
  - readable 로 유효한 record 를 가져옴. 값 비교를 해서 다르면 overwriteable 를 통해 쓰기 동작을 함. 쓰기 옵저버들에게 알림

## 오래된 기록 제거 또는 재사용하기 (Removing or reusing obsolete records)
- 다중 버전 동시성 제어에 의해 같은 상태의 여러가지 스냅샷이 있을수가 있는데 이중에 안쓰는거는 제거 한다
- 스냅샷은 open 상태 라는게 있음. open 상태가 되면 변경 기록들은 다른 스냅샷에서 read 가 불가능해지고 닫혀야 가능해짐
  - open 상태인 스냅샷은 별도 set 으로 관리
- record 재활용 가능 여부 이렇게
  - 가장 낮은 id 를 가지고 open 상태의 스냅샷을 찾음(id 값은 생성하는 순서대로 증가시키는 메카니즘)
  - record 가 유효하고 위에서 스냅샷에서 보이지 않는경우 재사용 가능한거로 판단함
- 스냅샷이 apply 되고 record 가 가려지면 재사용 가능
- 스냅샷이 apply 전에 버려지면 그 안에 모든 record 는 유효하지 않은상태가 되고 재사용 가능

## 변경 사항 전파하기 (Change propagation)
- 스냅샷을 닫으면 open 스냅샷 set 에서 id 가 제거되고 다른 스냅샷에서 state record 가 읽기가 가능해짐. 이때 상태 변경사항 전파함
- 스냅샷을 닫을때 즉시 새로운 스냅샷으로 교체하고 싶은경우 advancing 이라고 함. 이렇게 새로 만들어진 스냅샷을 open 스냅샷 set 에 id 가 추가됨
  - global 스냅샷이 이렇게 동작함
  - nested 스냅샷도 apply 할때 mutalbe 스냅샷을 advancing 가능
- mutable 스냅샷이 apply 하면 변경 사항이 상위 스냅샷을 전파됨
  - active 스냅샷 set 에서 제거됨. open 스냅샷 set 에서 제거됨
  - record 들은 유효화 되고 읽기가 가능해짐
- 위 작업중 상태 충돌(쓰기 작업중)이 있으면 충돌 해결 먼저 헤야함
  - 스냅샷 apply 시점에 다른 스냅샷의 변경 사항과 함께 추가되기 때문에 충돌 발생 가능성이 있음
  - 충돌이 있을거 같으면 이를 감지해서 가능한 머지를 수행
- 다른 스냅샷의 보류중이 변경사항이 없는 경우: 스냅샷 닫음처리, set 처리 이후 옵저버에게 알림
- 다른 스냅샷의 보류중이 변경사항이 있는 경우
  - id 가 최신인지 record 들의 값이 다른지 비교해서 값을 결정
  - 변경 값이 다르면 머지 수행. 머지후 새로운 record 를 state 객체의 링크드리스트 첫번째에다가 껴 넣음
- 스냅샷의 apply 가 실패하는 경우 위에 변경사항이 없는 경우랑 동일한 과정으로 옵저버에게 알림
- nested 스냅샷의 경우 부모에게 전파하기 때문에 부모가 가지고 있는 set 에서 id 를 제거함

### 쓰기 충돌 병합하기 (Merging write conflicts)
- 머지를 해야되는 경우 현재 값, 변경 apply 이전값, 변경 apply 이후 값으로 머지 시도함. 머지 정책은 state 객체에 의존함
- 현재 runtime 에는 적절한 머지 지원 정책이 없음. 충돌 나면 runtime exception 발생
  - 이를 방지 하기 위해 state 객체 접근시 고유키를 가지고 접근함
  - mutableStateOf 의 경우 머지에 StructuralEqualityPolicy(고유 키 포함 객체 값의 deep 비교) 를 사용
- SnapshotMutationPolicy interface 를 구체화해서 머지 정책을 직접 만들수 있음
  - fun equivalent(a: Int, b: Int): Boolean  // 이걸로 동일하다고 판단되면 아래 머지 호출 안함
  - fun merge(previous: Int, current: Int, applied: Int)
- 커스텀 머지 정책 예시

# 6. 이펙트 및 이펙트 핸들러 (Effects and effect handlers)

## 사이드 이펙트
- 사이드 이펙트 개념 설명.. 함수 외부의 레퍼를 사용한 동작을 한다던가 같은 인풋을 넣고 호출햇는데 다른 결과가 나온다던가

## Compose에서의 사이드 이펙트 (Side effects in Compose)
- composable 함수는 여러번 호출될수 있으므로.. 여러번 호출 하는게 부적절한경우, 또는 recomposition 와 관계 없이 상태가 필요한경우
- composable 의 composition 이 끝나는 경우 동작중이던 사이드 이펙트를 취소하고 리소스 해재하는 타이밍을 제공해줌 -> 이런 것들을 이펙트 핸들러 라고 한다

## 우리가 필요로 하는 것 (What we need)
- 즉 사이드 이펙트 라는거로 지원하는 매카니즘
- composable 생명주기에 맞춰서 동작함
- 적절한 코루틴내에서 동작
- composition 더이상 안할때(끝날때) 리소스 반환 타이밍 제공
- composition 더이상 안할때(끝날때) 알아서 동작중이던거 취소
- 상황에 따라 다시 실행 가능 해야함

## 이펙트 핸들러의 종류 (Effect Handlers)

### 비일시 중단 이펙트 (Non suspended effects)
- 인풋람다가 suspend 가 아닌거

#### DisposableEffect
- Composable이 composition에 들어갈 때 처음 실행, 그이후 인풋 키가 변경 될때마다 실행
- composition을 떠날 때 혹은 키가 변경됫을때 onDispose 콜백을 호출

#### SideEffect
- composition 이 실패 하지 않고 성공하면 인풋 람다 호출
- composition, 매번 recomposition 때마다 실행
- 상태로서 슬롯테이블에 저장되지 않음
- 외부의 상태 업뎃할대 유용

#### currentRecomposeScope
- invalidate() 호출 가능

### 일시 중단 이펙트 (Suspended effects)
- 인풋람다가 suspend 인거

#### rememberCoroutineScope
- CoroutineScope 을 만듬. AndroidUiDispatcher.Main 사용
- composition 떠날때 취소됨
- 주로 사용자 인터랙션 처리용

#### LaunchedEffect
- composition에 진입할 때 이펙트를 실행, 떠날대 취소
- 키가 변경되면 취소하고 다시 실행
- Applier 의 dispatcher 사용(AndroidUiDispatcher.Main)

#### produceState
- 내부적으로 LaunchedEffect 를 사용해서 value 를 상태로 리턴

### 서드 파티 라이브러리 어댑터 (Third party library adapters)
- LiveData.observeAsState()
- Observable.subscribeAsState()
- Flow.collectAsState()

# 7. Compose Runtime 고급 사용 사례 (Advanced Compose Runtime use cases)

## compose runtime & ui
- 멀티 플랫폼으로서 아래같은 라이브러리 흐름
  - compiler -> runtime -> ui -> android ui
  - compiler -> runtime -> ui -> desktop
  - compiler -> runtime -> web
- comose ios, desktop, web 등은 렌더링 계층을 skia 꺼로 사용
  - skia: 2d 그래픽 라이브러리. 각종 플랫폼의 2d 그리는 부분을 추상화를 함. 대표적으로 플러터에서 사용




