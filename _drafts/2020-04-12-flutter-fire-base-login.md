---
layout: post
title:  파이어베이스를 이용한 플루터 로그인
comments: true
---




```
FirebaseAuth auth = FirebaseAuth.instance;
GoogleSignIn googleSignIn = GoogleSignIn();
GoogleSignInAccount account = await googleSignIn.signIn();
GoogleSignInAuthentication authentication = await account.authentication;
AuthCredential credential = GoogleAuthProvider.getCredential(idToken: authentication.idToken, accessToken: authentication.accessToken);
AuthResult authResult = await auth.signInWithCredential(credential);

FirebaseUser user = authResult.user;

assert(!user.isAnonymous);
assert(await user.getIdToken() != null);

IdTokenResult idTokenResult = await user.getIdToken(refresh: false);

final FirebaseUser currentUser = await _auth.currentUser();
assert(user.uid == currentUser.uid);
``````