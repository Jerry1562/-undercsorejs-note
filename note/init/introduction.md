# 简介
在框架初始化的阶段，首先要完成以下几个任务：
+ 获取当前的全局对象 
+ 引用JavaScript原生对象的原型方法 
+ 创建一个对象用于保存所有的方法
+ 创建框架内部使用的高阶函数

这一步骤的主要目的是，声明一个用于保存所有方法的对象`_`并把这个对象绑定到全局对象中。为了使其能运行在各种环境下，我们需要做一些环境匹配。同时，为了方便后面的书写，还要为一些原生对象创建快速引用，最后完成内部高阶函数的制作。
