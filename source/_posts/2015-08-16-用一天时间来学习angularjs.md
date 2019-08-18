---
layout: lay_post
title: 用一天时间来学习angularjs
date: 2015-08-16 12:10:12
categories: 前端
tags: AngularJS
author: lvyafei
published: true
summary: ["/images/angularjs.jpg"]
---

* 目录
{:toc #meuid}

对于学习angulaJS来说，更多的是对整体的了解。看了Todd Motto 的一篇文章后，对这个框架有了一个比较清晰的认识。
[查看原文](http://toddmotto.com/ultimate-guide-to-learning-angular-js-in-one-day/)
<!-- more -->

## I.什么是AngularJS?

![](/images/angularjs.jpg)

Angular 是一个客户端MVC/MVVM框架，内置在JavaScript中，是现在流行的单页面web应用（甚至是网站）必不可少的东西。这篇文章是我学习angularjs的全部的经验，建议和最佳的做法。通过掌握它你会学会很多东西。

## II.Terminology(术语)

Angular 有一个短的学习路线，掌握了基本知识后你会发现它的优点和缺点，对学习这个框架来说，掌握这些术语和学会用MVC思考是非常重要的，下面是一些Angular中包含的一些高级并且重要的api和术语。

## III.MVC

你可能听说过MVC，用在许多编程语言中来构建应用程序/软件/架构的一种手段。这里是每个部分的简单含义：

**Model**：每一个应用中都离不开数据结构，通常放在JSON中。在开始Angular之前先要读懂JSON，因为它对了解server和view是如何通信的非常重要。例如，一组用户身份标识可以有以下模型：

    {
      "users" : [{
      "name": "Joe Bloggs",
      "id": "82047392"
    },{
      "name": "John Doe",
      "id": "65198013"
     }]
    }

然后你就可以从服务器通过XHR抓取信息（XMLHTTP请求），在Jquery中是通过$.ajax方法，在Angular中是$http,或者它会写进你的代码在页面解析（从一个数据仓库/数据库）时，然后你可以推送更新到你的模型，并把它发送回。

**View**: View是非常容易理解的，它是你的HTML页面或呈现的输出。使用MVC框架，你会使用Model数据来更新View和HTML中显示的相关数据。

**Controller**：Controller直接访问服务器中的View，作为中间人，你能够在Controller中更新Model中的数据来改变View。

## 0.建立一个AngularJS项目（基本要点）

首先，我们需要确认安装了AngularJS必备的基础环境，在我们开始前有一些事情是需要注意的，一般由ng-app声明来定义你的应用程序,一个Controller和View页面、一些DOM邦定还有Angular来交互。以下是基本要点：

一些带“ng-*” 声明的HTML

    <div ng-app="myApp">
      <div ng-controller="MainCtrl">
          <!-- controller logic -->
       </div>
    </div>
    
Angular的 Module 和 Controller:

	var myApp = angular.module('myApp', []);

	myApp.controller('MainCtrl', ['$scope', function ($scope) {
 	   // Controller magic
	}]);
    
在开始动手之前，我们需要创建一个包含所有逻辑的Angular module,有很多种方法来声明 modules,你可以将所有的逻辑像这样串联起来（我不喜欢这种做法）：

	angular.module('myApp', [])
	.controller('MainCtrl', ['$scope', function ($scope) {...}])
	.controller('NavCtrl', ['$scope', function ($scope) {...}])
	.controller('UserCtrl', ['$scope', function ($scope) {...}]);
    
在我的使用中，建立一个全局的Module被证明是一个非常好的方式。缺少分号和意外关闭的'连接'证明适得其反，会出现一些不必要的编译错误，像这样：

	var myApp = angular.module('myApp', []);
	myApp.controller('MainCtrl', ['$scope', function ($scope) {...}]);
	myApp.controller('NavCtrl', ['$scope', function ($scope) {...}]);
	myApp.controller('UserCtrl', ['$scope', function ($scope) {...}]);
    
我创建的每一文件都使用简单的myApp命名空间并自动的邦定到Application上，没错，我会为每个Controller, Directive, Factory 和其它的内容创建一个文件，关联和压缩这些文件到一个脚本文件中，然后引入到DOM中（使用Grunt或Gulp类似的工具）

## 1.Controllers(控制器)
 
现在你已经掌握了MVC设计模式的概念和基本设置，让我们看看Angular中是如何运用Controllers的。
从上面的例子中我们可以很简单的从一个Controller中拿到数据然后放入到DOM中，Angular采用一种模板风格 {{ handlebars }} 嵌在你的HTML中，你的HTML应该（最好）不包含物理文本或硬编码的值以最大限度地利用Angular，下面是一个将字符串推送到页面上的例子：

    <div ng-app="myApp">
	  <div ng-controller="MainCtrl">
          { { text } }
  	  </div>
    </div>

脚本：

	var myApp = angular.module('myApp', []);
	
	myApp.controller('MainCtrl', ['$scope', function ($scope) {
    
    	$scope.text = 'Hello, Angular fanatic.';
    
	}]);



演示：http://runjs.cn/code/jgevzqjd

这里的核心法则是“$scope”的概念，你会在一个特定的Controller里面邦定所有的功能。$scope代表的是DOM中当前的元素，封装一个非常好的观察能力，保持元素范围内的数据和逻辑完完美邦定。它提供了JavaScript的公共/私有范围的DOM，这很神奇。

$scope的范围概念似乎有点可怕，但这是你从服务器到DOM的交互方式（静态数据，如果你有过）！这个例子教你了一个如何把数据“推”到DOM的办法。

让我们看一个比较有代表性的数据结构，我们假设从服务器检索显示用户的登录信息。现在我将使用静态数据；我后边会告诉你如何获取动态JSON数据。

首先我们要编写JavaScript：

    var myApp = angular.module('myApp', []);

    myApp.controller('UserCtrl', ['$scope', function ($scope) {
    
        // Let's namespace the user details
        // Also great for DOM visual aids too
        $scope.user = {};
        $scope.user.details = {
            "username": "Todd Motto",
            "id": "89101112"
        };
    
    }]);
    
然后交给DOM来显示这些数据：

    <div ng-app="myApp">
        <div ng-controller="UserCtrl">
            <p class="username">Welcome, { { user.details.username } }</p>
            <p class="id">User ID: { { user.details.id } }</p>
        </div>
    </div>
    
演示：http://runjs.cn/code/4xrlzycm

重要的一点要记住，控制器只为数据，并创建函数（也包括事件函数）！它跟服务器交互并且推/拉JSON数据。这里没有DOM操作，所以不需要jQery工具。指令(Directives )是DOM操作，这是下一个要介绍的。

小提示：在Angular的官方文档中（我在写这篇文章时）他们的例子中使用这种方式创建Controllers：

    var myApp = angular.module('myApp', []);

    function MainCtrl ($scope) {
    //...
    };
    
…不要这样做。这将所有的功能都暴露在全局范围内，并不会让它们与应用程序很好地联系在一起。这也意味着你不能很容易的缩小你的代码或运行测试。不要滥用全局名称空间，要保证Controller只在您的应用程序内。


## 2.Directives(指令)

指令最简化形式是一段HTML模板，在应用程序需要的地方最好能多次使用。这是毫不费力的注入DOM到应用中最简单的方式，或执行自定义DOM交互。指令并不简单，想完全征服他们需要一个非常令人难以置信的学习路线，但通过这一小节的学习会让你基本掌握它。

那指令有什么用处？很多事情，包括DOM组件，例如制表符或导航元素会依赖你的应用程序中使用的UI。让我这样说吧，如果你有过ng-show或者ng-hide，这就是指令（虽然他们不注入DOM）。

对于这个练习，我想把它弄的很简单，创建一个自定义按钮（称为CustomButton）注入一些标记，我讨厌写很多字。有各种不同的方式定义DOM中的指令，这些可能看起来像这样：

    <!-- 1: as an attribute declaration -->
    <a custom-button>Click me</a>

    <!-- 2: as a custom element -->
    <custom-button>Click me</custom-button>

    <!-- 3: as a class (used for old IE compat) -->
    <a class="custom-button">Click me</a>

    <!-- 4: as a comment (not good for this demo, however) -->
    <!-- directive: custom-button -->
    
我更喜欢他们作为一个属性使用，自定义元素作为HTML5未来的一个Web组件特性，但Angular指出这在一些老的浏览器上会当成是一个BUG。

现在你知道在使用/注入指令的时候如何声明了吧，让我们创建这个自定义按钮。再次，我会回调我的全局命名空间MyApp的应用，这是最简形式的指令：

    myApp.directive('customButton', function () {
      return {
        link: function (scope, element, attrs) {
          // DOM manipulation/events here!
        }
      };
    });
    
我使用 .directive()方法来创建指令，传入指令的名称“CustomButton“。当你在指令的名称中使用一些字母，它会通过DOM中的连字符进行分割（如上图）。

一个指令简单的返回一个带有参数的对象。对我来说最重要的是：restrict, replace, transclude, template 和 templateUrl,当然还有 link 属性。让我把其它的也加入进来吧：

    myApp.directive('customButton', function () {
      return {
        restrict: 'A',
        replace: true,
        transclude: true,
        template: '<a href="" class="myawesomebutton" ng-transclude>' +
                    '<i class="icon-ok-sign"></i>' +
                  '</a>',
        link: function (scope, element, attrs) {
          // DOM manipulation/events here!
        }
      };
    });
    
演示： http://runjs.cn/code/xtupcxdh

验证你是否插入元素，请确认你是否注入附加标记。指令的属性解释：

**restrict**：[约束]看下上面的例子我们是怎么使用restrict的，如果你的项目需要支持IE的话，你可能需要声明attribute或class。使用“A”意味着你限制它作为一个Attribute，E代表Element，C代表Class，M代表Comment。默认是“EA”，对的，你可以限制多个。

**replace**：[替换]替换DOM为指令中定义的内容，在这个例子中，你会注意到初始DOM已经被指令的模板替换。

**transclude**：[嵌入]简单地说，使用嵌入允许现有的DOM内容被复制到指令中。你会看到单词'点击我'已被'移'到指令中，在显示的时候。

**template**：[模板]模板（如上）允许你将标记为被注入。这是一个好主意，用这一小段HTML片断。注入模板都通过Angular编译，这意味着你可以定义处理的方法已更好的邦定使用。

**templateUrl**：[模板Url]类似于一个模板，但保持在它自己的文件或<脚本>标签。你可以指定一个模板的URL，你会使用一个单独的文件来管理HTML模板，请指定路径和文件名，最好放在自己的模板目录中：

    myApp.directive('customButton', function () {
      return {
        templateUrl: 'templates/customButton.html'
        // directive stuff...
      };
    });
    
加入你的文件（文件名不敏感）：

    <!-- inside customButton.html -->
    <a href="" class="myawesomebutton" ng-transclude>
      <i class="icon-ok-sign"></i>
    </a>
    
这样做真的很好，浏览器会缓存HTML文件，干的漂亮！设置不缓存的另一个选择是在script标签中声明一个模板：

    <script type="text/ng-template" id="customButton.html">
    <a href="" class="myawesomebutton" ng-transclude>
      <i class="icon-ok-sign"></i>
    </a>
    </script>
    
你会告诉Angular它是一个模板并且还有一个ID，Angular会查找NG模板或*.HTML文件,他们很容易管理，提高性能和保持DOM很干净，你可以建立1个或100个指令，你可以很容易的浏览它们。

## 3.Services(服务)

服务通常是混淆点。从经验和研究上，他们更多的是一种风格的设计模式，而不是提供更多的功能差异。在深入到Angular源码后,他们指望通过相同的编译器运行，并共享大量的功能。从我的研究来看，你应该在单例，工厂以及其它更复杂的功能上使用服务，如对象字面量和更复杂的用例。

下面是一个Service例子,计算2个数字的乘积：

     myApp.service('Math', function () {
          this.multiply = function (x, y) {
              return x * y;
         };
    });
    
然后，你应该使用这样的内部控制器：

    myApp.controller('MainCtrl', ['$scope', function ($scope) {
        var a = 12;
        var b = 24;
       // outputs 288
        var result = Math.multiply(a, b);
    }]);
    
是的，乘法是很容易的，不需要一个服务，但你领会到了其中的要点。

当你创建一个服务（或工厂）时，你需要使用依赖注入来告诉Angular需要扑获新的服度，否则你会得到编译错误，你的控制器会中断。你可能已经注意到了这个函数($scope)的部分，现在，这是简单的依赖注入。看下你的代码,你也会注意到（$scope）之前的功能（$scope），我会到后面解释。这里是如何使用依赖注入来告诉你需要你的服务：

    // Pass in Math
    myApp.controller('MainCtrl', ['$scope', 'Math', function ($scope, Math) {
      var a = 12;
      var b = 24;
      // outputs 288
      var result = Math.multiply(a, b);
    }]);
    
## 4.Factories(工厂)

从服务到工厂现在应该是简单的，我们可以在工厂里创建一个对象字面量或简单地提供一些更深入的方法：

	myApp.factory('Server', ['$http', function ($http) {
	  return {
	    get: function(url) {
	      return $http.get(url);
	    },
	    post: function(url) {
	      return $http.post(url);
	    },
	  };
	}]);
	
下面，在依赖注入到一个控制器后，我创造了一个Angular的XHR自定义包装。这个例子很简单：

	myApp.controller('MainCtrl', ['$scope', 'Server', function ($scope, Server) {
	    var jsonGet = 'http://myserver/getURL';
	    var jsonPost = 'http://myserver/postURL';
	    Server.get(jsonGet);
	    Server.post(jsonPost);
	}]);
	
如果你想为了改变而轮询服务，你可以设置一个Server.poll(jsonPool),或者假如你正在使用一个Socket,你可以设置Server.socket(jsonSocket).它打开了大门，模块化代码以及创造您使用和保持代码内的控制器，作为一个最小的完整集合。

## 5.Filters(过滤器)
    
过滤器一般和数组一块使用，用在循环外，如果你遍历数组来筛选出特定的东西，在正确的地方，你也可以使用过滤器过滤用户想要的类型在<input>标签为例，有几个方法可以使用过滤器，在控制器里面或作为一个定义的方法。这里有使用的方法，您可以在全局使用：

	myApp.filter('reverse', function () {
	    return function (input, uppercase) {
	        var out = '';
	        for (var i = 0; i < input.length; i++) {
	            out = input.charAt(i) + out;
	        }
	        if (uppercase) {
	            out = out.toUpperCase();
	        }
        return out;
	    }
	});

	// Controller included to supply data
	myApp.controller('MainCtrl', ['$scope', function ($scope) {
	    $scope.greeting = 'Todd Motto';
	}]);
	
DOM内容：

	<div ng-app="myApp">
	    <div ng-controller="MainCtrl">
	        <p>No filter: { { greeting } }</p>
	        <p>Reverse: { { greeting | reverse } }</p>
	    </div>
	</div>
	
例子：http://runjs.cn/code/ina2n1lb

可以看到我们通过传递过滤符号“|”在greeting 数据上，对该数据进行逆向过滤。

在标签ng-repeat中的用法为：

	<ul>
	  <li ng-repeat="number in myNumbers |filter:oddNumbers">{{ number }}</li>
	</ul>

一个在controller中使用过滤器的简明例子：

	myApp.controller('MainCtrl', ['$scope', function ($scope) {
		$scope.numbers = [10, 25, 35, 45, 60, 80, 100];
		$scope.lowerBound = 42;
		// Does the Filters
		$scope.greaterThanNum = function (item) {
		    return item > $scope.lowerBound;
		};
	}]);
	
在ng-repeat标签中的用法为:

	<li ng-repeat="number in numbers | filter:greaterThanNum">
	  {{ number }}
	</li>
	
演示:http://runjs.cn/code/xiricwi3

这是AngularJS中的主体部分和API，虽然这只是冰山一角，但利用这些内容足以建立起你自己的angular应用。

## 6.Two-way data-binding(双向数据绑定)

当我第一次听说双向数据绑定时，我并不太明白它是什么。双向数据绑定是最好的理解为一个完全同步的数据循环：更新模型并更新视图，更新视图并更新模型。这意味着数据不需要做任何操作都是同步的。如果我将一个ng模型绑定到一个<input>标签然后开始输入，这将在同一时间创建（或更新一个现有的）模型。

我创建的一个<input>标签并绑定模型称为"myModel"，然后我可以利用angularjs中的语法来反射这个模型并同时更新视图：

	<div ng-app="myApp">
	    <div ng-controller="MainCtrl">
	        <input type="text" ng-model="myModel" placeholder="Start typing..." />
	        <p>My model data: {{ myModel }}</p>
	    </div>
	</div>
	
	myApp.controller('MainCtrl', ['$scope', function ($scope) {
	    // Capture the model data
	    // and/or initialise it with an existing string
	    $scope.myModel = '';
	}]);
	
示例：http://runjs.cn/code/fu074xjf

## 7.XHR/Ajax/$http calls and binding JSON

你已经掌握了如何把一个基础数据邦定到$scope上，并大致了解了这个model是如何工作的，包括数据的双向邦定。
现在是时候学习一些真正的XHR去调用一个服务。对于网站来说，这是没有必要的除非你有特别的Ajax要求，这主要是集中在抓取数据的Web应用程序。

当你在做本地开发时，你可能使用类似Java，ASP.NET，PHP或别的东西来运行一个本地服务器。无论您是和本地数据库通信还是实际使用服务器作为接口来与其他资源进行通信，这都是相同的。

输入$http,就可以愉快的开始了。$HTTP方法是从服务器访问数据的一个很好的Angular包装，你可以很方便的使用。这里是一个“GET”请求的简单例子，它从服务器获取数据。它的语法很像是jQuery，所以很好理解：

	myApp.controller('MainCtrl', ['$scope', '$http', function ($scope, $http) {
	  $http({
		method: 'GET',
		url: '//localhost:9000/someUrl'
	  });
	}]);
	
Angular然后返回内容(叫做承诺)，这是处理回调的一种更有效和可读的方式。承诺是链接function到他们从使用点.mypromise()。正如预期的那样，我们有错误和成功的处理程序：

	myApp.controller('MainCtrl', ['$scope', function ($scope) {
	  $http({
		method: 'GET',
		url: '//localhost:9000/someUrl'
	  })
	  .success(function (data, status, headers, config) {
		// successful data retrieval
	  })
	  .error(function (data, status, headers, config) {
		// something went wrong :(
	  });
	}]);
	
非常好的可读性。这就是我们合并视图和绑定模式或更新模型数据到DOM的服务器。让我们假设一个设置和推动一个用户名的DOM，通过Ajax调用。

理想情况下，我们应该首先建立和设计我们的JSON，这将影响我们如何构造我们的数据。让我们保持简单，这是一个后端开发者将开放一个API在应用程序中，您希望如下：

	{
	  "user": {
		"name": "Todd Motto",
		"id": "80138731"
	  }
	}
	
这意味着我们会得到一个从服务器返回的对象（化名就叫‘数据’[你可以看到数据被传递到我们的承诺处理]），并嵌有data.user属性。在data.user属性里面，有name和id,访问这些很容易的，我们需要寻找data.user.name这个属性时，它将返回这个属性的值。现在让我们来操作下吧！

JavaScript代码：

	myApp.controller('UserCtrl', ['$scope', '$http', function ($scope, $http) {
	  // create a user Object
	  $scope.user = {};
	  // Initiate a model as an empty string
	  $scope.user.username = '';
	  // We want to make a call and get
	  // the person's username
	  $http({
		method: 'GET',
		url: '//localhost:9000/someUrlForGettingUsername'
	  })
	  .success(function (data, status, headers, config) {
		// See here, we are now assigning this username
		// to our existing model!
		$scope.user.username = data.user.name;
	  })
	  .error(function (data, status, headers, config) {
		// something went wrong :(
	  });
	}]);

DOM代码：

	<div ng-controller="UserCtrl">
	  <p>{{ user.username }}</p>
	</div>
	
它会打印username的值。下面我们将进一步了解声明式数据绑定，这是非常令人兴奋的。

## 8.Declarative data-binding(声明式数据邦定)

Angular的理念是创建动态HTML.它是一个强大的功能,代替了你在客户端做的大量工作。这正是他的目的所在。

让我们想象一下，我们刚刚作了一个Ajax请求得到一些电子邮件的列表包含主题，发送日期，现在想把它们展示在DOM中。这里会体现出Angular的优势。首先，我需要设置一个电子邮件的controller：

	myApp.controller('EmailsCtrl', ['$scope', function ($scope) {
	  // create a emails Object
	  $scope.emails = {};
	  // pretend data we just got back from the server
	  // this is an ARRAY of OBJECTS
	  $scope.emails.messages = [{
		    "from": "Steve Jobs",
		    "subject": "I think I'm holding my phone wrong :/",
		    "sent": "2013-10-01T08:05:59Z"
		},{
		    "from": "Ellie Goulding",
		    "subject": "I've got Starry Eyes, lulz",
		    "sent": "2013-09-21T19:45:00Z"
		},{
		    "from": "Michael Stipe",
		    "subject": "Everybody hurts, sometimes.",
		    "sent": "2013-09-12T11:38:30Z"
		},{
		    "from": "Jeremy Clarkson",
		    "subject": "Think I've found the best car... In the world",
		    "sent": "2013-09-03T13:15:11Z"
		}];
	}]);
	
现在我们需要把它插入到我们的HTML中。这时我们会使用声明式绑定宣布什么应用程序将创建我们的第一个动态HTML。我们要用到Angular的内置指令ng-repeat，它会遍历数据，完全没有回调或状态变化的渲染输出，这都是非常灵活的：

	<ul>
	  <li ng-repeat="message in emails.messages">
		<p>From: {{ message.from }}</p>
		<p>Subject: {{ message.subject }}</p>
		<p>{{ message.sent | date:'MMM d, y h:mm:ss a' }}</p>
	  </li>
	</ul>
	
示例：http://runjs.cn/code/q9beqc9s

我使用了一个日期过滤器，从中你可以看到如何处理UTC日期。

深入Angular的ng-*指令,你将完全释放出angular的声明式绑定的能力，这里展示了如何从服务器到模型，以及模拟和渲染数据。


## 9.Scope functions(范围函数)

作为声明性绑定的延续，范围函数是在创建应用程序时的下一个层次。这里展示了一个基本功能，删除我们的电子邮件中的一个数据：

	myApp.controller('MainCtrl', ['$scope', function ($scope) {
	  $scope.deleteEmail = function (index) {
		$scope.emails.messages.splice(index, 1)
	  };
	}]);
	
友情提示：重要的一点是要考虑从模型中删除数据。不删除元素或任何实际的DOM相关，angular是一个MVC框架，将为您处理这一切与它的双向绑定和回调的自由世界，你只需要设置你的代码可以让它响应你的数据！

	<a ng-click="deleteEmail($index)">Delete email</a>
	
这是一种不同的内联点击处理程序，因为许多原因。这将很快覆盖。你会看到我也在通过$index，angular知道你正在删除的项目（这样可以节省你多少代码和逻辑！）。

示例：http://runjs.cn/code/jm84qnh3

## 10.Declarative DOM methods(陈述性DOM方法)

现在我们要转移到的DOM方法，这些也都是些指令和模拟功能的DOM，你通常会写出更多的逻辑脚本。这个例子是一个简单的切换导航。用一个简单的ng-show和一个简单的ng-click设置，我们可以创建一个完美无瑕的切换导航：

	<a href="" ng-click="toggle = !toggle">Toggle nav</a>
	<ul ng-show="toggle">
	  <li>Link 1</li>
	  <li>Link 2</li>
	  <li>Link 3</li>
	</ul>
	
我们开始进入MVVM的学习，你会发现脚本里没有controller的代码，我们会很快进入MVVM。

示例：http://runjs.cn/code/r5tlnqrb

## 11.Expressions(表达式)

我最喜欢Angular的一部分是，你会用JavaScript写出很多重复的代码。

你尝试过这个吗？

	elem.onclick = function (data) {
	  if (data.length === 0) {
		otherElem.innerHTML = 'No data';
	  } else {
		otherElem.innerHTML = 'My data';
	  }
	};
	
这可能是一个GET请求回调，你会改变基于数据的DOM。Angular给你这个自由，你就可以做到无需编写任何内置的JavaScript！

	<p>{{ data.length > 0 && 'My data' || 'No data' }}</p>

这会动态更新自己而没有回调应用程序获取数据。如果没有数据，它会告诉你，如果有数据，它会显示数据。针对Angular处理自动双向绑定数据，会有很多的使用情况。

示例：http://runjs.cn/code/ahl0hfcc

## 12.Dynamic views and routing(动态视图和路由)

单页面web应用(也包括网站)背后的理念是: 你有一个标题，页脚，侧边栏，和中间的内容,通过设置URL可以神奇地注入新的内容。

Angular通过一个简单的设置来配置我们所说的动态视图。动态视图依靠URL注入每个视图，通过$routeprovider。一个简单的设置如下：

	myApp.config(['$routeProvider', function ($routeProvider) {
	  /**
	   * $routeProvider
	   */
	  $routeProvider
	  .when('/', {
		templateUrl: 'views/main.html'
	  })
	  .otherwise({
		redirectTo: '/'
	  });
	}]);

你会看到，当URL是"/"(即该站点的根目录),你会注入main.html。当你在一个单页面应用中已经有一个index.html页面，这是一个好的方法去调用内部的main.html视图而不是index.html。依靠URL去添加更多的视图是如此简单：

	myApp.config(['$routeProvider', function ($routeProvider) {
	  /**
	   * $routeProvider
	   */
	  $routeProvider
	  .when('/', {
		templateUrl: 'views/main.html'
	  })
	  .when('/emails', {
		templateUrl: 'views/emails.html'
	  })
	  .otherwise({
		redirectTo: '/'
	  });
	}]);
	
我们可以在emails.html上简单的加载HTML去生成我们的邮件列表,最终你可以毫不费力的创建一个复杂的应用程序。

## 13.Global static data(全局静态数据)

Gmail处理很多内部初始化数据是通过将JSON加入到页面中(右键查看源代码)。如果你想立即在你的页面设置数据，它会加快渲染时间并且Angular将更快的执行。

当我们开发应用程序时，页面渲染时，Java标签放置在DOM中，数据从后台发送过来。[我没有Java开发经验，你需要知道一个下面的声明，虽然在服务器上你可以使用任何语言。]这里是如何把JSON加入到网页中，然后把它传给一个控制器,为即时邦定使用：

	<!-- inside index.html (bottom of page ofc) -->
	<script>
	window.globalData = {};
	globalData.emails = <javaTagHereToGenerateMessages>;
	</script>
	
我定义的java标签使数据能在页面中渲染，Angular会轻松的渲染你的emails数据，只要把你的数据放入到controller中即可:

	myApp.controller('EmailsCtrl', ['$scope', function ($scope) {
		$scope.emails = {};
		// Assign the initial data!
		$scope.emails.messages = globalData.emails;
	}]);
	
## 14.Minification(精简)

这一段是关于精简你的angular代码内容。在这方面你可能已经尝试过了，有时跑一个精简过的代码，会遇到了一个错误！

精简你的AngularJS代码很简单，你需要指定你的依赖注入的内容在函数数组：

	myApp.controller('MainCtrl',
	['$scope', 'Dependency', 'Service', 'Factory',
	function ($scope, Dependency, Service, Factory) {
	  // code
	}]);
	
精简后:

	myApp.controller('MainCtrl',
	['$scope', 'Dependency', 'Service', 'Factory',
	function (a,b,c,d) {
	  // a = $scope
	  // b = Dependency
	  // c = Service
	  // d = Factory
	  // $scope alias usage
	  a.someFunction = function () {...};
	}]);
	
请记住，以保持参数的顺序出现，否则你可能会导致你和你的团队费劲。

## 15.Differences in MVC and MVVM(MVC和MVVM的差异)

这里简要介绍两者的区别:

**MVC**: 用一个controller交互,关系是Model-View-Controller
**MVVM**: 封装声明数据邦定,Model-View-View-Model,Model和View交互,View也可以和Model交互,Angular的双向数据绑定允许它在不做任何事情时保持同步。它还允许你在没有控制器的情况下,编写逻辑代码！

一个简单的例子，你可以创建一个ng-repeat标签去提供数据,而不需要Controller:

	<li ng-repeat="number in [1,2,3,4,5,6,7,8,9]">
	  {{ number }}
	</li>
	
对于快速测试这是很有帮助的，但对与复杂的情况时，我建议使用一个Controller。

## 16.HTML5 Web Components(HTML5组件)

在早些时候你会发现AngularJS允许你创建自定义的元素:

	<myCustomElement></myCustomElement>

这实际上是符合HTML5未来的特性。HTML5 Web部件和模板元素，Angular让我们今天就能这样用。Web部件包括通过动态JavaScript注入生成的自定义元素！

## 17.Scope comments(范围注释)

范围注释,我觉得真的是一个不错特性,代替了HTML注释，像这样的块：

	<!-- header -->
	<header>
	  Stuff.
	</header>
	<!-- /header -->
	
引入Angular后，让我们思考的是Views和Scopes，而不是DOM，除非你故意共享控制器间的数据，我发现使用范围是一个很大的帮助：

	<!-- scope: MainCtrl -->
	<div class="content" ng-controller="MainCtrl">
	</div>
	<!-- /scope: MainCtrl -->
	
## 18.Debugging AngularJS(调试)

对于调试AngularJS的话，可以使用Chrome的一个扩展工具-Batarang,你可以去下载下看看。

## 结束
到此本文结束,希望能够对大家学习angular有一个好的指导，更多关于angular的细节可以参看[Todd Motto](http://toddmotto.com/) 的系列文章 
再次感谢原作者的整理，文中如有不对的地方可以指出。
