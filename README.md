# Login
회원가입,로그인 구현 후 쿠키적용 및 세션을 구현 해보고 servlet.http 세션도 사용해본다. 그리고 filter와 interceptor도 구현해본다

향후 web을 다른 기술로 바꾸어도 도메인은 그대로 유지할 수 있어야 한다.<br/>
이렇게 하려면 web은 domain을 알고있지만 domain은 web을 모르도록 설계해야 한다. 이것을 web은
domain을 의존하지만, domain은 web을 의존하지 않는다고 표현한다.<br/> 예를 들어 web 패키지를 모두
삭제해도 domain에는 전혀 영향이 없도록 의존관계를 설계하는 것이 중요하다.<br/> 반대로 이야기하면
domain은 web을 참조하면 안된다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205494150-37010ea8-9fc2-4114-94c9-6237a9062e6c.png)
![image](https://user-images.githubusercontent.com/69129562/205494225-9246c191-e8fa-4a97-bca8-61491ec859a2.png)
쿠키에는 영속 쿠키와 세션 쿠키가 있다.<br/>
영속 쿠키: 만료 날짜를 입력하면 해당 날짜까지 유지<br/>
세션 쿠키: 만료 날짜를 생략하면 브라우저 종료시 까지만 유지<br/>
브라우저 종료시 로그아웃이 되길 기대하므로, 우리에게 필요한 것은 세션 쿠키이다.<br/>
<br/>
쿠키 생성 로직<br/>
Cookie idCookie = new Cookie("memberId", String.valueOf(loginMember.getId()));<br/>
response.addCookie(idCookie);<br/>
로그인에 성공하면 쿠키를 생성하고 HttpServletResponse 에 담는다.<br/> 쿠키 이름은 memberId 이고, 값은
회원의 id 를 담아둔다.<br/> 웹 브라우저는 종료 전까지 회원의 id 를 서버에 계속 보내줄 것이다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205495848-d661f7dd-ba7d-4a31-92c3-ae3893071d8f.png)
<br/>
쿠키와 보안 문제<br/>
쿠키를 사용해서 로그인Id를 전달해서 로그인을 유지할 수 있었다. 그런데 여기에는 심각한 보안 문제가
있다.<br/>

쿠키 값은 임의로 변경할 수 있다.<br/>
클라이언트가 쿠키를 강제로 변경하면 다른 사용자가 된다.<br/>
실제 웹브라우저 개발자모드 Application Cookie 변경으로 확인<br/>
Cookie: memberId=1 Cookie: memberId=2 (다른 사용자의 이름이 보임)<br/>
쿠키에 보관된 정보는 훔쳐갈 수 있다.<br/>
만약 쿠키에 개인정보나, 신용카드 정보가 있다면?<br/>
이 정보가 웹 브라우저에도 보관되고, 네트워크 요청마다 계속 클라이언트에서 서버로 전달된다.<br/>
쿠키의 정보가 나의 로컬 PC에서 털릴 수도 있고, 네트워크 전송 구간에서 털릴 수도 있다.<br/>
해커가 쿠키를 한번 훔쳐가면 평생 사용할 수 있다.<br/>
해커가 쿠키를 훔쳐가서 그 쿠키로 악의적인 요청을 계속 시도할 수 있다.<br/>
대안<br/>
쿠키에 중요한 값을 노출하지 않고, 사용자 별로 예측 불가능한 임의의 토큰(랜덤 값)을 노출하고, 서버에서
토큰과 사용자 id를 매핑해서 인식한다. 그리고 서버에서 토큰을 관리한다.<br/>
토큰은 해커가 임의의 값을 넣어도 찾을 수 없도록 예상 불가능 해야 한다.<br/>
해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 해당 토큰의 만료시간을 짧게(예: 30분)
유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 토큰을 강제로 제거하면 된다<br/>
<br/>
앞서 쿠키에 중요한 정보를 보관하는 방법은 여러가지 보안 이슈가 있었다. 이 문제를 해결하려면 결국
중요한 정보를 모두 서버에 저장해야 한다. 그리고 클라이언트와 서버는 추정 불가능한 임의의 식별자
값으로 연결해야 한다.<br/>
이렇게 서버에 중요한 정보를 보관하고 연결을 유지하는 방법을 세션이라 한다.<br/>
<br/>
![image](https://user-images.githubusercontent.com/69129562/205496284-d1e06174-11ea-4ae7-95b3-4c04f354a06b.png)
세션 ID를 생성하는데, 추정 불가능해야 한다.<br/>
UUID는 추정이 불가능하다.<br/>
Cookie: mySessionId=zz0101xx-bab9-4b92-9b32-dadb280f4b61<br/>
생성된 세션 ID와 세션에 보관할 값( memberA )을 서버의 세션 저장소에 보관한다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205496353-15638105-97e4-46f2-ab1b-fcd6b2ff41b7.png)
클라이언트와 서버는 결국 쿠키로 연결이 되어야 한다.<br/>
서버는 클라이언트에 mySessionId 라는 이름으로 세션ID 만 쿠키에 담아서 전달한다.<br/>
클라이언트는 쿠키 저장소에 mySessionId 쿠키를 보관한다.<br/>
중요<br/>
여기서 중요한 포인트는 회원과 관련된 정보는 전혀 클라이언트에 전달하지 않는다는 것이다.<br/>
오직 추정 불가능한 세션 ID만 쿠키를 통해 클라이언트에 전달한다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205496430-d92be0d8-6f7c-436e-8263-c1f5d29bf03b.png)
클라이언트는 요청시 항상 mySessionId 쿠키를 전달한다.<br/>
서버에서는 클라이언트가 전달한 mySessionId 쿠키 정보로 세션 저장소를 조회해서 로그인시 보관한
세션 정보를 사용한다.<br/>
정리<br/>
세션을 사용해서 서버에서 중요한 정보를 관리하게 되었다. 덕분에 다음과 같은 보안 문제들을 해결할 수
있다.<br/>
쿠키 값을 변조 가능, 예상 불가능한 복잡한 세션Id를 사용한다.<br/>
쿠키에 보관하는 정보는 클라이언트 해킹시 털릴 가능성이 있다. 세션Id가 털려도 여기에는 중요한
정보가 없다.<br/>
쿠키 탈취 후 사용 해커가 토큰을 털어가도 시간이 지나면 사용할 수 없도록 서버에서 세션의
만료시간을 짧게(예: 30분) 유지한다. 또는 해킹이 의심되는 경우 서버에서 해당 세션을 강제로
제거하면 된다.<br/>
<br/>
ConcurrentHashMap : 동시 요청에 안전한 HashMap<br/>
<br/>
세션이라는 것이 뭔가 특별한 것이 아니라 단지 쿠키를 사용하는데, 서버에서 데이터를 유지하는 방법<br/>
<br/>
서블릿이 제공하는 HttpSession 을 사용하도록 개발해보자<br/>
//세션이 있으면 있는 세션 반환, 없으면 신규 세션 생성<br/>
 HttpSession session = request.getSession();<br/>
 //세션에 로그인 회원 정보 보관<br/>
 session.setAttribute(SessionConst.LOGIN_MEMBER, loginMember);<br/>
 session.invalidate() : 세션을 제거한다<br/>
 request.getSession(false) : request.getSession() 를 사용하면 기본 값이 create: true<br/>
이므로, 로그인 하지 않을 사용자도 의미없는 세션이 만들어진다. 따라서 세션을 찾아서 사용하는 시점에는
create: false 옵션을 사용해서 세션을 생성하지 않아야 한다.<br/>
<br/>
스프링은 세션을 더 편리하게 사용할 수 있도록 @SessionAttribute 을 지원한다<br/>
<br/>
세션 타임아웃 설정<br/>
세션은 사용자가 로그아웃을 직접 호출해서 session.invalidate() 가 호출 되는 경우에 삭제된다.<br/>
그런데 대부분의 사용자는 로그아웃을 선택하지 않고, 그냥 웹 브라우저를 종료한다.<br/> 문제는 HTTP가 비
연결성(ConnectionLess)이므로 서버 입장에서는 해당 사용자가 웹 브라우저를 종료한 것인지 아닌지를
인식할 수 없다.<br/> 따라서 서버에서 세션 데이터를 언제 삭제해야 하는지 판단하기가 어렵다.<br/>
이 경우 남아있는 세션을 무한정 보관하면 다음과 같은 문제가 발생할 수 있다.<br/>
세션과 관련된 쿠키( JSESSIONID )를 탈취 당했을 경우 오랜 시간이 지나도 해당 쿠키로 악의적인 요청을
할 수 있다.<br/>
세션은 기본적으로 메모리에 생성된다. 메모리의 크기가 무한하지 않기 때문에 꼭 필요한 경우만 생성해서
사용해야 한다. 10만명의 사용자가 로그인하면 10만개의 세션이 생성되는 것이다.<br/>
세션의 종료 시점<br/>
세션의 종료 시점을 어떻게 정하면 좋을까? 가장 단순하게 생각해보면, 세션 생성 시점으로부터 30분
정도로 잡으면 될 것 같다.<br/> 그런데 문제는 30분이 지나면 세션이 삭제되기 때문에, 열심히 사이트를
돌아다니다가 또 로그인을 해서 세션을 생성해야 한다 그러니까 30분 마다 계속 로그인해야 하는
번거로움이 발생한다.<br/>
더 나은 대안은 세션 생성 시점이 아니라 사용자가 서버에 최근에 요청한 시간을 기준으로 30분 정도를
유지해주는 것이다.<br/> 이렇게 하면 사용자가 서비스를 사용하고 있으면, 세션의 생존 시간이 30분으로 계속
늘어나게 된다.<br/> 따라서 30분 마다 로그인해야 하는 번거로움이 사라진다. HttpSession 은 이 방식을
사용한다.<br/>
스프링 부트로 글로벌 설정<br/>
application.properties<br/>
server.servlet.session.timeout=60 : 60초, 기본은 1800(30분)<br/>
(글로벌 설정은 분 단위로 설정해야 한다. 60(1분), 120(2분), ...)<br/>
세션 타임아웃 발생<br/>
세션의 타임아웃 시간은 해당 세션과 관련된 JSESSIONID 를 전달하는 HTTP 요청이 있으면 현재 시간으로
다시 초기화 된다. 이렇게 초기화 되면 세션 타임아웃으로 설정한 시간동안 세션을 추가로 사용할 수 있다.<br/>
session.getLastAccessedTime() : 최근 세션 접근 시간<br/>
LastAccessedTime 이후로 timeout 시간이 지나면, WAS가 내부에서 해당 세션을 제거한다<br/>
실무에서 주의할 점은 세션에는 최소한의 데이터만 보관해야 한다는 점이다. 보관한 데이터 용량 * 사용자
수로 세션의 메모리 사용량이 급격하게 늘어나서 장애로 이어질 수 있다.<br/> 추가로 세션의 시간을 너무 길게
가져가면 메모리 사용이 계속 누적 될 수 있으므로 적당한 시간을 선택하는 것이 필요하다. 기본이 30
분이라는 것을 기준으로 고민하면 된다.<br/>

애플리케이션 여러 로직에서 공통으로 관심이 있는 있는 것을 공통 관심사(cross-cutting
concern)라고 한다.<br/> 여기서는 등록, 수정, 삭제, 조회 등등 여러 로직에서 공통으로 인증에 대해서 관심을
가지고 있다.<br/>
이러한 공통 관심사는 스프링의 AOP로도 해결할 수 있지만, 웹과 관련된 공통 관심사는 지금부터 설명할
서블릿 필터 또는 스프링 인터셉터를 사용하는 것이 좋다.<br/> 웹과 관련된 공통 관심사를 처리할 때는 HTTP의
헤더나 URL의 정보들이 필요한데, 서블릿 필터나 스프링 인터셉터는 HttpServletRequest 를 제공한다.<br/>
필터 흐름<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러<br/>
필터를 적용하면 필터가 호출 된 다음에 서블릿이 호출된다. 그래서 모든 고객의 요청 로그를 남기는
요구사항이 있다면 필터를 사용하면 된다.<br/> 참고로 필터는 특정 URL 패턴에 적용할 수 있다. /* 이라고
하면 모든 요청에 필터가 적용된다.<br/>
참고로 스프링을 사용하는 경우 여기서 말하는 서블릿은 스프링의 디스패처 서블릿으로 생각하면 된다.<br/>
필터 제한<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 컨트롤러 //로그인 사용자<br/>
HTTP 요청 -> WAS -> 필터(적절하지 않은 요청이라 판단, 서블릿 호출X) //비 로그인 사용자<br/>
필터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를
체크하기에 딱 좋다.<br/>
필터 체인<br/>
HTTP 요청 -> WAS -> 필터1 -> 필터2 -> 필터3 -> 서블릿 -> 컨트롤러<br/>
필터는 체인으로 구성되는데, 중간에 필터를 자유롭게 추가할 수 있다.<br/> 예를 들어서 로그를 남기는 필터를
먼저 적용하고, 그 다음에 로그인 여부를 체크하는 필터를 만들 수 있다.<br/>
<br/>
![image](https://user-images.githubusercontent.com/69129562/205662152-69fd8774-a770-4b85-b38f-5c6a654992b8.png)
<br/>
doFilter(ServletRequest request, ServletResponse response, FilterChain chain)<br/>
HTTP 요청이 오면 doFilter 가 호출된다.<br/>
ServletRequest request 는 HTTP 요청이 아닌 경우까지 고려해서 만든 인터페이스이다. HTTP를
사용하면 HttpServletRequest httpRequest = (HttpServletRequest) request; 와 같이
다운 케스팅 하면 된다.<br/>

필터를 등록하는 방법은 여러가지가 있지만, 스프링 부트를 사용한다면 FilterRegistrationBean 을
사용해서 등록하면 된다.
![image](https://user-images.githubusercontent.com/69129562/205661822-4204da69-c1a1-43c1-bb12-3b8c17e5c2bd.png)
> 참고
> @ServletComponentScan @WebFilter(filterName = "logFilter", urlPatterns = "/*") 로
필터 등록이 가능하지만 필터 순서 조절이 안된다. 따라서 FilterRegistrationBean 을 사용하자.

스프링 인터셉터 - 소개<br/>
스프링 인터셉터도 서블릿 필터와 같이 웹과 관련된 공통 관심 사항을 효과적으로 해결할 수 있는 기술이다.<br/>
서블릿 필터가 서블릿이 제공하는 기술이라면, 스프링 인터셉터는 스프링 MVC가 제공하는 기술이다.<br/> 둘다
웹과 관련된 공통 관심 사항을 처리하지만, 적용되는 순서와 범위, 그리고 사용방법이 다르다.<br/>
스프링 인터셉터 흐름<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러<br/>
스프링 인터셉터는 스프링 MVC가 제공하는 기능이기 때문에 결국 디스패처 서블릿 이후에 등장하게 된다.<br/>
스프링 MVC의 시작점이 디스패처 서블릿이라고 생각해보면 이해가 될 것이다.<br/>
스프링 인터셉터에도 URL 패턴을 적용할 수 있는데, 서블릿 URL 패턴과는 다르고, 매우 정밀하게 설정할
수 있다.<br/>
스프링 인터셉터 제한<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터 -> 컨트롤러 //로그인 사용자<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 스프링 인터셉터(적절하지 않은 요청이라 판단, 컨트롤러 호출
X) // 비 로그인 사용자<br/>
인터셉터에서 적절하지 않은 요청이라고 판단하면 거기에서 끝을 낼 수도 있다. 그래서 로그인 여부를
체크하기에 딱 좋다.<br/>
스프링 인터셉터 체인<br/>
HTTP 요청 -> WAS -> 필터 -> 서블릿 -> 인터셉터1 -> 인터셉터2 -> 컨트롤러<br/>
스프링 인터셉터는 체인으로 구성되는데, 중간에 인터셉터를 자유롭게 추가할 수 있다. 예를 들어서 로그를
남기는 인터셉터를 먼저 적용하고, 그 다음에 로그인 여부를 체크하는 인터셉터를 만들 수 있다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205666519-d0393880-87a7-4442-a0d2-68feddd1bcec.png)
서블릿 필터의 경우 단순하게 doFilter() 하나만 제공된다. 인터셉터는 컨트롤러 호출 전( preHandle ),
호출 후( postHandle ), 요청 완료 이후( afterCompletion )와 같이 단계적으로 잘 세분화 되어 있다.<br/>
서블릿 필터의 경우 단순히 request , response 만 제공했지만, 인터셉터는 어떤 컨트롤러( handler )가
호출되는지 호출 정보도 받을 수 있다. 그리고 어떤 modelAndView 가 반환되는지 응답 정보도 받을 수
있다<br/>
![image](https://user-images.githubusercontent.com/69129562/205666966-522ad3c1-e239-427e-b006-a3aa26efdece.png)
preHandle : 컨트롤러 호출 전에 호출된다. (더 정확히는 핸들러 어댑터 호출 전에 호출된다.)<br/>
preHandle 의 응답값이 true 이면 다음으로 진행하고, false 이면 더는 진행하지 않는다. false<br/>
인 경우 나머지 인터셉터는 물론이고, 핸들러 어댑터도 호출되지 않는다. 그림에서 1번에서 끝이
나버린다.<br/>
postHandle : 컨트롤러 호출 후에 호출된다. (더 정확히는 핸들러 어댑터 호출 후에 호출된다.)<br/>
afterCompletion : 뷰가 렌더링 된 이후에 호출된다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205667169-5500bddb-2b38-4733-bdee-56746d6eb3ff.png)
예외가 발생시<br/>
preHandle : 컨트롤러 호출 전에 호출된다.<br/>
postHandle : 컨트롤러에서 예외가 발생하면 postHandle 은 호출되지 않는다.<br/>
afterCompletion : afterCompletion 은 항상 호출된다. 이 경우 예외( ex )를 파라미터로 받아서 어떤
예외가 발생했는지 로그로 출력할 수 있다.<br/>
afterCompletion은 예외가 발생해도 호출된다.<br/>
예외가 발생하면 postHandle() 는 호출되지 않으므로 예외와 무관하게 공통 처리를 하려면
afterCompletion() 을 사용해야 한다.<br/>
예외가 발생하면 afterCompletion() 에 예외 정보( ex )를 포함해서 호출된다.<br/>
정리<br/>
인터셉터는 스프링 MVC 구조에 특화된 필터 기능을 제공한다고 이해하면 된다. 스프링 MVC를 사용하고,
특별히 필터를 꼭 사용해야 하는 상황이 아니라면 인터셉터를 사용하는 것이 더 편리하다.<br/>

![image](https://user-images.githubusercontent.com/69129562/205931308-0a906612-0fde-40e2-827f-ff1bdfb54077.png)
request.setAttribute(LOG_ID, uuid)<br/>
서블릿 필터의 경우 지역변수로 해결이 가능하지만, 스프링 인터셉터는 호출 시점이 완전히 분리되어
있다. 따라서 preHandle 에서 지정한 값을 postHandle , afterCompletion 에서 함께 사용하려면
어딘가에 담아두어야 한다.<br/> LogInterceptor 도 싱글톤 처럼 사용되기 때문에 맴버변수를 사용하면
위험하다. 따라서 request 에 담아두었다. 이 값은 afterCompletion 에서
request.getAttribute(LOG_ID) 로 찾아서 사용한다.<br/>
![image](https://user-images.githubusercontent.com/69129562/205931907-bf48389e-14f3-4eed-bd52-19c5111e9df4.png)
HandlerMethod<br/>
핸들러 정보는 어떤 핸들러 매핑을 사용하는가에 따라 달라진다. 스프링을 사용하면 일반적으로
@Controller , @RequestMapping 을 활용한 핸들러 매핑을 사용하는데, 이 경우 핸들러 정보로
HandlerMethod 가 넘어온다.<br/>
ResourceHttpRequestHandler<br/>
@Controller 가 아니라 /resources/static 와 같은 정적 리소스가 호출 되는 경우
ResourceHttpRequestHandler 가 핸들러 정보로 넘어오기 때문에 타입에 따라서 처리가 필요하다.
