# weex conclution
Weex是阿里开源的一套高性能跨平台移动开发框架。通过前端语法实现同一份代码，在ios，android和web下都能完整工作。weex仿早餐页面，发现使用weex开发的可实现范围和存在的局限。

## 1. 开发语言的选择
weex从Weex从0.9.5版本开始集成了Vue.js的框架代码，将Vue2.0作为内置的前端框架，并强推用vue 2.0语法来开发native应用。
weex本身也有一套自己的语法规则，是以.we结尾的文件，该文件格式的写法在当前的weex版本仍然有效。
官方的weex项目源码里或者github上一些早期实践weex项目里也有大量用.we来写的组件和页面。如果想在.we和.vue
文件的写法上进行切换，可以借助工具 weex-vue-migration自动进行语法转换。也可以参考语法差异表来过渡。详情戳[Weex和Vue2.x的差异](https://weex.incubator.apache.org/cn/references/migration/difference.html)
[如何将原有Weex项目改造成Vue版本](https://weex.incubator.apache.org/cn/references/migration/migration-from-weex.html)

## 2. 使用weex-toolkit创建调试项目
weex-toolkit是官方提供的一个脚手架命令行工具，主要作用有：
*   初始化项目
*   直接编译.we和.vue文件，配合使用playground实时预览
*   调试weex页面
*   打包weex文件
playground是官方提供的对weex文件预览的工具，将该App安装在Android机或者iphone上，通过扫描二维码来
预览编译后的.we或.vue文件。 应用下载地址：[Weex Playground](https://weex.apache.org/cn/playground.html)

### 2.1 初始化
*   使用命令 weex init breakfast 创建一个早餐模板项目
*   进入项目所在路径，使用npm intall 安装项目依赖
*   运行npm run dev和npm run server开启watch和静态服务器
*   打开浏览器，进入http://localhost:8080/index.html，可以看到weex h5页面
想要直接在android或iphone上进行开发调试预览，可以在上面步骤的第二步之后，直接coding和debug .we或.vue。略去后两步骤。

### 2.2 使用weex命令编译.we/.vue文件
使用weex breakfast.we 或者 weex breakfast.vue执行编译，会自动在chrome打开一个页面，网页下方有一
个二维码，再借助playground来扫码查看页面。

### 2.3 调试weex页面
*   启动debug server。执行命令weex debug  ，chrome浏览器会自动打开weex devtool的页面，网页下方有
二维码。如果执行命令weex debug breakfast.we ，则会自动打包编译.we文件，并在最底部右侧生成另一
个二维码，用playground扫描该二维码，就可以显示breakfast.we的页面。
*   使用playground扫码。扫描左下角的二维码启动Debugger，会出现当前设备信息。
*   点击Debugger按钮打开debugger页面，根据页面提示打开该页JavaScript控制台，选择sources这个tab
。
*   设置断点调试。用playground扫描weex devtool页面右下角的二维码，在sources会出现相应的js文件，
设置断点，刷新playground就可以开始调试了。

### 2.4 打包weex文件
使用weex compile  breakfast.we dist 命令对单个文件或整个项目的打包。将生成的js文件拷贝到android项
目或者ios项目的assets目录下。

## 3. 动能点的实现
### 3.1 顶部日期栏在列表向上滑动时sticky到顶部，并横向滚动日期菜单
使用<list>组件支持最外层的滚动。直接内嵌子组件<header>，header组件在到达屏幕顶部时，会吸附在屏
幕顶部。<header>组件内放入一个横向滚动的<scroller>来支持日期菜单横向滚动。
```
<template>
	<div class="container">
		<list>
      <cell>
      	<slider class="slider" interval="3000" auto-play="false">
      		<div class="slider-pages" repeat="{{itemList}}">
      			<image class="slider-img" src="{{pictureUrl}}"></image>
      		</div>
      	</slider>
      </cell>
			<header>
	      <div class="header">
	        <scroller scroll-direction="horizontal" class="menuScroller">
          	<div repeat="{{menuList}}" class="weekCell {{menuIndex == $index ? 'active_menu' : 'normal_menu'}}" onclick="onWeekCellClick($index)">
          		<text style="font-size: 30px;">{{ title }}</text>
          		<text class="header-menu-item-desc">{{ description }}</text>
          	</div>
          </scroller>
	      </div>
	    </header>
      <cell>
        //餐品列表
      </cell>
    </list>
  </div>
</template>
```
### 3.2 餐品列表区域侧滑
餐品列表的侧滑的实现通过监听Swipe手势，当用户在屏幕上滑动时触发，一次连续的滑动只触发一次swiper
手势，在handleSwipe方法中，通过eventProperties参数来获取到滑动的方向，通过判断用户滑动的左右滑动
方向来更新当前激活的日期菜单项并更新对应的餐品数据。
```
<template>
	<div class="container">
		<list>
      <cell>
      	<slider class="slider" interval="3000" auto-play="false">
      		//banner
      	</slider>
      </cell>
			<header>
	      //header container
	    </header>
      <cell class="foodItemContainer" repeat="{{foods in menuList[menuIndex].newFoods}}" if="true">
				<div class="foodItem" repeat="{{ item in foods}}" onclick="onFoodItemClick(item.name)" onswipe="handleSwipe" >
				  //...
				</div>
      </cell>
    </list>
  </div>
</template>
<script>
  module.exports = {
		data: {
			menuIndex: 0, //当前选中菜单的索引值
		},
		method: {
			handleSwipe: function(eventProperties) {
				if (eventProperties.direction == 'left') {
					this.menuIndex =  this.menuIndex + 1 > this.menuList.length ? this.menuIndex : this.menuIndex + 1;
				} else if (eventProperties.direction == 'right') {
					this.menuIndex = this.menuIndex - 1 < 0 ? this.menuIndex : this.menuIndex - 1;
				}
			},    
		}
  }
</script>
```
### 3.3 小球抛落动画
使用Transform在transform中添加贝塞尔曲线cubic-bezier的timing-function实现小球抛落的效果
<script>
var animation = weex.requireModule('animation');
module.exports = {
	method: {
		onroll: function() {
			animation.transition(this.$el('ballWrapper'), {
				styles: {
					color: '#FF0000',
					transform: 'translate(400px, 0)',
					transformOrigin: 'center center',
				},
				duration: 500,
				timingFunction: 'linear',
				delay: 0
			}, () => {
				//callback
			});

			animation.transition(this.$el('ball'), {
				styles: {
					color: '#FF0000',
					transform: 'translate(0, 800px)',
					transformOrigin: 'center center',
				},
				duration: 500,
				timingFunction: 'cubic-bezier(1, 0, 1, 1)',
				delay: 0
			}, () => {
				//callback
			});
		},
	}
}
</script>
### 3.4 购物车容器和餐品详细内容页
Weex目前不支持z-index设置元素层级关系，但靠后的元素层级更高，所以对于层级高的元素可将它放在后
面。如购物车容器和餐品详细内容页都是覆盖在餐品列表之上。

## 4. 存在的问题
*   同向滚动。Weex目前不支持同方向的<list>或者 <scroller>互相嵌套。
*   Weex目前没有提供滚动监听的Api，且在Android上不支持在滚动类型的元素上监听手势，阻碍了一些方法的实现。
*   小球出现和落入购物车需获取定位位置，从而绘制曲线。目前Weex尚未提供相关Api：
getBoundingClientRect，不能获取到元素的坐标。
*   选择楼宇页面的“地图打点”功能，需要根据环境分别使用native和js的sdk来实现

## 5. Weex优势和局限
*   移动端开发人员可以通过简单的前端代码写出native的性能体验
*   提供了playground，在集成到移动端之前，可以不用建立native工程，直接结合playground来开发调试和预览。
*   web端css属性，api支持不够，如果要实现某些特定的功能，就需要native配合，或者折中需求。
*   社区规模小，活跃人数少，填坑速度不够快。

## 6. Weex跟进
*   对于web标准的支持，weex按照组件使用频率来跟进开发，目前虽然支持有限，但是在不断更新和支持。
*   工具，调试，开发，渲染性能优化中。
