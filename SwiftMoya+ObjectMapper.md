## Moya + ObjectMapper 运用
### · [Moya](https://github.com/Moya/Moya) 和 [ObjectMapper](https://github.com/Hearst-DD/ObjectMapper)
	Moya: 网络抽象层，它在底层将Alamofire进行封装，对外提供更简洁的接口供开发者调用。
	ObjectMapper: 在 Swift 下数据转模型的非常好用，并且很 Swift 的一个框架
#### · 在项目中导入
	控制台分别 pod search Moya 和 ObjectMapper 
	不加版本号每次pod都会更新到最新版本，若使用固定版本的pod 应加入版本号：pod 'xxx', '~> x.x.x'
	pod 'ObjectMapper'
	pod 'Moya/RxSwift'

![image](https://github.com/DRCHENHAN/DRCHENHAN/blob/master/images/Moya.jpg)


### · Moya 和 ObjectMapper 的使用
#### 先声明一个请求enum，请求api列表
	public enum RequestAPI {
		case channels  //获取频道列表
		case playlist(String) //获取歌曲
	 }
#### 然后扩展这个enum 并遵循 TargetType 协议

##### `TestProvider.swift`
```
import Foundation
import Moya
import ObjectMapper

let TestProvider = MoyaProvider<RequestAPI> ()

public enum RequestAPI {
    case channels  //获取频道列表
    case playlist(String) //获取歌曲
}

extension RequestAPI: TargetType {
    public var baseURL: URL {
        switch self {
        case .channels:
            return URL(string: "https://www.douban.com")!
        case .playlist(_):
            return URL(string: "https://douban.fm")!
        }
    }
    
    //各个请求的具体路径
    public var path: String {
        switch self {
        case .channels:
            return "/j/app/radio/channels"
        case .playlist(_):
            return "/j/mine/playlist"
        }
    }

    //请求类型
    public var method: Moya.Method {
        return .get
    }
    
    //请求任务事件（这里附带上参数）
    public var task: Task {
        switch self {
        case .playlist(let channel):
            var params: [String: Any] = [:]
            params["channel"] = channel
            params["type"] = "n"
            params["from"] = "mainsite"
            return .requestParameters(parameters: params,
                                      encoding: URLEncoding.default)
        default:
            return .requestPlain
        }
    }
    
    //是否执行Alamofire验证
    public var validate: Bool {
        return false
    }
    
    //这个就是做单元测试模拟的数据，只会在单元测试文件中有作用
    public var sampleData: Data {
        return "{}".data(using: String.Encoding.utf8)!
    }
    
    //请求头
    public var headers: [String: String]? {
        return nil
    }
}
```
    
#### 使用ObjectMapper , 添加扩展

##### `Observable+ObjectMapper.swift`
```
import Foundation
import RxSwift
import ObjectMapper

extension Observable {
    func mapObject<T:Mappable>(Type:T.Type) -> Observable<T> {
        return self.map { response in
            //if response is a dictionary, then use ObjectMapper to map the dictionary
            //if not throw an error
            guard let dict = response as? [String: Any] else {
                throw RxSwiftMoyaError.ParseJSONError
            }
            
            return Mapper<T>().map(JSON: dict)!
        }
    }
    
    func mapArray<T: Mappable>(type: T.Type) -> Observable<[T]> {
        return self.map { response in
            //if response is an array of dictionaries, then use ObjectMapper to map the dictionary
            //if not, throw an error
            guard let array = response as? [Any] else {
                throw RxSwiftMoyaError.ParseJSONError
            }
            
            guard let dicts = array as? [[String: Any]] else {
                throw RxSwiftMoyaError.ParseJSONError
            }
            
            return Mapper<T>().mapArray(JSONArray: dicts)
        }
    }
}

enum RxSwiftMoyaError: String {
    case ParseJSONError
    case OtherError
}

extension RxSwiftMoyaError: Swift.Error { }
```
    
#### 创建请求封装，简化请求
##### `SwiftNetWork.swift`
```
import Foundation
import Moya

struct SwiftNetWork {
    
    // 请求成功的回调
    typealias successCallback = (_ result: Response) -> Void
    // 请求失败的回调
    typealias failureCallback = (_ error: MoyaError) -> Void
    
    // 单例
    static let provider = MoyaProvider<RequestAPI>()
    
    // 发送网络请求
    static func request(
        target: RequestAPI,
        success: @escaping successCallback,
        failure: @escaping failureCallback
        ) {
        
        provider.request(target) { result in
            switch result {
            case let .success(response):
                do {
                    try success(response) // 测试用JSON数据
                } catch {
                    failure(MoyaError.jsonMapping(response))
                }
            case let .failure(error):
                failure(error)
            }
        }
    }
}
```
	
#### 创建数据模型
###### 推荐使用 JSONExport 软件

###### `TestModel.swift`
```
import Foundation 
import ObjectMapper


class TestModel : NSObject, NSCoding, Mappable{

	var channels : [TestChannelList]?


	class func newInstance(map: Map) -> Mappable?{
		return TestModel()
	}
	required init?(map: Map){}
	private override init(){}

	func mapping(map: Map)
	{
		channels <- map["channels"]
		
	}

    /**
    * NSCoding required initializer.
    * Fills the data from the passed decoder
    */
    @objc required init(coder aDecoder: NSCoder)
	{
         channels = aDecoder.decodeObject(forKey: "channels") as? [TestChannelList]

	}

    /**
    * NSCoding required method.
    * Encodes mode properties into the decoder
    */
    @objc func encode(with aCoder: NSCoder)
	{
		if channels != nil{
			aCoder.encode(channels, forKey: "channels")
		}

	}

}
```

##### `TestChannelList.swift`
```
import Foundation 
import ObjectMapper


class TestChannelList : NSObject, NSCoding, Mappable{

	var abbrEn : String?
	var channelId : String?
	var name : String?
	var nameEn : String?
	var seqId : Int?


	class func newInstance(map: Map) -> Mappable?{
		return TestChannelList()
	}
	required init?(map: Map){}
	private override init(){}

	func mapping(map: Map)
	{
		abbrEn <- map["abbr_en"]
		channelId <- map["channel_id"]
		name <- map["name"]
		nameEn <- map["name_en"]
		seqId <- map["seq_id"]
		
	}

    /**
    * NSCoding required initializer.
    * Fills the data from the passed decoder
    */
    @objc required init(coder aDecoder: NSCoder)
	{
         abbrEn = aDecoder.decodeObject(forKey: "abbr_en") as? String
         channelId = aDecoder.decodeObject(forKey: "channel_id") as? String
         name = aDecoder.decodeObject(forKey: "name") as? String
         nameEn = aDecoder.decodeObject(forKey: "name_en") as? String
         seqId = aDecoder.decodeObject(forKey: "seq_id") as? Int

	}

    /**
    * NSCoding required method.
    * Encodes mode properties into the decoder
    */
    @objc func encode(with aCoder: NSCoder)
	{
		if abbrEn != nil{
			aCoder.encode(abbrEn, forKey: "abbr_en")
		}
		if channelId != nil{
			aCoder.encode(channelId, forKey: "channel_id")
		}
		if name != nil{
			aCoder.encode(name, forKey: "name")
		}
		if nameEn != nil{
			aCoder.encode(nameEn, forKey: "name_en")
		}
		if seqId != nil{
			aCoder.encode(seqId, forKey: "seq_id")
		}

	}

}
```

#### 请求数据
##### `ViewController.swift`
```
import UIKit
import ObjectMapper
import Moya
import RxSwift
import RxCocoa

class ViewController: UIViewController {

    let disposeBag = DisposeBag()
    
    override func viewDidLoad() {
        super.viewDidLoad()
        // Do any additional setup after loading the view, typically from a nib.

        TestProvider.rx.request(.channels)
            .filterSuccessfulStatusCodes()
            .mapJSON()
            .asObservable()
            .mapObject(Type: TestModel.self).subscribe(onNext :{ (result) in
                print(result)
            }, onError: { (error) in
                print(error)
            }, onCompleted: nil, onDisposed: nil).disposed(by: disposeBag)
        
    }

    override func didReceiveMemoryWarning() {
        super.didReceiveMemoryWarning()
        // Dispose of any resources that can be recreated.
    }

}
```
		