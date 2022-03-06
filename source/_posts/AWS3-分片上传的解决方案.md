---
title: AWS3 分片上传的解决方案
date: 2021-08-27 17:30:03
categories: "iOS"
tags:
  - iOS
---


## 背景
> - 背景：最近一次业务需求是通过本地录制或者相册视频上传到亚马逊服务
> - 研究：前端小伙伴尝试接入 SDK 发现 AWS3 的上传部分还是需要做很多工作，比如切片部分 > 5M 及 ETAG 处理等
> - 决策：为了减少前端工作，决定采用后端调用 S3 SDK 方式，前端通过后端预签名后的 URL 直接进行文件分段上传

Github：[ TWMultiUploadFileManager ](https://github.com/zeqinjie/TWMultiUploadFileManager)
## 安装
 使用Cocoapods安装，或手动拖入项目

```ruby
pod 'TWMultiUploadFileManager'
```


## 方案
后端执行执行 AWS3 SDK API，前端通过后端预签名后的 URL 直接进行文件分段上传

### 步骤

- step1:  前端对文件选取后进行切片 (> 5M )
- step2: 请求后端上传 aws3 资源 url
- step3: 切片后的文件足个上传 aws3 服务器
- step4: 请求后端对上传完毕的资源文件做校验

### 流程图

![](/images/2021/AWS3-分片上传的解决方案/1.png)

## TWMultiUploadFileManager 组件封装
### 功能
封装了对文件分片处理，以及上传功能

- `具体功能` ☑️
   - maxConcurrentOperationCount：上传线程并发个数（默认3 ）
   - maxSize：文件大小限制（默认2GB ）
   - perSlicedSize：每个分片大小（默认5M）
   - retryTimes：每个分片上传尝试次数（默认3）
   - timeoutInterval：請求時長 （默認 120 s）
   - headerFields：附加 header
   - mimeType：文件上传类型 不为空 （默认 text/plain）
- `TODO` ⏳
   - 上传文件最大时长（秒s）默认7200
   - 最大缓冲分片数（默认100，建议不低于10，不高于100）
   - 附加参数， 目前封装 put 请求，后续会补充 post 请求
- 并发队列管理依赖 [Queuer](https://github.com/FabrizioBrancati/Queuer) 库 
   - 根据场景：自定义 `TWConcurrentOperation`



### 步骤

#### step 1


从相册中选择视频源（文件）


```swift
// MARK: - Action
/// 选择影片
fileprivate func selectPhotoAction(animated: Bool = true) {
    let imagePicker: TZImagePickerController! = TZImagePickerController(maxImagesCount: 9, delegate: self)
    imagePicker.allowPickingVideo = true
    imagePicker.allowPreview = false
    imagePicker.videoMaximumDuration = Macro.videoMaximumDuration
    imagePicker.maxCropVideoDuration = Int(Macro.videoMaximumDuration)
    imagePicker.allowPickingOriginalPhoto = false
    imagePicker.allowPickingImage = false
    imagePicker.allowPickingMultipleVideo = false
    imagePicker.autoDismiss = false
    imagePicker.navLeftBarButtonSettingBlock = { leftButton in
        leftButton?.isHidden = true
    }
    present(imagePicker, animated: animated, completion: nil)
}
    
/// 获取视频资源
fileprivate func handleRequestVideoURL(asset: PHAsset)  {
    /// loading
    print("loading....")
    self.requestVideoURL(asset: asset) { [weak self] (urlasset, url) in
        guard let self = self else { return }
        print("success....")
        self.url = url
        self.asset = asset
        self.uploadVideoView.play(videoUrl: url)
    } failure: { (info) in
        print("fail....")
    }
}
```


对视频源文件进行切片并创建上传资源对象（文件）


```swift
/// 上传影片
fileprivate func uploadVideoAction() {
    guard let url = url, let asset = asset ,let outputPath: String = self.fetchVideoPath(url: url) else { return }
    let relativePath: String = TWMultiFileManager.copyVideoFile(atPath: outputPath, dirPathName: Macro.dirPathName)
    // 创建上传资源对象, 对文件进行切片
    let fileSource: TWMultiUploadFileSource = TWMultiUploadFileSource(
        configure: self.configure,
        filePath: relativePath,
        fileType: .video,
        localIdentifier: asset.localIdentifier
    )
    // 📢 上传前需要从服务端获取每个分片的上传到亚马逊 url ，执行上传
    // fileSource.setFileFragmentRequestUrls([])
    
    uploadFileManager.uploadFileSource(fileSource)
}
```


切片的核心逻辑


```objectivec
/// 切片处理
- (void)cutFileForFragments {
    NSUInteger offset = self.configure.perSlicedSize;
    // 总片数
    NSUInteger totalFileFragment = (self.totalFileSize%offset==0)?(self.totalFileSize/offset):(self.totalFileSize/(offset) + 1);
    self.totalFileFragment = totalFileFragment;
    NSMutableArray<TWMultiUploadFileFragment *> *fragments = [NSMutableArray array];
    for (NSUInteger i = 0; i < totalFileFragment; i++) {
        TWMultiUploadFileFragment *fFragment = [[TWMultiUploadFileFragment alloc] init];
        fFragment.fragmentIndex = i+1; // 从 1 开始
        fFragment.uploadStatus = TWMultiUploadFileUploadStatusWaiting;
        fFragment.fragmentOffset = i * offset;
        if (i != totalFileFragment - 1) {
            fFragment.fragmentSize = offset;
        } else {
            fFragment.fragmentSize = self.totalFileSize - fFragment.fragmentOffset;
        }
        /// 关联属性
        fFragment.localIdentifier = self.localIdentifier;
        fFragment.fragmentId = [NSString stringWithFormat:@"%@-%ld",self.localIdentifier, (long)i];
        fFragment.fragmentName = [NSString stringWithFormat:@"%@-%ld.%@",self.localIdentifier, (long)i, self.fileName.pathExtension];
        fFragment.fileType = self.fileType;
        fFragment.filePath = self.filePath;
        fFragment.totalFileFragment = self.totalFileFragment ;
        fFragment.totalFileSize = self.totalFileSize;
        [fragments addObject:fFragment];
    }
    self.fileFragments = fragments;
}
```


#### step 2


- 业务逻辑：通过后端调用 AWS3 SDK 获取资源文件分片上传的 urls, 后端配合获取上传 aws3 的 url
- 📢 这里也可以上传到自己服务端的 urls ,组件已封装的上传逻辑 put 请求，具体按各自业务修改即可
```swift
// 📢 上传前需要从服务端获取每个分片的上传到亚马逊 url ，执行上传
fileSource.setFileFragmentRequestUrls([])
```

#### step 3


```swift
/// 执行上传到 AWS3 服务端
uploadFileManager.uploadFileSource(fileSource)
```


设置代理回调，当然也支持 block


```swift
extension ViewController: TWMultiUploadFileManagerDelegate {
    /// 准备开始上传
    func prepareStart(_ manager: TWMultiUploadFileManager!, fileSource: TWMultiUploadFileSource!) {
        
    }
    
    /// 文件上传中进度
    func uploadingFileManager(_ manager: TWMultiUploadFileManager!, progress: CGFloat) {
        
    }
    
    /// 完成上传
    func finish(_ manager: TWMultiUploadFileManager!, fileSource: TWMultiUploadFileSource!) {
    
    }
    
    /// 上传失败
    func fail(_ manager: TWMultiUploadFileManager!, fileSource: TWMultiUploadFileSource!, fail code: TWMultiUploadFileUploadErrorCode) {
        
    }
    
    /// 取消上传
    func cancleUploadFileManager(_ manager: TWMultiUploadFileManager!, fileSource: TWMultiUploadFileSource!) {
        
    }
    
    /// 上传中某片文件失败
    func failUploadingFileManager(_ manager: TWMultiUploadFileManager!, fileSource: TWMultiUploadFileSource!, fileFragment: TWMultiUploadFileFragment!, fail code: TWMultiUploadFileUploadErrorCode) {
        
    }
}
```

#### step 4

业务逻辑：最后资源上传完毕后，请求后端接口；对上传完毕的资源文件做校验

### 📢 说明


- 业务逻辑是各自的业务方处理， 本组件封装的是上传功能：包括切片，重试次数，文件大小，分片大小，最大支持分片数等
- 具体看上传资源的配置对象



```objectivec
@interface TWMultiUploadConfigure : NSObject
/// 同时上传线程 默认3
@property (nonatomic, assign) NSInteger maxConcurrentOperationCount;
/// 上传文件最大限制（字节B）默认2GB
@property (nonatomic, assign) NSUInteger maxSize;
/// todo: 上传文件最大时长（秒s）默认7200
@property (nonatomic, assign) NSUInteger maxDuration;
/// todo:  最大缓冲分片数（默认100，建议不低于10，不高于100）
@property (nonatomic, assign) NSUInteger maxSliceds;
/// 每个分片占用大小（字节B）默认5M
@property (nonatomic, assign) NSUInteger perSlicedSize;
/// 每个分片上传尝试次数（默认3）
@property (nonatomic, assign) NSUInteger retryTimes;
/// 請求時長 默認 120 s
@property (nonatomic, assign) NSUInteger timeoutInterval;
/// todo: 附加参数, 目前封装 put ，后续会补充 post 请求
@property (nonatomic, strong) NSDictionary *parameters;
/// 附加 header
@property (nonatomic, strong) NSDictionary<NSString *, NSString *> *headerFields;
/// 文件上传类型 不为空 默认 text/plain
@property (nonatomic, strong) NSString *mimeType;
@end
```

## 业务使用案例
### 场景
![](/images/2021/AWS3-分片上传的解决方案/2.png)


### 业务代码示例
> 项目基于 [ReactorKit](https://github.com/ReactorKit/ReactorKit) 框架


定义视频各个状态

```swift
/// 当前的各种状态
enum SaleHouseVideoUploadStatus: Equatable {
    /// 默认是未选视频的状态
    case unseleted
    /// 开始选择视频，为了解决两次选择视频失败没有变化无法进入监听回调里
    case beginSeletedVideo
    /// 选择视频失败
    case seletedVideoFail(code: SaleHouseVideoUploadStatusSeletedVideoFailCode)
    /// 选择视频成功，准备上传   导航: "上傳"
    case seletedVideoSuccess(asset: PHAsset, url: URL)
    /// 点击上传，准备获取服务器提交信息
    case requestUploadData
    /// 获取上传信息成功
    case requestUploadDataSuccess(uploadInfo: SaleHouseVideoUploadInfoModel)
    /// 分片校验 & 获取上传信息失败 导航: "重新上传"； code: 错误码后端返回用于统计
    case requestUploadDataFail(code: String?)
    /// 上传中， 导航: "取消上传"
    case uploading(progress: CGFloat)
    /// 上传失败，导航: "重新上传"
    case uploadFail(code: TWMultiUploadFileUploadErrorCode)
    /// 文件已上传 aws3 下一步 提交合并
    case uploadFilesComplete
    /// 上传完成后提交服务失败 合并 aws3； code: 错误码后端返回用于统计
    case requestMergeFilesFail(code: String?)
    /// 上传完成后提交合并 aws3 服务成功  导航: "重新上传" "提交"
    case requestMergeFilesComplete(mergeInfo: SaleHouseVideoMergeFilesCompleteModel)
    /// 上传成功，但提交失败 ； code: 错误码后端返回用于统计
    case requestCommitFail(mergeInfo: SaleHouseVideoMergeFilesCompleteModel, code: String?)
    /// 上传成功，并提交成功  导航: "重新上传" "提交"
    case requestCommitSuccess(mergeInfo: SaleHouseVideoMergeFilesCompleteModel)
    ///  取消上传
    case cancel
    /// 处于编辑状态成功，删除 && 编辑影片未完成预处理预览 导航: "删除影片" "重新上传"
    case requestEditInfoSuccess(fileInfo: SaleHouseVideoEditInfoModel)
    /// 获取编辑状态失败
    case requestEditInfoFail
    /// 删除成功， 删除失败还是之前状态
    case deleteSuccess
}

```

SaleHouseVideoUploadReactor 实现 `Reactor` 协议

```swift
// MARK: - Reactor
extension SaleHouseVideoUploadReactor: Reactor {
    enum Action {
        /// 视频校验
        case checkSelectedVideo(asset: PHAsset)
        /// 点击上传获取上传数据
        case requestUploadData
        /// 重置视频选择
        case resetSelectedVideo
        /// 取消上传
        case cancleUploadVideo
        /// 上传完成后，提交视频
        case commitUploadedVideo
        /// 上传失败，重新上传
        case reuploadVideo
        /// 删除影片
        case deleteVideo
        /// 获取影片信息
        case requestEditVideoInfo
        /// 恢复原编辑视频信息
        case resetEditVideoInfo
    }
    
    enum Mutation {
        case setStatus(SaleHouseVideoUploadStatus)
        case setHUDAction(HUDAction)
    }
    
    struct State {
        /// 当前的操作的状态，默认是未选视频
        var status: SaleHouseVideoUploadStatus = .unseleted
    }
    
    func mutate(action: Action) -> Observable<Mutation> {
        switch action {
        case let .checkSelectedVideo(asset: asset):
            return checkSelectedVideo(asset: asset)
        case .cancleUploadVideo:
            return cancleUploadVideo()
        case .resetSelectedVideo:
            return updateStatus(.unseleted)
        case .requestUploadData:
            return requestUploadData()
        case .commitUploadedVideo:
            return commitUploadedVideo()
        case .reuploadVideo:
            return reuploadVideo()
        case .deleteVideo:
            return deleteVideo()
        case .requestEditVideoInfo:
            return fetchEditVideoInfoRequest()
        case .resetEditVideoInfo:
            return resetEditVideoInfo()
        }
    }
    
    func reduce(state: State, mutation: Mutation) -> State {
        var newState = state
        switch mutation {
        case let .setStatus(status):
            newState.status = status
        case let .setHUDAction(action):
            hudAction.accept(action)
        }
        return newState
    }
}
```

### 步骤

#### step 1

基于 TZImagePickerController 获取视频，触发 checkSelectedVideo 校验视频事件

```swift
/// 单个视频选择回调
func imagePickerController(_ picker: TZImagePickerController!, didFinishPickingPhotos photos: [UIImage]!, sourceAssets assets: [Any]!, isSelectOriginalPhoto: Bool) {
    picker.dismiss(animated: true, completion: nil)
    guard let asset = assets.first as? PHAsset else { return }
    self.pickerDissmissActionType = .dismiss
    self.reactor?.action.onNext(.checkSelectedVideo(asset: asset))
}
```

```swift
/// 校验视频资源
fileprivate func checkSelectedVideo(asset: PHAsset) -> Observable<Mutation>  {
    DLog("assets :\(asset)")
    let beginSeletedVideo: Observable<Mutation> = updateStatus(.beginSeletedVideo)
    if asset.duration > Macro.videoMaximumDuration {
        let seletedVideoFail = updateStatus(.seletedVideoFail(code: .overTime))
        return .concat([beginSeletedVideo, seletedVideoFail])
    }
    let size = SaleHouseVideoUploadHelper.requestVideoSize(asset)
    if size > Macro.videoMaximumSize {
        let seletedVideoFail: Observable<Mutation> = updateStatus(.seletedVideoFail(code: .overSize))
        return .concat([beginSeletedVideo, seletedVideoFail])
    }
    switch asset.mediaType {
    case .video:
        let showLoading = Observable.just(Mutation.setHUDAction(.showHint(Macro.loadingVideoSourceTips)))
        let requestVideoURL = handleRequestVideoURL(asset: asset)
        return .concat([showLoading, requestVideoURL])
    default:
        return .empty()
    }
}

/// 请求视频资源
fileprivate func handleRequestVideoURL(asset: PHAsset) -> Observable<Mutation> {
    return Observable.create { observer in
        SaleHouseVideoUploadHelper.requestVideoURL(asset: asset) { [weak self] (urlasset, url) in
            self?.url = url
            self?.asset = asset
            // 视频选择成功                                                 
            observer.onNext(.setStatus(.seletedVideoSuccess(asset: asset, url: url)))
            observer.onNext(.setHUDAction(.hide))
            observer.onCompleted()
        } failure: { (info) in
            observer.onNext(.setHUDAction(.hideHint(Macro.loadFailVideoSourceTips)))
            observer.onCompleted()
        }
        return Disposables.create()
    }
}
```


用户点击上传，获取切片上传资源

```swift
///  对上传 aws3 的 rxswift 封装
/// - Parameters:
///   - uploadFileManager: 分片上传管理类
///   - fileSource: 分片资源类
///   - continueUpload: 是否继续上传
/// - Returns: Observable
static func startUpload(
    uploadFileManager: TWMultiUploadFileManager,
    fileSource: TWMultiUploadFileSource,
    continueUpload: Bool = false
) -> Observable<SaleHouseVideoUploadStatus> {
    return Observable.create { observer in
        // 准备开始上传
        uploadFileManager.prepareStartUploadBlock = { (manager, fileSource) in
            observer.onNext(.uploading(progress: 0))
        }
        // 文件上传中进度
        uploadFileManager.uploadingBlock = { (manager, progress) in
            observer.onNext(.uploading(progress: progress))
        }
        // 完成上传
        uploadFileManager.finishUploadBlock = { (manager, fileSource) in
            observer.onNext(.uploadFilesComplete)
            observer.onCompleted()
        }
        // 上传失败
        uploadFileManager.failUploadBlock = { (manager, fileSource, failErrorCode) in
            observer.onNext(.uploadFail(code: failErrorCode))
            observer.onCompleted()
        }
        // 取消上传
        uploadFileManager.cancleUploadBlock = { (manager, fileSource) in
            observer.onNext(.cancel)
            observer.onCompleted()
        }
        // 上传中某片文件失败
        uploadFileManager.failUploadingBlock = { (manager, fileSource, fileFragment, failErrorCode) in
            observer.onNext(.uploadFail(code: failErrorCode))
        }
        if continueUpload {
            uploadFileManager.continue(fileSource)
        } else {
            uploadFileManager.uploadFileSource(fileSource)
        }
        return Disposables.create()
    }
}

/// step1 点击上传获取切片上传资源文件
fileprivate func requestUploadData(continueUpload: Bool = false) -> Observable<Mutation> {
    let fetchUploadDataFail: Observable<Mutation> = .concat([setRequestUploadDataStatus(), requestUploadDataFail()])
    guard let url = url, let asset = asset else { return fetchUploadDataFail }
    guard let outputPath: String = SaleHouseVideoUploadHelper.fetchVideoPath(url: url) else { return fetchUploadDataFail }
    let fetchUploadData = Observable<Mutation>.deferred { [weak self] in
        guard let self = self else { return .empty() }
        if !continueUpload { // 不是继续上传当前这个资源
            // 直接移动文件到指定目录(相对路径)
            let relativePath: String = TWMultiFileManager.copyVideoFile(atPath: outputPath, dirPathName: Macro.dirPathName)
            DLog("relativePath ===> \(relativePath)")
            self.deleteFile() // 删除无效文件, 并对视频进行切片，创建上传资源对象
            let fileSource: TWMultiUploadFileSource = TWMultiUploadFileSource(
                configure: self.configure,
                filePath: relativePath,
                fileType: .video,
                localIdentifier: asset.localIdentifier
            )
            self.currentFileSource = fileSource // 更新文件
        }
        guard let fileSource = self.currentFileSource else  { return fetchUploadDataFail }
        let fetchUploadInfoModel = SaleHouseVideoFetchUploadInfoModel(
            filename: fileSource.fileName,
            category: SaleHouseVideoUploadHeader.video,
            part: TWSwiftGuardValueString(fileSource.totalFileFragment),
            size: TWSwiftGuardValueString(fileSource.totalFileSize),
            pathExtension: TWSwiftGuardValueString(fileSource.pathExtension)
        )
        return self.fetchUploadDataRequest(fetchUploadInfoModel: fetchUploadInfoModel, continueUpload: continueUpload)
    }
    return .concat([setRequestUploadDataStatus(), fetchUploadData])
}
```
#### step 2

```swift
/// step2 获取上传 urls 信息請求
fileprivate func fetchUploadDataRequest(
    fetchUploadInfoModel: SaleHouseVideoFetchUploadInfoModel,
    continueUpload: Bool = false
) -> Observable<Mutation>  {
    // loading 获取服务 url
    let showLoading = Observable.just(Mutation.setHUDAction(.showLoading))
    let params: [String: Any] = [
        "filename" : TWSwiftGuardNullString(fetchUploadInfoModel.filename),
        "part" : TWSwiftGuardNullString(fetchUploadInfoModel.part),
        "size" : TWSwiftGuardNullString(fetchUploadInfoModel.size),
        "category" : TWSwiftGuardNullString(fetchUploadInfoModel.category),
        "type" : TWSwiftGuardNullString(fetchUploadInfoModel.type),
    ]
    let fetchData = TWSwiftHttpTool.rx.request(
        type: .RequestPost,
        url: APIAWSUploadPartUtil,
        parameters:params,
        isCheckLogin: true,
        checkNeedLoginHandler: checkNeedLoginHandler()
    )
    .mapResult()
    .flatMap { [weak self] (status, result) -> Observable<Mutation> in
        guard let self = self else  { return .empty() }
        var hud: Observable<Mutation> = .just(.setHUDAction(.hide))
        var uploadInfoStatus: Observable<Mutation> = self.updateStatus(.requestUploadDataFail(code: nil)) // 默认获取失败
        switch status {
        case let .success(isSuccessStatus, data, msg, _):
            if isSuccessStatus {
                if let uploadInfoModel = self.getUploadInfoModel(data: data)  {
                    uploadInfoStatus = self.startUploadVideo(uploadInfoModel: uploadInfoModel, continueUpload: continueUpload)
                } else {
                    hud = .just(.setHUDAction(.hideHint(Macro.requestUploadDataFailTips)))
                }
            } else {
                hud = .just(.setHUDAction(.hideHint(msg)))
                uploadInfoStatus = self.updateStatus(.requestUploadDataFail(code: self.fetchErrorCode(data: data)))
            }
        case let .error(msg, _):
            hud = .just(.setHUDAction(.hideHint(msg)))
        case .noNet:
            hud = .just(.setHUDAction(.hideHint(TWSwiftHttpTool.Macro.ErrorStr.noNet)))
        }
        return .concat([hud, uploadInfoStatus])
    }
    return .concat([showLoading, fetchData])
}
```

#### step 3

```swift
/// step3 设置获取上传 urls ，并开始上传
/// - Parameters:
///   - uploadInfoModel: 上传资源对象
///   - continueUpload: 是否断点继续上传
fileprivate func startUploadVideo(
    uploadInfoModel: SaleHouseVideoUploadInfoModel?,
    continueUpload: Bool = false
) -> Observable<Mutation>  {
    var uploadInfoStatus: Observable<Mutation> = self.requestUploadDataFail() // 默认获取失败
    if let uploadInfoModel = uploadInfoModel {
        guard let parts = uploadInfoModel.parts , let fileSource = self.currentFileSource else { return uploadInfoStatus }
        uploadInfoStatus = updateStatus(.requestUploadDataSuccess(uploadInfo: uploadInfoModel))
        // step2 设置上传服务的 urls
        fileSource.setFileFragmentRequestUrls(parts.map({ $0.url}))
        // step3 开始上传 aws3
        let uploadStatus = SaleHouseVideoUploadHelper.startUpload(uploadFileManager: self.uploadFileManager, fileSource: fileSource, continueUpload: continueUpload)
            .flatMap { [weak self] status -> Observable<Mutation> in
                var mutation: Observable<Mutation> = .empty()
                guard let self = self else { return mutation }
                switch status {
                case .uploadFilesComplete: // 上传完成，继续下一步,合并操作
                    mutation = self.requestMergeFiles()
                default:
                    break
                }
                return .concat([
                    self.updateStatus(status),
                    mutation
                ])
            }
        return .concat([uploadInfoStatus, uploadStatus])
    }
    return uploadInfoStatus
}
```


#### step 4

上传 aws3 完毕后，请求后端服务接口对资源做合并校验

```swift
/// step4 完成上传提交合併請求
/// - Parameters:
///   - bucketKey: 存储桶路径
///   - uploadId: 上传唯一标识，step1获得
///   - category: 分类，如video
///   - parts: [{"partNumber":1,"Etag":"xxxxx"},{"partNumber":2,"Etag":"xxxxx"}]
fileprivate func mergeFilesUploadRequest(_ mergeFilesUploadInfoModel: SaleHouseVideoMergeFilesUploadInfoModel) -> Observable<Mutation>  {
    var params: [String: Any] = [
        "key" : TWSwiftGuardNullString(mergeFilesUploadInfoModel.key),
        "upload_id" : TWSwiftGuardNullString(mergeFilesUploadInfoModel.upload_id),
        "category" : TWSwiftGuardNullString(mergeFilesUploadInfoModel.category),
        "file_id" : TWSwiftGuardValueNumber(mergeFilesUploadInfoModel.file_id),
    ]
    // etga 校验
    if let parts = mergeFilesUploadInfoModel.parts {
        params["parts"] = parts.mj_JSONString()
    }

    let fetchData = TWSwiftHttpTool.rx.request(
        type: .RequestPost,
        url: APIAWSUploadComplete,
        parameters:params,
        isCheckLogin: true,
        checkNeedLoginHandler: checkNeedLoginHandler()
    )
    .mapResult()
    .flatMap { [weak self] (status, result) -> Observable<Mutation> in
        guard let self = self else  { return .empty() }
        var hud: Observable<Mutation> = .just(.setHUDAction(.hide))
        var mergeFilesInfoStatus: Observable<Mutation> = self.updateStatus(.requestMergeFilesFail(code: nil)) // 默认文件合并失败
        switch status {
        case let .success(isSuccessStatus, data, msg, _):
            if isSuccessStatus {
                mergeFilesInfoStatus = self.getMergeUploadStatus(data: data)
            } else {
                hud = .just(.setHUDAction(.hideHint(msg)))
                mergeFilesInfoStatus = self.updateStatus(.requestMergeFilesFail(code: self.fetchErrorCode(data: data)))
            }
        case let .error(msg, _):
            hud = .just(.setHUDAction(.hideHint(msg)))
        case .noNet:
            hud = .just(.setHUDAction(.hideHint(TWSwiftHttpTool.Macro.ErrorStr.noNet)))
        }
        return .concat([hud, mergeFilesInfoStatus])
    }
    return .concat(fetchData)
}

```

## 参考

- [如何使用 AWS CLI 将文件分段上传到 Amazon S3？](https://aws.amazon.com/cn/premiumsupport/knowledge-center/s3-multipart-upload-cli/)
- [AWS3 API_UploadPart](https://docs.aws.amazon.com/zh_cn/AmazonS3/latest/API/API_UploadPart.html)
- [Amazon S3 Transfer Utility for iOS](https://aws.amazon.com/cn/blogs/mobile/amazon-s3-transfer-utility-for-ios/)



