---
title: 基于Jenkins-Gitlab-蒲公英-附上shell脚本
date: 2019-04-11 19:03:00
categories: "CI/CD"
tags:
  - iOS
---
# Jenkins 持续集成

## 自动化打包分发 
> 基于Jenkins + Gitlab + 蒲公英

### 部署流程

#### 安装Jenkins
1. 安装Java环境，目前jenkins只支持jdk8 [下载地址](https://www.oracle.com/technetwork/java/javase/downloads/index.html)
> 1. 安装完使用命令 java -version检查当前版本
> 2. java选择如图

![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/1.png)

2. 安装 Jenkins
- 先确保已安装[Homebrew](https://brew.sh/index_zh-cn) 
- 命令安装 brew install jenkins
- 设置开机自启动
    - 创建一个链接到开机启动文件夹里
    
    ```
    ln -sfv /usr/local/opt/jenkins/*.plist ~/Library/LaunchAgents
    ```

    - 手动启动jenkins 
    
    ```
    launchctl load ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
    ```

    - 手动关闭jenkins
    
    ```
    launchctl unload ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
    ```

3. 启动 Jenkins
> 浏览器打开连接
- http://localhost:8080

4. 按照提示，找到/Users/wsh/.jenkins/secrets/initialAdminPassword 这个目录下的initialAdminPassword文件
5. Install suggested plugins 安装推荐插件
6. 安装GitLab Plugin 和 Gitlab Hook Plugin

#### 配置shell脚本
> 注意 将shell脚本保存到工程目录下  保存为 zzq.sh

```
SECONDS=0

#默认使用的语言是英文
export LANG=zh_CN.UTF-8
export LC_ALL=zh_CN.UTF-8

#假设脚本放置在与项目相同的路径下
project_dir=$(pwd)

#编辑上传的文件全部放于此路径下,不影响原工程 ，保存项目目录地址
upload_dir="/Users/addcn/Documents/591TestAndUpload"

if [ -d "$upload_dir" ]; then
    echo "编辑上传的文件输出文件目录存在,目录为:$upload_dir" 
else 
    echo "编辑上传的文件目录不存在" 
    mkdir -pv $upload_dir
    echo "创建打包文件目录${upload_dir}成功"
fi


#打包配置环境
configuration="Release"

#判断是用的xcodeproj还是直接xcworkspace，xcworkspace设置为true，否则设置为false
isWorkSpace=true

#项目名称
scheme=`find . -name *.xcodeproj | awk -F "[/.]" '{print $(NF-1)}'`

projectName="${scheme}.xcworkspace"

#确定工程名称，如果用了cocopods，则使用xcworkspace，但是没有该文件则使用xcodeproj进行

if [ -a "$project_dir/$projectName" ]; then
	isWorkSpace=true
	projectName="${scheme}.xcworkspace"
else
	isWorkSpace=false
	projectName="${scheme}.xcodeproj"
fi

#指定项目地址
project_path="$project_dir/$projectName"

#确认输出日期
buildDate=$(date +%Y%m%d%H%M%S)

#指定输出路径
output_path="$upload_dir/package$buildDate"

#指定输出归档文件地址
archive_path="$output_path/${scheme}.xcarchive"

#指定输出ipa名称
ipa_name="${scheme}.ipa"

#指定输出ipa地址
ipa_path="$output_path/$ipa_name"

#指定xarchive文件导出授权样式 bundle id
provisioningProfileName="XC iOS: com.xxxx.xxxxxxx"

#指定打包所使用的输出方式，目前支持app-store, package, ad-hoc, enterprise, development, 和developer-id，即xcodebuild的method参数
export_method='ad-hoc'

###############获取版本号,bundleID
infoPlist="$project_dir/$scheme-Info.plist"
bundleVersion=`/usr/libexec/PlistBuddy -c "Print CFBundleShortVersionString" $infoPlist`
bundleIdentifier=`/usr/libexec/PlistBuddy -c "Print CFBundleIdentifier" $infoPlist`
bundleBuildVersion=`/usr/libexec/PlistBuddy -c "Print CFBundleVersion" $infoPlist`
displayname=`/usr/libexec/PlistBuddy -c "Print CFBundleDisplayName" $infoPlist`

#输出设定的变量值
echo  "项目名:$projectName"

echo "===项目路径: ${project_path}==="

echo "===打包xarchive文件路径: ${archive_path}==="

echo "===打包ipa文件路径: ${ipa_path}==="

echo "~~~~~~~~~~~~~~~~~~~开始编译~~~~~~~~~~~~~~~~~~~"

#处理没有输出打包文件目录的情况
if [ -d "$output_path" ]; then
    echo "打包文件输出文件目录存在,目录为:$output_path" 
else 
    echo "打包文件目录不存在" 
    mkdir -pv $output_path
    echo "创建打包文件目录${output_path}成功"
fi

#处理编译文件目录的情况
cd $upload_dir
rm -rf ./build
buildAppToDir="$upload_dir/build" #编译打包完成后.app文件存放的目录

#重新进入当前项目路径下
cd $project_dir

security unlock-keychain -p "addcn" /Users/addcn/Library/Keychains/login.keychain

#开始编译app
if $isWorkSpace ; then  #判断编译方式
    echo  "开始编译workspace...." 
    xcodebuild  -workspace $projectName -scheme $scheme  -configuration $configuration clean build SYMROOT=$buildAppToDir
else
    echo  "开始编译target...."
    xcodebuild  -target  $projectName  -configuration $configuration clean build SYMROOT=$buildAppToDir
fi

#判断编译结果
if test $? -eq 0
then
echo "~~~~~~~~~~~~~~~~~~~编译成功~~~~~~~~~~~~~~~~~~~"
else
echo "~~~~~~~~~~~~~~~~~~~编译失败~~~~~~~~~~~~~~~~~~~"
exit 1
fi

echo "开始打包$scheme.app成$scheme.ipa....."
cd $upload_dir
findFolderName=`find . -name "$configuration-*" -type d |xargs basename` #查找目录
appDir=$buildAppToDir/$findFolderName  #app所在路径

#重新进入当前项目路径下
cd $project_dir

echo "打包xarchive文件,路径为:$archive_path"
if $isWorkSpace ; then  #判断编译方式
    echo  "开始打包workspace...." 
    xcodebuild -workspace $projectName -scheme $scheme -destination generic/platform=iOS archive -configuration ${configuration} ONLY_ACTIVE_ARCH=NO -archivePath $archive_path
else
    echo  "开始打包target...."
    xcodebuild -target $projectName -scheme $scheme -destination generic/platform=iOS archive -configuration ${configuration} ONLY_ACTIVE_ARCH=NO -archivePath $archive_path
fi

#检查文件是否存在
if [ -a "$archive_path" ]; then
echo "打包$scheme.xcarchive成功."
else
echo "打包$scheme.xcarchive失败."
exit 1
fi

# echo "开始导出$scheme.xcarchive成$scheme.ipa....."
# xcodebuild -exportArchive -exportFormat ipa -archivePath $archive_path -exportPath $appDir/$ipa_name -exportProvisioningProfile $provisioningProfileName

echo "开始打包$scheme.app成$scheme.ipa....."
xcrun -sdk iphoneos PackageApplication -v $appDir/$scheme.app -o $appDir/$ipa_name #将app打包成ipa

echo "检查打包文件$appDir/${ipa_name}是否存在"
if [ -f "$appDir/$ipa_name" ];then
echo "打包${ipa_name}成功."
else
echo "打包${ipa_name}失败."
exit 1
fi

cp -f -p $appDir/$ipa_name $ipa_path   #拷贝ipa文件
echo "复制${ipa_name}到${ipa_path}成功"


echo "~~~~~~~~~~~~~~~~~~~结束打包，处理成功~~~~~~~~~~~~~~~~~~~"
#输出总用时
echo "===Finished.编译加打包共花费时间: ${SECONDS}s==="

 
echo "正在上传至蒲公英平台"

echo "版本:${bundleVersion}"

#蒲公英的ukey apikey
curl -F "file=@${ipa_path}" -F "uKey=xxxxxxxxxxxxx" -F "_api_key=xxxxxxxxxxxxxxx" https://www.pgyer.com/apiv1/app/upload > $output_path/code.text

result=`cat $output_path/code.text`

result1=`echo "${result##*"appQRCodeURL"}"`

length=`expr ${#result1} - 6`

result2=`echo ${result1:3:$length}`

result3=`echo $result2 | sed 's:\\\/:\/:g'`

if [ ! $result3 ];
then
   echo "~~~~~~~~~~~~~~~~~~~上传失败~~~~~~~~~~~~~~~~~~~"
else
   echo "~~~~~~~~~~~~~~~~~~~上传成功~~~~~~~~~~~~~~~~~~~"
   echo "生成的二维码链接为:${result3}"
fi

# echo "删除编译文件包和打包"
# rm -rf $buildAppToDir
# rm -rf $output_path

#输出总用时
echo "===Finished.共花费时间: ${SECONDS}s==="

```

#### 配置Jenkins
1. 构建任务
    1. 选择新建任务
    2. 如建远程develop。保存名称为remove_develop 项目
    
![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/2.png)
    
2. 配置任务-源码管理

![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/3.png)

3. 配置任务 - 定时触发器配置(可选)[参考](https://www.jianshu.com/p/509c59391b3b)


![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/4.png)


4. 配置任务 - 构建执行shell脚本命令


![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/5.png)



#### Jenkins配置结果
##### 项目任务建立 （本地dev 远程dev 远程master）


![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/6.png)

##### 执行构建

![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/7.png)

##### 执行历史

![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/8.png)

#### 蒲公英平台 

![](/images/2019/基于Jenkins-Gitlab-蒲公英-附上shell脚本/9.png)


## 局域网ip + 端口访问失败

> 修改 
- homebrew.mxcl.jenkins.plist 的 httpListenAddress 为 0.0.0.0
> 目录地址
- ~/Library/LaunchAgents/homebrew.mxcl.jenkins.plist
> 重启
- brew services stop jenkins
- brew services start jenkins

## 打包失败
> 如报异常:
xcrun: error: unable to find utility "PackageApplication", not a developer tool or in PATH
则根据如下链接操作:
> 1. Xcode脚本自动化打包问题：xcrun: error: unable to find utility "PackageApplication", not a developer tool or in PATH
> 2. 后面根据对比发现新版的Xcode少了这个PackageApplication（转注：PackageApplication在前几个版本已被标识为废弃，在8.3版本彻底移除了）
先去找个旧版的Xcode里面copy一份过来
> 3. 放到下面这个目录：
/Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/
> 4. 然后执行命令：
> - sudo xcode-select -switch /Applications/Xcode.app/Contents/Developer/
> - chmod +x /Applications/Xcode.app/Contents/Developer/Platforms/iPhoneOS.platform/Developer/usr/bin/PackageApplication

- 附上PackageApplication下载地址：
- https://pan.baidu.com/s/1jHJF2Lo
- [参考链接](https://www.cnblogs.com/Crazy-ZY/p/7115076.html)
