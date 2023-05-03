JSSP Language: A Brief Introduction
=============
### JSSP is a language designed for efficient server and client-side development. This document provides a brief introduction to the basic syntax and main features of JSSP.

## 1. Defining Classes, Structs, and Entities
Classes can be defined using the class keyword, and class properties and methods can be added using curly braces. JSSP provides a convenient feature that automatically generates name-based setters for struct and entity types. This makes it easy to copy data between instances of these types with matching property names.

```C#
class User {
  userId string;
  email Email;
  nickname string;

  func getName() string {
    return nickname;
  }
}

struct UserInfo {
  userId string;
  email Email;
  nickname string;
}

entity UserVo {
  userId string;
  email Email;
  nickname string;
}
```


## 2. Enums
Enumerations can be defined using the enum keyword, followed by the name and a list of values inside curly braces.
```C#
enum RoleType {
  Admin 0;
  User 1;
}
```

## 3. Queries
JSSP allows you to define SQL queries within your code using the query keyword.
```C#
query selectUserInfoByEmail(email Email) UserVo {
  select user_id as userId, email, nickname
  from user
  where email = :email;
}
```

## 4. API Design
API endpoints can be defined using the define opcode syntax.
```C#
define opcode[controller=Auth, baseUrl='/api/v1/auth'] {
    [method=post, action='signin', noauth, overwrite=true]
    signIn 100 'sign in'
}
req signIn
{
    [required=false]
    email     Email;
    
    [required=false]
    password  Password;
}
res signIn
{
    userInfo    UserInfo;
    accessToken   string;
    refreshToken  string;
}
handler signIn {
  func execute(user CustomUserDetails, req @req) (res @res) {
    // Your logic for handling the sign-in process goes here
  }
}
```

### 4.1 Handler class
In JSSP, the handler class plays a crucial role in converting server-side code. It is used to define the logic for handling API requests and processing the data received from clients. When you write a handler class in JSSP, the code gets translated into the target platform's language, such as Java for Spring Boot, C# for .NET, or JavaScript/TypeScript for Node.js.

Here's an example of a handler class in JSSP:
```C#
handler signIn {
  func execute(user CustomUserDetails, req @req) (res @res) {

    try {

      var email = req.email;
      var password = req.password;

      var userVo UserVo = @query.selectUserInfoByEmail(bizNo);

      if(userVo == null) {
        res.code = @error.NotFoundUser;
        return;
      }

      var userId = userVo.userId;

      var userInfo = @service.UserService.getUserInfo(userId);
      if(userInfo == null) {
        res.code = @error.NotFoundUser;
        return;
      }

      var n = @service.DeviceService.updateUserDevice(userId, deviceId, deviceType, deviceToken);
      if(n < 0) {
        res.code = @error.InternalServerError;
        return;
      } else if(n > 0) {
        @class.FirebaseUtils.subscribeTopic("coders", deviceToken);
      }
           
      res.userInfo = userInfo;
      res.accessToken = @class.AuthUtils.generateToken(userVo.userId, @enum.TokenType.Access);
      res.code = @error.Success;
    
      return;
    } catch (e) {
      res.code = @error.InternalServerError;
      @console.error(e);
      return;
    }
  }
}
```
In this example, the signIn handler class contains a single function called execute, which takes a user object, a req object representing the incoming request, and returns a res object representing the response.

When you run the JSSP build process using the coders tool, the handler class will be translated into the appropriate language for the target platform. This allows you to write server-side code using a single, unified syntax and generate code for different platforms without having to rewrite your logic for each one.

In summary, the handler class in JSSP enables you to write server-side code that can be easily translated to various languages and platforms, making it more efficient and maintainable.

## 5. Services
JSSP allows you to define service classes, which can execute database transactions.

```C#
service UserService {
  func getUserInfo(userId UserID) (userInfo UserInfo) {
    var userVo = @query.selectUserInfoByUserId(userId);
    userInfo.set(userVo);
    return;
  }
}
```
## 6. Widgets
widget is a class in JSSP designed for creating client-side user interfaces (UI). It is based on Flutter's widget syntax, but with some improvements to make it more convenient to use. The main changes include removing commas and using curly braces {} and semicolons ; instead. These adjustments help avoid issues caused by deeply nested commas, making it easier to write and maintain the code.

Here's an example of a JSSP widget class:
```C#
  widget UserLoginPage {
  func build(context BuildContext) widget {
    return Scaffold {
      body: Center {
        child: Column {
          mainAxisAlignment: MainAxisAlignment.center;
          children: [
            Text('User Login Page') {
              style: Theme.of(context).textTheme.headline4;
            }
          ]
        }
      }
    }
  }
}
```
In this example, the LoginPage widget class contains a single function called build, which takes a BuildContext object and returns a widget. The code is easier to read and write compared to the traditional Flutter syntax, thanks to the use of curly braces and semicolons instead of commas.

When you use JSSP to write client-side UI code, the build process will generate the corresponding Flutter Dart code. This allows you to maintain a single, unified syntax for your UI code and generate platform-specific code seamlessly.

In summary, the widget class in JSSP simplifies the process of writing client-side UI code by improving upon Flutter's widget syntax, making it more convenient and maintainable.

## 7. Stores
JSSP provides a store syntax for client-side data storage.
```C#
store app {
    [localStorage=true, secure=true]
    accessToken string;

    [localStorage=true]
    deviceId string;

    [localStorage=true]
    deviceType int32;

    [localStorage=true]
    deviceToken string;
}
```

## 8. External Classes
JSSP allows you to integrate external classes using the external class syntax.
```C#
external class {
  @platform.springboot[import='com.example.{projectName}.api.controller.utils.CryptoUtils']
  CryptoUtils {
    static func encryptPassword(string, string) string;
  }
}
```

## 9. Localization
JSSP provides syntax for managing client-side text in multiple languages.
```C#
// English
define string[locale="en"] {
  msg_unknown_error: 'An unknown error has occurred.';
  hello: 'Hello {name}';
  msg_not_found_user: 'User not found.';
}

// Korean
define string[locale="ko"] {
  msg_unknown_error: "알 수 없는 오류 입니다.";
  hello: "안녕하세요 {name}";
  msg_not_found_user: "사용자를 찾을 수 없습니다";
}
```

## 10. Using the Coders Tool
The Coders tool is used to interpret JSSP language and convert it to the appropriate language for various platforms. Install the Coders tool using the following command:
```bash
dotnet tool install -g coders
```
Use the Coders tool to generate
initial JSSP template code and convert JSSP code to the appropriate language for various platforms.
### 10.1. Initializing a JSSP Project
Use the 'coders init' command to create an initial JSSP template code.
```bash
coders init
```
### 10.2. Creating Project Files for Different Platforms
Use the 'coders create' command to copy initial files for different platforms and set up the project structure.
```bash
coders create
```
### 10.3. Building and Converting Code
Use the 'coders build' command to convert JSSP code to the appropriate language for different platforms. Modify your JSSP files and repeat this step as needed.
```bash
coders build
```


This process allows you to write your code using JSSP syntax and easily convert it to the required language for the target platform, simplifying cross-platform development.

In conclusion, JSSP is a versatile language designed to streamline server and client-side development across multiple platforms. With its syntax and features tailored for various use cases, JSSP simplifies the development process and enables a more efficient workflow.

