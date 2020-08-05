# Firebase 備忘録チートシート

## Firestore

### データを取得してリスト表示したい

```
.
.
return Scaffold(
      body: StreamBuilder(
        // streamにはfirestoreのsnapshotsを設定
        stream: Firestore.instance
            .collection(COLLECTION_NAME)
            .snapshots(),
        // streamに変更があると画面が更新される
        builder: (ctx, streamSnapshot) {
          // 読み込みが終わるまではグルグルを表示
          if (streamSnapshot.connectionState == ConnectionState.waiting) {
            return Center(
              child: CircularProgressIndicator(),
            );
          }
          // 読み込みが終わったらdocumentsとして使用
          final documents = streamSnapshot.data.documents;
          // 表示部分
          return ListView.builder(
            itemCount: streamSnapshot.data.documents.length,
            itemBuilder: (ctx, index) => Container(
              padding: EdgeInsets.all(8),
              child: Text(documents[index][DOCUMENT_KEY_NAME]),
            ),
          );
        },
      ),
    );
.
.
```

### データを追加したい(ID は自動)

```
.
.
floatingActionButton: FloatingActionButton(
        child: Icon(Icons.add),
        onPressed: () {
          Firestore.instance
              .collection(COLLECTION_NAME)
              .add({KEY: VALUE});
        },
      ),
.
.
```

## Firebase Auth

### メールアドレス認証（エラーハンドリング付き）

```
.
.
void _submitAuthForm(
    String email,
    String password,
    bool isLogin,
    BuildContext ctx,
  ) async {
    AuthResult authResult;

    try {
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
    // その他のエラーハンドリング
    } catch (error) {
      print(error);
    }
  }
.
.
```
