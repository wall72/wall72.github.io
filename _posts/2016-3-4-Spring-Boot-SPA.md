---
layout: post
title: Spring Boot SPA
---

#### 목표
* Spring Boot 를 이용하여 프로젝트를 생성한다.
* Web, JPA, H2, WebJar (Angular.js, Bootstrap) 사용설정을 추가한다.
* DBMS에 CRUD를 수행하는 서버 기능을 작성한다.
* SPA (Single-Page Application) 샘플을 추가한다..
* Spring Boot Application을 실행하고 결과를 확인한다.

#### 개발환경 (Environment)
* STS 3.7.2
* JDK 1.8.0_65
* Mac OS X (El Capitan)

#### 소스 코드 (Source)
* 본 Tutorial에서 완성된 코드가 포함된 Release
* [SpringBootSPA](https://github.com/wall72/SpringBootSPA "Spring Boot SPA")

#### 1. Create Project
* New > Spring Starter Project
* Packaging: War (웹 프로젝트를 위한 패키징)
* Dependencies: Web (Spring Web, MVC 사용), JPA (JPA를 사용한 데이터저장 처리), H2 (저장소로 H2 사용)

#### 2. build.gradle dependency 추가
* WebJars를 통해 Angular.js, Bootstrap 사용을 위해 추가

```
compile('org.webjars.bower:bootstrap:3.3.6')
compile('org.webjars.bower:angular:1.5.0')
compile('org.webjars.bower:angular-resource:1.5.0')
```

* WebJars를 통한 Bower 디펜던시 참조 추가는 http://www.webjars.org/bower 에서 찾는다.

* Dependencies를 Update한다. (Maven의 Update Project와 동일)
* {Project} > Gradle > Refresh All

#### 3. Entity 클래스 추가
* 자료 저장을 위한 저장단위 클래스 선언을 위해 domain 패키지에 Entity 클래스를 추가한다.

```java
@Entity
public class Item {
    @Id
    @GeneratedValue
    private Integer id;

    @Column
    private boolean checked;

    @Column
    private String description;

    public Integer getId() {
        return id;
    }

  // {Getters, Setters}
}
```

#### 4. Repository 인터페이스 추가
* JPA 를 통한 자료 저장을 위해 domain 패키지에 JpaRepository 를 상속한 인터페이스를 추가한다.

```java
public interface ItemRepository extends JpaRepository<Item, Integer> {

}
```

#### 5. Controller 클래스 추가
* 클라이언트에서 요청한 리퀘스트를 처리하기 위해 web 패키지에 Controller 클래스를 추가한다.
* 참조 튜토리얼에도 서비스 클래스는 없으므로 일단 간단하게 Controller 만으로 처리한다.

```java
@RestController
@RequestMapping("/items")
public class ItemController {
    @Autowired
    private ItemRepository repo;

    @RequestMapping(method = RequestMethod.GET)
    public List findItems() {
        return repo.findAll();
    }

    @RequestMapping(method = RequestMethod.POST)
    public Item addItem(@RequestBody Item item) {
        item.setId(null);
        return repo.saveAndFlush(item);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.PUT)
    public Item updateItem(@RequestBody Item updatedItem, @PathVariable Integer id) {
        updatedItem.setId(id);
        return repo.saveAndFlush(updatedItem);
    }

    @RequestMapping(value = "/{id}", method = RequestMethod.DELETE)
    public void deleteItem(@PathVariable Integer id) {
        repo.delete(id);
    }
}
```

* REST 테스트를 위해 Chrome 을 이용한다.
* [Advanced REST Client](https://chrome.google.com/webstore/detail/advanced-rest-client/hgmloofddffdnphfgcellkdfbfbjeloo "Advanced REST Client")

* 조회

```
URL : http://localhost:8080/items
Method : GET
```
![Screenshot #1](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-spa-01.png?raw=true)

* 입력

```
URL : http://localhost:8080/items
Method : POST
Header : Content-Type: application/json
Body : {"checked":false, "description":"My First Task"}
```
![Screenshot #2](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-spa-02.png?raw=true)

* 수정

```
URL : http://localhost:8080/items/1
Method : PUT
Header : Content-Type: application/json
Body : {"checked":false, "description":"My First Task updated"}
```
![Screenshot #3](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-spa-03.png?raw=true)

* 삭제

```
URL : http://localhost:8080/items/1
Method : DELETE
Header : Content-Type: application/json
```
![Screenshot #4](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-spa-04.png?raw=true)

#### 6. UI 파일 추가
* 정적 파일은 static 폴더에 추가한다.
* src/main/resources/static

* src/main/resources/static/index.html

```html
<!DOCTYPE html>
<html lang="en">
  <head>
    <link rel="stylesheet" href="/webjars/bootstrap/3.3.6/dist/css/bootstrap.min.css" />
  </head>
  <body ng-app="myApp">
    <div class="container" ng-controller="AppController">
      <div class="page-header">
        <h1>A checklist</h1>
      </div>
      <div class="alert alert-info" role="alert" ng-hide="items && items.length > 0">
        There are no items yet.
      </div>
      <form class="form-horizontal" role="form" ng-submit="addItem(newItem)">
        <div class="form-group" ng-repeat="item in items">
          <div class="checkbox col-xs-9">
            <label>
              <input type="checkbox" ng-model="item.checked" ng-change="updateItem(item)"/> {{item.description}}
            </label>
          </div>
          <div class="col-xs-3">
            <button class="pull-right btn btn-danger" type="button" title="Delete"
              ng-click="deleteItem(item)">
              <span class="glyphicon glyphicon-trash"></span>
            </button>
          </div>
        </div>
        <hr />
        <div class="input-group">
          <input type="text" class="form-control" ng-model="newItem" placeholder="Enter the description..." />
          <span class="input-group-btn">
            <button class="btn btn-default" type="submit" ng-disabled="!newItem" title="Add">
              <span class="glyphicon glyphicon-plus"></span>
            </button>
          </span>
        </div>
      </form>
    </div>
    <script type="text/javascript" src="/webjars/angular/1.5.0/angular.min.js"></script>
    <script type="text/javascript" src="/webjars/angular-resource/1.5.0/angular-resource.min.js"></script>
    <script type="text/javascript" src="./app/app.js"></script>
    <script type="text/javascript" src="./app/controllers.js"></script>
    <script type="text/javascript" src="./app/services.js"></script>
  </body>
</html>
```

* src/main/resources/static/app/app.js

```javascript
(function(angular) {
  angular.module("myApp.controllers", []);
  angular.module("myApp.services", []);
  angular.module("myApp", ["ngResource", "myApp.controllers", "myApp.services"]);
}(angular));
```

* src/main/resources/static/app/services.js

```javascript
(function(angular) {
  var ItemFactory = function($resource) {
    return $resource('/items/:id', {
      id: '@id'
    }, {
      update: {
        method: "PUT"
      },
      remove: {
        method: "DELETE"
      }
    });
  };

  ItemFactory.$inject = ['$resource'];
  angular.module("myApp.services").factory("Item", ItemFactory);
}(angular));
```

* src/main/resources/static/app/controllers.js

```javascript
(function(angular) {
  var AppController = function($scope, Item) {
    Item.query(function(response) {
      $scope.items = response ? response : [];
    });

    $scope.addItem = function(description) {
      new Item({
        description: description,
        checked: false
      }).$save(function(item) {
        $scope.items.push(item);
      });
      $scope.newItem = "";
    };

    $scope.updateItem = function(item) {
      item.$update();
    };

    $scope.deleteItem = function(item) {
      item.$remove(function() {
        $scope.items.splice($scope.items.indexOf(item), 1);
      });
    };
  };

  AppController.$inject = ['$scope', 'Item'];
  angular.module("myApp.controllers").controller("AppController", AppController);
}(angular));
```

#### 7. 테스트
* http://localhost:8080

![Screenshot #5](https://github.com/wall72/wall72.github.io/blob/master/images/spring-boot-spa-05.png?raw=true)

#### 8. Reference Sites
* [Rapid prototyping with Spring Boot and AngularJS](http://g00glen00b.be/prototyping-spring-boot-angularjs "Rapid prototyping with Spring Boot and AngularJS")

