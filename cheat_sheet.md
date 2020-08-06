# Firebase 備忘録チートシート

## Firestore

### データを取得してリスト表示したい

```
import 'package:flutter/material.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class Messages extends StatelessWidget {
  @override
  Widget build(BuildContext context) {
    return StreamBuilder(
      stream: Firestore.instance.collection('messages').snapshots(),
      builder: (ctx, messageSnapshot) {
        // 読み込み中
        if (messageSnapshot.connectionState == ConnectionState.waiting) {
          return Center(
            child: CircularProgressIndicator(),
          );
        }
        // 読み込めたらデータを格納
        final messageDocs = messageSnapshot.data.documents;
        // 表示
        return ListView.builder(
          itemCount: messageSnapshot.data.documents.length,
          itemBuilder: (ctx, index) => Text(messageDocs[index]['text']),
        );
      },
    );
  }
}
.
.
```

### データを追加したい（ID は自動）

```
.
.
floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          Firestore.instance
              .collection('message')
              .add({'text': 'hello'});
        },
      ),
.
.
```

### データを追加したい（ID を指定）

```
.
.
floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          Firestore.instance
              .collection('message')
              .document('ABCDEFG')
              .setData({'text': 'hello'});
        },
      ),
.
.
```

### 取得するデータの並べ替え(昇順)

```
.
.
Firestore.instance
          .collection('messages')
          .orderBy('createdAt')
          .snapshots(),
.
.
```

### 取得するデータの並べ替え(降順)

```
.
.
Firestore.instance
          .collection('messages')
          .orderBy('createdAt', descending: true)
          .snapshots(),
.
.
```

### Firestore ルール

```
rules_version = '2';
service cloud.firestore {
  match /databases/{database}/documents {

    // 認証済みかつ、リクエストのUIDがドキュメントのUIDと一致する場合に書き込み可能
    match /users/{uid} {
        allow write: if request.auth != null && request.auth.uid == uid
    }

    // 認証済みならば読み込み可能
    match /users/{uid} {
        allow read: if request.auth != null;
    }

    // 認証済みならばドキュメントの読み込み、新規作成可能
    match /messages/{document=**} {
        allow read, create: if request.auth != null;
   }
  }
}
```

## Firebase Auth

### 認証によって画面を切り替える

```
.
.
return MaterialApp(
      title: 'Flutter Demo',
      theme: ThemeData(
        primarySwatch: Colors.pink,
      ),
      home: StreamBuilder(
        stream: FirebaseAuth.instance.onAuthStateChanged,
        builder: (ctx, userSnapshot) {
          if (userSnapshot.hasData) {
            // 認証ずみ
            return HomeScreen();
          } else {
            // 認証のための画面
            return AuthScreen();
          }
        },
      ),
    );
.
.
```

### AppBar でログアウトをする（ドロップダウンボタン）

```
.
.
appBar: AppBar(
        title: Text('Sample'),
        actions: <Widget>[
          DropdownButton(
            icon: Icon(
              Icons.more_vert,
              color: Theme.of(context).primaryIconTheme.color,
            ),
            items: [
              DropdownMenuItem(
                child: Container(
                  child: Row(
                    children: <Widget>[
                      Icon(Icons.exit_to_app),
                      SizedBox(
                        width: 8,
                      ),
                      Text('Logout'),
                    ],
                  ),
                ),
                value: 'logout',
              )
            ],
            onChanged: (itemIdentifier) {
              if (itemIdentifier == 'logout') {
                FirebaseAuth.instance.signOut();
              }
            },
          ),
        ],
      ),
.
.
```

### メールアドレス認証したい（エラーハンドリング付き+ローディング考慮）

```
.
.
final _auth = FirebaseAuth.instance;
var _isLoading = false;

void _submit(
    String email,
    String password,
    bool isLogin,
    BuildContext ctx,
  ) async {
    AuthResult authResult;

    try {
      setState(() {
        _isLoading = true;
      });
      if (isLogin) {
        // ログイン処理
        authResult = await _auth.signInWithEmailAndPassword(
          email: email,
          password: password,
        );
      } else {
        // 登録処理
        authResult = await _auth.createUserWithEmailAndPassword(
          email: email,
          password: password,
        );
      }
      // 認証が終われば画面遷移するため_Loadingのフラグ変更不要
    } on PlatformException catch (error) {
      // 一応エラーメッセージを設定
      var message = 'エラー発生';
      // 適切なエラーメッセージがあればそれを設定
      if (error.message != null) {
        message = error.message;
      }

      // スナックバーでユーザに通知
      Scaffold.of(ctx).showSnackBar(
        SnackBar(
          content: Text(message),
          backgroundColor: Theme.of(ctx).errorColor,
        ),
      );

      setState(() {
        _isLoading = false;
      });
    // その他のエラーハンドリング
    } catch (error) {
      print(error);
      setState(() {
        _isLoading = false;
      });
    }
  }
.
.
```

### 登録と同時に Firestore へユーザ情報を登録したい

```
.
.
authResult = await _auth.createUserWithEmailAndPassword(
          email: email,
          password: password,
        );
        await Firestore.instance
            .collection('users')
            .document(authResult.user.uid)
            .setData({
          'username': username,
          'email': email,
        });
.
.
```
