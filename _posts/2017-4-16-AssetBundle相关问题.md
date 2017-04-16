---
layout: post
title: AssetBundle QA
---

最近项目中碰到了关于AssetBundle的一些问题，决定记录下来：

1. ab内同名资源的处理<br/>

&nbsp;&nbsp;&nbsp;&nbsp;项目里存在一个资源A.fbx和A.prefab，并且A.prefab依赖于A.fbx，在ab打包时，这2个文件会放到一个ab文件中，导致在使用ab.LoadAsset(A)接口时，竟然会返回A.fbx的object。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;为了解决这个问题，查询了一下ab.LoadAsset的接口参数，发现是可以传递存于manifest中的全路径名称，解决方法也就很简单了，制作一份资源的名称和全路径映射关系表，排除掉不需要直接加载的资源，在ab.LoadAsset时查找到全路径，然后加载，保证取得正确的资源。<br/>

2. 针对5.5Unity提高abbuild的效率<br/>

&nbsp;&nbsp;&nbsp;&nbsp;升级Unity5.5后，发现每次abBuild的耗时特别长，特别是ab数量很多的情况下，Unity官方也表示的确存在这个问题。在和其他团队交流中了解到，找到一个提高abbuild的效率的方法，<a href="https://github.com/unity3d-jp/SeparatedAssetBundleBuild">https://github.com/unity3d-jp/SeparatedAssetBundleBuild</a><br/>

&nbsp;&nbsp;&nbsp;&nbsp;其思路大致是，比如原本有4000个ab文件，一次build可能是几个小时，那么按照300个一组，分批build，全部build完之后，将所有组的manifest进行合并打成最终的一个manifest，因为Unity5.5制作ab速度慢主要还是出现在ab数量太多的问题上，所以通过每次build减少ab数量进行加快ab打包。<br/>

3. 尽量使用abbuild，而不是给资源打上标签<br/>

&nbsp;&nbsp;&nbsp;&nbsp;在项目开发后期，发现资源越来越多，每一次ab都开始越来越耗时。我们打包ab的策略，会把对应的资源的ab 标签加上，也就是获得AssetImporter，然后设置assetbundlename。

&nbsp;&nbsp;&nbsp;&nbsp;向其他团队请教后发现，BuildPipeline.BuildAssetBundles的接口里，其实可以传递一个叫做AssetBundleBuild的对象数组，对象内直接传递assetbundleName和其包含的资源路径名称即可，可以极快的增加abBuild的时间。<br/>

4. 出AssetBundle游戏包，需要移除Resources的问题<br/>

&nbsp;&nbsp;&nbsp;&nbsp;在开发中使用Resource.Load加载资源，应该是必须的，增加开发效率，但是项目真机测试或者线上运营时，一般都采用AssetBundle加载模式，需要支持资源热更新的功能。但是Unity在打包的时候，没有办法一键帮助切换这2种资源形式，已经打成AssetBundle的资源，需要从Resource目录下移除。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;我们项目资源不是全路径加载的，路径结构类似于xx/xx/resources/x.prefab， resources文件夹下不会有其他的目录。在打ab模式的游戏包时，会遍历所有resources目录下的资源，然后打成ab，然后移除掉所有的resources目录， 最后执行buildPlayer，打包结束，再把resources目录移回来。非常的繁琐。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;和同事讨论想到一个解决方案，所有的resources目录更名resources_x，记录下所有的资源对应的全路径，在Editor模式下，不再使用Resource.Load方式加载，改用AssetDatabase.FindAssets传递全路径获取资源。<br/>

&nbsp;&nbsp;&nbsp;&nbsp;这样子，项目内没有了Resources目录，也就不用再繁琐的去移动他了。只需要再写一个脚本，一键把所有的resources_x替换成resources即可重回Resource模式的版本。<br/>
