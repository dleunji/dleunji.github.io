---
Title : "Firebase Realtime Database"
layout : post
date : 2021-05-12
category : Internship
blog : true
author : dleunji
description : Firebase
---

그동안 SQL, NoSQL 문을 작성하거나 이론적으로 공부한 게 다인데, 드디어 실제 데이터를 다루어볼 기회가 생겼다. 우리 회사는 ainize 관련된 데이터는 Firebase Realtime Database로 관리한다. Firebase Realtime Database는 대표적인` NoSQL` 데이터베이스로 듣기만 했지, 이렇게 다룰 줄이야 `설렌당 히히♡` 

처음으로 다루는 툴이고 직접 실전에서 써야해서 빠르게 암기하고 익혀야한다. 다행히도 많은 양이 아니니 블로그에 일부 코드를 아카이빙해두고자 한다.

### JSON 트리

모든 Firebase 실시간 데이터베이스 데이터는 JSON 객체로 저장된다. 데이터베이스를 클라우드 호스팅 JSON 트리라고 생각하면 된다. SQL 데이터베이스와 달리 테이블이나 레코드가 없으며, JSON 트리에 추가된 데이터는 연결된 키를 갖는 기존 JSON 구조의 노드가 된다. 사용자 ID 또는 의미 있는 이름과 같은 고유 키로 직접 지정할 수도 있고, [`push()`](https://firebase.google.com/docs/reference/js/firebase.database.Reference#push)를 사용하여 자동으로 지정할 수도 있다.

```json
{
  "users": {
    "alovelace": {
      "name": "Ada Lovelace",
      "contacts": { "ghopper": true },
    },
    "ghopper": { ... },
    "eclarke": { ... }
  }
}
```



## 데이트베이스 구조화

### 데이터 구조 권장사항

- 데이터 중첩 배제
  - 중첩을 허용하나 특정 위치에서 데이터를 가져오면 모든 하위 노드가 함께 검색된다.
  - 특정 노드에 대한 읽기 또는 쓰기 권한을 부여하면 해당 노드에 속한 모든 데이터에 대한 권한이 함께 부여된다.
  - 실제 구현에서는 데이터 구조를 최대한 평면화하는 것이 좋다.
- 데이터 구조 평면화
  - 비정규화를 통해 데이터를 서로 다른 경로로 분할하면 필요에 따라 별도의 호출을 통해 효율적으로 다운로드할 수 있다.

```json
{
  // Chats contains only meta info about each conversation
  // stored under the chats's unique ID
  "chats": {
    "one": {
      "title": "Historical Tech Pioneers",
      "lastMessage": "ghopper: Relay malfunction found. Cause: moth.",
      "timestamp": 1459361875666
    },
    "two": { ... },
    "three": { ... }
  },

  // Conversation members are easily accessible
  // and stored by chat conversation ID
  "members": {
    // we'll talk about indices like this below
    "one": {
      "ghopper": true,
      "alovelace": true,
      "eclarke": true
    },
    "two": { ... },
    "three": { ... }
  },

  // Messages are separate from data we may want to iterate quickly
  // but still easily paginated and queried, and organized by chat
  // conversation ID
  "messages": {
    "one": {
      "m1": {
        "name": "eclarke",
        "message": "The relay seems to be malfunctioning.",
        "timestamp": 1459361875337
      },
      "m2": { ... },
      "m3": { ... }
    },
    "two": { ... },
    "three": { ... }
  }
}
```

- 확장 가능한 데이터 만들기
  - 앱을 빌드할 때는 목록의 일부만 다운로드하는 것이 나을 때가 많다. 목록에 수천 개의 레코드가 포함된 경우에 특히 그러하다. 데이터의 관계가 정적이며 일방적인 경우에는 상위 객체 아래에 하위 객체를 중첩하면 간단히 해결된다.

  - 경우에 따라서 관계가 동적이거나 데이터를 비정규화해야할 수 있다. 대체적으로 쿼리를 사용하여 데이터를 검색하면 데이터를 비정규화할 수 있다.

  - 이 방법도 충분하지 않을 수 있다. 사용자와 그룹 간의 양방향 관계를 예로 들 수 있다. 사용자는 그룹에 속할 수도 있으며, 그룹은 사용자의 목록으로 구성된다. 이 때 사용자가 어떠한 그룹에 속할지를 결정하려면 문제가 복잡해진다

    → 이러한 경우 특정 사용자가 속하는 그룹을 나열하고 해당 그룹의 데이터만 가져오는 깔끔한 방법이 필요하다. 이 때 **그룹 색인**이 상당한 도움이 된다.

```json
// An index to track Ada's memberships
{
  "users": {
    "alovelace": {
      "name": "Ada Lovelace",
      // Index Ada's groups in her profile
      "groups": {
         // the value here doesn't matter, just that the key exists
         //✰✰ 그룹 색인!! key를 활용
         "techpioneers": true,
         "womentechmakers": true
      }
    },
    ...
  },
  "groups": {
    "techpioneers": {
      "name": "Historical Tech Pioneers",
      "members": {
        "alovelace": true,
        "ghopper": true,
        "eclarke": true
      }
    },
    ...
  }
}
```

Ada의 레코드와 그룹 모두에 관계를 저장하면 일부 데이터가 중복됨을 알 수 있다.  `alovelace`는 그룹 아래에 색인이 생성되었고 `techpioneers`는 Ada의 프로필에 나열되었다. 따라서 Ada를 그룹에서 삭제하려면 두 위치에서 업데이트가 이루어져야 한다.

이러한 중복성은 양방향 관계에서 불가피하다. 사용자 또는 그룹 목록이 수백만 개로 늘어나거나 실시간 데이터베이스 보안 규칙으로 인해 일부 레코드에 액세스할 수 없더라도 이 중복성 덕분에 Ada의 소속 그룹을 빠르고 효율적으로 확인할 수 있다.

✰✰✰**이와 같이 ID를 키로 나열하고 값을 true로 설정하여 데이터를 반전하는 방식을 사용하면 키를 확인할 때 `/users/$uid/groups/$group_id`를 읽어서 `null`인지만 확인하면 되니 간단하다. 색인은 데이터 쿼리 또는 검색보다 속도가 빠르고 훨씬 효율적이다.**✰✰✰



## 데이터 읽기 및 쓰기

Firebase 로컬 에뮬레이터 도구 모음으로 제작 및 테스트 가능하나 우선은 대략적인 구조를 보기 위해 에뮬레이터 사용은 미뤄두겠다!

- **데이터베이스 참조 가져오기**

```javascript
var database = firebase.database();
```

- **데이터 쓰기**

Firebase 데이터를 검색하려면 `firebase.database.Reference`에 비동기 리스너를 연결한다. 리스너는 데이터의 초기 상태가 확인될 때 한 번 트리거된 후 데이터가 변경될 때마다 다시 트리거된다.

기본 쓰기 작업은 `set()`코드를 사용하여 지정된 참조에 데이터를 저장하고 해당 경로의 기존 데이터를 모두 바꾼다.

```javascript
function writeUserData(userId, name, email, imageUrl){
  firebase.database().ref('users/' + userId).set({
    username : name,
    email : email,
    profile_picture : imageUrl
  });
}
```

`set()`를 사용하면 지정된 위치에서 하위 노드를 포함한 데이터를 덮어쓴다.

- **데이터 읽기**

경로의 데이터를 읽고 변경사항을 수신 대기하려면 `firebase.database.Reference` 의 `on()` 또는 `once()`를 사용하여 이벤트를 관찰한다.

`value` 이벤트를 사용하여 **이벤트 발생 시점에 특정 경로에 있던 콘텐츠의 정적 스냅샷을 읽을 수 있다.** 이 메서드는 리스너가 연결될 때 한 번 트리거된 후 하위 요소를 포함한 데이터가 변경될 때마다 다시 트리거된다. 하위 데이터를 포함하여 해당 위치의 모든 데이터를 포함하는 스냅샷이 이벤트 콜백에 다시 전달된다. 데이터가 없는 경우 스냅샷은 `exists()` 호출 시 false를 반환하고 `val()`호출 시 null을 반환한다.

```javascript
//데이터베이스에서 게시물의 별표 수를 검색하는 소셜 블로깅 애플리케이션
var starCountRef = firebase.database().ref('posts/' + postId + '/starCount');
starCountRef.on('value',(snapshot)=>{
  const data = snapshot.val();
  updateStarCount(postElement, data);
});
```

on 리스너는 value이벤트 발생 시점에 데이터 베이스에서 지정된 위치에 있던 데이터를 포함하는 snapshot을 수신한다. val()메서드를 사용하여 snapshot의 데이터를 확인할 수 있다.

- 

- 데이터 한 번 읽기
  - get()을 사용하여 데이터 한 번 읽기
  - 데이터가 한 번만 필요한 경우 `get()`을 사용하여 데이터베이스에서 데이터의 스냅샷을 가져올 수 있다. 어떠한 이유로든 `get()`이 서버 값을 반환할 수 없는 경우 클라이언트는 로컬 스토리지 캐시를 프로브하고 값을 여전히 찾을 수 없으면 오류를 반환한다.
  - 불필요한 `get()` 사용은 대역폭 사용을 증가시키고 성능 저하를 유발할 수 있지만 위와 같이 실시간 리스너를 사용하면 이를 방지할 수 있다.

```javascript
const dbRef = firebase.database().ref();
dbRef.child("users").child(userId).get().then((snapshot)=>{
  if(snapshot.exists()){
    console.log(snapshot.val());
  } else {
    console.log("No data available");
  }
}).catch((error)=>{
  console.error(error);
})
```

- 관찰자를 사용하여 데이터 한 번 읽기
  - 경우에 따라 서버의 업데이트된 값을 확인하는 대신 로컬 캐시의 값을 즉시 반환하고 싶을 수 있다. 이 경우에는 `once()`를 사용하여 로컬 디스크 캐시에서 데이터를 즉시 가져올 수 있다.
  - 한 번 로드된 후 자주 변경되지 않거나 능동적으로 수신 대기할 필요가 없는 데이터에 유용하다. 예를 들어 위 예시의 블로깅 앱에서는 사용자가 새 글을 작성하기 시작할 때 이 메서드로 사용자의 프로필을 로드한다.

```javascript
var userId = firebase.auth().currentUser.uid;
return firebase.database().ref('/users/' + userId).once('value').then((snapshot)=>{
  var username = (snapshot.val() && snapshot.val().username) || 'Anonymous';
})
```



## 데이터 업데이트 또는 삭제

- **특정 필드 업데이트** 

다른 하위 노드를 덮어쓰지 않고 특정 하위 노드에 동시에 쓰려면 `update()` 메서드를 사용한다. `update()`를 호출할 때 키 경로를 지정하여 더 낮은 수준의 하위 항목 값을 업데이트할 수 있다. 확장성 개선을 위해 데이터를 열 위치에 저장한 경우 [데이터 팬아웃](https://firebase.google.com/docs/database/web/structure-data#fanout)을 사용하여 해당 데이터의 모든 인스턴스를 업데이트할 수 있다.

e.g. 소셜 블로깅 애플리케이션에서 글을 만든 후 최근 활동 피드 및 게시자의 활동 피드에 동시에 업데이트

```javascript
function writeNewPost(uid, username, picture, title, body){
  // A post entry
  var postData = {
    author: username,
    uid: uid,
    body: body,
    title: title,
    starCount: 0,
    authorPic: picture
  };
  
  //Get a key for a new post
  var newPostKey = firebase.database().ref().child('posts').push().key;
  
  //Write the new post's data simultaneously in the posts list and the user's post list.
  var updates = {};
  updates['/posts' + newPostKey] = postData;
  updates['/user-posts' + uid + '/' + newPostKey] = postData;
  
  return firebase.database().ref().update(updates);

}
```

`push()`를 사용하여 모든 사용자의 게시물을 포함하는 노드(`/posts/$postid`)에서 게시물을 작성하는 동시에 키를 검색한다.

그런 다음 이 키를 사용하여 `/user-posts/$userid/$postid` 에서 사용자의 게시물에 두 번째 항목을 작성한다.

이 경로를 사용하면 이 예시에서 두 위치에 새 게시물을 생성한 것처럼 `update()`를 한 번만 호출하여 JSON 트리의 여러 위치에서 동시에 업데이트를 수행할 수 있다. 이러한 동시 업데이트는 원자적인 성격을 갖는다. 즉, 모든 업데이트가 한꺼번에 성공하거나 실패한다.

- **완료 콜백 추가**

데이터가 커밋되는 시점을 파악하려면 완료 콜백을 추가한다. `set()` 및`update()` 는 모두 쓰기가 데이터베이스에 커밋되었을 때 호출되는 선택적 완료 콜백을 사용한다. 호출이 실패하면 콜백에 실패 이유를 나타내는 오류 객체가 전달된다.

```javascript
//데이터 쓰기
firebase.database().ref('/users/' + userId).set({
  username: name,
  email: email,
  profile_picture: imageUrl
}, (error)=>{//두번째 파라미터로 완료 콜백 추가
  if(error){
    //The write failed...
  } else {
    //Data saved successfully!
  }
});
```

- 데이터 삭제

데이터를 삭제하는 가장 간단한 방법은 해당 데이터 위치의 참조에 `remove()`를 호출하는 것이다.

`set()` 또는 `update()` 등의 다른 쓰기 작업 값으로 null을 지정하여 삭제를 할 수도 있다.

`update()`에 이 방법을 사용하면 API 호출 한 번으로 여러 하위 항목을 삭제할 수 있다.

#### Promise 수신

데이터가 Firebase 실시간 데이터베이스 서버에 커밋되는 시점을 확인하려면 `Promise`를 사용하면 된다. `set()`와 `update()`는 쓰기가 데이터베이스에 커밋되는 시점을 알기 위해 사용할 수 있는 `Promise`를 반환할 수 있다.



#### 리스너 분리

Firebase 데이터베이스 참조에 대해 `off()` 메서드를 호출하면 콜백이 삭제된다.

리스너 하나를 삭제하려면 `off()`에 매개변수로 전달한다. 인수 없이 `off()` 를 호출하면 해당 위치의 모든 리스너가 삭제된다. 

상위 리스너에 `off()`를 호출해도 하위 노드에 등록된 리스너가 자동으로 삭제되지 않는다. 하위 리스너에도 `off()`를 호출하여 콜백을 삭제해야 한다.



#### 데이터를 트랜잭션으로 저장

증가 카운터와 같이 동시 수정으로 인해 손상될 수 있는 데이터를 다루는 경우 [트랜잭션 작업](https://firebase.google.com/docs/reference/js/firebase.database.Reference#transaction)을 사용할 수 있다. 이 작업에 업데이트 함수 및 선택적 완료 콜백을 지정할 수 있다. 업데이트 함수는 데이터의 현재 상태를 인수로 취하고 이 데이터에 새로 기록하려는 상태를 반환한다. 새 값이 기록되기 전에 다른 클라이언트에서 해당 위치에 기록하면 업데이트 함수가 새로운 현재 값으로 다시 호출되고 쓰기가 다시 시도된다.

예를 들어 소셜 블로깅 앱에서는 다음과 같이 사용자가 게시물에 별표표시를 하거나 별표를 삭제할 수 있게 하고 게시물이 몇 개의 별표를 받았는지 집계할 수 있다.

```javascript
function toggleStar(postRef, uid){
  postRef.transaction((post)=>{
    if(post){
      if(post.stars && post.stars[uid]){
        post.StarCount--;
        post.stars[uid] = null;
      } else {
        //post.stars == null or post.stars[uid] == null
        post.starCount++;
        if(!post.stars){
          post.stars = {};
        }
        post.stars[uid] = true;
      }
    }
    return post;
  });
}
```

트랜잭션을 사용하면 여러 사용자가 같은 게시물에 **동시에** 별표표시를 하거나 클라이언트 데이터의 동기화가 어긋나도 별표표시를 하거나 클라이언트 데이터의 동기화가 어긋나도 별표가 잘못 집계되지 않는다. 트랜잭션이 거부되면 서버에서 현재 값을 클라이언트에 반환하며, 클라이언트에 반환하며, 클라이언트는 업데이트된 값으로 트랜잭션을 다시 실행한다. 트랜잭션이 성공하거나 취소될 때까지 이 과정이 반복된다.

**✰ 참고:** 업데이트 함수는 여러 번 호출되므로 `null` 데이터를 처리할 수 있어야 합니다. 원격 데이터베이스에 기존 데이터가 있더라도 트랜잭션 함수를 실행할 때 로컬에 캐시된 데이터가 없으면 초깃값이 `null`일 수 있습니다.



#### 서버 측 원자적 증분

위의 사용 사례에서는 두 가지 값, 즉 별표 표시를 하거나 별표를 삭제한 사용자의 ID와 증분된 별표 수를 데이터베이스에 쓴다. 사용자가 게시물에 별표표시를 하고있다는 것을 이미 안다면 트랜잭션 대신 원자적 증분 작업을 사용할 수 있다.

```javascript
function addStar(uid, key){
  const updates = {};
  updates[`posts/${key}/stars/${uid}`] = true;
  updates[`posts/${key}/starCount`] = firebase.database.ServerValue.increment(1);
  udpates[`user-posts/${key}/stars/${uid}`] = true;
  updates[`user-posts/${key}/starCount`] = firebase.database.ServerValue.increment(1);
  firebase.database().ref().update(updates);
}
```

이 코드는 트랜잭션 작업을 사용하지 않으므로 충돌하는 업데이트가 있는 경우 코드가 자동으로 다시 실행되지는 않는다. 하지만 증분 작업은 데이터베이스 서버에서 직접 이루어지므로 충돌이 발생할 가능성이 없다.

사용자가 이전에 이미 별표표시한 게시물에 별표를 표시하는 상황처럼 애플리케이션별 충돌을 감지하고 거부하려면 해당 사용 사례에 대한 커스텀 보안 규칙을 작성해야한다.



#### 오프라인으로 데이터 작업하기

클라이언트의 네트워크 연결이 끊겨도 앱은 계속 정상작동한다.

Firebase 데이터베이스에 연결된 모든 클라이언트는 자체적으로 활성 데이터의 내부 버전을 유지한다. 데이터를 쓰면 우선 로컬 버전에 기록된다. 그런 다음 Firebase 클라이언트가 해당 데이터를 원격 데이터베이스 서버 및 다른 클라이언트와 '최선을 다해' 동기화한다.

이와 같이 데이터베이스에 대한 모든 쓰기 작업은 로컬 이벤트를 즉시 발생시키며, 그 이후에 서버에 데이터가 기록된다. 따라서 앱은 네트워크 지연 또는 연결 여부에 관계없이 응답성을 유지한다.

네트워크에 다시 연결되면 앱에서 적합한 이벤트 세트를 수신하며 클라이언트가 현재 서버 상태와 동기화하므로 커스텀 코드를 별도로 작성할 필요가 없다.



## 데이터 목록 다루기

### 목록 읽기 및 쓰기

- **데이터 목록에 추가**

멀티 사용자 애플리케이션에서 `push()`메서드를 사용하여 목록에 데이터를 추가한다.  `push()`메서드는 지정된 Firebase참조에 새 하위 요소가 추가될 때마다 고유 키를 생성한다. 목록의 새 요소마다 이러한 자동 생성 키를 사용하면 여러 클라이언트에서 쓰기 충돌 없이 동시에 같은 위치에 하위 요소를 추가할 수 있다.  `push()`가 생성하는 **고유키는 타임스탬프에 기반**하므로 목록 항목은 시간순으로 자동 정렬된다.

 `push()`메서드가 반환하는 새 데이터에 대한 참조를 사용하여 하위 요소의 자동 생성 키 값을 가져오거나 하위 데이터를 설정할 수 있다. `push()` 참조의 `.key` 속성에는 자동 생성 키가 포함된다.

(악 어려워 이게 무슨 말이야! 백문이불여일견)

e.g.  `push()`를 사용하여 소셜 애플리케이션의 게시물 목록에 새 게시물을 추가할 수 있다.

```javascript
//Create a new post reference with an auto-generated id
var postListRef = firebase.database().ref('posts');
var newPostRef = postListRef.push(); //->데이터 추가 
newPostRef.set({//set() -> 데이터 쓰기
  //...
});
```

- **하위 이벤트 수신 대기**

하위 이벤트는 노드의 하위 요소에서 발생하는 특정 작업에 대응하여 트리거된다. 하위 요소가 `push()`메서드를 통해 새로 추가되거나 `update()`메서드를 통해 업데이트되는 경우가 그 예시이다.

| 이벤트          | 일반적인 용도                                                |
| :-------------- | :----------------------------------------------------------- |
| `child_added`   | 항목 목록을 검색하거나 항목 목록에 대한 추가를 수신 대기합니다. 이 이벤트는 기존 하위 요소마다 한 번씩 발생한 후 지정된 경로에 하위 요소가 새로 추가될 때마다 다시 발생합니다. 리스너에는 새 하위 요소의 데이터를 포함하는 스냅샷이 전달됩니다. |
| `child_changed` | 목록의 항목에 대한 변경사항을을 수신 대기합니다. 이 이벤트는 하위 노드가 수정될 때마다 발생합니다. 여기에는 하위 노드의 하위에 대한 수정이 포함됩니다. 이벤트 리스너에 전달되는 스냅샷에는 하위 요소의 업데이트된 데이터가 포함됩니다. |
| `child_removed` | 목록의 항목 삭제를 수신 대기합니다. 이 이벤트는 바로 아래 하위 요소가 삭제될 때 트리거됩니다. 콜백 블록에 전달되는 스냅샷에는 삭제된 하위 요소의 데이터가 포함됩니다. |
| `child_moved`   | 순서가 지정된 목록의 항목 순서 변경사항을 수신 대기합니다. `child_moved` 이벤트는 항상 현재의 정렬 기준에 따라 항목 순서를 변경하도록 한 `child_changed` 이벤트 뒤에 나옵니다. |

이러한 메서드는 데이터베이스의 특정 노드에 대한 변경사항을 수신대기하는 데 유용할 수 있다.

e.g. 소셜 블로깅 앱으로 게시물의 댓글 활동 모니터링

```javascript
var commentsRef = firebase.database().ref('post-comments/' + postId);
commentsRef.on('child_added',(data)=>{
  addCommentElement(postElement, data.key, data.val().text, data.val().author);
});

commentsRef.on('child_changed',(data)=>{
  setCommentValues(postElement, data.key, data.val().text, data.val().author);
});

commentsRef.on('child_removed', (data)=>{
  deleteComment(postElement, data.key);
});
```

- **값 이벤트 수신 대기**

데이터 목록 읽기에 권장되는 방법은 하위 이벤트를 수신 대기하는 것이지만 상황에 따라서는 목록 참조의 값 이벤트를 수신 대기하는 방법이 유용하다.

데이터 목록에 `value`관찰자를 연결하면 전체 데이터 목록이 단일 스냅샷으로 반환되며, 이 객체를 루프 처리하여 개별 하위 요소에 액세스할 수 있다.

쿼리에 일치하는 항목이 단 한 개인 경우에도 스냅샷은 목록으로 표시된다. 단지 항목이 한 개 있을 뿐이다. 이 항목에 액세스하려면 결과를 루프 처리해야 한다.

```javascript
ref.once('value', (snapshot)=>{
  snapshot.forEach((childSnapshot)=>{
    var childKey = childSnapshot.key;
    var childData = childSnapshot.val();
    //...
  });
});
```

하위 추가 이벤트를 추가로 수신 대기하는 대신 **한 번의 작업으로** 목록의 모든 하위 항목을 가져오려는 경우 이 패턴이 유용하다.

- **데이터 정렬 및 필터링**

실시간 데이터베이스의 `Query`클래스를 사용하여 키,값, 또는 하위 요소 값으로 정렬된 데이터를 검색할 수 있다. 정렬된 결과를 특정 개수로 필터링하거나 키 또는 값 범위에 따라 필터링할 수도 있다.

정렬된 데이터를 검색하려면 우선 정렬 기준 메서드 중 하나를 지정하여 결과를 어떤 순서로 정렬할지 결정합니다.

| 메서드           | 용도                                                         |
| :--------------- | :----------------------------------------------------------- |
| `orderByChild()` | 지정된 하위 키 또는 중첩된 하위 경로의 값에 따라 결과를 정렬합니다. |
| `orderByKey()`   | 하위 키에 따라 결과를 정렬합니다.                            |
| `orderByValue()` | 하위 값에 따라 결과를 정렬합니다.                            |

정렬 기준 메서드는 한 번에 하나만 사용가능하다.

e.g. 별표 수를 기준으로 사용자의 최상위 게시물 목록을 검색하는 방법을 보여준다.

```javascript
var myUserId = firebase.auth().currentUser.uid;
var topUserPostsRef = firebase.database().ref('user-posts/' + myUserId).orderByChild('starCount');
```

쿼리를 하위 리스너와 함께 사용하여 사용자ID를 기준으로 데이터베이스의 특정 경로에 있는 사용자의 게시물을 각 게시물이 받는 별표 수에 따라 정렬한 결과를 클라이언트와 동기화하도록 쿼리를 정의한다. 이와 같이 **ID를 색인 키로 사용하는 기법**을 **데이터 팬아웃**이라고 한다. 

`orderByChild()`메서드를 호출할 때는 결과를 정렬하는 기준이 될 하위 키를 지정한다. 여기에서는 각 게시물의 "startCount" 하위 요소 값에 따라 게시물이 정렬된다. 다음과 같은 데이터가 있다면 중첩된 하위 요소에 따라 쿼리를 정렬할 수도 있다.

```javascript
"posts": {
  "ts-functions": {
    "metrics": {
      "views" : 1200000,
      "likes" : 251000,
      "shares": 1200,
    },
    "title" : "Why you should use TypeScript for writing Cloud Functions",
    "author": "Doug",
  },
  "android-arch-3": {
    "metrics": {
      "views" : 900000,
      "likes" : 117000,
      "shares": 144,
    },
    "title" : "Using Android Architecture Components with Firebase Realtime Database (Part 3)",
    "author": "Doug",
  }
},
```

이 예시에서는 `orderByChild()`호출에서 중첩된 하위 항목에 대한 상대 경로를 지정하여 `metrics` 키 아래에 중첩된 값에 따라 목록 요소를 정렬할 수 있다.

```javascript
var mostViewedPosts = firebase.database().ref('posts').orderByChild('metrics/views');
```

다른 데이터 유형을 정렬하는 방법에 대한 자세한 내용은 [쿼리 데이터 정렬 방법](https://firebase.google.com/docs/database/web/lists-of-data#data-order)을 참조

- **데이터 필터링**

데이터를 필터링하려면 제한 또는 범위 메서드를 정렬 기준 메서드와 조합하여 쿼리를 작성한다.

| 메서드           | 용도                                                         |
| :--------------- | :----------------------------------------------------------- |
| `limitToFirst()` | 정렬된 결과 목록의 시작 부분부터 반환할 최대 항목 수를 설정합니다. |
| `limitToLast()`  | 정렬된 결과 목록에서 맨 끝부터 반환할 최대 항목 개수를 설정합니다. |
| `startAt()`      | 선택한 정렬 기준 메서드에 따라 지정된 키 또는 값보다 크거나 같은 항목을 반환합니다. |
| `startAfter()`   | 선택한 정렬 기준 메서드에 따라 지정된 키 또는 값보다 큰 항목을 반환합니다. |
| `endAt()`        | 선택한 정렬 기준 메서드에 따라 지정된 키 또는 값보다 작거나 같은 항목을 반환합니다. |
| `endBefore()`    | 선택한 정렬 기준 메서드에 따라 지정된 키 또는 값보다 작은 항목을 반환합니다. |
| `equalTo()`      | 선택한 정렬 기준 메서드에 따라 지정된 키 또는 값과 동일한 항목을 반환합니다. |

정렬 기준 메서드와 달리 여러 개의 제한 또는 범위 함수를 조합할 수 있다.

##### 결과수 제한

`limitToFirst()` 및 `limitToLast()`메서드를 사용하여 특정 이벤트에서 동기화할 하위 요소의 최대 개수를 설정할 수 있다.

e.g. `limitToFirst()` 를 사용하여 제한을 100개로 설정하면 처음에 최대 100개의 `child_added`이벤트만 수신한다. 만약 Firebase 데이터베이스에 저장된 항목이 100개 미만이면 모든 항목에 `child_added`이벤트가 발생한다.

항목이 변경됨에 따라 쿼리에 새로 포함되는 항목에 대해 `child_added`이벤트, 쿼리에서 제외되는 항목에 대해 `child_removed`이벤트가 수신되며 총 개수는 100개로 유지된다.

e.g. 블로깅 앱이 모든 사용자의 게시물 중 최근 100개의 목록을 검색하는 쿼리를 정의하는 방법

```javascript
var recentPostRef = firebase.database().ref('posts').limitToFirst(100);
```

**이 예시에서는 쿼리를 정의하기만 하므로 데이터를 실제로 동기화하려면 리스너를 연결해야한다.**



##### 키 또는 값으로 필터링

`startAt()`, `startAfter()`, `endAt()`, `endBefore()`, `equalTo()`를 사용하여 쿼리의 시작, 종료, 동일 지점을 임의로 선택할 수 있다. 이 메서드는 특정값을 갖는 하위 요소로 데이터를 페이지로 나누거나 항목을 찾는 데 유용하다.



#### 쿼리 데이터의 순서

##### orderByChild

1. 지정된 하위 값이 null
2. 지정된 하위 값이 false
3. 지정된 하위 값이 true
4. 숫자 값을 갖는 하위 요소가 그 다음에 나오며 오름차순 정렬. 지정된 하위 노드의 숫자 값이 동일한 하위 요소가 여러 개면 키에 따라 정렬된다.
5. 숫자 다음에는 문자열이 나오며 사전순, 오름차순으로 정렬.지정된 하위 노드의 값이 동일한 하위 요소가 여러 개면 키에 따라 사전 순으로 정렬된다.
6. 마지막으로 객체가 나오며 키에 따라 사전순, 오름차순으로 정렬된다.

##### orderByKey

데이터가 **키에 따라** 오름차순으로 반환

1. 키가 32비트 정수로 파싱될 수 있는 하위 요소가 맨 처음에 나오며 오름차순으로 정렬된다.
2. 키가 문자열 값인 하위 요소가 그 다음에 나오며 사전순, 오름차순으로 정렬된다.

##### orderByValue

하위 요소가 **값에 따라** 정렬. 정렬 기준은 orderByChild()와 동일하지만 지정된 하위 키의 값 대신 노드의 값이 사용된다.
