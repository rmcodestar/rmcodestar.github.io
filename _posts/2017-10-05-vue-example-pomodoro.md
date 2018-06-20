---
layout: post
title: VUE js를 이용하여 pomodoro 만들기
category: vue
tag: [vue.js]
---

![ScreenShot](https://github.com/rmcodestar/study-vue-js/blob/master/img/pomodoro.png?raw=true)



## [STEP 1] 포모도로 예제 만들기 ([commit](https://github.com/rmcodestar/study-vue-js/commit/ce221fe23e2fd17b4ac2976dccb1ccafeb6b99e2))



### 1. vue 인스턴스 만들기

```javascript
new Vue({
	el : "#app"
	, data : {
        minute : MINUTES_WORKIG_TIME
        , second : 0
        , state : STATE.WORK
        , timestamp : 0
      }
    , methods : {
        start : function() {
            this._tick();
            this.interval = setInterval(this._tick, ONE_SECOND);
        }
        , _tick : function() {
        ...
        }
    }
})
```



### 2. 데이터 바인딩

데이터 중 분과 초를 이중 중괄호를 이용하여 바인딩(단방향)

```html
<div class="well">
	<div class="pomodoro-timer">
		<span>{{minute}}</span>:<span>{{second}}</span>
	</div>
</div>
```



### 3. 이벤트 청취 

`v-on:click` 혹은 `@click` 디렉티브를 이용하여 이벤트 핸들링

```html
<div id="app">
	<div class="page-header">
	      <h1>
		<span>POMODORO</span>
		<button type="button" class="btn btn-default" @click="start">
		    <span class="glyphicon glyphicon-play"></span>
		</button>
	      </h1>
	</div>
  ...생략...
</div>
```


## [STEP2] 조건부 랜더링 : 휴식 중 일때 고양이 보여주기 ([commit](https://github.com/rmcodestar/study-vue-js/commit/50ae657ae5b30d78deb788752d90988667bd6903))

### 1. 계산된 속성

computed에 속성을 추가하자

* title : 포모도로 상태에 따라 표현할 타이틀
* remainingMinute : 일단 가볍게 0으로 패딩을 구현한 분
* remainingSecond: 위와 동일하게 0으로 패딩한 초 (ex. 05) 
* isWorking : 일하고 있는지 boolean으로 반환 ==> 추후 조건부 랜더링에 사용할 속성

```javascript
new Vue({
    el : "#app"
    , data : data
    , computed : {
        title : function() {
		return this.state == STATE.WORK ? "working" : "rest";
        }
        , remainingMinute : function() {
		return (this.minute < 10)? "0" + this.minute : this.minute;
        }
        , remainingSecond : function() {
                return (this.second < 10)? "0" + this.second : this.second;
        }
        , isWorking : function() {
		return this.state == STATE.WORK;
        }
    }
});
```



### 2.조건부 랜더링 (v-if)

`isWorking`속성에 따라 고양이를 보여줄 지 말지 결정하자

```Html
<div class="well" v-if="!isWorking">
	<img src="kitty.png">
</div>
```

일단 내가 그린 허접한 고양이를 보여줌, 추후 고양이 api를 이용해서 다양한 고양이를 감상하자



## [STEP3] 컴포넌트로 분리 ([commit](https://github.com/rmcodestar/study-vue-js/commit/20bb152995eecd3d68caf860136b7968a7730cca))



목표 : 4개의 컴포넌트로 분리하기

- countDownComponent
- stateTitleComponent
- controlComponent
- kittyComponent



### 1. 컴포넌트로 만들기

그 중 countDownComponent를 컴포넌트 만들기

```javascript
var countDownComponent = Vue.extend({
	template : "#countDownComponent"
	, props : ["minute", "second"]
	, computed : {
        remainingMinute : function() {
        	return (this.minute < 10)? "0" + this.minute : this.minute;
        }
        , remainingSecond : function() {
        	return (this.second < 10)? "0" + this.second : this.second;
        }
	}
});
```

그리고 root vue 인스턴스에 컴포넌트로 등록하기

```javascript
 new Vue({
            el : "#app"
            , data : function() {
                return pomodoro;
            }
            , components : {
                "control-component" : controlComponent
                , "state-title-component" : stateTitleComponent
                , "count-down-component" : countDownComponent
                , "kitty-component" : kittyComponent
            }
   			...생략...
});
```

### 3. props으로 자식 컴포넌트에게 데이터 전달하기
하위 컴포넌트에서 props으로 전달받을 값을 명시하기
```javascript
var kittyComponent = Vue.extend({
    template : "#kittensComponent"
    , props : ["isVisble", "imageSource"]
});
```

부모 컴포넌트에서 자식 컴포넌트에게 전달할 데이터 명시하기

```html
<div id="app">
    <kitty-component :is-visble="isWorking" :image-source="kittyImage"></kitty-component>
</div>
```
참고) isWorking, imageSource props를 명시할 때 html이 대소문자를 구별하지 않는 특성때문에 kebab-case으로 변경해서 적어둔다


### 4. 부모자식간 컴포넌트 관계에서 이벤트는 어떻게 하나요?
vue2에서 양방향 바인딩이 안되므로 이벤트로 해결해야 한다.
부모 컴포넌트에게 자식 컴포넌트의 감지하고 있고(`v-on`), 자식 컴포넌트는 이벤트를 트리거(`emit`)하는 방식으로 구현해야 한다

....continue....




**추후 업데이트할 내용**
* 스타일 바인딩
* 필터 사용하기
* Cat api 연동하기

***

### [결과물](http://jsfiddle.net/rmcodestar/8hznuy6r/1/)
<script async src="//jsfiddle.net/rmcodestar/8hznuy6r/embed/js,html,result/"></script>
