# 추상 클래스보다는 인터 페이스를 우선하라

> 자바 8부터 인터페이스도 default method를 제공할 수 있게되어, 이제는 두 메커니즘 모두 인스턴스 메서드를 구현 형태로 제공할 수 있다.

<details>
  <summary>디폴트 메서드(Default Method)</summary>

[참고](https://velog.io/@heoseungyeon/%EB%94%94%ED%8F%B4%ED%8A%B8-%EB%A9%94%EC%84%9C%EB%93%9CDefault-Method)
    
</details>

## 추상클래스와 인터페이스

- 추상클래스 : 다중 상속이 불가능, 구현체와 상위-하위 클래스 관계를 갖는다.
- 인터페이스 : 다중 상속이 가능, 구현한 클래스와 같은 타입으로 취급된다.

## 인터페이스의 장점

### 1. 기존 클래스에 손쉽게 새로운 인터페이스를 구현할 수 있다.

> 인터페이스가 요구하는 메서드를 (아직 없다면) 추가하고, 클래스 선언에 implements 구문만 추가하면 끝이다. <br>
반면 기존 클래스 위에 새로운 추상 클래스를 끼워넣기는 어려운 게 일반적이다.

```
class Dog { 
    void bark() { System.out.println("멍멍!"); }
}

// 새로운 인터페이스 추가
interface Animal {
    void eat(); 
}

class DogWithAnimal extends Dog implements Animal {
    @Override
    public void eat() {
        System.out.println("먹는 중...");
    }
}
```

```
abstract class Animal { 
    abstract void eat(); 
}

class Dog extends Animal { // 이미 Animal을 상속받음
    @Override
    void eat() { System.out.println("먹는 중..."); }
}

// ❌ 새로운 추상 클래스 추가 불가 (Java는 다중 상속X)
abstract class Mammal extends Animal { } 
class Dog extends Mammal { } // 불가능!

```

### 2. 인터페이스는 믹스인(mixin) 정의에 안성맞춤이다.

<details>
  <summary>믹스인(Mixin)</summary>

    기존 클래스에 추가 기능을 덧붙이는 용도로 사용되는 개념.
    ex) Comparable, Iterable, AutoCloseable

</details>

- 인터페이스가 믹스인에 적합한 이유

> 다중 구현 가능 → 기존 클래스에 쉽게 추가 가능 <br>
디폴트 메서드 제공 가능 → 기본 기능을 정의해 재사용 가능 <br>
상속 관계와 무관 → 기존 클래스 구조를 바꾸지 않고 기능 추가 가능

### 3. 인터페이스로는 계층구조가 없는 타입 프레임워크를 만들 수 있다.

```
public interface Singer {
	AudioClip sing(Song s);
}

public interface Songwriter {
	Song compose(int chartPosition);
}
```

```
public interface SingerSongwriter extends Singer, Songwriter {
	AudioClip strum();
	void actSensitive();
}
```

> 이를 추상클래스로 만드려면 가능한 조합 전부를 각각의 클래스로 정의한 고도비만 계층구조가 만들어질 것이다. <br>
→ 속성이 n개라면 지원해야 할 조합의 수는 $2^n$개 (노래가능한 작곡가, 노래 불가능한 작곡가, 작곡가능한 가수, 작곡 불가능한 가수)

## 인터페이스와 Wrapper 클래스

- 래퍼 클래스와 인터페이스를 함께 사용한다면 인터페이스의 기능을 향상, 더욱 안전하고 강력하게 하는 방법
- 타입을 추상클래스로 정의한다면, 그 타입에 기능을 추가하는 방법은 상속 뿐
- 상속해서 만든 클래스는 래퍼 클래스보다 활용도가 떨어지고 규칙이 깨지기 더욱 쉽다. 

## 인터페이스와 default 메서드

- default 메서드를 제공할 때 규칙 

> 상속하는 클래스를 위해 @implSpec 자바독 태그를 붙여 문서화 <br>
equals , hashCode 같은 Object 메서드는 디폴트 메서드로 작성해선 안된다


## 추상 골격 구현 클래스(Skeletal Implementation Class)의 장점

>  인터페이스에서 구현해 줄 수 있는 것들은 디폴트 메서드로 구현하고, 구현해 줄 수 없는 것들은 추상 골격 클래스에서 나머지 메서드를 구현한다.

### 1.  인터페이스로는 타입을 정의하고, 필요하면 디폴트 메서드 몇 개도 함께 제공한다. 그리고 골격 구현 클래스는 나머지 메서드들까지 구현한다.

> 이렇게 해두면 단순히 골격 구현을 확장하는 것만으로 이 인터페이스를 구현하는 데 필요한 일이 대부분 완료된다.

### 2. 시뮬레이트한 다중 상속(simulated multiple inheritance)
>  다중 상속의 많은 장점을 제공하는 동시에 단점은 피하게 해준다.

## 추상 골격 구현 클래스 예시

> 인터페이스 - 골격 구현(Skeletal Implementation) 클래스 - 구체(Concrete) 클래스 구조

### 1. BaseFilter에서는 필터 클래스에서 공통적으로 사용하는 메소드를 정의
```
public interface BaseFilter {
    
    /**
     * 실제 필터 로직 수행하는 메소드
     * @param request
     * @param response
     */
    void doFilterLogic(HttpServletRequest request, HttpServletResponse response);

    FilterExceptionHandler getFilterExceptionHandler();
}
```

### 2.AbstractFilter에서는 OncePerRequestFilter를 상속받고 BaseFilter를 구현하는 골격 구현 클래스를 정의
> 여기서 하위 클래스에서 모두 공통적으로 사용하는 OncePerRequestFilter의 doFilterInternal을 오버라이딩하여 공통 내용을 작성한 후, 하위 클래스에서 작성할 로직과 필터 ExceptionHandler를 인터페이스에서 정의한 메소드로 사용하였습니다.

```
public abstract class AbstractFilter extends OncePerRequestFilter implements BaseFilter {
    
    /**
     * 실제 필터 로직 수행하는 메소드 구현
     * @param request
     * @param response
     * @param filterChain
     * @throws IOException
     */
    @Override
    protected void doFilterInternal(
            @NonNull HttpServletRequest request,
            @NonNull HttpServletResponse response,
            @NonNull FilterChain filterChain
    ) throws IOException {
        try{
            doFilterLogic(request,response);
            filterChain.doFilter(request,response);
        }
        catch (ValidationException e){
            logger.error(e.getMessage());
            getFilterExceptionHandler().setErrorResponse(e.getCode(),e.getMessage(),response);
        }
        catch (Exception e){
            logger.error(e.getMessage());
            getFilterExceptionHandler().sendErrorToSlack(request,response,e);
        }
    }
}
```

### 3. 마지막으로 JwtFilter에서 골격 구현 클래스에서 사용한 인터페이스 메소드를 오버라이딩
```
@Component
@RequiredArgsConstructor
public final class JwtFilter extends AbstractFilter implements BaseFilter {
    private final FilterExceptionHandler filterExceptionHandler;
    private final UriProvider uriProvider;
    private final TokenProvider tokenProvider;

    @Override
    public void doFilterLogic(HttpServletRequest request, HttpServletResponse response) {
        // 2. 헤더의 토큰이 존재하는지 체크
        logger.info("2. 토큰 유효성 검사");

        // 토큰이 필요 없는 API는 패스
        String uri = uriProvider.getURI(request);
        if(!uriProvider.isValidationPass(uri)){
            String jwt = tokenProvider.resolveToken(request, TokenProvider.HEADER_NAME);

            // 토큰 유효성 검증 후 SecurityContext에 저장
            Claims claims = tokenProvider.getAuthenticationClaims(jwt);
            Authentication authentication = tokenProvider.getAuthentication(jwt, claims);
            SecurityContextHolder.getContext().setAuthentication(authentication);
        }
    }

    @Override
    public FilterExceptionHandler getFilterExceptionHandler() {
        return filterExceptionHandler;
    }
}
```

> **공통 부분이 깔끔하게 정리되며 필터를 추가할 경우 확장하기 편하고 코드의 중복이 줄어들어 유지 보수의 편리성이 증가하였습니다.**

[예시참고](https://velog.io/@gale4739/Spring-Boot-Interface-%EA%B3%A8%EA%B2%A9-%EA%B5%AC%ED%98%84-%ED%81%B4%EB%9E%98%EC%8A%A4-%ED%81%B4%EB%9E%98%EC%8A%A4-%EA%B5%AC%EC%A1%B0-%EB%B3%80%EA%B2%BDFeat.-Composition)

## 결론
> 일반적으로 다중 구현용 타입에는 인터페이스가 가장 적합하다. <br>
복잡한 인터페이스라면 골격 구현을 함께 제공하는 방법을 고려하자. <br>
골격 구현은 '가능한' 인터페이스의 디폴트 메서드로 제공하여 그 인터페이스를 구현한 모든 곳에서 활용하도록 하는 것이 좋다. <br>
인터페이스에 걸려있는 구현상 제약 때문에 골격 구현을 추상 클래스로 제공하는 경우가 더 흔하다. <br>