## · DCBannerView
### · introduce

###### · DCBannerViewDataModel
```
    struct DCBannerViewData {
        let imageUrl: String?
        let imageLink: String?
    
        init(url: String, link: String) {
            self.imageUrl = url
            self.imageLink = link
        }
    }
```

###### · convenience init
``` 
    /// - Parameters:
    ///   - width: banner width
    ///   - height: banner height
    ///   - dataSource: dataSource
    ///   - callback: callBack
    convenience init(width: CGFloat, height: CGFloat, dataSource: [DCBannerViewData], callback: @escaping callbackBlock) {
        self.init(frame: CGRect(x: 0, y: 0, width: width, height: height))
        _width = width
        _height = height
        imagesData = dataSource
        _bannerViewClick = callback
    }
```

###### · Used
``` 
    let data = [DCBannerViewData(url: "image01", link: "https://www.baidu.com"),
                DCBannerViewData(url: "image02", link: "https://m.vmall.com/index"),
                DCBannerViewData(url: "image03", link: "https://www.microsoftstore.com.cn/cart#"),
                DCBannerViewData(url: "image04", link: "https://www.apple.com"),
                DCBannerViewData(url: "image05", link: "https://www.yahoo.com")];
                    
    let bannerView = DCBannerView.init(width: UIScreen.main.bounds.size.width, height: 200, dataSource: data) { (link) in
        print("link:\(link)")
    }
```

###### · others
    DCBannerView can support network images. But you need to Pod network image loading tool. 
    eg: pod 'Kingfisher'
        pod 'SDWebImage'
