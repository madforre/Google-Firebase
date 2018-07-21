# Firebase CLI 설치

    npm install firebase-tools -g 
    (어디에서든 사용 가능하게 글로벌 설치)

    *안될시 sudo 이용하여 관리자 권한으로 실행

# firebase login 

    로그인 창이 뜬다.

    CLI에 정상적으로 로그인되면 success가 뜬다.

# firebase list

    firebase에 있는 실제 프로젝트의 정보가 뜬다.
    보인다면 정상적으로 연결이 된 것이다.

# firebase init

    초기화->여러가지 설정 선택 가능

    여기서 database 와 hosting을 선택해주자.

# firebase serve

    로컬 서버를 열어주는 기능

파이어베이스는 데이터베이스를 바로 더미식으로 받아와서
forEach 돌려서 렌더링 하는 방법이 아닌 비동기식, 콜백으로 데이터를 받아온다.

# 데이터베이스 관련

출처 https://gist.github.com/singun/bdceaa99ad61ee1296204454f797d579

## 정규화
관계형 데이터베이스의 설계에서 중복을 최소화하게 데이터를 구조화하는 프로세스를 정규화라고 한다. 데이터베이스 정규화의 목표는 이상이 있는 관계를 재구성하여 작고 잘 조직된 관계를 생성하는 것에 있다. 일반적으로 정규화란 크고, 제대로 조직되지 않은 테이블들과 관계들을 작고 잘 조직된 테이블과 관계들로 나누는 것을 포함한다. 정규화의 목적은 하나의 테이블에서의 데이터의 삽입, 삭제, 변경이 정의된 관계들로 인하여 데이터베이스의 나머지 부분들로 전파되게 하는 것이다.

## 비정규화
정규화는 되도록 중복된 데이터를 제거해서 성능 향상에 도움을 주것을 목표로 하고 있다. 그러나 과도한 정규화로 인해서 테이블의 수가 증가하게 되면, 다수의 JOIN이 발생함에 따라서 성능 저하가 발생할 수 있다. 보통의 경우 정규화 과정을 모두 거친 다음 마지막 단계에서 비정규화를 실시한다. 단, 테이블을 합치는 것만이 비정규화는 아니다.

* 그룹에 대한 합계와 같은 값을 미리 계산하여 테이블에 저장해 둔다. (얍 플레이스를 예로 들면, 사용자가 좋아요를 하거나 리뷰를 남기면 통계 관련 작업을 해주는데, 통계 테이블을 생성하여 관리하는 것 또한 비정규화라고 해석할 수 있다.)
* 하나의 테이블에서 자주 사용되는 행(레코드)와 그렇지 않은 행들을 분리하여 두 개의 테이블로 나눈다. (이런 경우에는 UNION으로 다시 연결시킨다.)
* 다른 테이블에 의존적이지만 자주 JOIN하여 사용하는 컬럼을 중복하여 테이블 안에 하나 더 생성한다.

# const db = firebase.database();

    const events = db.child('events/fm');
    const attendees = db.child('eventAttendees/fm');

    events.on('value', snap => {
        // render data to HTML
    })

    attendees.on('child_added', snap => {
        // append attendees to list
    })

## Listener

A value listener works best for the single event because value events work really well when you're
synchronizing objects.

And a child listener really works well for the attendees because attendees is a list, and child events work well for lists.


# Firebase Database Querying 101

    const db = firebase.database(); // 데이터베이스 인스턴스 가져와서 변수에 담음. Get the database object.
    const eventsRef = db.child('events'); // 데이터베이스 인스턴스안의 이벤트를 참조. Create a reference to the parent key which is events.

    eventsRef.orderFunction().queryFunction();

* example 1 ) 

    eventsRef.orderByKey().limitToFirst(10);

## orderFunction()

There are actually four different ordering functions that you can use.
    
1. orderByKey();
2. orderByChild('child_property'); 제일많이 쓰인다.(Specify the property.)
3. orderByValue();
4. orderByPriority(); 구식방식. orderByChild를 통해서 사용하도록 하자.

## queryFunction()

1. startAt('value') - You can start at a value or a key and then move until you hit this ending value.
2. endAt('value')
3. equalTo('child_key') - EqualTo is like your WHERE clause.
4. limitToFirst(10) - this is how you limiting. (select these top 10 rows.)
5. limitToLast(10) - select these bottom 10 or whatever number you need rows.

* example 2 )

    const db = firebase.database();
    const events = db.child('events');
    const query = events
                    .orderBychild('name')
                    .equalTo('Firebase Meetup')
                    .limitToFirst(1);

    query.on('value', snap => {
        // render data to HTML
    });

# Eight SQL Queries -> Firebase Queries

To do this, we would create a route reference first.
every single query can also use this root reference variable.

    const rootRef = firebase.database().ref();

1. Select a user by UID

    SELECT * FROM Users WHERE UID = 1; // SQL

    const oneRef = rootRef.child('users').child('1'); // Firebase
    const oneRef = rootRef.child('users').child('1').on('value' ~~~) 이렇게 뒤에 리스너도 사용 가능

2. Find a user by email address

    SELECT * FROM Users WHERE Email = 'alice@email.com'; // SQL

    const twoRef = rootRef.child('users').orderByChild('email').eqaulTo('alice@email.com'); // Firebase

3. Limit to 10 users

    SELECT * FROM Users LIMIT 10; // SQL

    const threeRef = rootRef.child('users').orderByKey().limitToFirst(10); // Firebase
    const threeRef = rootRef.child('users').limitToFirst(10); (Realtime Database는 이러한 경우 오더바이키 생략해줘도 스마트하게 작동한다.)

4. Get all users names that start with 'D'

    SELECT * FROM Users WHERE Name LIKE 'D%';

    const fourRef = rootRef.child('users').orderByChild('name').startAt('D').endAt('D\uf8ff');
    
    // 쿼리에 사용된 \uf8ff 문자는 유니코드 범위에서 매우 높은 코드 포인트입니다. 이 문자는 유니코드에서 대부분의 정규 문자보다 뒤에 나오므로 쿼리는 b로 시작되는 모든 값과 일치합니다. 

5. Get all users who are less than 50

    SELECT * FROM Users WHERE Age < 50;

    const fiveRef = rootRef.child('users').orderByChild('age').endAt(49);

6. Get all users who are grater than 50

    SELECT * FROM Users WHERE Age > 50;

    const sixRef = rootRef.child('users').orderByChild('age').startAt(51);

7. Get all users who are between 20 and 100

    SELECT * FROM Users WHERE Age >= 20 && Age <= 100;

    const sevenRef = rootRef.child('users').orderByChild('age').startAt(20).endAt(100);

8. Get all users who are 28 and live in Berlin

    SELECT * FROM Users WHERE Age = 28 && Location = 'Berlin';

    const eightRef = rootRef.child('users').orderByChild('age').equalTo(28)
                                           .orderByChild('location').equalTo('Berlin');  ---> Error 데이터 비정규화 필요

    const eightRef = rootRef.child('users').orderByChild('age_location').equalTo('28_Berlin');

* You can only use one ordering function in the Firebase SDK

# Joins in the Firebase Database

firebase method - once() 한 번만 함수를 호출