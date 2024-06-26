# riverpodapi


HomePage
import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';

import 'home_screen.dart';

void main() {
  runApp(ProviderScope(child: const MyApp()));
}

class MyApp extends StatelessWidget {
  const MyApp({Key? key}) : super(key: key);

  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      // Remove the debug banner
      debugShowCheckedModeBanner: false,
      theme: ThemeData(primarySwatch: Colors.amber),
      title: "RP",
      home: HomeScreen(),
    );
  }
}


home_screen.dart

import 'package:flutter/material.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:river1/state/post_state.dart';

import 'model/post.dart';

class HomeScreen extends ConsumerStatefulWidget {
  const HomeScreen({super.key});

  @override
  ConsumerState<HomeScreen> createState() => _HomeScreenState();
}

class _HomeScreenState extends ConsumerState<HomeScreen> {
  @override
  Widget build(BuildContext context) {
    return Scaffold(
        floatingActionButton: FloatingActionButton(
          onPressed: () {
            ref.read(postProvider.notifier).fetchApiService();
          },
          child: Icon(Icons.add),
        ),
        body: Center(
          child: Consumer(builder: (contex, ref, child) {
            PostState state = ref.watch(postProvider);
            if (state is InitialPageLoding) {
              return Center(
                child: Text("Press FAB to load data"),
              );
            }

            if (state is PostLodingPostState) {
              return CircularProgressIndicator();
            }

            if (state is ErrorPostState) {
              return Text(state.message);
            }

            if (state is PostLoadedPostState) {
              return _postLoaded(state);
            }

            return Text("Nothing found");
          }),
        ));
  }

  _postLoaded(PostLoadedPostState state) {
    return ListView.builder(
        itemCount: state.posts.length,
        itemBuilder: (context, index) {
          Post post = state.posts[index];
          return ListTile(
            leading: CircleAvatar(
              child: Text(post.id.toString()),
            ),
            title: Text(post.title.toString()),
          );
        });
  }
}


post_provider

import 'package:flutter/material.dart';

import 'package:flutter/foundation.dart';
import 'package:flutter_riverpod/flutter_riverpod.dart';
import 'package:river1/model/services/http_get_service.dart';

import '../model/post.dart';

final postProvider = StateNotifierProvider<PostNotifier, PostState>((ref) {
  return PostNotifier();
});

@immutable
abstract class PostState {}

class InitialPageLoding extends PostState {}

class PostLodingPostState extends PostState {}

class PostLoadedPostState extends PostState {
  PostLoadedPostState({required this.posts});
  final List<Post> posts;
}

class ErrorPostState extends PostState {
  final String message;

  ErrorPostState({required this.message});
}

class PostNotifier extends StateNotifier<PostState> {
  PostNotifier() : super(InitialPageLoding());

  final HttpGetPost _httpGetPost = HttpGetPost();

  Future fetchApiService() async {
    try {
      state = InitialPageLoding();
      List<Post> post = await _httpGetPost.getService();
      state = PostLoadedPostState(posts: post);
    } catch (e) {
      state = ErrorPostState(message: e.toString());
    }
  }
}


service.dart
import 'dart:convert';

import 'package:http/http.dart';

import '../post.dart';

class HttpGetPost {
  final String url = "https://jsonplaceholder.typicode.com/posts";
  Future<List<Post>> getService() async {
    final List<Post> posts = [];

    try {
      final response = await get(Uri.parse(url));

      if (response.statusCode == 200) {
        final postList = jsonDecode(response.body);

        for (var postItem in postList) {
          Post post = Post.fromMap(postItem);
          posts.add(post);
        }
      }
    } catch (e) {
      print(e.toString());
    }

    return posts;
  }
}


post.dart

import 'dart:convert';

// ignore_for_file: public_member_api_docs, sort_constructors_first
class Post {
  Post({
    required this.userId,
    required this.id,
    required this.title,
    required this.body,
  });

  final int userId;
  final int id;
  final String title;
  final String body;

  Post copyWith({
    int? userId,
    int? id,
    String? title,
    String? body,
  }) {
    return Post(
      userId: userId ?? this.userId,
      id: id ?? this.id,
      title: title ?? this.title,
      body: body ?? this.body,
    );
  }

  Map<String, dynamic> toMap() {
    return <String, dynamic>{
      'userId': userId,
      'id': id,
      'title': title,
      'body': body,
    };
  }

  factory Post.fromMap(Map<String, dynamic> map) {
    return Post(
      userId: map['userId'] as int,
      id: map['id'] as int,
      title: map['title'] as String,
      body: map['body'] as String,
    );
  }

  String toJson() => json.encode(toMap());

  factory Post.fromJson(String source) =>
      Post.fromMap(json.decode(source) as Map<String, dynamic>);
}
