### OCHBannerView

##### * dataModel

```
struct OCHBannerViewData {
    let imageUrl: String?
    let imageLink: String?
    
    init(url: String, link: String) {
        self.imageUrl = url
        self.imageLink = link
    }
}
```

### used
##### * init(width: CGFloat, height: CGFloat, dataSource: [OCHBannerViewData], callback: @escaping callbackBlock){}

```
let data = [OCHBannerViewData(url: "image01", link: "https://www.baidu.com"),
            OCHBannerViewData(url: "image02", link: "https://m.vmall.com/index"),
            OCHBannerViewData(url: "image03", link: "https://www.microsoftstore.com.cn/cart#"),
            OCHBannerViewData(url: "image04", link: "https://www.apple.com"),
            OCHBannerViewData(url: "image05", link: "https://www.yahoo.com")]  
            
let bannerView = OCHBannerView.init(width: UIScreen.main.bounds.size.width, height: 200, dataSource: data) { (link) in
            print("link:\(link)")
        }
```
