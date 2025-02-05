# TouchEn nxKey: 키로깅 방지를 위해 키로깅하는 솔루션

* :kr: *번역상태*: 번역 완료 (2023-01-31)

```
이 글은 저자의 허락을 받아 영문으로 된 원문을 한국어로 번역한 글입니다.
번역 오류 및 잘못된 내용에 대한 수정은 Pull Request를 부탁드려요.
다음 글도 번역에 참여하실 분은 코멘트 남겨주세요. (혼자 다 하려니 쉽지 않네요 ^^)
```

## 번역 관련 정보

- 아래 글 번역에 참여한 이:
  - [@alanleedev](https://github.com/alanleedev)
  - [@MerHS](https://github.com/MerHS)
  - [@KENNYSOFT](https://github.com/KENNYSOFT)
  - [@ilsubyeega](https://github.com/ilsubyeega)
  - [@sh-cho](https://github.com/sh-cho)
  - [@kunggom](https://github.com/kunggom)

- 원본 문서: [TouchEn nxKey: The keylogging anti-keylogger solution](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/) (2023-01-09)
- 번역 문서 최초 작성일: 2023-01-15 (PR을 통해 계속 업데이트 중)

---

일주일 전에 [한국에서 필수적인 소위 보안 프로그램이라 불리는 것](https://palant.info/2023/01/02/south-koreas-online-security-dead-end/)에 대해 글을 썼다. 이 여정은 라온시큐어에서 만든 TouchEn nxKey에 관심을 가지면서 시작되었는데, 왜냐하면 이 브라우저 확장 프로그램은 1,000만 명 이상의 사람이 사용하고 있어서였다. 사실 크롬 웹 스토어에서 보여주는 다운로드 수는 최대 1,000만까지지만, 이건 한국에서 사용되는 대다수의 컴퓨터에 설치되기 때문에 실제 사용자 수는 훨씬 더 많을 것으로 추정된다.

이렇게나 사용자가 많은 것은 사람들이 이 제품을 너무 좋아해서가 아니다. 실제 평가는 매우 좋지 않아서 별점 5점 만점에 평균 1.3점을 받았고, 많은 사용자가 제발 이걸 없애달라고 요청하고 있다. 하지만 한국에서 온라인 뱅킹 등을 하려면 해당 제품의 사용이 필수적이다.

이 소프트웨어를 설치하도록 하는 은행에서는 이걸 통해 보안이 향상된다고 주장한다. 하지만 사용자들은 이것을 [악성 프로그램 혹은 키로거라고 부른다](https://www.reddit.com/r/korea/comments/9qwucv/comment/e8ch6yn/). 내가 시간을 투자해 이 제품의 내부 동작을 분석해 봤는데, 후자의 주장이 진실에 더 가깝다고 판단된다. 이 프로그램에는 실제로 설계상 키로깅 기능이 포함되어 있으며, 외부에서 이 기능에 접근하는 것을 제대로 막고 있지 않다. 또한 단순한 서비스 거부부터 원격 코드 실행에 이르기까지 다양한 범위에 걸친 버그가 있다. 나는 이 제품에서 전부 7가지 보안 취약점을 신고했다.

## 목차

- [배경](#배경)
- [TouchEn nxKey는 실제로 어떻게 동작하는가?](#touchen-nxkey는-실제로-어떻게-동작하는가)
- [웹사이트는 어떻게 TouchEn nxKey와 통신하는가?](#웹사이트는-어떻게-touchen-nxkey와-통신하는가)
- [TouchEn 확장 프로그램을 악용하여 은행 웹사이트 공격하기](#touchen-확장-프로그램을-악용하여-은행-웹사이트-공격하기)
  - [Side-note: TouchEn과 유사한 브라우저 확장기능](#side-note-touchen과-유사한-브라우저-확장-프로그램)
- [웹사이트에서 키로깅 기능 사용하기](#웹사이트에서-키로깅-기능-사용하기)
- [애플리케이션 자체를 공격하기](#애플리케이션-자체를-공격하기)
- [도우미 (helper) 애플리케이션 악용하기](#도우미-helper-애플리케이션-악용하기)
- [드라이버의 키로깅 기능에 직접 접근하기](#드라이버의-키로깅-기능에-직접-접근하기)
  - [Side-note: 드라이버가 죽다 (crash)](#side-note-드라이버가-죽다-crash)
- [과연 문제가 수정될까?](#과연-문제가-수정될까)
  - [Side-note: 정보 유출 (information leak)](#side-note-정보-유출-information-leak)
- [nxKey의 동작 방식이 효과가 있긴 할까?](#nxkey의-동작-방식이-효과가-있긴-할까)

## 배경

[한국의 현재 상황에 대해서 요약한 글](https://palant.info/2023/01/02/south-koreas-online-security-dead-end/)을 쓴 후, 다양한 한국 웹사이트에서 내 글에 관해서 토론하기 시작했다. [특히 댓글 하나](https://www.clien.net/service/board/news/17827726?c=true#140131396)에서 내가 놓쳤던 꽤 중요한 정보를 발견했는데, 2005년에 한국 외환은행에서 발생했던 해킹 사건이었다. [[1]](https://news.kbs.co.kr/news/view.do?ncd=735696) [[2]](https://news.kbs.co.kr/news/view.do?ncd=735697) 이 기사는 기술적인 내용에 대해서 자세히 서술하지 않았지만, 이 사건에 대해 내가 이해한 바를 설명하도록 하겠다.

> 역주: 2005년 5월 10일, 외환은행에서 인터넷 뱅킹 해킹 사건이 발생했습니다. 범인은 4인조 일당으로, 재테크 관련 Daum 카페 게시물에 악성코드를 숨겨놓아 게시물을 클릭하면 NetDevil이라는 트로이 목마가 피해자의 PC에 설치되게끔 하였습니다. 그런 다음 해당 악성코드의 키로거 기능을 통해 피해자의 인터넷 뱅킹 인증용 정보를 탈취하여 계좌에서 5,000만 원을 빼돌렸습니다.
> 처음에 해당 은행에서는 피해 보상을 거부했으나, 금융감독원 조사 결과 윈도우 95 및 98 환경에서는 은행에서 제공하는 “보안” 프로그램이 실행되고 있어도 해당 키로거를 막지 못한다는 점이 드러나자 결국 여론에 떠밀린 은행이 피해를 전액 보상했습니다. 이는 한국에서 사용자의 중대 과실이 없는 것으로 판정된 첫 번째 인터넷 뱅킹 해킹 사건이었습니다.
> 그러나 정부와 금융권의 [후속 대책](https://m.boannews.com/html/detail.html?idx=79860)은 오히려 은행이 제공하는 “보안” 프로그램 사용을 의무화하는 식으로 흘러갔고, 이후 발생한 인터넷 뱅킹 해킹 사건의 피해자들은 도리어 아무런 보상도 받지 못하게 되었습니다.

이 사건은 2005년 당시 한국에선 상당히 큰 사건이었던 것으로 보인다. 사이버 범죄조직이 원격 접속 트로이 목마([Remote Access Trojan](https://www.malwarebytes.com/blog/threats/remote-access-trojan-rat))를 사용해서 고객의 은행 계좌에서 5,000만원(당시 기준으로 약 5만 달러)을 빼돌렸다. 그들은 이 방법으로 고객의 로그인 정보뿐만 아니라 보안카드에 대한 정보도 같이 얻을 수 있었다. 내가 알기로 보안카드는 EU(유럽연합)에서 2차 인증 수단으로 사용하던 indexed TAN과 유사한데, EU에서는 트로이 목마를 통해 쉽게 뚫린다는 정확히 그 이유로 2012년에 폐기되었다.

고객의 컴퓨터는 어떤 경로를 통해 악성 프로그램에 감염되었을까? 설명된 내용을 보면, 브라우저의 보안 취약점을 공략하는 악성 웹사이트에 방문했을 때 감염되는 드라이브 바이 다운로드([drive-by download](https://en.wikipedia.org/wiki/Drive-by_download)) 공격이 사용된 것으로 보인다. 아니면 사용자가 직접 어떤 프로그램을 설치하도록 속였을 수도 있다. 사용된 브라우저의 이름은 명시되어 있지 않지만, 인터넷 익스플로러일 것이 뻔하다. 당시 한국에서 다른 브라우저는 거의 사용되지 않았기 때문이다.

위 기사는 고객이 온라인 뱅킹 로그인 정보를 분실하거나 누구에게 준 적이 없으므로, 고객은 잘못한 것이 없다고 강조하고 있다. 대신 온라인 뱅킹 자체의 전반적인 안전성에 대해서 의문을 제시하고, 은행이 충분한 보안 조치를 구현하지 않았다고 비판하였다.

2005년 당시에는 다른 나라에서도 유사한 이야기들이 많이 나돌았다. 오늘날에도 이러한 문제가 완전히 사라졌다고 말하기는 어렵지만, 이제는 훨씬 찾아보기 어렵다. 웹 브라우저의 보안이 많이 향상되기도 했지만, 한편으로는 은행에서 더 나은 2차 인증 방법을 도입했기 때문이다. 적어도 유럽에서는 2차 인증 장치가 있어야 거래를 완료할 수 있으며, 거래 승인 시에 거래의 세부 내용을 볼 수 있기 때문에 실수로 악성 거래를 승인하기는 힘들다.

하지만 분노한 사람들이 빠른 해결책을 요구했고, 한국은 다른 길을 선택했다. 위의 두 번째 기사가 당시의 문제점을 조명하고 있는데, 보안 애플리케이션을 통해 공격을 막을 수는 있지만 사용자는 이를 필수적으로 실행할 필요는 없었고 은행은 이를 허용했다. 결과적으로 은행은 “해킹 방지” 애플리케이션을 사용자에게 제공하고, 고객들에게 이를 필수적으로 실행하도록 하기로 하였다.

그렇기 때문에 TouchEn Key에 대한 첫 번째 정보가 2006~2007년도부터 나오기 시작하는 것은 결코 우연이 아닌 것으로 보인다. TouchEn Key는 취급에 주의가 필요한 정보를 웹 페이지에 입력할 때 이를 보호해 줄 수 있다고 주장한다. 시간이 지나 마이크로소프트 유래의 브라우저가 아닌 타사 브라우저에 대한 지원이 추가되었고, 내가 살펴본 제품도 이것이다.

## TouchEn nxKey는 실제로 어떻게 동작하는가?

대중에 공개된 TouchEn nxKey에 대한 모든 자료에서는 이 제품이 키로거를 방지하기 위해 모종의 방법으로 키보드 입력을 암호화한다고 주장한다. 내가 찾을 수 있는 기술적인 내용은 이게 다였다. 그래서 동작 방식을 스스로 알아내야 했다.

TouchEn nxKey에 의존하는 웹사이트는 nxKey SDK를 사용하는데, 이는 웹사이트에서 실행되는 자바스크립트 코드와 서버에서 실행되는 코드 2가지로 나뉘어져 있다. 동작 방식은 다음과 같다.

1. 사용자가 nxKey SDK를 사용하는 웹사이트에서 패스워드 필드에 패스워드를 입력한다.
2. nxKey SDK 내의 자바스크립트 코드가 이를 인식하고 로컬에서 실행 중인 nxKey 애플리케이션에 알려준다.
3. nxKey 애플리케이션이 윈도우 커널 상의 디바이스 드라이버를 활성화한다.
4. 이제부터 디바이스 드라이버가 모든 키보드 입력을 가로챈다. 키보드 입력은 시스템에서 처리되는 대신 nxKey 애플리케이션으로 전송된다.
5. nxKey 애플리케이션은 키보드 입력을 암호화하여 nxKey SDK의 자바스크립트 코드로 보낸다.
6. 자바스크립트 코드는 암호화된 데이터를 숨겨진 폼의 필드에 입력한다. 실제 암호 입력란에는 더미 텍스트만이 입력된다.
7. 사용자가 로그인 정보 입력을 마치고 “로그인” 버튼을 클릭한다.
8. 암호화된 키보드 입력 정보가 다른 데이터와 함께 서버로 보내진다.
9. 서버상에서 동작하는 nxKey SDK는 입력 정보를 복호화해 평문으로 된 패스워드를 얻는다. 이제 일반적인 로그인 절차로 넘어간다.

그러니까 이론적으로는, 웹사이트에 입력된 데이터를 가로채려는 키로거는 암호화된 데이터만 볼 수 있다는 것이다. 웹사이트가 사용하는 공개키를 볼 수는 있지만, 거기에 상응하는 비공개키를 얻을 수는 없다. 그러니까 복호화는 불가능하고 패스워드는 안전할 것이다.

그렇다. 이론적으로는 꽤 괜찮은 아이디어이다.

## 웹사이트는 어떻게 TouchEn nxKey와 통신하는가?

웹사이트는 어떻게 특정 애플리케이션이 컴퓨터에 설치가 되어 있는지 알 수 있을까? 그리고 어떻게 그것과 통신하는 것일까?

이와 관련되어서는 패러다임(역자 주: 구현 혹은 동작 방식)의 변화가 일어나고 있는 것처럼 보인다. 원래 TouchEn nxKey는 자체 브라우저 확장 프로그램을 설치하는 것이 필수였다. 브라우저 확장 프로그램이 웹사이트의 요청 내용을 [native messaging](https://developer.mozilla.org/en-US/docs/Mozilla/Add-ons/WebExtensions/Native_messaging)을 사용해서 애플리케이션에 전달하고, 그 응답 내용을 다시 웹사이트로 되돌려 주었다.

하지만 중간 매개체로 브라우저 확장 프로그램을 사용하는 것은 더 이상 최신 방식이 아니다. 요즘 방식은 웹사이트가 웹소켓 API([WebSockets API](https://developer.mozilla.org/en-US/docs/Web/API/WebSockets_API))를 사용해 애플리케이션과 직접 통신하는 것이다. 더 이상 브라우저 확장 프로그램이 필요하지 않다.

![busanbank.co.kr 웹사이트는 `touchenex_nativecall()`을 사용해서 TouchEn 브라우저 확장 프로그램과 통신하는 것으로 보인다. 이 확장 프로그램은 Native Messaging을 통하여 `CrossEXChrome`이라는 애플리케이션과 통신한다. 다른 한편으로 citibank.co.kr 웹사이트는 웹소켓 주소 `127.0.0.1:34581`을 사용해 `CrossEXService` 애플리케이션과 직접 통신한다.](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/website_communication.png)

이런 동작 방식의 변화가 언제 발생했는지 모르겠지만, 아직 완성과는 거리가 멀다. 한국씨티은행 같은 몇몇 웹사이트의 경우 웹소켓을 사용한 방식만 쓰고 있지만, 부산은행과 같은 다른 웹사이트는 아직 브라우저 확장 프로그램에 의존하는 예전 방식을 유지하고 있다.

이것은 사용자가 여전히 브라우저 확장 프로그램을 설치해야 한다는 것만을 의미하지 않는다. 소프트웨어를 설치해도 제대로 인식이 되지 않는다는 잦은 불만에 관해서도 설명하고 있는 것이다. 이러한 사용자들은 바로 웹소켓 통신을 지원하지 않는 옛날 버전의 소프트웨어를 설치한 것이다. 자동 업데이트 기능도 없다. 일부 은행은 아직도 옛날 버전을 다운로드하도록 하고 있는데, 나도 처음에 잘 모르고 옛날 버전을 다운로드했다.

## TouchEn 확장 프로그램을 악용하여 은행 웹사이트 공격하기

TouchEn 브라우저 확장 프로그램은 정말 작고 최소한의 기능만 있다. 그렇기 때문에 악용할 소지가 별로 없어야 하지만, 코드를 보면 다음과 같이 주석 처리된 내용이 있다.

```js
result = JSON.parse(result);
var cbfunction = result.callback;

var reply = JSON.stringify(result.reply);
var script_str = cbfunction + "(" + reply + ");";
//eval(script_str);
if(typeof window[cbfunction] == 'function')
{
  window[cbfunction](reply);
}
```

어떤 사람이 무언가를 하기 위해 아주 끔찍한 방식(즉, 위험하다는 뜻)으로 설계했다. 그러고 나서 `eval()`을 사용하지 않고도 기능 구현이 가능한 것을 깨달았거나, 혹은 누군가 그 부분을 지적한 것으로 추정된다. 하지만 나쁜 코드를 삭제하는 대신에, 혹시 모른다는 이유로 주석 처리만 했다. 내가 보기에, 이건 솔직히 말해서 자바스크립트, 보안, 버전 관리에 관해 아주 잘못 이해했다고 밖에 해석할 수가 없다. 나만 그런 건지도 모르겠지만, 나라면 엄격한 감독 없이는 이 코드를 작성한 사람이 보안 관련 제품을 만들지 못하도록 할 것이다.

어쨌든 위험한 `eval()` 호출은 이미 브라우저 확장 프로그램에서 제거되었다. 은행 웹사이트에서 사용하는 nxKey SDK의 자바스크립트 부분은 그리 많지 않아서, 아직 큰 문제가 발생하지는 않았다. 하지만 이렇게 낮은 품질의 코드를 보면, 다른 문제들도 더 많을 것으로 예상된다.

그리고 역시나 유사한 문제를 콜백 처리 방식에서도 발견하였다. 웹사이트에서는 이벤트 등록을 위해 애플리케이션에 `setcallback` 요청을 호출할 수 있다. 이벤트가 발생하면 애플리케이션은 확장 프로그램에 해당 페이지에 등록된 콜백 함수를 호출하도록 지시한다. 간단히 말하면 해당 페이지에 있는 어떤 글로벌 함수도 이름으로 호출할 수 있다.

그러면 악성 웹사이트가 다른 웹사이트를 대신해 콜백을 등록할 수 있을까? 이것을 위해서는 2가지 장애물이 있다.

1. 목표 웹페이지가 `id="setcallback"`을 포함한 엘리먼트(element)를 가지고 있어야 한다.
2. 콜백은 특정 탭으로만 전달된다.

첫 번째 장애물은 기본적으로 nxKey SDK를 사용하는 웹사이트만 공격받을 수 있다는 의미이다. 브라우저 확장 프로그램을 통해서 통신할 때는 이것을 통해 필요한 엘리먼트를 생성한다. 웹소켓을 사용하여 통신할 때는 이 엘리먼트가 생성되지 않으므로, 최신 nxKey SDK를 사용하는 웹사이트는 영향을 받지 않는다.

두 번째 장애물은 현재 브라우저 탭에서 로딩된 페이지(예를 들면 프레임 안에 로딩된 페이지)만 공격할 수 있다는 뜻이다. 만약에 nxKey 애플리케이션을 속여서 응답에 잘못된 `tabid` 값을 설정하지 않는다면 말이다.

이 공격방식은 놀라울 정도로 쉬웠다. 애플리케이션은 제대로 된 JSON 파서를 사용해 입력되는 데이터를 처리하지만, 응답은 [`sprintf_s()`](https://learn.microsoft.com/en-us/cpp/c-runtime-library/reference/sprintf-s-sprintf-s-l-swprintf-s-swprintf-s-l)를 호출해서 생성한다. 이스케이프 처리 같은 건 전혀 없다. 그렇기 때문에 응답에 포함된 일부 속성(property)을 조작해서 따옴표를 추가하면 임의의 JSON 속성을 삽입할 수 있다.

```js
touchenex_nativecall({
  …
  id: 'something","x":"y'
  …
});
```

이 `id` 속성은 애플리케이션의 응답 내용에 복사된다. 즉, 응답 내용에 원래 없던 JSON 속성인 `x`가 갑자기 추가된다는 뜻이다. 이 취약점을 통해서 `tabid`에 아무 값이나 넣을 수 있다.

그렇다면 악성 웹페이지는 은행 웹사이트를 접속한 브라우저 탭을 어떻게 알 수 있을까? 자신의 tab ID(TouchEn 확장 프로그램에서 노출해 줌)를 사용해서 다른 tab ID를 추측해볼 수 있다. 아니면 그냥 공란으로 남길 수도 있다. 이럴 경우 브라우저 확장 프로그램이 의도치 않은 도움을 준다.

```js
tabid = response.response.tabid;
if (tabid == "")
{
  chrome.tabs.query({active: true, currentWindow: true}, function(tabs) {
    chrome.tabs.sendMessage(tabs[0].id, response, function(res) {});
  });
}
```

따라서 `tabid`의 값이 비어 있다면 현재 활성화된 탭에 메시지를 전달한다.

이것으로 다음과 같은 공격이 가능할 수 있다.

1. 은행 웹사이트를 새 탭에서 연다. 그러면 그것이 활성화된 탭이 된다.
2. 해당 페이지가 로딩될 때까지 기다리면, `id="setcallback"` 엘리먼트가 생겨난다.
3. TouchEn 확장 프로그램을 통해 `setcallback` 메시지를 보내어 어떤 함수를 콜백으로 등록하면서, 동시에 `"tabid":""`와 `"reply":"malicious payload"`라는 JSON 속성을 덮어쓴다.

첫 번째 콜백 호출은 즉시 실행된다. 따라서 은행 웹사이트가 콜백 함수를 호출하면서 그 매개변수로 악성 페이로드가 포함된 `reply` 속성을 사용하게 된다.

거의 끝까지 왔으니 조금만 더 따라오시라. 실행할 수 있는 콜백 함수 중에 `eval`이 있지만, 마지막 장애물이 있다. TouchEn은 `reply` 속성을 콜백으로 넘기기 전에 해당 속성을 `JSON.stringify()`로 처리한다. 그래서 실제로는 `eval("\"malicious payload\"")`이 실행되니까 아무런 동작도 하지 않는다.

그런데 만약에 목표 페이지가 jQuery를 사용한다면? `$('"<img src=x onerror=alert(\'Hi,_this_is_JavaScript_code_running_on_\'+document.domain)>"')`을 호출하면 원하는 결과를 얻을 수 있다.

![gbank.busanbank.co.kr says: Hi,_this_is_JavaScript_code_running_on_busanbank.co.kr](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/xss.png)

공격이 성공하려면 jQuery가 있어야 한다는 가정은 비현실적일까? 그렇지 않다. TouchEn nxKey를 사용하는 웹사이트는 TouchEn Transkey(화상 키보드)도 같이 사용할 가능성이 높은데, 이 물건은 jQuery를 사용하고 있다. 전반적으로 한국의 모든 은행 사이트는 jQuery에 대한 의존도가 매우 높아 보이는데, 이는 [좋지 않다(bad idea)](https://palant.info/2020/03/02/psa-jquery-is-bad-for-the-security-of-your-project/).

하지만 nxKey SDK에서 사용하는 지정 콜백인 `update_callback` 또한 JSON 문자열로 변환된 임의의 자바스크립트 코드를 실행하는 데 악용할 수 있다. 다음과 같이 `update_callback('{"FaqMove":"javascript:alert(\'Hi, this is JavaScript code running on \'+document.domain)"}')` 코드를 호출할 경우 `javascript:` 링크로 리다이렉트 시도를 하며, 그 부수효과로 임의의 코드를 실행할 수 있다.

![gbank.busanbank.co.kr says: Hi, this is JavaScript code running on busanbank.co.kr](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/xss2.png)

이 공격으로 악성 웹사이트가 TouchEn 확장 프로그램을 사용하는 임의의 웹사이트를 탈취(compromise)할 수 있다. 그리고 한국의 은행에서 필수적으로 설치하도록 하는 어떤 “보안” 애플리케이션도 이 공격을 감지하거나 막을 수 없다.

### Side-note: TouchEn과 유사한 브라우저 확장 프로그램

내가 테스트를 시작했을 무렵, 크롬 웹스토어에는 TouchEn 확장 프로그램이 2개가 있었다. 대체적으로 유사하지만, 사용자가 더 적은 확장기능은 그 뒤 삭제되었다.

이것이 이 이야기의 끝은 아니다. 나는 이것과 거의 동일한 확장 프로그램 3개를 더 발견했다. 이니세이프(INISAFE)에서 만든 CrossWeb EX와 Smart Manager EX, 그리고 이니라인(iniLINE)이 만든 CrossWarpEX가 있다. 이 중에서 가장 인기 있는 것은 CrossWeb EX로, 현재 무려 400만 명 이상의 사용자가 있는 것으로 나타난다. 이러한 확장 프로그램들도 유사하게 웹사이트를 공격에 노출시킨다.

나는 처음에 라온시큐어와 이니세이프가 같은 그룹에 속한 계열사인 줄 알았다. 하지만 그런 것 같아 보이지는 않는다.

그러고 나서 iniLINE 소프트웨어 개발사의 [이 페이지](http://www.iniline.co.kr/about/about_04.jsp)를 보게 되었다.

![이니테크 및 라온시큐어의 로고를 보여주는 페이지](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/partners.png)

이 목록을 보면 이니테크와 라온시큐어가 파트너로 명시되어 있다. 이걸 봐서는, 문제가 되는 브라우저 확장 프로그램의 개발사는 이니라인으로 보인다. 또 다른 흥미로운 사실은, 맨 위의 “주요고객사” 중 첫 번째가 바로 국방부라는 것이다. 적어도 국방과 관련된 업무의 결과물은 다른 파트너들에 건네주는 코드보다는 더 높은 품질이길 바랄 뿐이다.

## 웹사이트에서 키로깅 기능 사용하기

만약 악성 웹사이트가 하나 있다고 치자. 그리고 그 웹사이트에서 TouchEn nxKey에 다음과 같이 이야기한다고 치자. “안녕? 지금 사용자가 패스워드 필드에 뭘 쓰고 있네. 거기에 입력되는 데이터 좀 알려줘.” 그렇다면 그 웹사이트가 모든 키보드 입력을 가로챌 수 있을까?

당연히 그렇다! 지금 어떤 브라우저 탭이 활성화되어 있는지, 아니면 아예 브라우저 자체가 활성화되어 있는지에 관계없이 사용자가 입력하는 모든 타이핑 내용을 가져갈 수 있다. nxKey 애플리케이션은 단순히 요청을 그대로 들어줄 뿐, 그 요청이 말이 되는지는 전혀 확인하지 않는다. 사실 [UAC 프롬프트](https://en.wikipedia.org/wiki/User_Account_Control)에 입력된 관리자 패스워드까지 고스란히 넘겨줄 수 있다.

하지만 넘어야 할 장애물이 있긴 하다. 먼저, 해당 웹사이트는 유효한 라이선스가 필요하다. 애플리케이션의 기능을 사용하기 전에 먼저 `get_versions`를 호출하여 그 라이선스를 전달해야 한다.

```js
socket.send(JSON.stringify({
  "tabid": "whatever",
  "init": "get_versions",
  "m": "nxkey",
  "origin": "https://www.example.com",
  "lic": "eyJ2ZXJzaW9uIjoiMS4wIiwiaXNzdWVfZGF0ZSI6IjIwMzAwMTAxMTIwMDAwIiwicHJvdG9jb2xfbmFtZSI6InRvdWNoZW5leCIsInV1aWQiOiIwMTIzNDU2Nzg5YWJjZGVmIiwibGljZW5zZSI6IldlMkVtUDZjajhOUVIvTk81L3VNQXRVd0EwQzB1RXFzRnRsTVQ1Y29FVkJpSTlYdXZCL1VCVVlHWlY2MVBGdnYvVUJlb1N6ZitSY285Q1d6UUZWSFlCcXhOcGxiZDI3Z2d0bFJNOUhETzdzPSJ9"
}));
```

이 라이선스는 `www.example.com`에서만 유효하다. 그러므로 `www.example.com` 웹사이트에서만 사용이 가능하다. 아니면 `www.example.com`이라고 주장하는 웹사이트이거나.

위 코드에서 `origin` 속성이 보이는가? TouchEn nxKey는 HTTP 헤더의 `Origin`을 확인하는 대신, 위에서처럼 주어진 값을 그대로 받아들인다. 그래서 nxKey를 사용하는 어떤 웹사이트에서 라이선스를 가져와서 마치 그 웹사이트인 것처럼 행세할 수 있다. 가짜 라이선스를 만들 필요조차 없다.

또 다른 장애물 하나. 악성 웹사이트가 받게 되는 데이터는 암호화되어 있지 않나? 그걸 어떻게 복호화할 수 있을까? 공격자 자신이 비공개 키를 가지고 있는 별도의 공개 키를 사용할 수 있을 것이다. 그렇다면 암호화 알고리즘만 알면 데이터를 복호화할 수 있다.

하지만 사실 이 중 어떤 것도 필요하지 않다. TouchEn nxKey는 공개 키를 받지 못하면 그냥 암호화 자체를 전혀 하지 않는다. 그러면 악성 웹사이트는 키보드 입력 내용을 암호화되지 않은 평문(clear text)으로 받을 수 있다.

내가 만든 개념증명(PoC: Proof of Concept) 페이지를 보라. (HTML 상용구 코드를 포함해도 3 kB 미만)

![웹페이지 스크린샷: 이 페이지는 당신이 다른 애플리케이션으로 입력하는 것을 다 알 수 있다! 아무 프로그램에서나 입력한 내용이 여기 뜨는 것을 확인해 보라: I AM TYPING THIS INTO A UAC PROMPT](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/typing_page.png)

이 취약점의 심각성을 줄이는 세 번째 장애물이 있다. 악성 웹페이지에서 키보드 입력을 가로채면 그 내용은 원래 목적지에 도달하지 못한다. 사용자가 패스워드를 입력하기 시작했는데 텍스트 필드에 아무것도 표시되지 않으면 뭔가 의심을 할 것이다. 내가 nxKey 애플리케이션을 분석한 결과 키보드 입력은 악성 웹페이지로 가거나 아니면 원래 목적지로 전달되지만, 절대로 둘 다 동시에 전달되지는 않는다.

## 애플리케이션 자체를 공격하기

이 제품의 자바스크립트 코드를 작성한 사람이 누군지는 몰라도, 이 언어에 별로 능숙하지 않다는 것을 앞에서 이미 살펴봤다. 하지만 이게 코드 작성자의 전문 분야가 C++이기 때문이라면? 개발자가 자바스크립트를 최대한 빨리 벗어나서 모든 동작을 C++ 코드에 맡기려는 이러한 모습은 [예전에도](https://palant.info/2018/11/30/maximizing-password-manager-attack-surface-leaning-from-kaspersky/) 본 적이 있다.

아쉽게도 이건 내가 확인할 수 없는 추측일 뿐이다. 나는 바이너리 코드보다는 자바스크립트를 분석하는 것에 훨씬 익숙하다. 하지만 애플리케이션 자체도 유사한 문제들이 많아 보인다. 사실 C++보다는 C 언어에서 많이 사용하는 기법들을 쓰고 있다. 그래서 수동으로 메모리 관리를 하는 부분이 많다.

앞에서 이미 `sprintf_s()` 사용에 관하여 언급했다. `sprintf_s()`나 `strcpy_s()` 같은 함수의 흥미로운 점은 이것들이 `sprintf()` 또는 `strcpy()`의 “메모리 안전” 버전이라 버퍼 오버플로우를 방지하긴 하지만, 그럼에도 불구하고 여전히 사용하기 까다롭다는 점이다. 만약 충분히 큰 버퍼를 할당하지 않는다면, 이 함수는 잘못된 매개변수 처리기(invalid parameter handler)를 호출한다. 그리고 이 처리기의 기본 동작은 애플리케이션을 죽여버리는(crash) 것이다.

그거 아는가? nxKey 애플리케이션은 버퍼 크기가 충분한지 대부분 확인하지 않는다. 그리고 기본 동작을 변경하지도 않는다. 그래서 매우 큰 값을 집어넣으면 대개 애플리케이션이 그냥 죽어버린다. 버퍼 오버플로우가 생기는 것보다는 애플리케이션이 죽는 것이 그나마 낫지만, 애플리케이션이 죽어버리면 원래 하고자 했던 일을 하지 못한다. 그 결과, 인터넷 뱅킹 로그인 창은 정상 동작하는 것처럼 보이지만 패스워드를 암호화되지 않은 상태로 받게 된다. 그리고 사용자는 입력한 양식을 제출하고 나서 오류 메시지가 뜨기 전까지는 뭔가 잘못되었다는 것을 전혀 눈치챌 수 없다. 이 취약점을 활용하면 서비스 거부 공격([Denial-of-Service attacks](https://owasp.org/www-community/attacks/Denial_of_Service))을 할 수 있다.

다른 사례를 보자. 여러 JSON 파서가 있음에도 불구하고, nxKey 애플리케이션 개발자는 [C 언어로 작성된 파서](https://github.com/json-parser/json-parser/)를 채택했다. 그뿐만 아니라, 2014년 1월경의 특정한 리포지토리 상태를 임의로 가져온 뒤 업데이트를 한 적이 전혀 없다. 이 때문에 [2014년 6월에 수정된 null 포인터 역참조 문제](https://github.com/json-parser/json-parser/commit/dec8f04414d2b4a754b2309147665ef341c5f90b)가 그대로 남아 있다. 그래서 정상적인 JSON 데이터 대신 `]`(닫는 대괄호 하나)를 보내는 것만으로도 애플리케이션이 죽는다. 서비스 거부 공격을 허용하는 또 하나의 취약점인 셈이다.

웹사이트와 연결하기 위한 웹소켓 서버는 어떤가? 이건 OpenSSL을 사용한다. 어떤 버전일까? OpenSSL 1.0.2c 버전이다. 그렇다. 여기 있는 모든 보안 전문가들이 다들 하나같이 한숨을 쉬는 소리가 들리는 듯하다. OpenSSL 1.0.2c는 출시된 지 7년이나 지났다. 사실 1.0.2 브랜치에 대한 지원은 이미 3년 전인 2020년 1월 1일부로 종료되었다. 마지막 릴리즈는 OpenSSL 1.0.2u인데, 이건 그 뒤로 버그와 보안 문제를 수정하는 패치가 18번이나 더 있었다는 뜻이다. 이러한 수정 사항이 nxKey 애플리케이션에는 모두 빠져 있다.

제품이 죽는 것보다 좀 더 흥미로운 내용을 살펴보자. 위에서 언급한 애플리케이션 라이선스는 base64로 인코딩된 데이터이다. 애플리케이션에서는 이것을 디코딩해야 한다. 디코딩 함수는 아래와 유사하다.

```c
size_t base64_decode(char *input, size_t input_len, char **result)
{
  size_t result_len = input_len / 4 * 3;
  if (str[input_len - 1] == '=')
    result_len--;
  if (str[input_len - 2] == '=')
    result_len--;
  *result = malloc(result_len + 1);

  // Decoding input in series of 4 characters here
}
```

이 함수를 어디서 가져왔는지는 모르겠지만, CycloneCRYPTO 라이브러리의 base64 디코더와 상당히 유사하다. 하지만 CycloneCRYPTO는 결과를 미리 할당해둔 버퍼에 쓴다. 그러니 여기의 버퍼 할당 로직은 nxKey 개발자가 직접 추가했을 수도 있다.

그리고 그 로직에 문제가 있다. `input_len`이 항상 4의 배수라고 가정하고 있다. 따라서 `abcd==`와 같은 입력값의 경우 3바이트 크기의 결과가 출력되지만, 위 코드에서는 2바이트의 버퍼만 할당하고 만다.

겨우 1바이트의 힙 오버플로우(heap overflow)를 악용할 수 있는가? 그렇다. Project Zero의 [이 블로그 글](https://googleprojectzero.blogspot.com/2014/08/the-poisoned-nul-byte-2014-edition.html)이나 Javier Jimenez의 [이 글](https://sensepost.com/blog/2017/linux-heap-exploitation-intro-series-the-magicians-cape-1-byte-overflow/)에서 설명하고 있다. 하지만 그 취약점을 공략하는 코드를 작성하는 것은 내 능력치 밖의 일이다.

그 대신에, 내 개념증명 페이지에서는 그냥 무작위로 생성한 라이선스 문자열을 nxKey 애플리케이션에 보내기만 했다. 이것만으로도 애플리케이션을 몇 초만에 죽일 수 있었다. 디버거를 연결해 보면 메모리 손상(corruption)의 증거가 명백히 드러났다. 애플리케이션이 죽은 이유는 가짜 메모리 주소에서 데이터를 읽거나 쓰려고 시도했기 때문이다. 어떤 경우, 이 메모리 주소들은 내 웹사이트가 보낸 데이터로부터 오기도 했다. 따라서 충분한 실력이 있는 사람이 시간을 투자하면 분명 이 취약점을 원격 코드 실행에 악용할 수 있을 것이다.

최신 운영체제는 이러한 버퍼 오버플로우를 악용하여 임의 코드를 실행하기 어렵게 만드는 기능을 갖추고 있다. 하지만 이런 기능도 실제로 쓰지 않으면 아무 소용이 없다. 그런데도 nxKey의 개발자는 이 애플리케이션이 불러오는 DLL 중 2개에서는 ASLR([Address space layout randomization](https://en.wikipedia.org/wiki/Address_space_layout_randomization))을 비활성화해 두었고, 4개의 DLL에서는 DEP([Data Execution Prevention](https://learn.microsoft.com/en-us/windows/win32/memory/data-execution-prevention))를 꺼 두었다.

## 도우미 (helper) 애플리케이션 악용하기

여기까지는 웹 기반 공격에 관한 내용이었다. 하지만 악성 프로그램이 시스템에 이미 침투했고, 거기서 좀 더 높은 권한을 얻고자 시도하고 있는 경우에는 과연 어떨까? TouchEn nxKey는 이런 악성 프로그램을 막기 위한 애플리케이션치고는 놀라울 정도로 자신이 존재하는 목적에 도움이 안 된다.

예를 들어, nxKey가 키보드 입력을 가로챌 때마다 `CKAgentNXE.exe`라는 도우미 애플리케이션이 실행된다. 이것의 목적은 nxKey가 키 입력을 처리하지 않을 때 목표 애플리케이션에 그 입력 내용이 전달되게끔 하는 것이다. 메인 애플리케이션에서 사용하는 `TKAppm.dll` 라이브러리의 로직은 대략 다음과 같다.

```c
if (IsAdmin())
  keybd_event(virtualKey, scanCode, flags, extraInfo);
else
{
  AgentConnector connector;

  // An attempt to open the helper’s IPC objects
  connector.connect();

  if (!connector.connected)
  {
    // Application isn’t running, start it now
    RunApplication("CKAgentNXE.exe");

    while (!connector.connected)
    {
      Sleep(10);
      connector.connect();
    }
  }

  // Some IPC dance involving a mutex, shared memory and events
  connector.sendData(2, virtualKey, scanCode, flags, extraInfo);
}
```

nxKey 애플리케이션은 사용자 권한으로 실행되므로, 대부분의 경우 `CKAgentNXE.exe`를 실행하게 될 것이다. 그리고 그 도우미 애플리케이션은 명령 코드 2를 받으면 [`SendInput()`](https://learn.microsoft.com/en-us/windows/win32/api/winuser/nf-winuser-sendinput)을 호출한다.

왜 이런 식으로 만들었는지 이해하는 데는 시간이 좀 필요했다. nxKey나 `CKAgentNXE.exe`나 똑같은 권한 수준으로 실행되고 있는데, 어째서 그냥 직접 `SendInput()`을 호출하지 않는가? 왜 이렇게 우회하는 게 필요한가?

그러다가 `CKAgentNXE.exe`가 IPC 객체의 보안 설명자(security descriptor) 설정을 바꾸어서 무결성 수준(integrity level)이 낮음(Low)인 프로세스의 접근을 허용하게끔 한다는 것을 알게 되었다. 또한 설치 프로그램이 `HKEY_LOCAL_MACHINE\SOFTWARE\Microsoft\Internet Explorer\Low Rights\ElevationPolicy` 밑에 레지스트리 항목을 추가하여 `CKAgentNXE.exe`의 권한을 자동으로 높이게끔 한다는 것도 알게 되었다. 그제야 깨닫게 되었다. 이건 모두 인터넷 익스플로러의 샌드박스(sandbox) 때문이었다.

> 역주: ‘integrity level’이라는 용어는 [Microsoft의 한국어 문서](https://learn.microsoft.com/ko-kr/windows/win32/secauthz/mandatory-integrity-control)에서는 “무결성 수준”으로 직역되어 있지만, [일부 자료](https://www.slideshare.net/hajimaru1/01windows-20150613)에서는 “신뢰도”라고 번역하는 경우도 있습니다. 개인적으로는 “신뢰도”라고 번역하는 쪽이 더 이해하기 쉽다고 생각하지만, 일단 Microsoft 문서의 번역을 추종하였습니다. 이하 다른 용어 또한 다른 이유가 없는 한 Windows 운영체제의 개발사인 Microsoft의 한국어 문서에 나오는 번역을 우선으로 선택합니다.

TouchEn Key가 인터넷 익스플로러에서 ActiveX로 실행될 때는 낮은 무결성 수준으로 실행된다. 이렇게 샌드박스 안에서 갇힌 채 실행될 때는 `SendInput()`을 호출하는 것이 불가능하다. 이 제한을 우회하기 위해 인터넷 익스플로러의 샌드박스에서 `CKAgentNXE.exe`를 실행하고 권한을 자동으로 높일 수 있게 한 것이다. 일단 도우미 애플리케이션이 실행되고 나면, 샌드박스 안에 있는 ActiveX에서 이것과 연결하여 무언가 하게끔 요청할 수 있다. 예를 들면 `SendInput()`을 호출한다던가.

인터넷 익스플로러 밖에서야 이런 방식이 아무런 의미도 없지만, 그런데도 TouchEn nxKey는 계속해서 똑같이 작업을 `CKAgentNXE.exe`에 맡긴다. 그리고 이건 보안에 악영향을 미친다.

낮은 무결성 수준으로 실행된 악성코드(Malware)가 있다고 치자. 브라우저 취약점을 사용해서 침투에 성공하거나 했겠지만, 이제는 샌드박스에 갇히고 말았다. 무엇을 더 할 수 있을까? 그저 `CKAgentNXE.exe`가 실행되기만을 기다리면 (조만간 일어날 가능성 있음) 이걸 써서 탈출할 수 있다!

내가 만든 개념증명 애플리케이션에서는 `CKAgentNXE.exe`에게 다음과 같은 가짜 키보드 입력을 생성하도록 요청하였다. 윈도우 키, 그리고 C, M, D, 엔터 키. 그 결과 명령 프롬프트가 열렸는데, 이건 중간 무결성 수준(기본값)으로 실행된 것이다. 만약에 이게 진짜 악성코드였다면 여기에다가 어떤 명령어를 입력시켜 샌드박스 바깥에서 임의의 코드를 실행했을 것이다.

진짜 악성코드라면 이렇게 눈에 쉽게 띄는 방법을 쓰지는 않을 것이다. `CKAgentNXE.exe`는 명령 코드 5를 받으면 임의의 DLL을 아무 프로세스에나 로드하게 해 주는 기능도 있다. 시스템을 감염시키기에는 이게 더 괜찮은 방법 아닌가?

적어도 이번에는 필수 설치 보안 애플리케이션 하나가 그나마 유용하게 동작해서 위험을 감지했다.

![AhnLab Safe Transaction 애플리케이션에서 'C:\Temp\test.exe' 파일이 'Malware/Win.RealProtect-LS.C5210489' 악성코드에 감염되었다고 경고하는 창](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/antivirus_warning.png)

악성코드를 만드는 자는 어떻게 이 경고가 발생하는지 파악하여 우회할 수 있을 것이다. 아니면 웹소켓 연결을 통해 실제 은행 웹사이트에서와는 달리 안랩 애플리케이션이 실행되지 않는 상태에서 `CKAgentNXE.exe`를 실행시킬 수 있을지도 모른다. 하지만 그렇게 번거롭게 할 필요가 있을까? 이건 그냥 경고 메시지일 뿐이고, 공격을 능동적으로 막지는 않는다. 사용자가 악성코드 치료하기 버튼을 눌렀을 때는 너무 늦었다. 공격은 이미 성공했다.

## 드라이버의 키로깅 기능에 직접 접근하기

위에서 언급한 것처럼 TouchEn nxKey 애플리케이션(드라이버에서 받는 키보드 입력을 암호화하는 녀석)은 사용자 권한으로 실행된다. 높은 권한으로 실행되는 것이 아니므로 특별한 권한이 없다. 그렇다면 드라이버의 기능에 대한 접근은 어떻게 제한할까?

정답은 물론 ‘하지 않는다’이다. 시스템의 어떤 애플리케이션도 이 기능에 접근할 수 있다. 그냥 nxKey가 드라이버와 어떻게 통신하는지만 알면 된다. 궁금해할 것 같아서 말해두자면, 그 통신 프로토콜은 그렇게 많이 복잡하지는 않다.

대체 무슨 생각으로 이렇게 만든 것인지 잘 모르겠다. 드라이버와 통신할 때 쓰는 라이브러리인 `TKAppm.dll`은 Themida를 통해서 난독화(obfuscation)되어 있다. Themida를 만든 업체는 다음과 같이 약속한다.

> Themida®는 SecureEngine® 보호 기술을 사용하여 가장 높은 권한으로 실행될 때 이전에 보지 못한 보호 기법을 구현하여 고도의 소프트웨어 크래킹으로부터 애플리케이션을 보호합니다.

어쩌면 nxKey 개발자들은 이것으로 충분히 리버스 엔지니어링을 막을 수 있다고 생각했을지도 모르겠다. 그러나 런타임에 디버거를 연결해 보니 메모리에 이미 난독화가 해제된(decrypted) `TKAppm.dll`이 올라와 있었기 때문에, 이걸 그대로 저장해서 Ghidra에 띄워 분석할 수 있었다.

> 역주: [Ghidra](https://www.nsa.gov/ghidra)는 미국 정보기관인 NSA에서 개발하여 오픈소스로 공개한 것으로 유명한 소프트웨어 리버스 엔지니어링 도구 모음집입니다. 사이버 보안 분야에서 악성 코드나 소프트웨어 취약점을 분석하기 위해 쓰이고 있습니다.

![TouchEn nxKey에서 띄운 메시지 창. 텍스트 내용: 디버거가 탐지되었습니다. 디버거를 종료하고 다시 시도해 주세요. TouchEn nxKey는 이후 키 입력에서 작동하지 않습니다. (만약 가상머신일 경우 실제 PC로 사용하시기 바랍니다.)](https://palant.info/2023/01/09/touchen-nxkey-the-keylogging-anti-keylogger-solution/debugging.png)

미안하지만 너무 늦었다. 내가 필요한 건 이미 다 가져갔다. 그리고 안전 모드 부팅 때 애플리케이션이 동작을 거부하는 것도 소용없다.

어쨌든 나는 이 드라이버에 연결하여 시스템의 모든 키보드 입력 내용을 가로챌 수 있는 아주 작은 (코드 70줄 미만) 애플리케이션을 만들 수 있었다. 관리자 권한도 필요 없이 사용자 권한만으로도 잘 작동했다. 그리고 앞서 봤던 웹페이지에서의 데모와는 달리, 이 앱은 입력 내용이 원래 목적지로도 전달되도록 할 수 있기 때문에 사용자는 아무것도 눈치챌 수 없다. 키로거 만들기가 이렇게 쉬울 수가 없다!

가장 좋은 점은 이 키로거가 nxKey 애플리케이션과 잘 연동된다는 점이다. 그래서 nxKey가 키보드 입력을 받아 암호화한 데이터를 웹사이트로 보낼 때, 내가 만든 작은 키로거는 동일한 키보드 입력 내용을 암호화되지 않은 평문으로 기록할 것이다.

### Side-note: 드라이버가 죽다 (crash)

커널 드라이버를 개발할 때 알아야 할 점이 하나 있다. 드라이버가 죽으면 시스템 전체가 죽는다. 그래서 드라이버 코드에서는 절대로 오류가 발생하지 않도록 특히 꼼꼼한 확인이 필요하다.

nxKey가 사용하는 드라이버에서 오류가 발생할 수 있을까? 그렇게 자세히 살펴본 것은 아니지만, 우연히 그럴 수 있다는 것을 발견했다. 애플리케이션에서는 [`DeviceIoControl()`](https://learn.microsoft.com/en-us/windows/win32/api/ioapiset/nf-ioapiset-deviceiocontrol)을 사용해서 입력 버퍼의 포인터를 달라고 드라이버에게 요청한다. 그러면 드라이버는 이 포인터 생성을 위해 [`MmMapLockedPagesSpecifyCache()`](https://learn.microsoft.com/en-us/windows-hardware/drivers/ddi/wdm/nf-wdm-mmmaplockedpagesspecifycache)를 호출한다.

그렇다. 이 입력 버퍼는 시스템의 모든 애플리케이션에서 다 볼 수 있다. 하지만 지금 말하고자 하는 건 그게 아니다. 만약 애플리케이션에서 한 번 더 포인터를 달라고 요청하면 어떻게 될까? 드라이버에서는 단순히 `MmMapLockedPagesSpecifyCache()`를 한 번 더 호출한다.

이걸 무한루프로 돌리자, 20초 정도 지난 시점에서 전체 가상 메모리 주소가 소진되어 `MmMapLockedPagesSpecifyCache()`가 `NULL`을 반환했다. 그러자 드라이버가 반환된 값을 확인하지 않았기 때문에 죽어버리고 말았다. 따라서 운영체제가 자동으로 재부팅되었다.

내가 보기에 이 문제점을 악용하는 것은 가능한 것 같지는 않지만 (참고로 나는 바이너리 악용에 관한 전문가가 아니다) 그래도 아주 형편없는 문제점이다.

## 과연 문제가 수정될까?

일반적으로 내가 취약점을 공개할 때는 문제가 이미 수정된 후이다. 그러나 이번에는 안타깝게도 그렇지 않다. 내가 알기로는 지금까지 해결된 문제가 하나도 없다. 제품 공급업체가 언제 이 문제를 고칠 계획인지도 모른다. 특히 은행이 현재 릴리스보다 최소 세 버전 이상 뒤처진 빌드를 배포하고 있다는 점을 고려할 때, 업데이트된 제품을 사용자에게 어떻게 배포할 계획인지도 알 수 없다. 기억하고 있겠지만, 자동 업데이트 기능은 없다.

이러한 문제를 신고하는 것조차 복잡했다. 라온시큐어는 보안 전문 기업임에도 불구하고 보안 관련 연락처를 전혀 기재하고 있지 않았다. 사실 라온시큐어는 서울에 있는 전화번호 하나를 제외하고는 어떠한 종류의 연락처도 공개하고 있지 않다. 나는 한국에 직접 전화해서 거기 영어 할 줄 아는 사람이 있는지 물어볼 생각은 없다.

다행히도 KrCERT는 외국인이 사용할 수 있는 [취약점 신고 양식](https://www.krcert.or.kr/krcert/contact/vulnerability.do)을 제공한다. 이 양식은 종종 오류가 발생하여 모든 내용을 다시 입력해야 하기도 하고, 일부 신고는 명백한 이유 없이 웹 방화벽에 걸리기도 하지만, 적어도 보안 관련 연락처를 찾는 부담은 다른 사람이 떠안게 된다.

내가 찾은 모든 취약점을 2022년 10월 4일에 KrCERT에 신고했다. 그 뒤에도 라온시큐어 경영진에게 직접 연락을 시도했지만, 아무런 답변도 받지 못했다. 일단 KrCERT는 약 2주 후에 내 신고 내용을 라온시큐어에 전달했다고 회신해 줬다. 또한 라온시큐어에서 나에게 연락하기 위해 내 이메일 주소를 물어봤다고도 전해줬다. 하지만 나는 연락받은 적이 없다.

이것이 전부이다. 취약점 신고 후 90일간의 공개 유예 마감일은 이미 일주일 전에 지났다. TouchEn nxKey 1.0.0.78은 내가 취약점을 보고한 날인 2022년 10월 4일에 릴리스된 것으로 보인다. 이 글을 쓰는 현시점에서 가장 최신 버전이지만, 여기에 설명된 모든 취약점이 여전히 존재한다. 수많은 사람이 사용하는 TouchEn 브라우저 확장 프로그램의 최신 버전은 여전히 5년 전 2018년 1월에 출시된 버전 그대로이다.

**업데이트** (2023-01-10): 라온시큐어는 한국 언론에 해당 취약점을 수정했으며, 고객에게 업데이트를 배포했다고 주장했다. 그러나 현 시점에서 이 주장이 사실인지 확인할 수가 없다. 이 회사의 자체 다운로드 서버는 여전히 TouchEn nxKey 1.0.0.78을 배포하고 있다.

### Side-note: 정보 유출 (information leak)

그들이 수정 작업을 하고 있다는 것을 내가 어떻게 알 수 있었을까? 사실 내가 한 번도 겪어보지 못한 일 때문에 가능했다. 그들이 내 개념증명 내용(즉, 해당 취약점에 대한 거의 완전한 익스플로잇)을 공개 유예 마감일 전에 유출했다.

예전에는 신고할 때 파일을 직접 첨부했었다. 하지만 이렇게 하니 보안 소프트웨어가 과하게 열심히 일해서 첨부파일을 지우거나 파괴하는 일이 자주 발생했다. 그래서 지금은 문제를 입증하는 데 필요한 모든 파일을 내 서버에 올려둔다. 내 서버로 연결되는 링크는 항상 동작하니까. 그렇게 하니까 또 다른 장점이 있다. 나에게 직접 연락하지 않는 업체의 경우에도 로그를 보면 해당 개념증명 파일에 접근했는지 알 수 있어서, 내 신고가 제대로 전달이 되었는지 확인할 수 있다.

며칠 전에 TouchEn nxKey 관련 파일에 대한 접근 로그를 확인했다. 첫눈에 구글봇이 눈에 들어왔다. 아니나 다를까, 이 파일들은 구글 검색 엔진에 노출되어 있었다.

나는 무작위로 생성된 폴더 이름을 사용하기 때문에 이걸 그냥 알아낼 수는 없다. 그리고 이 링크는 오직 해당 업체에만 공유한 것이다. 그 말은, 해당 업체가 이 취약점이 담긴 링크를 어딘가 공개된 곳에 올렸을 거라는 것이다.

실제로 그들이 그렇게 한 것이 확인되었다. 나는 아무나 볼 수 있고 심지어 구글에서 검색도 되는 개발 서버를 발견했다. 이 서버는 원래 내가 만든 개념증명 페이지를 링크하고 있었던 것 같다. 하지만 내가 이 서버를 발견했을 때는 해당 업체가 수정한 사본이 호스팅되고 있었다.

구글봇의 첫 번째 접속은 2022년 10월 17일이었다. 따라서 이 취약점은 공개 유예 마감일보다 2개월 이상 전부터 구글에서 검색할 수 있었다고 가정해야 할 것이다. 이 취약점 페이지에 접속이 여러 번 있었기 때문에, 오직 이 제품 개발자만 여기 접속한 것인지는 알기 어렵다.

이 문제를 신고하니까 문제의 개발 서버는 즉시 공개된 인터넷에서 사라졌다. 하지만, 민감한 보안 정보를 이렇게 부주의하게 다루는 경우는 난생처음 본다.

## nxKey의 동작 방식이 효과가 있긴 할까?

TouchEn nxKey 애플리케이션에서 여러 가지 취약점을 살펴보았다. nxKey 개발자는 키로거를 방지한다는 명목으로 완벽한 키로깅 도구를 만든 다음, 정작 거기에 누가 접근하지 못하게 막는 데는 실패했다. 하지만 아이디어는 괜찮지 않은가? 혹시 제대로 만들면 유용한 보안 도구가 될 수도 있지 않을까?

그렇다면 여기서 막으려는 키로거는 과연 어느 수준에서 실행되는 것인가? 내가 보기에는 4가지 가능성이 있다.

1. 브라우저 안에서. 온라인 뱅킹 웹페이지에서 실행된 악성 자바스크립트 코드가 패스워드를 훔치려고 시도하는 경우다. 그러한 코드는 nxKey가 활성화되는 것을 쉽게 막을 수 있다.
2. 시스템 내부에서 사용자 권한으로. 이 권한이면 동일한 사용자 권한으로 실행되고 있는 `CrossExService.exe`를 충분히 죽일 수 있다. 이는 내가 보여줬던 서비스 거부 공격과 동일하게 보호 기능을 무력화하는 것이다.
3. 시스템 내부에서 관리자 권한으로. 이는 nxKey 드라이버를 내려버리고 대신 트로이 목마를 올리기에 충분한 권한이다.
4. 하드웨어에서. 그냥 게임 끝이다. 이걸 소프트웨어적으로 막고자 한다면, 그저 행운을 빈다.

따라서 nxKey가 어떤 보호 기능을 제공하든, 이는 공격자가 nxKey의 존재 및 동작 방식을 전혀 모른다는 가정에만 의존하고 있다. 어쩌면 일반적인 공격은 막을 수 있을지 모르겠지만, 한국의 은행이나 정부 기관을 구체적으로 노린 공격을 막는 데는 그다지 효과적이지 않을 것 같다.

이 4가지 가능성 중, 2번은 *어쩌면* 고칠 수 있을지도 모른다. `CrossEXService.exe` 애플리케이션을 관리자 권한으로 실행되게끔 하는 것이다. 그러면 (사용자 권한으로 실행된) 악성코드는 이 프로세스를 건드릴 수 없다. 하지만 이건 악성코드가 사용자의 브라우저에 침입할 수 없다는 전제하에서만 유효하다.

그리고 다른 가능성의 경우, 이러한 동작 방식으로는 신뢰성 있게 악성코드를 막을 방법이 있는지 도저히 알 수 없다.

---

## 역자 추가 내용

### 원글에 대한 댓글/토론이 있는 웹페이지

- 원글 Hacker News: [TouchEn nxKey: A keylogging anti-keylogger solution](https://news.ycombinator.com/item?id=34307602)
- 원글 Reddit: [TouchEn nxKey: The keylogging anti-keylogger solution](https://www.reddit.com/r/korea/comments/107bpsm/touchen_nxkey_the_keylogging_antikeylogger/)
- 원글 GeekNews: [TouchEn nxKey 취약점 분석](https://news.hada.io/topic?id=8211)
- 번역글 GeekNews: [TouchEn nxKey: 키로깅 방지를 위해 키로깅하는 솔루션](https://news.hada.io/topic?id=8274)
- 개드립 다른 한국어 번역: [(요약있음,강스압,번역) TouchEn nxKey 7개 보안 취약점 발견](https://www.dogdrip.net/456327682)
- 루리웹 원문 요약: [장문) 한국 온라인 보안업체들의 민낯을 적나라하게 까발리고 있는 보안전문가](https://bbs.ruliweb.com/community/board/300143/read/59978508)
- 클리앙: [TouchEn nxKey 보안 취약점 공개되었네요...](https://www.clien.net/service/board/park/17839993)
- PGR21:
  - [TouchEn nxKey 취약점 공개](https://www.pgr21.com/freedom/97663)
  - [TouchEn nxKey 취약점 공개에 대한 개발사의 입장](https://www.pgr21.com/freedom/97695)

### 언론 보도 기사

- 매일경제: [독일 개발자 “한국 은행 사이트, 매우 불편하고 위험해”](https://www.mk.co.kr/news/it/10599751)
- 연합뉴스: [해외개발자, 국내 금융보안시스템 비판…"해킹 못막고 버그도"](https://www.yna.co.kr/view/AKR20230110073700017)
- 이뉴스투데이: [‘취약점’ 저격 당한 국내 보안업계 “국내 환경 이해 부족”](http://www.enewstoday.co.kr/news/articleView.html?idxno=1630290)
