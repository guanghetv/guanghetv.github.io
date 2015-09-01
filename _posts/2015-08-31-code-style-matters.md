------

layout: post

title: Code Style Matters

------



## **Why does it matters?**

这个就不大做文章了，代码规范的目的主要是提高代码可读性/一致性/可维护性。持续沿用一致的代码规范，对团队协作和代码维护会越便利。



引自Twitter Bootstrap作者之一前端大神的@mdo的Golden Rule:

>  Every line of code should appear to be written by a single person, no matter the number of contributors.



当然代码规范是参考性的，非强制性，不是说代码规范就一定是对的是唯一的Coding选择，视乎个人习惯以及对规范接受的程度，以循序渐进的形式逐步改善Coding。

## Problems

以下代码从自同一人之手（图片链接有可能会失效）：

``` javascript
		Chapter.find({
        	$or: [{_id: {$in: cvGrade.chapters}, state: 'published'}, {
                _id: {$in: cvGrade.chapters},
                state: 'updating'
            }]
        }, function (err, chapters) {
        	// async 2nd chapters
            async.each(chapters, function (chapter, callback) {
            	Video.find({_id: chapter.guideVideo}, function (err, gv) {
                	if (gv[0]) {
                    	target.push({
                        	chapter: chapter.name,
							topic: "章节引入",
							topicIcon: "guide.png",
							task: "guide",
							activity: chapter.name + '的引入',
                           	// change to the video thumbnail in case of database ready
                            actThumbnail: chapter.icon,
							video: gv[0].url
                      	});
                   	}
           	});
        });
```

![](http://7xi815.com2.z0.glb.qiniucdn.com/diff_object.png)

可能你会觉得上面的代码没有错，不就是几处声明对象的代码不一样罢了。代码确实没有错，功能或许也是正常可用的，但代码规范并不是要掀谁的错要责问谁，而是要改善随意Coding无视一些小细节而形成的恶性循环的心态，从而逐步形成Golden Rule的秩序。其它问题我不一一列举了。

## Convenience

本来想把不同的开源项目不同的模块不同的人写的代码贴上来，做个对比，以突出代码规范的好处，想想就算了，反正你们不看。我这么正儿八经写个blog也是他妈的少有啊，掉光的节操都在这里了。

下面是干货。

## [EditorConfig](http://editorconfig.org)

如果你们用Vim/Emacs/Sublime/Eclipse/Webstorm/Atom/Textmate等等不同的IDE/编辑器，在项目的根目录加上个[.editorconfig](https://raw.githubusercontent.com/facebook/react/master/.editorconfig)文件。

## Code Style Guides

- Javascript([ES5](https://github.com/airbnb/javascript/tree/master/es5)/[ES6](https://github.com/airbnb/javascript))
  - 引自airbnb，在Github上的star数量已经说明了其代表性，包含了ES5/6的规范，所以不用担心ES6时代的来临
- [HTML/CSS](http://codeguide.co/)
  - 引自@mdo大神，抬头就是Golden Rule
- [Google Code Style Guides](https://github.com/google/styleguide)
  - 其它的语言在google-code-style-guides中基本上都有，当然也包括js/html/css，但这部分的Guides上面两个更好

## Pain

其实最大的问题是Code Style的执行，如果说让你先看Code Style，估计你看几眼就睡着了，目前我没有太好的建议和意见，先从Code Review抓起吧。