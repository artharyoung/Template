# ActionBar使用归纳

最近在自己的项目[MDVideo](https://github.com/AndroidTips/MDVideo)中添加了半透明的StatusBar效果，索性把官方文档中关于这一部分的讲解总结一下，
加强一下这一块的记忆。之前做了一年半的TV应用开发，由于交互上只处理OnKey的事件，所以应用基本都是采用FullScreen样式。并且由于当时使用的Eclipse
对support V7包的支持完全令人无语，导致这一块细节的了解还是比较陌生的。

## 添加ActionBar

> Action bar 最基本的形式，就是为 Activity 显示标题，并且在标题左边显示一个 app icon。即使在这样简单的形式下，action bar对于所有的 activity 来说是十分有用的。它告知用户他们当前所处的位置，并为你的 app 维护了持续的同一标识。

设置一个基本的ActionBar,需要APP使用一个Activity主题。主题必须是ActionBar可用的。主题的申明取决于APP所支持的版本targetSdkVersion与 minSdkVersion。
