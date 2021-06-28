# Spring作用域

1. Singleton（单例）：单例bean只在容器中存在一个实例，在spring内部通过HashMap来维护bean的缓存。
2. Prototype（原型）：每次获取bean的时候都会创建一个全新的bean实例。
3. Request：每次请求都会创建一个全新的bean实例，该类型作用于web类型的spring容器。
4. Session：每个会话创建一个全新的bean实例，该类型作用于web类型的spring容器。
5. GlobalSession：类似于session作用域，只是其用于portlet环境的web应用。如果非portlet环境视为session作用域。

其中主要了解Singleton、Prototype作用域，该作用域属于SpringBean的基本作作用域。request、session、globalsession属于web应用环境的作用域，必须有web应用环境的支持。
