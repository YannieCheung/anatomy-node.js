# Anatomy node.js

大多数的Node.js核心API都是围绕一个异步的事件驱动模型而编写的。生成器对象(emitter)生成一些特定的事件，然后通过事件循环机制执行回调函数，这些回调函数称为监听器(listener)。
