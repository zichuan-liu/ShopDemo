## 前言
项目Github地址：https://github.com/775269512/ShopDemo
仿小米应用商店的实现，目前完成界面显示和基本功能的开发，代码一般般，后面会继续完善代码。

数据的来源是利用Jsoup直接抓取小米应用商店官网的数据（很尴尬，不会利用java去爬虫，现在就只能先这样弄了）。
主要还是抱着一种学习的态度，熟悉第三方库的使用和熟悉基本Android的开发流程。

部分功能还未实现，做得不完善的地方，希望大家多给些建议。

## 环境依赖
    * com.android.tools.build:gradle:2.2.2
    * compileSdkVersion 26
    * buildToolsVersion "26.0.0"
    * minSdkVersion 15
    * targetSdkVersion 28

## 开源框架

* Jsoup解析 https://github.com/jhy/jsoup
* Butterknife注解式框架 http://jakewharton.github.io/butterknife/
* Glide图片加载框架 https://github.com/bumptech/glide
* OkGo 网络框架 https://github.com/jeasonlzy/okhttp-OkGo
* Retrofi网络请求框架 https://github.com/square/retrofit
* SmartRefreshLayout下拉刷新上拉加载库 https://github.com/scwang90/SmartRefreshLayout
* Butterknife事件注解 https://github.com/JakeWharton/butterknife
* FlycoTabLayout库 https://github.com/H07000223/FlycoTabLayout


## 功能
* 完成基本界面
* 加载并显示不同分类的APP页面，利用Fragment和ViewPager实现懒加载
* 对比本地已安装应用，判断是否需要安装或者升级 （抄的）
* 搜索功能
* 多线程断点下载（暂时是假数据，因为抓取不到每个应用的apk的下载地址）
* 省流量模式

## 屏幕截图
<p>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/detail.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/menu.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/download.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/featured.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/rank.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/manage.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/search.jpg"/>
<img width="288" height="500" src="https://github.com/775269512/ShopDemo/blob/master/screenShot/setting.jpg"/>
</p>

## 具体代码介绍

### Jsoup获取数据

#### 简介：
Jsoup 是一款Java 的HTML解析器，可直接解析某个URL地址、HTML文本内容。它提供了一套非常省力的API，可通过DOM，CSS以及类似于jQuery的操作方法来取出和操作数据。

#### 设计：
项目中关于Jsoup的操作代码，是在一个JsoupUtil类中，网络请求获取到源代码后就调用JsoupUtil中的方法进行解析操作，每一个App的信息保存在AppInfo的实体类中。

#### 步骤：
* 比如用chorme浏览器，打开小米应用商店官网，然后按F12，就可以看到网站的代码了，然后我们查看要抓取的数据所在的节点，然后根据节点的属性（比如class或者id）获取到指定节点的元素。
* 以获取精品页面的App为例子，按F12查看，定位到App列表所在的节点，首先是一个class属性为applist-wrap的div节点中，然后是里面的一个叫class属性为applist的ul表格中，然后表格中每一个元素就是一个App的信息。
* 对于每一个子节点，根据需要获取相应的信息，保存到AppInfo的实体类中。具体代码如下：

'''

	public static List<AppInfo> getAPPInfoList(String result) {
       Log.d("test", "getAPPInfoList:" + result);
       List<AppInfo> appInfoList = new ArrayList<>();
       Document document = Jsoup.parse(result);
       Elements elements = document.select("div.applist-wrap").first().select("ul.applist").select("li");
       for (int i = 0; i < elements.size(); i++) {
           Element element = elements.get(i);
           AppInfo appInfo = new AppInfo();
           //抓取相应的信息
           appInfo.setAppName(element.select("h5").text());
           appInfo.setIcon(element.select("img").attr("data-src"));
           appInfo.setDetailUrl(element.select("a").attr("href"));
           appInfo.setCategory(element.select("p.app-desc").first().text());

           appInfoList.add(appInfo);
       }
       return appInfoList;
    }
   
'''


## 网络请求

主要是利用Retrofit（同时学习Retrofit的使用）
爬取的应用数据网站：
>小米的应用商城：http://app.mi.com
>>含有精品推荐和其他的排行、分类
    
#### 具体接口如下

'''

public interface XiaoMiInterface {

    public static final String BASE_URL = "http://app.mi.com";

    /**
     * 精品推荐
     *
     * @return
     */
    @GET("/allFeaturedList")
    Observable<String> getAllFeaturedList();

    /**
     * 应用的详情页面
     */
    @GET("/details")
    Observable<String> getDetailInfo(@Query("id") String detailUrl);
    
    /**
     * 应用排行
     *
     * @return
     */
    @GET("/topList")
    Observable<String> getRankList(@Query("page") int page);
    
    /**
     * 获取分类的排行
     */
    @GET("/catTopList/{category}")
    Observable<String> getCatTopList(@Path("category") int category, @Query("page") int page);

    /**
     * 获取分类的精品
     */
    @GET("/hotCatApp/{category}")
    Observable<String> getHotCatApp(@Path("category") int category);

    /**
     * 获取分类的新品
     */
    @GET("/category/{category}")
    Observable<String> getCatNewApp(@Path("category") int category,
                                    @Query("page") int page);
    
    /**
     * 搜索
     */
    @GET("/search")
    Observable<String> search(@Query("keywords") String keywords);
    
    /**
     * 应用的下载地址
     */
    @GET("/download/{id}")
    Observable<String> donwLoad(@Path("id") int id);

   }

'''
 然后再对网络请求返回的String字符串进行Jsoup解析，返回相应的结果，具体代码是在RetrofitManager的类中.
 
 ### 注：
 * 其他的库或代码借鉴的参考资料网址忘了,侵删。
 * windows系统下的目录树还不太会做。。
 
 ### 联系邮箱
 * 775269512@qq.com
