# rxstudy_essentials
## 도서는 Rx Java Essentials(에이콘 출판사) 로 학습하였습니다. 위 소스 예제는 Rx Java Essentials 셈플 소스에서 개인적인 학습으로 사용되었습니다.
# 기본
## Observable.create()
- call() 함수에서 발행을 구현할 수 있다

<pre><code>
//자바 표현식
Observable.create(new Observable.OnSubscribe<AppInfo>() {
            @Override
            public void call(Subscriber<? super AppInfo> subscriber) {
                subscriber.onNext(new AppInfo(name, iconPath, appInfo.getLastUpdateTime()));
                subscriber.onCompleted();
            }
        });
//람다 표현식
Observable.create(subscriber -> { } );

</code></pre>

## Observable.just()
- 기존 코드를 Obserable로 변환
- 아래 예시는 onNext() 가 3번 불린다.
<pre><code>
        Observable<AppInfo> observable = Observable.just(appOne, appTwo, appThree);
        observable.subscribe(appInfo -> {
                    mAddedApps.add(appInfo);
                    mAdapter.addApplication(mAddedApps.size() - 1, appInfo);
                    L.d("onNext->appInfo=" + appInfo);
                }, error -> {
                    L.d("onError->error=" + error);
                }, () -> {
                    L.d("onCompleted");
                }
        );
</code></pre>

## Observable.repeat(), defer(), range(), interval(), timer()
- 반복, 구독전까지 옵저버블을 미루고 싶을때, X개중 N개 발행시, 3초동안 주기적으로, 3초 후 발행

## 필터링
- filter() 로 리스트 중 원하는 조건일때 바로 발행된다
<pre><code>
Observable.from(apps)
                .filter((appInfo) -> {
                    //true인 경우 onNext() 호출. 아닌경우 onNext()는 호출되지 않음
                    L.d("filter->"+appInfo.getName());
                    return appInfo.getName().startsWith("C");
                })
                .subscribe(new Observer<AppInfo>() {
                    @Override
                    public void onCompleted() {
                        L.d("onCompleted");
                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(AppInfo appInfo) {
                        L.d("onNext->appInfo="+appInfo);
                    }
                });
//아래처럼 null 객체를 필터링하여 구독자는 별도의 null 값을 체크할 필요없이 앱로직만 신경쓰면된다.(분리/추상화)
ArrayList<AppInfo> testList = new ArrayList<>();
        testList.add(null);
        testList.add(new AppInfo("aaaa", "bbbb", 1234));
        Observable.from(testList)
                .filter(new Func1<AppInfo, Boolean>() {
                    @Override
                    public Boolean call(AppInfo appInfo) {
                        L.d("filter->"+(appInfo == null ? "isNull" : "isNotNull"));
                        return appInfo != null;
                    }
                })
                .subscribe(new Observer<AppInfo>() {
                    @Override
                    public void onCompleted() {

                    }

                    @Override
                    public void onError(Throwable e) {

                    }

                    @Override
                    public void onNext(AppInfo appInfo) {
                        L.d("onNext->appInfo="+appInfo);
                    }
                });
</code></pre>

## take(), takeLast()
- 처음 2개 요소만 발행
- 마지막 1개 요소만 발행

##distinct(), distinctUntilChanged()
- 중복 제거
- 이전에 발행했던 값과 다른 새로운 값을 발행한 경우에만 onNext() 호출

## first(), last()
- 첫번째 한번 만
- 마지막 한번 만

## skip(), skipLast()
- skip(2) : 2개를 건너뛴 후 나머지는 발행
- skipLast(2) : 마지막 2개
- 목적에 부합하지 않는 제한해야 할 요소로 시작하거나 끝나는 경우

## elementAt(), sample()
- 특정 번째 요소만 발행시
- 지정한 시간 간격에서 최근 아이템을 발행한 것중 마지막 아이템을 발행하는 옵저버블 생성

## timeout()
- 지정한 시간내에 아무런 값을 바지 못할 경우 에러(onError)
- 타임아웃 후 도작한 아이템은 발행되지 않음

## debounce()
- 옵저버블이 아이템을 발행한 후 일정 시간 동안 발행되지 않은경우 마지막 아이템을 발행한다
- 일정 시간이 끝나기전 아이템이 발행되면 내부 타이머는 재시작된다

## observeOn,subscribeOn
[http://tiii.tistory.com/18](http://tiii.tistory.com/18)
- subscribeOn: observable의 작업을 시작하는 쓰레드 지정
- observeOn: subscribe 의 쓰레드 또는 observeOn 이후에 나오는 오퍼레이터의 스케줄러 지정
- 쓰레드 스케줄러는 위 링크 참고


<!-- 03.24 -->

# Observables 결합
- 여러 개의 소스 Observable들을 하나의 Observable로 만드는 연산자들

## merge()
- 복수 개의 Observable들이 배출하는 항목들을 머지시켜 하나의 Observable로 만든다
- 다중입력 > 단일출력
- 다수의 Observable를 병합하여 이벤트가 발생 된 순서대로 이벤트를 전달, 
  한 Observable에서 error가 발생하면 다른 Observable에서 발생된 이벤트는 전달하지 않음
  > 따라서, mergeDelayError()를 해야 에러를 던지더라도 다른 이벤트 중단하지 않으려면 사용해야함.
  
## zip()
- 각 observable에서 발생한 순번의 것들을 서로 묶어준다.
- 2개의 옵저버블과 Func2로 구성된 3개의 인자를 갖는다.

## join()
- A Observable과 B Observable이 배출한 항목들을 결합하는데, 이때 B Observable은 배출한 항목이 타임 윈도우를 가지고 있고 
  이 타임 윈도우가 열린 동안 A Observable은 항목의 배출을 계속한다. Join 연산자는 B Observable의 항목
  을 배출하고 배출된 항목은 타임 윈도우를 시작시킨다. 타임 윈도우가 열려 있는 동안 A Observable은 자신의 항목들을 계속 배출하여 
  이 두 항목들을 결합한다
  >> timer 2 확인 필요
  
## combineLatest()
https://realm.io/kr/news/rxandroid-3/
- checks1과 testExists 두가지 옵저버블 값을 가져오는데 둘 중 하나의 값이 변경될 때 마다 뒤의 람다 함수 
  (check, exist) -> !check || exist가 호출이 되어 Observable<Boolean>에 데이터가 흘러가게 된다.
- combileLatest를 이용하면 UI가 변경되었을 때 복합적으로 동작하는 UI를 쉽게 다룰 수 있다.

## and() then() when()
http://reactivex.io/documentation/ko/operators/and-then-when.html
- 두 개 이상의 Observable들이 배출한 항목들을 'Pattern'과 'Plan' 중계자를 이용해서 결합한다

## switch()
- Observable들을 배출하는 Observable을 싱글 Observable로 변환하다. 
  변환된 싱글 Observable은 변환 전 소스 Observable들이 배출한 항목들을 배출한다

## startwith()
- 소스 Observable이 항목을 배출하기 전에 다른 항목들을 앞에 추가한 후 배출한다



# 추가 학습 링크
- [https://realm.io/kr/news/rxandroid/](https://realm.io/kr/news/rxandroid/)
-  http://chuumong.tistory.com/entry/RxJava-정리
