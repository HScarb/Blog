title: cocos2d-x v2 和 v3 对照手册
date: 2016-01-28 21:30:36
author: Scarb
postid: $POSTID
slug: $SLUG
nicename: 
attachments: $ATTACHMENTS
posttype: post
poststatus: publish
tags: cocos2d-x
category: game

>本文转载自 zrong's blog
原文地址：http://zengrong.net/post/2193.htm

**2015-05-04** 更新： 加入 OpenGL 和 MenuItem 的相关变化。

----

本文的内容来自于对其它几篇文章的翻译、修改和合成，同时，我也会不断增加自己的内容。

下面这部分内容来自对这篇文章的翻译：[cocos2d-x v2 to v3 mapping guide][1]

但这篇文章有一些老了，还有一些内容已经在 cocos2d-x 3.3 中过时。因此，我并没有进行完全对照翻译。对原文中的错误，我也进行了一些修改。

----

[我的新项目开始使用 cocos2d-x v3][3] 。cocos2d-x v3 和 v2 相比有非常大的改变。我把踩过的坑列在下面，以方便后来之人。 <!--more-->

# cocos2d-x 常用类名改变

下面的表格中的类名的转换方式主要是直接删除了 CC 前缀。

|#|v2|v3|
|----|----|----|
|1|CCAction|Action|
|2|CCPoint|Point|
|3|CCAnimation|Animation|
|4|CCSprite|Sprite|
|5|CCLabel|Label|
|6|CCMenu|Menu|
|7|CCObject|Ref|
|8|CCNode|Node|
|9|CCScene|Scene|
|10|CCLayer|Layer|
|11|CCSpriteBatchNoe|SpriteBatchNode|
|12|CCTMXTiledMap|TMXTiledMap|

# cocos2d-x 类名改变

下面表格中的类名的转换就比较大了。

|#|v2|v3|
|----|----|----|
|1|CCDictionary|ValueMap|
|2|CCArray|ValueVector|
|3|CCString|Value|

# CCString 用法改变

之前：

	:::C++
    CCString* str = CCString::createWithFormat("%s.png","picture");

现在：

	:::C++
    std::string str = StringUtils::format("%s.png","picture");

# CCDictinoary 用法改变

之前：

	:::C++
    CCDictionary* dict = CCDictionary::createWithContentsOfFile("name.plist");
    CCArray* arr = (CCArray*) data->objectForKey("Levels");

现在：

	:::C++
    std::string path = FileUtils::getInstance()->fullPathForFilename("name.plist");
    ValueMap dict = FileUtils::getInstance()->getValueMapFromFile(path);
    ValueVector arrLevels = data.at("Levels").asValueVector();

# CCArray 用法改变

这里就是 C++ vector 容器的标准用法了。

|#|v2|v3|
|----|----|----|
|1|CCArray* sprites;|Vector<Sprint*> sprites;|
|2|sprites->addObject(sprite);|sprites.pushBack(sprite);|
|3|sprites->removeObject(sprite);|sprites.eraseObject(sprite);|
|4|sprites->removeObjectAtIndex(i);|sprites.erase(i);|
|5|sprites->objectAtIndex(i);|sprites.at(i);|
|6|sprites->count();|sprites.size();|

----

**下面这部分内容来自 [这里][2] 。**

----

# 触摸用法改变

|#|v2|v3|
|----|----|----|
|1|ccTouchBegan|onTouchBegan|
|2|ccTouchMoved|onTouchMoved|
|3|ccTouchEnded|onTouchEnded|

# 单例类用法改变

|#|v2|v3|
|----|----|----|
|1|CCEGLView::sharedOpenGLView();|Director::getInstance()->getOpenGLView();|
|2|CCTextureCache::sharedTextureCache();|Director::getInstance()->getTextureCache();|
|3|CCNotificationCenter::sharedNotificationCenter();|Director::getInstance()->getEventDispatcher();|

# CCTime 用法改变

CCTime cocos2d-x v3 中已经被删除了。

|#|v2|v3|
|----|----|----|
|1|cc_timeval|timeval|
|2|CCTime::gettimeofdayCocos2d|gettimeofday|
|3|CCTime::timesubCocos2d|getTimeDiffenceMS|

范例：

	:::C++
    static inline float getTimeDifferenceMS(timeval& start, timeval& end)
    {
        return ((((end.tv_sec - start.tv_sec)*1000.0f + end.tv_usec) - start.tv_usec) / 1000.0f);
    }

----

**下面的内容为 zrong 原创。**

----

# OpenGL 的用法变化

|#|v2|v3|
|----|----|----|
|1|CCGLProgram|GLProgram|
|3|kCCUniformPMatrix_s|GLProgram::UNIFORM_NAME_P_MATRIX|
|4|kCCUniformMVMatrix_s|GLProgram::UNIFORM_NAME_MV_MATRIX|
|5|kCCUniformMVPMatrix_s|GLProgram::UNIFORM_NAME_MVP_MATRIX|
|6|kCCUniformTime_s|GLProgram::UNIFORM_NAME_TIME|
|7|kCCUniformSinTime_s|GLProgram::UNIFORM_NAME_SIN_TIME|
|8|kCCUniformCosTime_s|GLProgram::UNIFORM_NAME_COS_TIME|
|9|kCCUniformRandom01_s|GLProgram::UNIFORM_NAME_RANDOM01|
|10|kCCUniformSampler_s|GLProgram::UNIFORM_NAME_SAMPLER0|
|11|kCCUniformAlphaTestValue|GLProgram::UNIFORM_NAME_ALPHA_TEST_VALUE|
|12|kCCAttributeNameColor|GLProgram::ATTRIBUTE_NAME_COLOR|
|13|kCCAttributeNamePosition|GLProgram::ATTRIBUTE_NAME_POSITION|
|14|kCCAttributeNameTexCoord|GLProgram::ATTRIBUTE_NAME_TEX_COORD|

<a name="menu"></a>
# MenuItem 的用法变化

以前我写过一篇 [Cocos2d-x中的事件调用方式汇总][4] ，其中介绍了 cocos2d-x 中的回调函数。而在 v3 版本中，这些回调函数已经完全废弃了。

在 cocos2d-x v3 中，使用的是 C++11 提供的标准的 [std::bind][5] 功能来实现回调。

让我们看看 `base/ccMacros.h` 中的几个宏：

``` c++
// new callbacks based on C++11
#define CC_CALLBACK_0(__selector__,__target__, ...) std::bind(&__selector__,__target__, ##__VA_ARGS__)
#define CC_CALLBACK_1(__selector__,__target__, ...) std::bind(&__selector__,__target__, std::placeholders::_1, ##__VA_ARGS__)
#define CC_CALLBACK_2(__selector__,__target__, ...) std::bind(&__selector__,__target__, std::placeholders::_1, std::placeholders::_2, ##__VA_ARGS__)
#define CC_CALLBACK_3(__selector__,__target__, ...) std::bind(&__selector__,__target__, std::placeholders::_1, std::placeholders::_2, std::placeholders::_3, ##__VA_ARGS__)
```

所以，对于 cocos2d-x v2 中这样的调用：

``` c++
CMenuItemImage *item1 = CCMenuItemImage::create(s_pPathB1, s_pPathB2, this, menu_selector(ActionsDemo::backCallback));
```
在 cocos2d-x v3 中应该是这样的：

``` c++
MenuItemImage *item1 = MenuItemImage::create(s_pPathB1, s_pPathB2, CC_CALLBACK_1(ActionsDemo::backCallback, this));
```

[1]: http://www.redtgames.com/blog/cocos2d-x-v2-to-v3-mapping-guide/
[2]: http://discuss.cocos2d-x.org/users/fradow/activity
[3]: http://zengrong.net/post/2188.htm
[4]: http://zengrong.net/post/1964.htm
[5]: http://en.cppreference.com/w/cpp/utility/functional/bind