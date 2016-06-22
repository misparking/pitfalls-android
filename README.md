# pitfalls-android

###31,百度加固后，运行再小米2S等低版本手机会出现崩溃的问题。
	
现象：在小米2S中，一旦通过百度加固后，就会出现崩溃。
解决办法：通过逐步排查，发现只有是在Android Studio的项目中才会出现，通过二次排查，发现是在我们的gradle中配置了 android:debuggable = true 就会导致运行崩溃。但通过资料调研后，发现	android:debuggable = true与百度加密的崩溃并不会有直接关系，通过排除法再次分析，认为问题只可能出现在百度加固的这个过程中了。联系百度技术人员后，百度人员成功复现，并给出的解释为：

在mi2s上失败的原因，是因为mi2s集成了Lbe，lbe会在应用启动的时候注入应用进程，它的行为和百度加固的逻辑有冲突
Lbe要获取你们dex里的类，加固过后，他获取的时候，你们的dex里的类还没有被壳加载起来
之前lbe的问题我们联系了他们，他们不维护了，只能我们做兼容。

通过一波多折的多次迭代过后，测试通过。最终确定问题为百度加固过程中的不兼容性导致小米2S的手机崩溃。后续如果遇到此类问题，首先需要调试我们程序中的debuggable;确认不是程序问题后，及时沟通第三方人员。

***

###30,Android6+系统，变声录音异常的解决办法：

1、权限改变

Android 6.0以前安装应用时就会弹出权限对话框给用户，而Android6.0以后再安装或打开应用时并不会弹出权限对话框，而是在你使用到当前功能（这里指的是录音功能时才会弹出）

2、系统不再支持8bit的编码率

上面已经说了，再低质量语音传输时8bit已经够了，再我们6.0以前使用8bit编码率对大部分的手机录音已经足矣，这里需要解释一下编码率到底是啥：

要算一个PCM音频流的码率是一件很轻松的事情，采样率值×采样大小值×声道数bps。一个采样率为44.1KHz，采样大小为16bit，双声道的PCM编码的WAV文件，它的数据速率则为 44.1K×16×2 =1411.2 Kbps。我们常说128K的MP3，对应的WAV的参数，就是这个1411.2 Kbps，这个参数也被称为数据带宽，它和ADSL中的带宽是一个概念。将码率除以8,就可以得到这个WAV的数据速率，即176.4KB/s。这表示存储一秒钟采样率为44.1KHz，采样大小为16bit，双声道的PCM编码的音频信号，需要176.4KB的空间，1分钟则约为10.34M，这对大部分用户是不可接受的。

所以有很多人为了再带宽上优化，增加采样率肯定是不可取的，所以就把16bit改成8bit，而对Andorid 6.0以前的影响并不会很大，但是在6.0以后，你再使用8bit就会出现异常了，这点一定要注意。

具体内容见：[android AudioRecord介绍与Android 6.0后的改变](http://blog.csdn.net/zhanggang740/article/details/51613983 "Link")   


***
###29， Android 5月份细节点总结
1. 一个View，如果既设了padding，又设了paddingTop，那么只有padding生效，paddingTop是无效的。

2. 在开发下载功能的时候，使用Service和DB是用来在activity死掉后，管理和记录下载状态。

3. 可以把ListView的adapter设置为null，这样就只显示ListView的headView，4.0以下也是没有问题的。

4. 要注意这样的问题：ListView的adapter是通过一个list关联其item的，如果子线程会动态修改这个list（即子线程和adapter引用同一个对象，子线程会修改这个对象的值），在滑动ListView的时候就会有异常抛出。

5. 要十分注意数据库降版本的情况。

6. ListView如果布局高度不确定的时候，会计算其或其父控件的高度，所以会造成其getView方法被重复调用的情况。

7. If an activity is paused or stopped, the system can drop it from memory either by asking it to finish (calling its finish() method), or simply killing its process. 

8. 如果ListView没有HeaderView或者FooterView的时候，与ListView相关联的Adapter就是传进来的参数Adapter。如果有，则原来的Adapter将被包装成HeaderViewListAdapter，通过getWrappedAdapter()方法可以获取原来的Adapter。

9.尽量少使用setBackGround()方法设置背景，由于版本问题会引起运行错误;在activity中通过getWindow().setBackgroundDrawable(null);可以减少一个层级。（getWindow().setBackgroundDrawable()还有另外一个用法就是输入法弹下去时背景为黑色，可以通过这个来改为想要的颜色）。

10.如何让EditText不自动获取焦点？在EditText的父Layout中,加入下面的两个属性即可 : android:focusable="true"，android:focusableInTouchMode="true"。    

### 28. java float类型比较细节 ：
举个例子    27.2 == 272/10.0;   求输出结果 ;
一般人第一眼看到肯定觉得很简单 , 返回就是true嘛 ;    当告知你答案是false ,你依然不解,急着在编译器上实践，结果发现真的是false ,为什么呢？

答案是 ： java中直接声明 27.2 默认是为double类型 ; 而272/10.0 返回时一个float类型 ;  double == float 自然返回false ; 
可换成这个写法  27.2f == 272/10.0 ; 直接声明一个float类型 ;    

###27.Android-G610手机出现的奇怪适配问题
机名：G610
Androd版本：4.2.2
主屏分辨率：960x540像素

问题：在此手机上出现适配问题如下图：
![pic_01](https://github.com/JackWaiting/pitfalls-android/blob/master/images_auto.png)

出现的原因：目前只能是猜测，由于手机和分辨率原因，导致下面的布局被拥挤，或某属性在此手机内部支持的可能性。如果大家有接触过类似的问题，欢迎指正。

最终的解决办法：更换其他布局，使用适配性更高的写法。
***


###26，RadioGroup调用check(id)方法时，onCheckedChanged(RadioGroup group, int checkedId)方法被执行多

多次调用经常会干扰到程序的正常逻辑，导致出现奇怪的问题。最初我会放弃RadioGroup的onCheckedChanged()的监听，而改用它的onClick()事件，但是onClick()又会存在多次点击的问题，依旧不是比较理想的解法。  
要想让它只回调一次而不是多次，正确的做法应该是：RadioButton.setChecked(true); [Link](http://stackoverflow.com/questions/10263778/radiogroup-calls-oncheckchanged-three-times "Link")   


###25，大图片裁剪终极解决方案  
APP在Android手机上实现拍照截图这一功能，虽然看起来非常简单，但是网上大多只是Demo水准，用在实际项目中问题漏洞百出，经常导致截取照片时程序异常的BUG。   
所幸在Google上搜到了一个不错的博客，才完美解决了这个问题。URL：[Android大图片裁剪终极解决方案](http://my.oschina.net/ryanhoo/blog/86842?fromerr=nYvxpdVH)   
总结一下最关键的原因，**截大图用URI，小图用Bitmap**，网上的Demo几乎都是用的Bitmap，而且并不提及如何使用URI。我们知道，现在的Android智能手机的摄像头都是几百万的像素，拍出来的图片都是非常大的。因此我们截图无论大图小图都要统一使用Uri进行操作。
关键代码： 

	private void cropImageUri(Uri uri, int outputX, int outputY, int requestCode){
	    Intent intent = new Intent("com.android.camera.action.CROP");
	    intent.setDataAndType(uri, "image/*");
	    intent.putExtra("crop", "true");
	    intent.putExtra("aspectX", 2);
	    intent.putExtra("aspectY", 1);
	    intent.putExtra("outputX", outputX);
	    intent.putExtra("outputY", outputY);
	    intent.putExtra("scale", true);
	    intent.putExtra(MediaStore.EXTRA_OUTPUT, uri);
	    intent.putExtra("return-data", false);
	    intent.putExtra("outputFormat", Bitmap.CompressFormat.JPEG.toString());
	    intent.putExtra("noFaceDetection", true); // no face detection
	    startActivityForResult(intent, requestCode);
	}

***

### 24.总结如何快速、高效、无错误的修改应用的包名
android如果想修改包名，如果牵扯到Manifest或者自定义控件带命名空间的。总会出现一些错误，比如，包名错乱、包名缺少、控件和控件交叉在一起。其实是可以避免这样错误的，总结如下：

**eclipse:**	
1.命名空间的书写问题		
例如：xmlns:myxmlns="http://schemas.android.com/apk/res/包名"，xmlns:app="http://schemas.android.com/apk/res-auto"第一个跟包名有很大关系，如果用第一个每次修改包名后，你要对应的xml里修改命名空间的包名。所以，不建议用第一种。	
2.xml结束符	
修改包名跟layout里面的控件。例如，		
<com.xxx.xxx.xxx.imageview></com.xxx.xxx.xxx.imageview>这样的方式很容易造成错误，建议用<com.xxx.xxx.xxx.imageview/>。这样更不容易出现错误。		
3.manifest文件最好复制出来，修改好包名再复制进去，然后进行修改。这样更好修改。如果直接不复制出来，你的service、receiver里面很容易出错。	
4.如果按照上面修改，在.java文件里也会出现错误，这个重新导下包就好了。		
**android studio:**		
android studio很好的解决了这个问题。	
修改好之后，重启启动下studio，在build.gradle里找到application id修改就好了。	

建议使用studio，eclipse修改不方便。很容易出现问题。	
***

### 23.布局优化
根据Android源码的分析(具体见文章)，RelativeLayout将对所有的子View进行两次measure，而LinearLayout在使用weight属性进行布局时也会对子View进行两次measure，如果他们位于整个View树的顶端时并可能进行多层的嵌套时，位于底层的View将会进行大量的measure操作，大大降低程序性能。因此，应尽量将RelativeLayout和LinearLayout置于View树的底层，并减少嵌套。  
配合`<include>`标签(布局重用)、`<merge>`标签(减少层级)、Viewstub(按需加载)。总之，关于布局优化，原则上尽量减少布局文件的层级。
***
### 22.Activity异常导致的资源回收
在编写代码时将数据放入文件中而不放static变量中，如登录的用户信息放在Sharepreference中，每次直接从其中取值。如果实在需要将数据放入static变量中，如模块信息，需要在activity的onSaveInstanceState执行保存。这样可以保证在执行内存清理后打开应用还能使用onRestoreInstanceState将数据找回。(OnRestoreInstanceState一旦被调用，其参数是一定有值的)。

***
### 21.For语句的使用
####(1)不要在for的第二个条件中调用任何方法.
如：```for (int j = 0; j < element.length; j++)
改为for(int j = 0 , k = element.length; j < k ; j++) ```
减少执行element.length的次数。
####(2)增强for循环.
能够用于实现了iterable接口的集合类及数组中。在集合类中，迭代器让接口调用hasNext()和next()方法。在ArrayList中，手写的计数循环迭代要快3倍(无论有没有JIT)，但其他集合类中，改进的for循环语法和迭代器具有相同的效率。    
***
### 20.Android 6.0系统注意事项(硬件设备)
根据Android官方文档：Android 6.0设备通过蓝牙和Wi-Fi扫描访问外部硬件设备时，你的应用需要添加ACCESS_FINE_LOCATION或者ACCESS_COARSE_LOCATION权限。    
 
**注意: 这两个权限在手机上提示为定位权限,用户看到扫描的时候提示是否允许定位很有可能会拒接,拒接后是扫描不到外部硬件的   **

***
以下为正序，以上为倒序。
***
##一、Wifi灯项目坑总结
### 1.radiobutton setChecked(false)后，再setChecked(true)无效，只有先选择其它的radiobutton才会有效。
原因：只设置了radiobutton的属性，并没有设置radiogroup的属性，所以对于radiogroup来说，它并不知道你的radiobutton已经设置成了false
<br>解决：调用radioGroup.clearCheck() 来清空选择

### 2，Fragment栈管理。加入进栈，清空栈。
        for (int i=0;i<getActivity().getSupportFragmentManager().getBackStackEntryCount();i++)
               getActivity().getSupportFragmentManager().popBackStack();	// 回退栈
         fragManager.beginTransaction()
                 .replace(layoutId, fragment)
                 .setTransition(FragmentTransaction.TRANSIT_FRAGMENT_OPEN)
                 // .addToBackStack(null)	// 是否将改动作添加到回退栈
                   .commit();

### 3，华为手机Intent不能传序列化的Object对象

### 4，Service没有在清单文件中配置，在开启该Service时，程序不会报错。

### 5，Fragment在退出后getActivity()方法返回为空，如在Handler中延迟执行一个事件，在触发之前退出改Fragment，触发时会出错。

### 6，点击事件变色的Shape
主要是android:state_focused="true" android:state_pressed="true"这两个属性，具体参考：[http://blog.csdn.net/u010134293/article/details/50161967](http://blog.csdn.net/u010134293/article/details/50161967 "自定义Drawable（文字按钮点击效果设置）")

### 7，APP界面图片显示错位或混乱，而资源文件的相关引用确定没有错误
问题：由于R文件生成错误导致
<br>解决：在发布项目之前**Clean Project**

### 8，for循环，ConcurrentModificationException异常。（并发修改）
	// 错误写法
	for (String item : set) {
		if ("bbbbbbb".equals(item)) {
    		set.remove(item);
		}
	}
	// 正确写法
	Iterator iterator = set.iterator();
	String item;
	while(iterator.hasNext()){
		item = (String) iterator.next();
		if ("bbbbbbb".equals(item)) {
			iterator.remove();
		}
	}

### 9，JNI加载so文件时，报java.lang.UnsatisfiedLinkError: dalvik.system.PathClassLoader错误，但在armeabi下有这个文件，并且文件名正确。
可能原因，还有其它的文件夹，如armeabi-v7a、x86、mips等文件夹，但是该文件夹下没有对应的so文件，可以通过删除其它文件来解决BUG。
<br>参考网址：[http://www.it165.net/embed/html/201410/2707.html](http://www.it165.net/embed/html/201410/2707.html "Androidndk开发打包时我们应该如何注意平台的兼容（x86,arm,arm-v7a）")

### 10， Apk安装错误：Installation error: INSTALL_FAILED_VERSION_DOWNGRADE
是因为 androidversionCode 的原因，我们手机里面的APP 的 versionCode 高于将要安装的 APP!
<br>将 androidversionCode 的版本号改成一个更高的版本。

***
##关于自定义控件小米2s的坑总结
在自定义控件的时候有两个方法特别重要，第一个是onMeasure,第二个是onSizeChanged。onMeasure做计算屏幕的工作，但是小米2s，却在这里做了更多的处理。在切换到其他屏幕的时候，会多次执行onMeasure，切换回原来界面也会多次执行onMeasure。如果你在这里处理了逻辑问题，很可能会出现很多问题。目前可以把逻辑写入到onSizeChanged里面去。他会在第一次进入界面的时候调用。还有一种就是屏幕发生变化的时候进行调用，比如华为p6，它下面会多出一块操作区域。

###11，关于gradle的使用
1、文件开头apply plugin是最新gradle版本的写法，以前的写法是apply plugin: ‘android’, 这里大家注意一下。 
2、buildToolsVersion这个需要你本地安装该版本才行，很多人导入新的第三方库，失败的原因之一是build version的版本不对，这个可以手动更改成你本地已有的版本或者打开 SDK Manager 去下载对应版本。 
3、applicationId代表应用的包名，也是最新的写法，这里就不在多说了。 
4、android 5.0开始默认安装jdk1.7才能编译，但是由于mac系统自带jdk的版本是1.6，所以需要手动下载jdk1.7并配置下，具体可以见我这篇博客Mac下安装和管理Java 
5、minifyEnabled也是最新的语法，很早之前是runProguard,这个也需要更新下。 
6、proguardFiles这部分有两段，前一部分代表系统默认的android程序的混淆文件，该文件已经包含了基本的混淆声明，免去了我们很多事，这个文件的目录在 sdk目录/tools/proguard/proguard-android.txt , 后一部分是我们项目里的自定义的混淆文件，目录就在 app/proguard-rules.txt , 如果你用Studio 1.0创建的新项目默认生成的文件名是 proguard-rules.pro , 这个名字没关系，在这个文件里你可以声明一些第三方依赖的一些混淆规则，由于是开源项目，SnailBulb_Basic_Android里并未进行混淆，具体混淆的语法也不是本篇博客讨论的范围。最终混淆的结果是这两部分文件共同作用的。 
具体参考：[http://blog.csdn.net/zhanggang740/article/details/49907745](http://blog.csdn.net/zhanggang740/article/details/49907745 "Gradle基础--认识Gradle")

###12，关于小米闹钟弹框的坑总结
这个问题是之前做音箱类应用的时候遇到的，功能就是在应用未杀死的情况下，闹钟响时能弹出提示框。后来发现其他的手机都可以弹出，唯独是小米不行。原因既然是小米把系统的悬浮窗给禁掉了，只有用户手动开打这个权限后才能弹，大家可以注意下这个问题。

###13，关于内存溢出的总结
最近已经写了一篇博客对这块进行了总结。
具体参考：[http://blog.csdn.net/zhanggang740/article/details/50393466](http://blog.csdn.net/zhanggang740/article/details/50393466 "关于内存溢出的总结")

***
##关于在tf卡下接听电话的坑总结
在tf卡下有电话进来会自动切换到a2dp模式，在a2dp模式下不用处理tf的音乐，因为a2dp下调用卡音乐的方法会在来电时自动接听，去电时会自动挂断。

###15,解决客户反馈打开应用就闪退的隐形坑
ListView嵌套GridView计算高度setGridViewHeightBasedOnChildren时，getView会计算到空值的情况，在这种情况下一定不要在convertView为null的情况下去调用hodler中刷新一些UI值，否则在某些手机下会出现空指针的情况，此时，如果调用View.setBackground方法会引起 NoSuchMethodError错误，这时我们一定要对这些手机版本进行控制，通过官方API提供的@TargetApi(Build.VERSION_CODES.JELLY_BEAN)进行控制即可，保证运行低版本时不会出现NoSuchMethodError错误。
如下图：
![pic_01](https://github.com/JackWaiting/pitfalls-android/blob/master/pitfalls.png)


###16，不要在Android的Application对象中缓存数据
在我们App中的很多地方都需要使用到数据信息，它可能是一个session token，一次费时计算的结果等等，通常为了避免Activity之间传递数据的开销，会将这些数据通过持久化来存储。

有人建议将这些数据放在Application对象中方便所有的Activity访问，这个解决方案简单、优雅并且是……完全错误的。

你如果你将数据缓存到Application对象中，如何你并未对这个值进行初始化，那么有可能你的程序最终会由于一个NullPointerException异常而崩溃掉。如果你已经对他进行初始化，很有可能会出现在这个值快速更新的情况下，他会变成你初始化过后的值。
####为什么会这样？
在上面这个例子中，程序之所以会崩溃掉是因为恢复之后APP的Application对象是全新的，所以缓存在Application中的用户名成员变量为空值，在程序调用String的toUpperCase()方法时由于NullPointerException而崩溃掉。

导致这个问题的主要原因是：Application对象并不是始终在内存中的，它有可能会由于系统内存不足而被杀掉。但Android在你恢复这个应用时并不是重新开始启动这个应用，它会创建一个新的Application对象并且启动上次用户离开时的activity以造成这个app从来没有被kill掉得假象。

我们以为可以通过Application来缓存数据，却没想到恢复APP时直接跑了B Activity而不是先启动A Activity，最终导致的结果是程序意外的崩溃掉了。

####有哪些替代方法可用呢？
1、对于数据缓存问题我也没有比较好的办法，但你可以按照下面其中一种方式来处理：

2、通过Intent在Activity之间来传递数据（但是请别传递大量数据，这有可能导致程序异常或者ANR）；

3、使用官方推荐的方法中的一种将数据持久化，存储在磁盘中；

4、在使用数据和句柄的时候做空值检测；


##17，四月Android开发细节点总结
1. 使用ListView时如果用到removeHeaderView，一定要确定ListView已经使用了setAdapter方法，不然会报NullPointException，addFooterView必须在setAdapter之前才会生效。

2. 使用ListView的时候，布局尽量使用fill_parent或者写死，如果使用wrap_content，它初始化的时候需要测量，会不断调用adapter的getView方法。

3. 使用ListView时如果要隐藏HeaderView，可以通过removeHeaderView来实现，也可以把headerView设为gone，然后headerView.setPadding(-headerView.height）来实现。

4. 使用ListView时尽量不使用onItemClick和onItemLongClick，而在adapter的getView中使用onClick和onLongClick。

5. Activity保存状态信息是应该在onPause时做，而不是onStop时做，以为可能因为内存紧张，可能不会调用onStop方法就已经被回收。

6. 在自定义一个UI控件的时候，一定要提供一个具有两个参数类型分别为Context和AttributeSet的构造函数，否则的话，该自定义控件就不可以在UI布局文件中使用。
 
7. 在定义Dialog，调用其dismiss和show方法的时候，一定要注意判断调用该Dialog的activity是不是已经为空或者已经finish了。

8. 单例中如果hold有context，一定要保证这个context是ApplicationContext，因为如果是Activity的context，会影响这个activity的回收。

9. 用SparseArray<E>代替HashMap能提高性能。

10. 当使用.9图做为一个view的background，如果在代码中动态修改了它的background，那么，这个view原先设置的padding将会失效。应先保存去padding值，然后等动态设置完background后再通过setPadding设置padding值。

##18，总结Testin中出现的BUG问题
1、对问题进行setBackgrond会导致低版本出现java.lang.NoSuchMethodError问题。
解决此类问题的版本之前已经提示过，但是好像无法彻底解决，因此建议大家在项目中不要直接再去使用setBackgrond，因此带来的闪退是用户无法接受的，建议使用setBackgroundDrawablue和setbackgroundResource代替，以此来设备低版本出现的闪退问题。

2、ImageLoader导致出现的运行异常问题
目前对此问题的解决方法是在调用displayImage时进行try{}catch{}进行捕获，至少保证程序不会闪退。

3、Fragment中调用getResource导致的空指针异常
通过分析，出现此异常的原因可能是android生命周期引起的非正常情况，在某些手机上一旦出现Activity还未加载就获取getResource便会导致此类问题出现，解决的办法是延长此Fragment的生命周期，调用ApplicationContext();

4、在加载Log日志的时候，出现的空指针问题
在我们的程序中，有时候会去打印一些集合或者实例化对象的某些属性，这些属性在某些情况下不做非空判断是会导致空指针出现的，一般我们会忽略输出Log日志的忘掉非空判断，这里提醒大家，平时一定要注意。

##19，Android 上传应用商店时出现ERROR getting 错误
最近出现一个bug，是上传应用商店的时候，部分应用商店会调用aapt工具获取apk信息，在获取信息时会出现错误。

这个错误并不长出现，只有一些国外的解决文章，还是花了一些时间才解决，这里记录一下了，如果少年们出现了类似的问题也可是试一下下面的解决方案。

首先我们如果出现上传问题以后可以使用aapt工具检测一下，工具在 sdk build-tools 文件夹下，用 cmd 的方式打开。
命令是 aapt dump badging xx.apk xx为应用名称，意思是获取apk相关信息。当程序出现问题就会出现和应用商店相同的提示信息。
ERROR getting 'android:name' attribute: attribute is not a string value
在我接手的项目里面出现这个问题的原因是，AndroidManiFest 中 activity 的 android:name= 用了@string的模式，这种相关的使用方式导致 aapt 无法识别。
修改方法就是把 @string 中的字符串复制到 android:name 中，然后使用 aapt 工具跑一下就可以解决问题了。如果重新打包的应用上传的应用商店时还出现错误提示，可以尝试刷新页面。

网上有国外的解决方案是吧 AndroidMainFest中所有 @string 都是用硬编码的方式写到文件里，这里其实并不需要的，只要没有提示 ERROR getting 'android:label' attribute: attribute is not a string value 或者是其他的类似提示，都只要修改 activity 里 android:name 就可以了。




