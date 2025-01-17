# Login-page-Flutter-
Flutter Signup/Login page With Firebase

A simple application design with flutter. Application is connected with Firebase to store and retrive user data data from firestore.





Code files are attached




main.dart 

import 'package:flutter/material.dart';
import 'package:firebase_core/firebase_core.dart';
import 'LoginPage.dart';


void main() async {
  WidgetsFlutterBinding.ensureInitialized();
  await Firebase.initializeApp();
  runApp(const MyApp());
}



class MyApp extends StatelessWidget {
  const MyApp({super.key});

  // This widget is the root of your application.
  @override
  Widget build(BuildContext context) {
    return MaterialApp(
      title: 'Login / Signup',
      theme: ThemeData(
        colorScheme: ColorScheme.fromSeed(seedColor: Colors.deepPurple),
        useMaterial3: true,
      ),
      home: LoginPage(),
    );
  }
}






LoginPage.dart

import 'package:cloud_firestore/cloud_firestore.dart';
import 'package:flutter/material.dart';
import 'package:flutter/gestures.dart';
import 'package:flutter_login_app/SignupPage.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:flutter_login_app/todolist.dart';

class LoginPage extends StatefulWidget {
  const LoginPage({super.key});

  @override
  State<LoginPage> createState() => _LoginState();
}

class _LoginState extends State<LoginPage> {
  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();
  bool _isPasswordVisible = false;
  bool isLoading =false;

  Future<void> _loginUser(String email, String password) async {
    setState(() {
    isLoading = true; // Start loading when login starts
  });

  try {
    // Sign in the user with email and password
    UserCredential userCredential = await FirebaseAuth.instance.signInWithEmailAndPassword(
      email: email,
      password: password,
    );

    // Check if user exists in Firestore
    var doc = await FirebaseFirestore.instance.collection('Users').doc(userCredential.user!.uid).get();
    if (doc.exists) {
      print('User data found: ${doc.data()}');
      // Navigate to the ToDoList page or any other page after successful login
      Navigator.pushReplacement(
        context,
        MaterialPageRoute(builder: (context) => const ToDoList()),
      );
    } else {
      print('User data not found in Firestore!');
      ScaffoldMessenger.of(context).showSnackBar(
        const SnackBar(content: Text('User data not found. Please check your credentials.')),
      );
    }
  } on FirebaseAuthException catch (e) {
    String errorMessage;
    if (e.code == 'user-not-found') {
      errorMessage = 'No user found for that email.';
    } else if (e.code == 'wrong-password') {
      errorMessage = 'Wrong password provided for that user.';
    } else {
      errorMessage = 'Login failed: ${e.message}';
    }
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(errorMessage)),
    );
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('An error occurred: ${e.toString()}')),
    );
  } finally {
    setState(() {
      isLoading = false; // End loading after attempt completes, success or failure
    });
    }
  }

  @override
  void dispose() {
    emailController.dispose();
    passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    double w = MediaQuery.of(context).size.width;
    double h = MediaQuery.of(context).size.height;

    return Scaffold(
      backgroundColor: Colors.white,
      body: SingleChildScrollView(
        child: Column(
          children: [
            // Top image container
            Container(
              width: w,
              height: h * 0.2,
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: AssetImage("images/blur.jpg"),
                  fit: BoxFit.cover,
                ),
              ),
            ),
            
            const SizedBox(height: 30),
            
            // Text fields container
            Container(
              margin: const EdgeInsets.symmetric(horizontal: 20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  const Text("Hello!", style: TextStyle(fontSize: 50, fontWeight: FontWeight.bold, color: Colors.blueGrey)),
                  const SizedBox(height: 30),
                  TextField(
                    controller: emailController,
                    decoration: const InputDecoration(
                      prefixIcon: Icon(Icons.email, color: Colors.blueAccent),
                      labelText: 'Email',
                      focusedBorder: OutlineInputBorder(
                        borderSide: BorderSide(color: Colors.indigo),
                      ),
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.all(Radius.circular(30)),
                      ),
                    ),
                  ),
                  const SizedBox(height: 20),
                  TextField(
                    controller: passwordController,
                    obscureText: !_isPasswordVisible,
                    decoration: InputDecoration(
                      prefixIcon: const Icon(Icons.password, color: Colors.blueAccent),
                      labelText: 'Password',
                      suffixIcon: IconButton(
                        icon: Icon(_isPasswordVisible ? Icons.visibility : Icons.visibility_off),
                        onPressed: () {
                          setState(() {
                            _isPasswordVisible = !_isPasswordVisible;
                          });
                        },
                      ),
                      focusedBorder: const OutlineInputBorder(
                        borderSide: BorderSide(color: Colors.indigo),
                      ),
                      border: const OutlineInputBorder(
                        borderRadius: BorderRadius.all(Radius.circular(30)),
                      ),
                    ),
                  ),

                ],
              ),
            ),
            
            const SizedBox(height: 30),
            
            // Login button container
            Container(
              alignment: Alignment.center,
              width: 130,
              height: 50,
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: AssetImage("images/blur.jpg"),
                  fit: BoxFit.cover,
                ),
                borderRadius: BorderRadius.all(Radius.circular(25)),
              ),
              child: TextButton(
                onPressed: () {
                  String email = emailController.text.trim();
                  String password = passwordController.text.trim();
                  _loginUser(email, password);
                },
                child: const Text('Login', style: TextStyle(fontSize: 25, color: Colors.white)),
              ),
            ),
            
            const SizedBox(height: 30),
            
            RichText(
              text: TextSpan(
                text: 'Don\'t have an account? ',
                style: const TextStyle(fontSize: 15, color: Colors.grey),
                children: [
                  TextSpan(
                    text: 'Create',
                    style: const TextStyle(fontSize: 15, color: Colors.black, fontWeight: FontWeight.bold),
                    recognizer: TapGestureRecognizer()
                      ..onTap = () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(builder: (context) => const SignupPage()),
                        );
                      },
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}







SignupPage.dart

import 'package:flutter/gestures.dart';
import 'package:flutter/material.dart';
import 'package:flutter_login_app/LoginPage.dart';
import 'package:firebase_auth/firebase_auth.dart';
import 'package:cloud_firestore/cloud_firestore.dart';

class SignupPage extends StatefulWidget {
  const SignupPage({super.key});

  @override
  State<SignupPage> createState() => _SignupState();
}

class _SignupState extends State<SignupPage> {
  final TextEditingController emailController = TextEditingController();
  final TextEditingController passwordController = TextEditingController();
  bool _isPasswordVisible = false;
  bool isLoading = false;

  Future<void> _registerUser(String email, String password) async {
    setState(() {
    isLoading = true; // Start loading when registration starts
  });

  try {
    // Create user with email and password using Firebase Authentication
    UserCredential userCredential = await FirebaseAuth.instance.createUserWithEmailAndPassword(
      email: email,
      password: password,
    );

    // Store user details in Firestore under the Users collection
    await FirebaseFirestore.instance.collection('Users').doc(userCredential.user!.uid).set({
      'email': email,
      'createdAt': FieldValue.serverTimestamp(),
    });

    // Debug print to verify UID and Firestore operation
    print('User UID: ${userCredential.user!.uid}');
    print('Saving user data to Firestore for UID: ${userCredential.user!.uid}');

    // Check if the document was created successfully
    var doc = await FirebaseFirestore.instance.collection('Users').doc(userCredential.user!.uid).get();
    if (doc.exists) {
      print('User data stored in Firestore: ${doc.data()}');
    } else {
      print('User data not found in Firestore!');
    }

    // Navigate to the LoginPage after successful signup
    Navigator.pushAndRemoveUntil(
      context,
      MaterialPageRoute(builder: (context) => const LoginPage()),
      (route) => false, // Clears previous routes
    );
  } on FirebaseAuthException catch (e) {
    String errorMessage;
    if (e.code == 'email-already-in-use') {
      errorMessage = 'This email is already registered. Please log in.';
    } else if (e.code == 'weak-password') {
      errorMessage = 'The password is too weak. Please choose a stronger password.';
    } else {
      errorMessage = 'Signup failed: ${e.message}';
    }
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text(errorMessage)),
    );
  } catch (e) {
    ScaffoldMessenger.of(context).showSnackBar(
      SnackBar(content: Text('An error occurred: ${e.toString()}')),
    );
  } finally {
    setState(() {
      isLoading = false; // End loading after attempt completes, success or failure
    });
  }
}

  @override
  void dispose() {
    emailController.dispose();
    passwordController.dispose();
    super.dispose();
  }

  @override
  Widget build(BuildContext context) {
    double w = MediaQuery.of(context).size.width;
    double h = MediaQuery.of(context).size.height;

    return Scaffold(
      backgroundColor: Colors.white,
      body: SingleChildScrollView(
        child: Column(
          children: [
            // Top image container
            Container(
              width: w,
              height: h * 0.2,
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: AssetImage("images/blur.jpg"),
                  fit: BoxFit.cover,
                ),
              ),
            ),
            
            const SizedBox(height: 30),
            
            // Text fields container
            Container(
              margin: const EdgeInsets.symmetric(horizontal: 20),
              child: Column(
                crossAxisAlignment: CrossAxisAlignment.center,
                children: [
                  const Text("Hello!", style: TextStyle(fontSize: 50, fontWeight: FontWeight.bold, color: Colors.blueGrey)),
                  const SizedBox(height: 30),
                  TextField(
                    controller: emailController,
                    decoration: const InputDecoration(
                      prefixIcon: Icon(Icons.email, color: Colors.blueAccent),
                      labelText: 'Email',
                      focusedBorder: OutlineInputBorder(
                        borderSide: BorderSide(color: Colors.indigo),
                      ),
                      border: OutlineInputBorder(
                        borderRadius: BorderRadius.all(Radius.circular(30)),
                      ),
                    ),
                  ),
                  const SizedBox(height: 20),
                  TextField(
                    controller: passwordController,
                    obscureText: !_isPasswordVisible,
                    decoration: InputDecoration(
                      prefixIcon: const Icon(Icons.password, color: Colors.blueAccent),
                      labelText: 'Password',
                      suffixIcon: IconButton(
                        icon: Icon(_isPasswordVisible ? Icons.visibility : Icons.visibility_off),
                        onPressed: () {
                          setState(() {
                            _isPasswordVisible = !_isPasswordVisible;
                          });
                        },
                      ),
                      focusedBorder: const OutlineInputBorder(
                        borderSide: BorderSide(color: Colors.indigo),
                      ),
                      border: const OutlineInputBorder(
                        borderRadius: BorderRadius.all(Radius.circular(30)),
                      ),
                    ),
                  ),
                  const SizedBox(height: 20),
                ],
              ),
            ),
            
            const SizedBox(height: 30),
            
            // Signup button container
            Container(
              alignment: Alignment.center,
              width: 130,
              height: 50,
              decoration: const BoxDecoration(
                image: DecorationImage(
                  image: AssetImage("images/blur.jpg"),
                  fit: BoxFit.cover,
                ),
                borderRadius: BorderRadius.all(Radius.circular(25)),
              ),
              child: TextButton(
                onPressed: isLoading
                ? null // Disable button when loading
                : () async {
                  String email = emailController.text.trim();
                  String password = passwordController.text.trim();

                  // Simple validation
                  if (email.isEmpty || password.isEmpty) {
                    ScaffoldMessenger.of(context).showSnackBar(
                      const SnackBar(content: Text('Please fill in all fields')),
                    );
                    return; // Exit the function if validation fails
                  }

                  // Call your register user method
                  await _registerUser(email, password);
                },
                child: isLoading
                  ? const CircularProgressIndicator(
                      valueColor: AlwaysStoppedAnimation<Color>(Colors.white), // Change color to match your theme
                    )
                  : const Text('Sign Up', style: TextStyle(fontSize: 25, color: Colors.white)),
              ),
            ),
            
            const SizedBox(height: 30),
            
            RichText(
              text: TextSpan(
                text: 'Already have an account? ',
                style: const TextStyle(fontSize: 15, color: Colors.grey),
                children: [
                  TextSpan(
                    text: 'Login',
                    style: const TextStyle(fontSize: 15, color: Colors.black, fontWeight: FontWeight.bold),
                    recognizer: TapGestureRecognizer()
                      ..onTap = () {
                        Navigator.push(
                          context,
                          MaterialPageRoute(builder: (context) => const LoginPage()),
                        );
                      },
                  ),
                ],
              ),
            ),
          ],
        ),
      ),
    );
  }
}







Todolist.dart

import 'package:flutter/material.dart';

class ToDoList extends StatefulWidget {
  const ToDoList({super.key});
  @override
  State<ToDoList> createState() => _ToDoState ();
}

class _ToDoState extends State<ToDoList> {

  final List<String> _todoList =[];
  final TextEditingController _textController =TextEditingController();


  // add new task to list 
  void _addTodoItem(String task)
  {
    if(task.isNotEmpty)
    {
      setState(() 
      {
        _todoList.add(task);
      });
      _textController.clear();
    }
  }

  // remove task from list
  void _removeTodoItem(int index)
  {
    setState(() 
    {
      _todoList.removeAt(index);
    });
  }

  @override
  Widget build(BuildContext context) {
    return Scaffold(
      appBar: AppBar(

        backgroundColor: Colors.blueGrey,
        title: const Text(
          'To Do List',
          style: TextStyle(
            fontSize: 25,
            color: Colors.white,
            fontWeight: FontWeight.bold,
            ),
           ),
        centerTitle: true,
      ),
      
      body:Column(
        children: [
          Padding(
            padding: const EdgeInsets.all(8.0),
            child: Row(
              children: [
                //input text 
                Expanded(
                  child: TextField(
                    controller: _textController,
                    decoration: const InputDecoration(
                      hintText: 'Enter new task',
                    ),
                  ),
                  ),

                  //button to add text

                  InkWell(
                    onTap: (){
                      _addTodoItem(_textController.text);
                    },
                    child: Container(

                      padding: const EdgeInsets.all(8.0),
                      color: Colors.lightBlue,
                      child: const Text ('Add',style: TextStyle(color: Colors.white)
                      ),
                    ),
                  )
                ],
              ),
            ),

            //Task List
            Expanded(
              child: ListView.builder(
    itemCount: _todoList.length,
    itemBuilder: (context, index) {
      return Container(
        margin: const EdgeInsets.symmetric(vertical: 5, horizontal: 10),
        padding: const EdgeInsets.all(10.0),
        color: Colors.red[50],
        child: Row(
          mainAxisAlignment: MainAxisAlignment.spaceBetween,
          children: [
            Expanded(
              child: Text(
                _todoList[index],
                style: const TextStyle(fontSize: 18),
                overflow: TextOverflow.ellipsis,
              ),
            ),
            InkWell(
              onTap: () {
                _removeTodoItem(index);
              },
              child: Container(
                padding: const EdgeInsets.all(8.0),
                color: Colors.redAccent,
                child: const Text('Delete', style: TextStyle(color: Colors.white)),
              ),
            ),
          ],
        ),
      );
    },
  ),    
            ),
        ],
      ) 
    );
  }
}
