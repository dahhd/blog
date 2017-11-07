---

title: KBPhotoSelector，一款iOS照片集选择、浏览、删除开源库

date: 2017-05-20 15:10:10

tags: iOS

categories: 混口饭吃

---


由于公司项目的一个新模块需要做一些社交之类的需求，类似于简单的朋友圈，可以支持用户发表纯文字内容、带有照片的配文。这时需要从系统相册中选择所需要的图片进行发表，开始这个需求之前网上找过一些轮子，但是都不太满意，功能不完善、细节处理的不够，包括只有选择照片，不支持多选，无最大照片选择数的根据需求自定义，选择之后不能进行删除、没有对其压缩等等。所以就有了下面这款新的轮子了，虽然某种意义上也是重复造轮子了，但为了更方便、用起来更舒心，我想还是有必要的，有需要的朋友可以尝试在您的项目中引用，你需要写的可能不超过三行代码。



## KBPhotoSelector

#### 简单介绍一下这款工具的优势：

- 支持照片多选，真正意义上的多选，可根据需求自定义选择数量
- 支持大图预览，预览的过程中可进行筛选
- 支持文件夹列表分类，便于进行分类选择，基于苹果iOS8推出的PhotoKit框架，以后可能会增加新特性
- 支持删除提供了专门删除的页面，这需要基于开发者自己模块具体操作，框架本身我已提供索引
- 支持回到分类页面，进行重新选择照片分类时，记录之前的所有选择，不会再需要重新手动选择一遍（目前这点我看了微信的照片选择器，并没有这么智能）


#### 项目地址：
<https://github.com/Bofearless/KBPhotoSelector.git>


#### 示例介绍具体用法：

框架已发布到cocoapods上，cocoapods是iOS上一款非常流行且强大的开源库管理工具，如果有不了解的同学，可以先去了解下，应该能极大的提高你的开发效率。这里友情链接一篇对cocoapods介绍较全面的[文章](http://blog.devtang.com/2014/05/25/use-cocoapod-to-manage-ios-lib-dependency/)，以供了解。

**两种安装方式(推荐使用pods)：**
1、podfile文件： pod 'KBPhotoSelector'
2、github下载后解压，拖入项目合适位置

<br/>


``` Swift

#import <KBPhotoSelector/KBPhotoSelector.h>
#import <KBPhotoSelector/KBBasePopView1.h>  //这个是弹起底部选择框的customView

    KBPhotoSelector *photoSelector = [[KBPhotoSelector alloc]init];
    photoSelector.maxSelectCount = count; //这里count就是你所想要支持选择照片的最多数量
    KBBasePopView1 *baseView = [KBBasePopView1 popupWithView:photoSelector];
    [photoSelector showPreviewPhotoWithSender:self animate:YES lastSelectPhotoModels:nil completion:^(NSArray<UIImage *> * _Nonnull selectPhotos, NSArray<KBSelectPhotoModel *> * _Nonnull selectPhotoModels) {
        [baseView hide];

           //这里提供了两个回调数组，第一个就是你所选择的照片数组，拿到数据后进行显示加载等后续操作就好；
           //第二个数组是PhotoKit对应的PHAsset类型封装的Model模型，有需要的同学也可用作其他操作。
        NSLog(@"selectPhotos is %@",selectPhotos);
    }];

    //取消按钮，hide弹起的popupView
    [photoSelector setCanceBlock:^{
        [baseView hide];
    }];
    [baseView show];

```

```Swift

//框架中，我其实还提供了选择完成后，在你所呈现的浏览页面上，点击每个itme，进行浏览、删除等操作
//具体的类是：KBPhotoBrowserDeleteController

#import <KBPhotoSelector/KBPhotoBrowserDeleteController.h>
//点击某张照片时，需要传的参数：当前所有找照片数组；当前被点击item(照片)的索引
    self.postsView.didSelectedItemBlock = ^(NSMutableArray *arr, NSInteger index) {
        KBPhotoBrowserDeleteController *nextVc = [[KBPhotoBrowserDeleteController alloc]init];
        nextVc.currentSelectedImgeArr = arr;
        nextVc.currentIndex = index;

//删除后回调
        [nextVc setCompeletionDeleteBlock:^(NSMutableArray<UIImage *> *callBackArr) {
            [weakSelf.postsView setUpdateDataSource:callBackArr];
        }];
        [weakSelf.navigationController pushViewController:nextVc animated:YES];
    };

```


好了，以上就是基本的使用示例了，集成起来很简单，实现几个暴露的API方法就能满足你的所有需求。上面是代码的示范，之后我会提供一下具体的截图样例，上传以供参考。







