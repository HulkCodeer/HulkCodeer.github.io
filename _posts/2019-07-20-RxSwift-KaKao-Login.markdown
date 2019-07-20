---
layout: post
title:  "RxSwift로 프로젝트에 이용해보기!"
date:   2019-07-20
description: RxSwift를 이용해서 프로젝트를 진행 하였습니다. 간단하게 로그인에만 적용했고, 차츰 다른 기능에도 적용해 나가 볼까 합니다.!!
---

 오늘은 RxSwift를 KaKao 로그인에 붙인 이야기 합시다!!
 
 Swfit를 처음 접하고 한달 뒤에 바로 힘난다라는 프로젝트를 하게 되었다.
 
 힘난다 프로젝트는 웹뷰 기반의 오더 플랫폼이다.
 
 기존 모든 프로젝트가 Objective-C로 되어 있었는데, 전임자 분께서
 
 Swift로 만들다가 도중에 퇴사를 하게 되셨다. 나는 Objective-C로 1년정도의 경력 뿐이였는데...
 
 한달정도 Swift를 공부 후 바로 프로젝트를 맡게 되었다. 아이폰 개발자가 구해지지 않아 계속 미루던 프로젝트라.....
 
 다행히 내게 한달정도의 공부시간이 주어진 것만으로도 다행이다.
 
 Swift를 공부하며 RxSwift도 같이 공부하게 되었고, 간단하게 로그인 기능 하나에만 RxSwift를 붙여보자 했다.
 
 아래는 지정되 API 서버에 푸쉬 토큰 키와, 회원 아이디를 기반으로 기존유저라면 업데이트를 
 
 아니라면 신규유저로 새로운 회원 아이디가 내려오는 API이다.
 
{% highlight swift %}
let fcmToken: String = GlobalShareManager.shared().getLocalData("fcmToken") as! String // 저장된 FCM 토큰
let mbrId: String = GlobalShareManager.shared().getLocalData(SHCEMA_HOST_MBRID) as! String // 저장된 회원아이디

Api.shared
    .request(LMS.UpdateUserInfo(mbrId:mbrId, pshTkn: fcmToken)) // 토큰값만 있을경우 신규 유저, 없을 경우 기존 유저 업데이트 API
    .retry(3) // 3번 재시도
    .subscribe(onSuccess: { (userInfo) in // 회원정보
        if let userMbrId = userInfo.data {
            GlobalShareManager.shared().setLocalData(dataValue: userMbrId, forKey: SHCEMA_HOST_MBRID)
        }
    }) { (error) in
        LogPrint(error) // Custom 로그
    }
    .disposed(by: disposeBag) // 옵저버 해제
{% endhighlight %}

API 관련된 라이브러리는 RxMoya를 썼으며, 추상화된 네트워크 정보가 .request와 LMS.UpdateUserInfo에 정의 되어 있으며

.retry(3)은 rx의 오퍼레이트다. 3번까지 재시도를 할 수 있게 만들어 준다.

.subscribe시 네트워크 통신 성공과 실패에 대한 처리만 필요 하므로 onSuccess의 결과 값만 구독하게 만들었다.

에러가 날 경우 subscribe의 마지막 인자 error 부분에 실패에 대한 로그를 찍도록 만들고,

.disposed는 메모리 해제를 위한 rx 오퍼레이트다.

간단한게 회원정보 업데이트 하는 부분에만 구현했지만, 이정도 구현하는데에도 오랜 시간의 공부가 필요 했다ㅠㅠ

넘나 어려운것....ㅠㅠ 

그래도 계속 다른 기능에 Rx를 추가 해서 만들어 볼 생각이며 최종적으로는 Rx를 활용한

MVVM 구조로 만들어 보는것이 최종 목표이다.
