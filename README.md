# flutter_ducafecat_news_getx
## Interface api description
  <img src="https://ducafecat.oss-cn-beijing.aliyuncs.com/ducafecat/video-ducafecat-banner.png" alt="Cat Brother Video Station" >
- Api address 

https://mock.apifox.cn/m1/1124717-0-default

- Documentation address

https://www.apifox.cn/apidoc/project-1124717/api-24266149

- Apifox import files

`doc/newsclient-api.apifox.json`

https://www.apifox.cn/

## Illustration

News Client Getx Version - Project Template

![](README/2021-07-24-14-47-42.png)

## Video demonstrations

https://space.bilibili.com/404904528/channel/detail?cid=177514&ctype=0

## How to improve code quality & efficiency

![](README/architecture.png)

### 1. Specifications
Use the same core concepts to develop the projects, such as but not limited to, Coding specifications, directory rules, model definitions, and layout schemes.

[Effective Dart: Style](https://dart.dev/guides/language/effective-dart/style)
[Flutter_Go Code Development Specification.md](https://github.com/alibaba/flutter-go/blob/master/Flutter_Go%20%E4%BB%A3%E7%A0%81%E5%BC%80%E5 %8F%91%E8%A7%84%E8%8C%83.md)

### 2. Templates

Common and common things are extracted, such as: routing, global data, authentication, authentication, offline login, interface management, data model, program upgrade, data verification, three-level cache, error collection, and behavior analysis.

### 3. Codebase

This is the business function. You can condense all of them in a single project which is recommended, instead of many packages, which are difficult to manage.

Common services include: welcome interface , registration, login, three-way login, chatting, video, photos, SKU, shopping cart, distribution, map, message push, comment, waterfall, category subscription, attribute table, as well as carousel.

## Supporting vscode plugin

- [GetX Snippets](https://marketplace.visualstudio.com/items?itemName=get-snippets.get-snippets)

   Required code hints, code blocks

- [Json to Dart Model](https://marketplace.visualstudio.com/items?itemName=hirantha.json-to-dart)

   Supports null safety, highly recommended

- [Flutter GetX Generator - Cat Brother](https://marketplace.visualstudio.com/items?itemName=ducafecat.getx-template)

   This plugin is used to quickly create `page` code, plans (json to dart, iconfont, test unit)

## refer to

- [get_cli](https://github.com/jonataslaw/get_cli)
- [getx_pattern](https://kauemurakami.github.io/getx_pattern/)
- [flutter-go](https://github.com/alibaba/flutter-go)
- [News first edition flutter_learn_news](https://github.com/ducafecat/flutter_learn_news)
- [写夜子 flutter-getx-template](https://github.com/xieyezi/flutter-getx-template)
- [Getx_quick_start](https://github.com/ducafecat/getx_quick_start)
- [flutter_use](https://github.com/CNAD666/flutter_use)
- [redux part-1-overview-concepts](https://redux.js.org/tutorials/essentials/part-1-overview-concepts)
- [todo_getx](https://github.com/loicgeek/todo_getx)

## Mock data

- api

https://yapi.ducafecat.tech/mock/11

- Check the interface mode

```
https://yapi.ducafecat.tech
api@ducafecat.tech
123456
```

## Directory Structure

![](README/catalog.png)

### Common components

| Name | Description |
| ----------- | -------------- |
| apis | Http interface definition |
| entities | Data model, instance |
| langs | Multilingual |
| middlewares | Middleware |
| routes | Routes |
| services | GetX global |
| utils | Tools |
| values | Values |
| widgets | Public components |

### Business page interface

![One-way data flow](README/one-way-data-flow-04fe46332c1ccb3497ecb04b94e55b97.png)

The interface code splitting also inherits the design idea of redux, splitting view, action, and state.

| Name | Description |
| --------------- | -------- |
| bindings.dart | Data Bindings |
| controller.dart | Controller |
| index.dart | Entry |
| state.dart | State |
| view.dart | View |
| widgets | Components |

## GetX up and down scroll list interface

![](README/2021-07-24-13-54-17.png)

### `RxList` to handle List collection

lib/pages/category/state.dart

```dart
class CategoryState {
   // Declared a list variable and made it observable (obs).
   RxList<NewsItem> newsList = <NewsItem>[].obs;
}
```

### `StatefulWidget` combined with `AutomaticKeepAliveClientMixin`

lib/pages/category/widgets/news_page_list.dart

```dart
class _NewsPageListState extends State<NewsPageList>
     with AutomaticKeepAliveClientMixin {
  // Unless you override AutomaticKeepAliveClientMixin, using a
  // non-existent member results in an error (not entirely sure what error it causes).
   @override
   bool get wantKeepAlive => true;

   final controller = Get. find<CategoryController>();

```

### `pull_to_refresh` drop-down component

lib/pages/category/widgets/news_page_list.dart

```dart
   @override
   /// Building the webpage
   Widget build(BuildContext context) {
     super. build(context);
     return GetX<CategoryController>(
       init: controller,
       builder: (controller) => SmartRefresher(
         enablePullUp: true,
         // Refers to page scrolling up and down controller. 
         controller: controller. refreshController,
         // Pull down page to scroll up and refresh data. 
         onRefresh: controller. onRefresh,
         // Pull up page to scroll down and load data.
         onLoading: controller. onLoading,
         // Dynamically builds each item based on the aspect ratio of user's device.
         child: CustomScrollView(
           slivers: [
             SliverPadding(
               padding: EdgeInsets.symmetric(
                 vertical: 0.w,
                 horizontal: 0.w,
               ), 
               sliver: SliverList(
                 delegate: SliverChildBuilderDelegate(
                   (content, index) {
                     var item = controller.state.newsList[index];
                     return newsListItem(item);
                   },
                   // childCount is the amount of data there is 
                   childCount: controller.state.newsList.length,
                 ),
               ),
             ),
           ],
         ),
       ),
     );
   }
```

`controller: controller.refreshController` up and down controller

`onRefresh: controller.onRefresh` Pull down to refresh data

`onLoading: controller.onLoading` pulls up and loads data

`SliverChildBuilderDelegate` dynamically builds each item, `childCount` tells the component how much data there is in total

### Write business in `controller`

lib/pages/category/controller.dart

- `onRefresh` Pull down to refresh

```dart
  
   void onRefresh() {
     fetchNewsList(isRefresh: true). then((_) {
       /// Refresh is completed if every element satisfies the refresh requirement(Changes in the API).
       refreshController. refreshCompleted(resetFooterState: true);
     }).catchError((_) {
       /// Refresh fails if not every element satisfies the refresh requirement (No change in API).
       refreshController. refreshFailed();
     });
   }
```

`refreshController.refreshCompleted()` refresh completed

`refreshController.refreshFailed()` Refresh failed

- `onLoading` pull-up loading

```dart
   void onLoading() {
     if (state. newsList. length < total) {
       fetchNewsList().then((_) {
         // Loading is completed if every element satisfies the loading requirement.
         refreshController. loadComplete();
       }).catchError((_) {
         // Loading produces an error if not every element satisfies the loading requirement.
         refreshController. loadFailed();
       });
     } else {
       /// Returns false / no data if API has no changes.
       refreshController.loadNoData();
     }
   }
```

`refreshController.loadComplete()` load complete

`refreshController.loadFailed()` failed to load

`refreshController.loadNoData()` has no data

- `fetch` all data

```dart
   /// Function to pull data.
   Future<void> fetchNewsList({bool isRefresh = false}) async {
     var result = await NewsAPI. newsPageList(
       params: NewsPageListRequestEntity(
         categoryCode: categoryCode,
         pageNum: curPage + 1,
         pageSize: pageSize,
       ),
     );

     if (isRefresh == true) {
       curPage = 1;
       total = result. counts!;
       state.newsList.clear();
     } else {
       curPage++;
     }
     // Need to ask what is the purpose of ! in (result.items!)
     state.newsList.addAll(result.items!);
   }
```

`state.newsList.addAll(result.items!);` Merge `list` collection `RxList` encapsulated

- `dispose` remembers to dispose

```dart
   /// Disposes releases memory? (Need to ask what is the purpose of this)
   @override
   void dispose() {
     super.dispose();
     // dispose releases the object
     refreshController. dispose();
   }
```

`refreshController.dispose()` This business is the drop-down control, and the video player, text box and other controllers must be released.

- `bindings` in `ApplicationBinding`

lib/pages/application/bindings.dart

```dart
class ApplicationBinding implements Bindings {
   @override
   void dependencies() {
     Get. lazyPut<ApplicationController>(() => ApplicationController());
     Get. lazyPut<MainController>(() => MainController());
     Get. lazyPut<CategoryController>(() => CategoryController());
   }
}
```

Because this `CategoryController` belongs to `Application` and is loaded by the route

## State Management

### `Bindings` autoload release

suitable for named routing

- Define `Bindings`

```dart
class SignInBinding implements Bindings {
   @override
   void dependencies() {
     Get. lazyPut<SignInController>(() => SignInController());
   }
}
```

- route definition

```dart
GetPage(
   name: AppRoutes. SIGN_IN,
   page: () => SignInPage(),
   binding: SignInBinding(),
),
```

- `Get.toNamed` automatically manages response data when loading screens

```sh
flutter: ** GOING TO ROUTE /home. isError: [false]
flutter: ** GOING TO ROUTE /count. isError: [false]
flutter: ** Instance "CountController" has been created. isError: [false]
flutter: ** Instance "CountController" has been initialized. isError: [false]
flutter: ** GOING TO ROUTE /count. isError: [false]
flutter: ** CLOSE TO ROUTE /count. isError: [false]
flutter: ** "CountController" onDelete() called. isError: [false]
flutter: ** "CountController" deleted from memory. isError: [false]
```

### `Get.put` `Get.find` manual management

Suitable for non-named routing, component instantiation

- `Get.put` initial

```dart
class StateDependencyPutFindView extends StatelessWidget {
   StateDependencyPutFindView({Key? key}) : super(key: key);

   final controller = Get. put<CountController>(CountController());
```

- `Get.find` call

```dart
/// Building for next page
class NextPageView extends StatelessWidget {
   NextPageView({Key? key}) : super(key: key);

   final controller = Get. find<CountController>();

   @override
   Widget build(BuildContext context) {
     return Scaffold(
       appBar: AppBar(
         title: Text("NextPage"),
       ),
       body: Center(
         child: Column(
           children: [
             GetX<CountController>(
               init: controller,
               initState: (_) {},
               builder: (_) {
                 return Text('value -> ${_.count}');
               },
             ),
             Divider(),
           ],
         ),
       ),
     );
   }
}
```

## Component Design

### Use the `GetView` component directly

The advantage is less code, directly use `controller` member variables to access

```dart
/// Not entirely sure what this does , need to ask
class HellowordWidget extends GetView<NotfoundController> {
   @override
   Widget build(BuildContext context) {
     return Center(
       child: Obx(() => Text(controller. state. title)),
     );
   }
}
```

### Using `Mixin` to customize

Use the `Mixin with` feature to directly wrap `StatefulWidget` `StatelessWidget`

This is inevitable

- AutomaticKeepAliveClientMixin

```dart
/// Reusing AutomaticKeepAliveClientMixin function from _NewsPageListState class
class _NewsPageListState extends State<NewsPageList>
     with AutomaticKeepAliveClientMixin {
   @override
   bool get wantKeepAlive => true;

   final controller = Get. find<CategoryController>();

   @override
   Widget build(BuildContext context) {
     super. build(context);
```

- TickerProviderStateMixin

```dart
/// Reusing TickerProviderStateMixin function from StaggerRoute class
class StaggerRoute extends StatefulWidget {
   @override
   _StaggerRouteState createState() => _StaggerRouteState();
}

class _StaggerRouteState extends State<StaggerRoute> with TickerProviderStateMixin {
   final controller = Get. find<StaggerController>();
```

### Don't respond to data overuse

- Many times, you may not need the response data

   - Single page data list
   - No boast page, boast component situation
   - form processing

- Recommended usage scenarios

   - Global data: user information, chat push, style color theme
   - Single-page multi-component interaction: chat interface
   - Multi-page switching: shopping cart

> Please make it clear that `GetX` is a component encapsulation method, it only includes `routing`, `state management`, `popup box`...

## Deep Linking way to open APP externally

- Effect

![](README/scheme.gif)

- refer to

   - https://flutter.dev/docs/development/ui/navigation/deep-linking
   - https://developer.android.com/codelabs/basic-android-kotlin-training-activities-intents#0
   - https://developer.android.com/reference/android/content/Intent
   - https://www.runoob.com/w3cnote/android-tutorial-intent-base.html
   - https://pub.flutter-io.cn/packages/uni_links

- android

> android/app/src/main/AndroidManifest.xml

```xml
<activity
   ... >
   ...
     <intent-filter>
         <action android:name="android.intent.action.VIEW"/>
         <category android:name="android.intent.category.DEFAULT"/>
         <category android:name="android.intent.category.BROWSABLE"/>
         <data
             android:scheme="newsgetx"
             />
     </intent-filter>
</activity>
```

- ios

> Runner -> TARGETS -> Info -> URL Types

![](README/2021-09-18-22-26-23.png)

- Plugin uni_links

```yaml
dependencies:
   ...
   uni_links: ^0.5.1
```

- flutter code

> lib/pages/application/controller.dart

```dart

   /// The URL is opened internally.
   bool isInitialUriIsHandled = false;
   StreamSubscription? uriSub;

   /// If URL is opened for the first time.
   Future<void> handleInitialUri() async {
     if (!isInitialUriIsHandled) {
       isInitialUriIsHandled = true;
       try {
         final uri = await getInitialUri();
         if (uri == null) {
           print('no initial uri');
         } else {
           // Getting the URL request here.
           print('got initial uri: $uri');
         }
       } on PlatformException {
         print('falied to get initial uri');
       } on FormatException catch (err) {
         print('malformed initial uri, ' + err.toString());
       }
     }
   }

   /// Intervene when the program from the URL opens.
   void handleIncomingLinks() {
     if (!kIsWeb) {
       uriSub = uriLinkStream. listen((Uri? uri) {
         // Get the URL request here
         print('got uri: $uri');

         if (uri != null && uri.path == '/notify/category') {
           Get.toNamed(AppRoutes. Category);
         }
       }, onError: (Object err) {
         print('got err: $err');
       });
     }
   }
   // Clears the URL cache 
   @override
   void dispose() {
     uriSub?. cancel();
     super.dispose();
   }
```

- Called in the web page

```html
<a href="newsgetx://com.tpns.push/notify/category"
   >newsgetx://com.tpns.push/notify/category</a
>

<a href="newsgetx://com.tpns.push/notify/message/123"
   >newsgetx://com.tpns.push/notify/message/123</a
>
```

- result

![](README/2021-09-18-22-52-54.png)

## Routing Design

Declare name, component, data binding, middleware through GetPage

document

`lib/common/routes/pages.dart`

```dart
// Setting up the different routes
class AppRoutes {
   static const INITIAL = '/';
   static const SIGN_IN = '/sign_in';
   static const SIGN_UP = '/sign_up';
   static const NotFound = '/not_found';

   static const Application = '/application';
   static const Category = '/category';
}
```

`lib/common/routes/names.dart`

```dart
class AppPages {
   static const INITIAL = AppRoutes. INITIAL;
   static final RouteObserver<Route> observer = RouteObservers();
   static List<String> history = [];

   static final List<GetPage> routes = [
     // If no login, redirect to welcome page.
     GetPage(
       name: AppRoutes. INITIAL,
       page: () => WelcomePage(),
       binding: WelcomeBinding(),
       middlewares: [
         RouteWelcomeMiddleware(priority: 1),
       ],
     ),
     ...
```

## middleware

### Login authentication

By inheriting `GetMiddleware` and overriding the `redirect` method, if you are not logged in, point to the login page.

`lib/common/middlewares/router_auth.dart`

```dart
/// Checks if you are logged in.
class RouteAuthMiddleware extends GetMiddleware {
   // If priority number is smaller , it has higher priority.
   @override
   // Need to ask what the ? in int? does. 
   int? priority = 0;

   RouteAuthMiddleware({required this.priority});

   @override
   // Need to ask what does the redirect here do
   RouteSettings? redirect(String? route) {
     if (UserStore.to.isLogin ||
         route == AppRoutes. SIGN_IN ||
         route == AppRoutes. SIGN_UP ||
         route == AppRoutes. INITIAL) {
       return null;
     } else {
       Future. delayed(
           Duration(seconds: 1), () => Get.snackbar("Prompt", "Login expired, please log in again"));
       return RouteSettings(name: AppRoutes. SIGN_IN);
     }
   }
}
```

### Welcome Screen

If it is the first time to log in, go to the welcome screen, if you have logged in before, you go to the home page, and if you have not logged in, go to the login page.

`lib/common/middlewares/router_welcome.dart`

```dart
/// Welcome page when you first log in.
class RouteWelcomeMiddleware extends GetMiddleware {
   // If priority number is smaller , it has higher priority.
   @override
   int? priority = 0;

   RouteWelcomeMiddleware({required this.priority});

   @override
   RouteSettings? redirect(String? route) {
     if (ConfigStore.to.isFirstOpen == true) {
       return null;
     } else if (UserStore. to. isLogin == true) {
       return RouteSettings(name: AppRoutes.Application);
     } else {
       return RouteSettings(name: AppRoutes. SIGN_IN);
     }
   }
}
```

## Global data

It mainly uses the global mechanism of `GetxService` to encapsulate some functions that need to be initialized and used globally, such as the local persistence here.

`lib/common/services/storage.dart`

```dart
/// Stores data in the storage.dart folder , efficient as data can be stored in 1 place , only needs to be called once to retrieve data
class StorageService extends GetxService {
   static StorageService get to => Get. find();
   late final SharedPreferences_prefs;

   Future<StorageService> init() async {
     _prefs = await SharedPreferences. getInstance();
     return this;
   }
```

> Note the singleton method here `static StorageService get to => Get.find();`
>
> You can use `StorageService.to.xxx` globally in the future

After the definition, complete the necessary initialization before `run man`, some of which can actually be lazy loaded, so as not to get stuck in `io`.

`lib/global.dart`

```dart

class Global {
   /// Initialization of global class.
   static Future init() async {
     WidgetsFlutterBinding.ensureInitialized();
     await SystemChrome.setPreferredOrientations([DeviceOrientation.portraitUp]);

     setSystemUi();
     Loading();

     await Get.putAsync<StorageService>(() => StorageService().init());

     Get. put<ConfigStore>(ConfigStore());
     Get. put<UserStore>(UserStore());
   }

   static void setSystemUi() {
     if (GetPlatform. isAndroid) {
       SystemUiOverlayStyle systemUiOverlayStyle = SystemUiOverlayStyle(
         statusBarColor: Colors.transparent,
         statusBarBrightness: Brightness. light,
         statusBarIconBrightness: Brightness. dark,
         systemNavigationBarDividerColor: Colors.transparent,
         systemNavigationBarColor: Colors. white,
         systemNavigationBarIconBrightness: Brightness. dark,
       );
       SystemChrome.setSystemUIOverlayStyle(systemUiOverlayStyle);
     }
   }
}

```

> The `init` method here is the method we want to execute first in `runApp`
>
> To initialize asynchronously, call `await Get.putAsync<StorageService>(() => StorageService().init());`
>
> Initialize the global object by `Get.put<ConfigStore>(ConfigStore());`

`lib/main.dart`

```dart
Future<void> main() async {
   await Global.init();
   runApp(MyApp());
}

class MyApp extends StatelessWidget {
   @override
   Widget build(BuildContext context) {
     return ScreenUtilInit(
       designSize: Size(375, 812),
       builder: () => RefreshConfiguration(
         headerBuilder: () => ClassicHeader(),
         footerBuilder: () => ClassicFooter(),
         hideFooterWhenNotFull: true,
         headerTriggerDistance: 80,
         maxOverScrollExtent: 100,
         footerTriggerDistance: 150,
         child: GetMaterialApp(
           title: 'News',
           theme: AppTheme. light,
           debugShowCheckedModeBanner: false,
           initialRoute: AppPages. INITIAL,
           getPages: AppPages. routes,
           builder: EasyLoading.init(),
           translations: TranslationService(),
           navigatorObservers: [AppPages. observer],
           localizationsDelegates: [
             GlobalMaterialLocalizations.delegate,
             GlobalWidgetsLocalizations. delegate,
             GlobalCupertinoLocalizations.delegate,
           ],
           supportedLocales: ConfigStore.to.languages,
           locale: ConfigStore.to.locale,
           fallbackLocale: Locale('en', 'US'),
           enableLog: true,
           logWriterCallback: Logger.write,
         ),
       ),
     );
   }
}
```

> I wrote a `main() async` that executes sequentially synchronously
>
> This `MyApp` is more typical, including `ScreenUtilInit` `RefreshConfiguration` `GetMaterialApp` `EasyLoading` `translations` `getPages` `theme` These initials, you can refer to

## Local data persistence

The component `shared_preferences` is used

Encapsulated into a global object `lib/common/services/storage.dart`

```dart
class StorageService extends GetxService {
   static StorageService get to => Get. find();
   late final SharedPreferences_prefs;
  
   // Initilization.
   Future<StorageService> init() async {
     _prefs = await SharedPreferences. getInstance();
     return this;
   }

   // Changing strings in the database.
   Future<bool> setString(String key, String value) async {
     return await _prefs.setString(key, value);
   }
   // Changing bools in the database.
   Future<bool> setBool(String key, bool value) async {
     return await _prefs.setBool(key, value);
   }
   // Changing lists in the database.
   Future<bool> setList(String key, List<String> value) async {
     return await _prefs.setStringList(key, value);
   }
   // Getting specific string from the database.
   String getString(String key) {
     return _prefs. getString(key) ?? '';
   }
   // Getting specific bool from the database.
   bool getBool(String key) {
     return _prefs. getBool(key) ?? false;
   }
   // Getting specific list from the database.
   List<String> getList(String key) {
     return _prefs. getStringList(key) ?? [];
   }
   // Removing data from the database.
   Future<bool> remove(String key) async {
     return await _prefs. remove(key);
   }
}

```

> Singleton access `StorageService.to.setString(xxxx)`

## Data Model

It is recommended that you use the three-party json to model plug-in

I am using [Paste JSON as Code](https://marketplace.visualstudio.com/items?itemName=quicktype.quicktype)

These instance objects are placed in the `lib/common/entities` directory

There is one thing I would like to suggest to everyone, that is, when the api interface is requested, the instance object should also be written to strictly control the type, so as to facilitate troubleshooting, otherwise it will be `map` and it will be difficult for everyone to maintain in the later stage.

Example `lib/common/entities/user.dart`

```dart
/// When a user registers.
class UserRegisterRequestEntity {
   String email;
   String password;

   UserRegisterRequestEntity({
     required this. email,
     required this.password,
   });

   factory UserRegisterRequestEntity. fromJson(Map<String, dynamic> json) =>
       UserRegisterRequestEntity(
         email: json["email"],
         password: json["password"],
       );

   Map<String, dynamic> toJson() => {
         "email": email,
         "password": password,
       };
}

// When a user logs in
class UserLoginRequestEntity {
   String email;
   String password;

   UserLoginRequestEntity({
     required this. email,
     required this.password,
   });

   factory UserLoginRequestEntity. fromJson(Map<String, dynamic> json) =>
       UserLoginRequestEntity(
         email: json["email"],
         password: json["password"],
       );

   Map<String, dynamic> toJson() => {
         "email": email,
         "password": password,
       };
}

// Login return.
class UserLoginResponseEntity {
   String? accessToken;
   String? displayName;
   List<String>? channels;

   UserLoginResponseEntity({
     this. accessToken,
     this.displayName,
     this. channels,
   });

   factory UserLoginResponseEntity. fromJson(Map<String, dynamic> json) =>
       UserLoginResponseEntity(
         accessToken: json["access_token"],
         displayName: json["display_name"],
         channels: List<String>.from(json["channels"].map((x) => x)),
       );

   Map<String, dynamic> toJson() => {
         "access_token": accessToken,
         "display_name": displayName,
         "channels":
             channels == null ? [] : List<dynamic>.from(channels!.map((x) => x)),
       };
}

```

> You can see that `UserRegisterRequestEntity` is the object when requesting

api interface code

```dart
/// User class.
class UserAPI {
   /// If login is successful.
   static Future<UserLoginResponseEntity> login({
     UserLoginRequestEntity? params,
   }) async {
     // Need to ask what this is for
     var response = await HttpUtil(). post(
       '/user/login',
       data: params?.toJson(),
     );
     return UserLoginResponseEntity. fromJson(response);
   }
```

> It can be seen that the input and output of this interface have been packaged, so that it is very convenient to debug later with strong typing.

## http pull data

I didn't use `GetConnect`, but used `dio`, mainly for robustness.

All operations are still encapsulated in `lib/common/utils/http.dart`

> I will subsidize the code, it is too long, please read it yourself
>
> Encapsulates common restful operations `get` `post` `put` `delete` `patch`
>
> Added `postForm` `postStream` for individual server components
>
> Error handling `onError` lists common errors

## User login logout & 401

When you log out, you need to clear the local cache, such as `token` `profile` data.

Specific code can refer to `lib/common/store/user.dart`

```dart
   // When user logs out.
   Future<void> onLogout() async {
     if (_isLogin. value) await UserAPI. logout();
     // Clears users cache.
     await StorageService.to.remove(STORAGE_USER_TOKEN_KEY);
     _isLogin. value = false;
     token = '';
   }
```

Let’s talk about 401. This is the unauthorized status returned by the server. After we get it, we need to pop up the login interface.

This operation can be placed in dio's error handling `lib/common/utils/http.dart`

```dart
// Error handling.
void onError(ErrorEntity eInfo) {
     print('error. code -> ' +
         eInfo.code.toString() +
         ', error. message -> ' +
         eInfo. message);
     switch (eInfo. code) {
       case 401:
         UserStore.to.onLogout();
         EasyLoading.showError(eInfo.message);
         break;
       default:
         EasyLoading.showError('Unknown error');
         break;
     }
   }
```

> Once the `eInfo.code` is found to be `401`, the onLogout operation will be performed directly, and a message prompt will pop up.

`ErrorEntity` is the encapsulated error message format

```dart
/// Error message.
   ErrorEntity createErrorEntity(DioError error) {
     switch (error. type) {
       case DioErrorType. cancel:
         return ErrorEntity(code: -1, message: "Request to cancel");
       case DioErrorType. connectTimeout:
         return ErrorEntity(code: -1, message: "Connection timed out");
       case DioErrorType. sendTimeout:
         return ErrorEntity(code: -1, message: "Request timed out");
       case DioErrorType.receiveTimeout:
         return ErrorEntity(code: -1, message: "Response timed out");
       case DioErrorType. cancel:
         return ErrorEntity(code: -1, message: "Request to cancel");
       case DioErrorType. response:
         {
           try {
             int errCode =
                 error.response != null ? error.response!.statusCode! : -1;
             // String errMsg = error. response. statusMessage;
             // return ErrorEntity(code: errCode, message: errMsg);
             switch (errCode) {
               case 400:
                 return ErrorEntity(code: errCode, message: "Request syntax error");
               case 401:
                 return ErrorEntity(code: errCode, message: "No permission");
               case 403:
                 return ErrorEntity(code: errCode, message: "The server refuses to execute");
               case 404:
                 return ErrorEntity(code: errCode, message: "Unable to connect to server");
               case 405:
                 return ErrorEntity(code: errCode, message: "The request method is forbidden");
               case 500:
                 return ErrorEntity(code: errCode, message: "Internal server error");
               case 502:
                 return ErrorEntity(code: errCode, message: "Invalid request");
               case 503:
                 return ErrorEntity(code: errCode, message: "The server is down");
               case 505:
                 return ErrorEntity(code: errCode, message: "HTTP protocol request is not supported");
               default:
                 {
                   // return ErrorEntity(code: errCode, message: "Unknown error");
                   return ErrorEntity(
                     code: errCode,
                     message: error. response != null
                         ?error.response!.statusMessage!
                         : "",
                   );
                 }
             }
           } on Exception catch (_) {
             return ErrorEntity(code: -1, message: "Unknown error");
           }
         }
       default:
         {
           return ErrorEntity(code: -1, message: error. message);
         }
     }
   }
```

## Dynamic permissions

If the permission check is involved on the front end, you can still write it in the routing middleware. Once you find a routing change, go to the authentication to see if you have permission. The user's permission can then pull `rules` in `profile` such information.

```dart
class AuthorityMiddleware extends GetMiddleware {
   // If priority number is smaller , it has higher priority.
   @override
   int? priority = 0;

   Authority Middleware({required this.priority});

   @override
   RouteSettings? redirect(String? route) {
      …
     implement here
   }
}
```

## User data

This requires globalizing `lib/common/store/user.dart`

Maintain the status of whether the user's `token` `profile` is logged in or not.

```dart
class UserStore extends GetxController {
   static UserStore get to => Get. find();

   // Whether to log in.
   final_isLogin = false. obs;
   // Token for cache.
   String token = '';
   // User's profile.
   final_profile = UserLoginResponseEntity().obs;

   bool get isLogin => _isLogin. value;
   UserLoginResponseEntity get profile => _profile. value;
   bool get hasToken => token.isNotEmpty;

   @override
   void onInit() {
     super.onInit();
     token = StorageService.to.getString(STORAGE_USER_TOKEN_KEY);
     var profileOffline = StorageService.to.getString(STORAGE_USER_PROFILE_KEY);
     if (profileOffline. isNotEmpty) {
       _profile(UserLoginResponseEntity. fromJson(jsonDecode(profileOffline)));
     }
   }

   // Saving the users token.
   Future<void> setToken(String value) async {
     await StorageService.to.setString(STORAGE_USER_TOKEN_KEY, value);
     token = value;
   }

   // Getting users profile.
   Future<void> getProfile() async {
     if (token. isEmpty) return;
     var result = await UserAPI. profile();
     _profile(result);
     _isLogin. value = true;
     // Retrieves users profile via JSON from API. 
     StorageService.to.setString(STORAGE_USER_PROFILE_KEY, jsonEncode(result));
   }

   // Saving users profile.
   Future<void> saveProfile(UserLoginResponseEntity profile) async {
     _isLogin. value = true;
     // Saves users profile by changing it via API
     StorageService.to.setString(STORAGE_USER_PROFILE_KEY, jsonEncode(profile));
   }

   // Log out function.
   Future<void> onLogout() async {
     if (_isLogin. value) await UserAPI. logout();
     // Removing users token / cache.
     await StorageService.to.remove(STORAGE_USER_TOKEN_KEY);
     _isLogin. value = false;
     token = '';
   }
}

```

## Some common mistakes

### The network has no permission under macOS

error message

```
SocketException: Connection failed (OS Error: Operation not permitted, errno = 1)
```

solve

`macos/Runner/DebugProfile.entitlements`

```
<key>com.apple.security.network.client</key>
<true/>
```

end

## From Raphael , What I need help with
CTRL + f "// Need to ask "
