# 💬 채팅 앱

이 프로젝트는 **Firebase Authentication**을 이용한 로그인 기능과 **Firestore**를 이용해 채팅 내용을 저장하는 Flutter 기반 채팅 앱이다. 사용자는 이메일과 비밀번호로 회원가입 및 로그인할 수 있으며, 로그인 후 실시간으로 메시지를 주고받고 저장할 수 있다.

## 🛠️ 1. 실행 과정

### 1.1 필수 패키지 설치
<details>
<summary>필수 패키지 설치</summary>
<div markdown="1">

앱에서 Firebase 서비스를 사용하기 위해 **firebase_auth**, **cloud_firestore**, **firebase_core** 등의 패키지를 사용한다. 아래 링크를 통해 각 패키지를 설치할 수 있다.:

- **firebase_auth 패키지**: [firebase_auth](https://pub.dev/packages/firebase_auth)
- **cloud_firestore 패키지**: [cloud_firestore](https://pub.dev/packages/cloud_firestore)
- **firebase_core 패키지**: [firebase_core](https://pub.dev/packages/firebase_core)

```yaml
dependencies:
  firebase_core: latest_version
  firebase_auth: latest_version
  cloud_firestore: latest_version
```
</div> </details>

### 1.2 Firebase 초기화
<details> <summary>Firebase 초기화 설정</summary> <div markdown="1">
앱 시작 시 Firebase를 초기화하여 Firebase Authentication 및 Firestore 기능을 사용할 수 있도록 한다.

```dart
void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(MyApp());
}
```
</div> </details>

### 1.3 로그인 및 회원가입 기능
<details> <summary>Firebase Authentication 이용</summary> <div markdown="1">
Firebase Authentication을 이용해 사용자는 이메일과 비밀번호로 회원가입 및 로그인을 할 수 있다.
또한 firestore에 user 컬렉션을 만들어서 유저의 정보를 저장한다.

```dart
class _NewMessageState extends State<NewMessage> {
  final _controller = TextEditingController();
  var _userEnterMessage = '';
  void _sendMessage()async{
    FocusScope.of(context).unfocus();
    final user = FirebaseAuth.instance.currentUser;
    final userData = await FirebaseFirestore.instance.collection('user')
        .doc(user!.uid).get();
    FirebaseFirestore.instance.collection('chat').add({
      'text' : _userEnterMessage,
      'time' : Timestamp.now(),
      'userID' : user.uid,
      'userName' : userData.data()!['userName'],
      'userImage' : userData['picked_image']
    });
    _controller.clear();
  }
```
</div> </details>

### 1.4 Firestore를 이용한 채팅 저장
<details> <summary>Firestore 이용한 메시지 저장 및 불러오기</summary> <div markdown="1">
채팅 내용을 Firestore에 저장하고 실시간으로 불러온다.

```dart
StreamBuilder(
      stream: FirebaseFirestore.instance
          .collection('chat')
          .orderBy('time', descending: true)
          .snapshots(),
      builder: (context,
          AsyncSnapshot<QuerySnapshot<Map<String, dynamic>>> snapshot) {
        if (snapshot.connectionState == ConnectionState.waiting) {
          return Center(
            child: CircularProgressIndicator(),
          );
        }
        final chatDocs = snapshot.data!.docs;

        return ListView.builder(
          reverse: true,
          itemCount: chatDocs.length,
          itemBuilder: (context, index) {
            return ChatBubbles(
                chatDocs[index]['text'],
                chatDocs[index]['userID'].toString() == user!.uid,
                chatDocs[index]['userName'],
                chatDocs[index]['userImage']
            );
          },
        );
      },
    );
```
</div> </details>
