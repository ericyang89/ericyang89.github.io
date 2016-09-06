# 1 每个模块添加注释说明，包括模块对应的位置、模块的描述、编写（负责）人。例如：
```JavaScript
/**
 * @module dyshow/page/index/app
 * @desc   鱼秀-首页
 * @author zhuzijian@douyu.tv
 */
 ```
# 2 所用模块都提供一个init作为初始化的方法；这也是对外暴露的唯一的一个方法；如果有参数，在模块内部会有默认值。
   + init方法一般包含：初始化dom（getDom）、事件绑定（bindEvent）和其他方法。
   + 初始化dom，把与dom紧耦合的代码放在一个位置方便管理；也减少多次dom查询。
   + 事件绑定，包含dom事件的绑定，包含监听的一些自定义事件（Observe）和 flash对js的调用。
   
 ```JavaScript
/**
 * @module dyshow/page/room/mod/patron
 * @desc   鱼秀直播间---守护
 * @author yangkun@douyu.tv
 */
define('dyshow/page/room/mod/patron', [
    'jquery',
    'shark/observer',

    'dyshow/context'
], function(
    $, Observer,
    Context
){
    'use strict';

    var toFixed=Number.prototype.toFixed,
        isPatroned=false;

    var Patron={
        init:function(){
            var self = this;
            self.getDoms();
            self.bindEvent();
            self.scrollable();
        },
        getDoms:function(){
            this.doms={
                patronBox:$('[data-rel="patron-box"]'),
                patronSum:$('[data-sum="patron"]')
            };
        },
        bindEvent:function(){
            var self=this,
                $patronBox=self.doms.patronBox;

            $patronBox.on('click','[data-rel="patron-add"]',function(){
                Observer.trigger('mod.patron.buy');
            });

            //渲染守护列表
            API.reg('room_data_patronList',function(datastr){
                var data=self.decodePatronList(datastr);
                self.render(data);
            });

            //获取用户余额
            Observer.on('mod.userinfo.userinfoready', function(data){
                _gold=data.gold||0;
                $('[data-login-user="gold"]').text(_gold);
            });

            //登录后查询守护列表
            Observer.on('mod.login.userinfo',function(){
                API.exe('js_patronList');
            });


        },
        scrollable:function(){        },
        destroy:function(){
            $('.show-dialog-mask').remove();
        }

    };

    return {
        init:function(){
            Patron.init();
        }
    };

});



 ```
 
# 3 涉及到逻辑的只用id选择器和data-属性选择器；所有的class选择器，都是和样式相关。例如：
 ```javascript
     //好
     this.doms={
                    $patronBox:$('[data-rel="patron-box"]'),
                    $patronSum:$('[data-sum="patron"]')
    };
    
    //不好
    this.doms={
                    $patronBox:$('.patron-box'),
                    $patronSum:$('patron-sum')
    };
```

# 4 模块间交互使用Observer，具体方法参见如下示例：
+ 在chat-user-manager模块，通过Observer，维护chat模块的黑名单
+ 在chat-msg模块获得chat-shield模块的数据，获取数据所以用fire，比trigger性能好。
+ ？？Observer 标识，取名;按照模块的路径？？//TODO
```JavaScript
    //chat模块
	Observer.on('mod.chat.blackList.set', function(id) {
		if ( undefined === id ) return;
		Vars.blackList.push(id);
	});
	
	//chat-user-manager 模块
	Observer.trigger('mod.chat.blackList.set', Vars.rel);
	
	
	<!--跨模块，取数据；fire只执行第一个监听-->
	//chat-shield 模块
	Observer.on('mod.chat.msg.shield', function() {
		return Vars.checkedMap;
	});
	
	//chat-msg模块（其他模块）
	Observer.fire('mod.chat.msg.shield').message
	
	
 ```
