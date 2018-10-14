# Firebase for Android
- https://www.udacity.com/course/firebase-in-a-weekend-by-google-android--ud0352


# Firebase가 왜 필요한가?
### Chat app case
- DB 서버: 메시지 내용을 저장
- File 서버: 사진같은 것을 저장
- App 서버: DB, File 서버를 중계
- Authentication Platform(인증): 접속자 모니터링/관리
- 통계적 접근 Tool: 가입자 분석, 광고, Push 서비스

### Firebase의 주요지원
- 인증, 권한, Data storage
- Realtime Database: 사용자와 sync를 유지
- Storage: 사용자가 업로드하는 컨텐츠를 저장
- Authentication(인증): 별도의 코드없이 인증을 관리함
- Analytics: 사용 통계 지원
- Notifications: web에서 쉽게 전체/그룹/개인 등으로 골라서 알림
- Remote configurations: 전체/그룹별 업데이트 없이 기능feature를 조절

### Firebase에 앱 등록
- 우선 Firebase에 가입
- Firebase 콘솔 에서 프로젝트 생성
- 앱 패키지명 등록
- Debug signing key의 SHA1 값 등록
    - keytool로 조회해서 SHA1 항목에 나오는 값을 복붙
    - For Mac/Linux
    ```
    $ keytool -exportcert -list -v alias androiddebugkey -keystore ~/.android/debug.keystore
    ```
    - For Windows
    ```
    $ keytool -exportcert -list -v alias androiddebugkey -keystore %USERPROFILE%\.android\debug.keystore
    ```
    - SHA1: SHA 해쉬값을 출력하는 알고리즘으로 미국 NIS 기관에서 만든 표준 중에 한가지
- google-services.json 파일을 받아 앱 모듈에 넣어두기
- gradle 파일에 google-services 빌드 추가 (Firebase 의 가이드를 참조)
    - project gradle에 google service 추가
    ```
    buildscript {
        dependencies {
            classpath 'com.google.gms:google-services:4.0.1'
        }
    }
    
    ```

    - app gradle에 google service 추가
    ```
    apply plugin: 'com.google.gms.google-services'
    ```
- 앱 실행하여 설치 확인
    - google-service만 추가하였기 때문에 설치확인이 되지 않는다. 현재 상태에선 skip.

### Firebase Realtime Database 연동
- gradle에 다음과 같이 추가
    - implementation 'com.google.firebase:firebase-database:16.0.1'
    - Firebase library: https://firebase.google.com/docs/android/setup
    
1. Firebase Realtime Database의 내부구조
- JSON형태로 관리된다.
- Root 하위에 여러개의 각기 다른 이름의 child(table 개념)가 생길 수 있다.
- 또한 child 밑에 또다른 child가 생길 수 있으며
- 각 child 하위에 중복이 생기면 시간+고유값(48bits for timestamp + 72bits for randomness = 120 bits) 개념의 Push ID가 생기고(row id 개념)
- 이 Push ID의 하위에 raw 개념의 key:value를 저장한다.

```
{  
   "questions": {  
      "ABCDakarandomkey": {  
         "question": "Who was the 13th president of the United States?",
         "choice_1": "Millard Fillmore",
         "choice_2": "Zachary Taylor",
         "choice_3": "Franklin Pierce",
         "choice_4" :"James K. Polk",
         "answer" :"choice_1"
      },
      "EFGHakarandomkey": {  
         "question": "In what year was the first gasoline combustion engine invented?",
         "choice_1": "1769",
         "choice_2": "1886",
         "choice_3": "1807",
         "choice_4": "1864",
         "answer": "choice_4"
      }
   },
   "players": {  
      "user_key_1": {  
         "name": "Person",
         "opponents": {  
            "IJKLakarandomkey": "user_key_2",
            "MNOPakarandomkey": "user_key_6"
         },
         "questions": {  
            "ABCDakarandomkey": "Correct",
            "EFGHakarandomkey": "Incorrect"
         }
      },
      "user_key_2": {  
         "name": "Mai",
         "opponents": {  
            "QRAAakarandomkey": "user_key_1",
            "SQUEakarandomkey": "user_key_6"
         },
         "questions": {  
            "ABCDakarandomkey": "Incorrect",
            "EFGHakarandomkey": "Incorrect"
         }
      }
   },
   "opponents": {  
      "couple_Key_1": "user_key_1_user_key_2",
      "user_1": "user_key_1",
      "user_2": "user_key_2",
      "winner": "user_key_1"
   }
}
```

2. Firebase Realtime Database 쓰기, 읽기
```Java
/*
    쓰기
 */
// FirebaseDatabase 객체 초기화
FirebaseDatabase mFirebaseDatabase = FirebaseDatabase.getInstance();
// 특정 child(table 개념)을 레퍼런스 할 수 있게 함
DatabaseReference mMessageDatabaseReference = mFirebaseDatabase.getReference().child("messages");

// 저장하려는 data 객체
FriendlyMessage friendlyMessage = new FriendlyMessage(mMessageEditText.getText().toString(), mUsername, null);
// data를 넣음
mMessageDatabaseReference.push().setValue(friendlyMessage);


/*
    읽기
    - 기존의 것도 추가된 것처럼 들어온다.
 */
ChildEventListener mChildEventListener = new ChildEventListener() {
    @Override
    public void onChildAdded(@NonNull DataSnapshot dataSnapshot, @Nullable String s) {
        // Deserialize the message
        FriendlyMessage friendlyMessage = dataSnapshot.getValue(FriendlyMessage.class);

        mMessageAdapter.add(friendlyMessage);
    }

    @Override
    public void onChildChanged(@NonNull DataSnapshot dataSnapshot, @Nullable String s) {
    }

    @Override
    public void onChildRemoved(@NonNull DataSnapshot dataSnapshot) {
    }

    @Override
    public void onChildMoved(@NonNull DataSnapshot dataSnapshot, @Nullable String s) {
    }

    @Override
    public void onCancelled(@NonNull DatabaseError databaseError) {
    }
};
mMessageDatabaseReference.addChildEventListener(mChildEventListener);
```

4. Firebase Realtime Database 보안 규칙
- root 및 그 이하 하위계층에 접근권한을 준다.
- 규칙 종류
    - ".read": 읽기 권한
    - ".write": 쓰기 권한
    - ".validate": 정해놓은 규격에 맞는 데이터로 쓰는지 확인(write 권한이 있어야 함)
    - ".indexOn": 특정 key를 자동으로 indexing하여 쿼리속도를 향상시킨다고 함
- 규칙 값: true/false 및 조건문에 의한 true/false
    - child까지 상속되어 적용된다.(Cascade)
- 변수들
    - 쓰임새를 좀더 알아보고 정리 필요
    
- 참조 https://firebase.google.com/docs/reference/security/database/
```
{
  "rules": {
    ".read": true,
    ".write": true
  }
}
``` 

### Firebase Authentication(로그인 인증)
1. 목적, 쓰임새
- 로그인단을 지원해주는 솔루션? 학습완료 후 작성...
- 

2. 적용방법
- FirebaseUI SDK dependencies 추가
- 로그인 상태를 감지할 listener 구현
- 로그인을 요청할 화면을 구성


# Firebase Feature Description
1. *Analytics
    - Firebase Analytics is a free and unlimited analytics tool to help you get insight on app usage and user engagement.Check out how PicCollage improves and analyzes their experiments with Firebase Analytics.

### Develop
2. Cloud Messaging
    - Firebase Cloud Messaging lets you deliver and receive messages across platforms reliably. Check out the Firebase Cloud Messaging documentation for more details.

3. *Authentication
    - Firebase Authentication is a key feature for protecting the data in your database and storage. Check out authentication in action on Bobon Profiles to help save the brewing profiles of users.

4. *Realtime Database
    - Firebase Realtime Database lets you sync data across all clients in realtime and remains available when your app goes offline. Check out how Skyscanner uses Firebase Realtime Database to keep clients in sync with the database.

5. *Storage
    - Firebase Storage lets you store and serve user-generated content, such as photos or videos. Firebase Storage is backed by Google Cloud Storage, used and trusted by many.

6. Hosting
    - Firebase Hosting provides fast and secure static hosting. Check out Firechat, one of the many web apps hosted with Firebase Hosting.

7. Test Lab for Android
    - Firebase Test Lab lets you test on Android devices hosted in the cloud. Check out the Firebase Test Lab for Android documentation for more details.

8. Crash Reporting
    - Firebase Crash Reporting provides detailed reports of errors and bugs in your app.Check out how Shazam uses Firebase Crash Reporting to find bugs to improve their user experience.

### Grow
9. *Notifications
    - Firebase Notifications helps you re-engage with users at the right moment.Check out the Firebase customers page to see real-world uses of Firebase Notifications.

10. *Remote Config
    - Firebase Remote Config can change the behavior and appearance of your app without publishing an app update. Check out how Fabulous uses Firebase Remote Config to streamline their processes.

11. App Indexing
    - With Firebase App Indexing, you can drive organic search traffic to your app, helping potential users of your app become your app’s biggest fans. Check out some App Indexing case studies from companies like Etsy and The Guardian.

12. Dynamic Links
    - Firebase Dynamic Links lets you pull users right to the content they were interested in, keeping them engaged and increasing the likelihood that they will continue to use the app. Check out the Firebase Developer Story on how NPR uses dynamic links.

13. Invites
    - Firebase Invites helps your users share your app with others. Check out some case studies on apps using Invites such as Yummly.

14. AdWords
    - AdWords help you search potential users with online ads. Check out the AdWords testimonials page to see how businesses use Google ads.

### Earn
15. AdMob
    - AdMob provides easy and powerful ad monetization with full support in Firebase. Check out the AdMob success stories page to see how businesses use AdMob.



# Firebase 단점 리서치
-
