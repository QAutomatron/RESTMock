# RESTMock
[![](https://jitpack.io/v/andrzejchm/RESTMock.svg)](https://jitpack.io/#andrzejchm/RESTMock)
[![Android Arsenal](https://img.shields.io/badge/Android%20Arsenal-RESTMock-green.svg?style=true)](https://android-arsenal.com/details/1/3468) [![Circle CI](https://circleci.com/gh/andrzejchm/RESTMock.svg?style=svg)](https://circleci.com/gh/andrzejchm/RESTMock)

REST API mocking made easy.
##About
RESTMock is a library working on top of Square's [okhttp/MockWebServer](https://github.com/square/okhttp/tree/master/mockwebserver). It allows you to specify [Hamcrest](https://github.com/hamcrest/JavaHamcrest) matchers to match HTTP requests and specify what response to return. It is as easy as:

```java
RESTMockServer.whenGET(pathContains("users/defunkt"))
            .thenReturnFile(200, "users/defunkt.json");
```
**Article**
- [ITDD - Instrumentation TDD for Android](https://medium.com/@andrzejchm/ittd-instrumentation-ttd-for-android-4894cbb82d37)

##Table of Contents
- [About](#about)
- [Setup](#setup)
- [Request verification](#request-verification)
- [Logging](#logging)
- [Unit Tests with Robolectric](#unit-tests-with-robolectric)
- [Android Sample Project](#android-sample-project)
- [TODO](#todo)
- [License](#license)

## Setup
Here are the basic rules to set up RESTMock for Android

####Step 1: Repository
Add it in your root build.gradle at the end of repositories:

```groovy  
allprojects {
	repositories {
		...
		maven { url "https://jitpack.io" }
	}
}
```
####Step 2: Dependencies
Add the dependency

```groovy  
dependencies {
	androidTestCompile 'com.github.andrzejchm.RESTMock:android:0.1.2'
}
```

####Step 3: Start the server
It's good to start server before the tested application starts, there are few methods:

##### a) RESTMockTestRunner
To make it simple you can just use the predefined `RESTMockTestRunner` in your UI tests. It extends `AndroidJUnitRunner`:

```groovy
defaultConfig {
		...
    	testInstrumentationRunner 'io.appflate.restmock.android.RESTMockTestRunner'
    }
```
##### b) RESTMockServerStarter
If you have your custom test runner and you can't extend `RESTMockTestRunner`, you can always just call the `RESTMockServerStarter`. Actually `RESTMockTestRunner` is doing exactly the same thing:

```java
public class MyAppTestRunner extends AndroidJUnitRunner {
	...
	@Override
	public void onCreate(Bundle arguments) {
		super.onCreate(arguments);
		RESTMockServerStarter.startSync(new AndroidAssetsFileParser(getContext()),new AndroidLogger());
		...
	}
	...
}

```


####Step 4: Specify Mocks

#####a) Files
By default, the `RESTMockTestRunner` uses `AndroidAssetsFileParser` as a mocks file parser, which reads the files from the assets folder. To make them visible for the RESTMock you have to put them in the correct folder in your project, for example:

	.../src/androidTest/assets/users/defunkt.json
This can be accessed like this:

```java
RESTMockServer.whenGET(pathContains("users/defunkt"))
            .thenReturnFile(200, "users/defunkt.json");
```

#####b) Strings
If the response You wish to return is simple, you can just specify a string:

```java
RESTMockServer.whenGET(pathContains("users/defunkt"))
            .thenReturnString(200, "{}");
```
#####c) MockResponse
If you wish to have a greater control over the response, you can pass the `MockResponse`
```java
RESTMockServer.whenGET(pathContains("users/defunkt")).thenReturn(new MockResponse().setBody("").setResponseCode(401).addHeader("Header","Value"));
```

####Step 5: Request Matchers
You can either use some of the predefined matchers from `RequestMatchers` util class, or create your own. remember to extend from `RequestMatcher`

####Step 6: Specify API Endpoint
The most important step, in order for your app to communicate with the testServer, you have to specify it as an endpoint for all your API calls. For that, you can use the ` RESTMockServer.getUrl()`. If you use Retrofit, it is as easy as:

```java
RestAdapter adapter = new RestAdapter.Builder()
                .baseUrl(RESTMockServer.getUrl())
                ...
                .build();
```
##Request verification
It is possible to verify which requests were called and how many times thanks to `RequestsVerifier`. All you have to do is call one of these:

```java
//cheks if the GET request was invoked exactly 2 times
RequestsVerifier.verifyGET(pathEndsWith("users")).exactly(2);

//cheks if the GET request was invoked at least 3 times
RequestsVerifier.verifyGET(pathEndsWith("users")).atLeast(3);

//cheks if the GET request was invoked exactly 1 time
RequestsVerifier.verifyGET(pathEndsWith("users")).invoked();

//cheks if the GET request was never invoked
RequestsVerifier.verifyGET(pathEndsWith("users")).never();
```

##Logging
RESTMock supports logging events. You just have to provide the RESTMock with the implementation of `RESTMockLogger`. For Android there is an `AndroidLogger` implemented already. All you have to do is use the `RESTMockTestRunner` or call

```java
RESTMockServerStarter.startSync(new AndroidAssetsFileParser(getContext()),new AndroidLogger());
```

or

```java
RESTMockServer.enableLogging(RESTMockLogger)
RESTMockServer.disableLogging()
```

## Unit Tests with Robolectric
If you want to write unit tests (no emulator or device necessary), you can use [Robolectric](http://robolectric.org)
to accomplish this. There is a sample project with Robolectric tests in [androidsample](androidsample/).

One change you will need to make is to the file parser that you use. Since Robolectric doesn't
create an actual device/emulator environment, you will need to use the local file system in lieu of
`getAssets()`. A file parser for this has been provided for you, called, `AndroidLocalFileParser`.
The `AndroidLocalFileParser` will look for files in the `resources/` directory, which should be a
child of `/test/`.
```
RESTMockServerStarter.startSync(new AndroidLocalFileParser(application),new AndroidLogger());
```

It is necessary to pass in the `application` variable at construction time, so that the Robolectric
runner knows where to find the base path for files. You can retrieve the application at runtime
using `RuntimeEnvironment.application` from within a Robolectric test.

## Android Sample Project
You can check out the sample Android app with tests [here](androidsample/)


##TODO
* Create API responses recorder that will store the responses in assets
* ~~setup CI~~
* ~~create some unit-tests~~
* ~~add something similar to Mockito's `verify()`~~
* ~~add android example~~

##License

	Copyright (C) 2016 Appflate.io

 	Licensed under the Apache License, Version 2.0 (the "License");
 	you may not use this file except in compliance with the License.
 	You may obtain a copy of the License at

	http://www.apache.org/licenses/LICENSE-2.0

	Unless required by applicable law or agreed to in writing, software
	distributed under the License is distributed on an "AS IS" BASIS,
	WITHOUT WARRANTIES OR CONDITIONS OF ANY KIND, either express or implied.
	See the License for the specific language governing permissions and
	limitations under the License.
