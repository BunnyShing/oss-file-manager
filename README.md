# oss-file-manager 
![npm](https://img.shields.io/npm/dt/oss-file-manager.svg)
![NPM](https://img.shields.io/npm/l/oss-file-manager)
![npm](https://img.shields.io/npm/v/oss-file-manager)

> 基于 Vue、ElementUI、OSS BrowserJS、OSS STS 的OSS在线浏览器

[View on GitHub](https://github.com/BunnyShing/oss-file-manager) 

## 特性
1. 类似 OSS Browser/Windows文件管理器 所见即所得的用户体验
2. 自带ElementUI界面，开箱即用，无需额外配置
3. 支持文件/文件夹拖曳上传
4. 文件默认分片上传，实时显示文件上传进度
5. 支持重命名文件夹
6. 支持删除文件夹及其子文件/文件夹

## 安装

``` bash
npm i oss-file-manager -S
```

## 引入
```javascript
import Vue from 'vue'
import OssFileManager from 'oss-file-manager'

Vue.use(OssFileManager)
```
> 强烈建议更新V1.1.0以上版本，1.1.0版本修正了很多bug及提高了代码健壮性，详见：[更新日志](#更新日志)

## 使用
```html
<!-- 调用示例 -->
<oss-file-manager
    @uploadSuccess="上传成功回调"
    @removeSuccess="删除成功回调"
    @renameSuccess="重命名成功回调"
    access-key-id="OSS STS返回的临时access id"
    access-key-secret="OSS STS返回的临时access secret"
    sts-token="OSS STS返回的token"
    bucket="OSS Bucket"
    root-dir="指定根目录">
</oss-file-manager>
```
> 参数access-key-id与access-key-secret可传入OSS长期密钥，但强烈建议通过OSS STS临时访问凭证来调用。  
> 暴露OSS长期密钥到前端存在巨大安全风险，详见：https://help.aliyun.com/document_detail/100624.htm

## 属性
| 参数 | 说明 | 类型 | 可选值 | 默认值	|
| :---- | :---- | :----: | :----: | :----: |
region | OSS区域 | String | 详见OSS说明文档 | oss-cn-shenzhen
access-key-id | STS临时Access Key Id/长期Access Key Id | String | - | -
access-key-secret | STS临时Access Key Secret/长期Access Key Secret | String| - | -
sts-token | STS Token | String | - | - 
bucket | Bucket名称 | String | - | - 
root-dir | 指定根目录 | String | - | / 
custom-oss-domain | 自定义OSS域名 | String | - | - 
https | OSS域名是否启用HTTPS | Boolean | true/false | true 
hidden-root | 是否隐藏根目录 | Boolean | true/false | true 
allow-upload-folder | 是否允许上传文件夹 | Boolean | true/false | true 
before-upload | 自定义上传文件前置函数，入参为上传的文件对象，返回false或返回Promise且被reject，则终止上传 | Function(file) | - | - 
headers | 上传时的请求头，透传给OSS Client 详见：https://help.aliyun.com/document_detail/383952.html | Object | - | - 
callback | 异步回调配置，透传给OSS Client 详见：https://help.aliyun.com/document_detail/383952.html | Object | - | -

## 事件
| 方法名 | 说明 | 参数 |
| :---- | :---- | :---- |
listObjectsFail | 获取对象列表失败回调 | (err: 错误信息)
uploadSuccess | 上传成功回调 | (res: OSS API 返回上传结果对象)
uploadFail | 上传失败回调 | (err: 错误信息)
removeSuccess | 删除成功回调 | (path: 对象OSS路径)
removeFail | 删除失败回调 | (err: 错误信息)
renameSuccess | 重命名成功回调 | (newPath: 对象新OSS路径,oldPath: 对象旧OSS路径)
renameFail | 重命名失败回调 | (err: 错误信息)

## 更新日志
### V1.1.0
本版本主要更新如下：  
1. 修正未正确监听OSS相关配置（Access Key Id、Access Key Secret、STS Token之类）的属性变动，导致初始化组件/更新access-key-id时未重载对象列表的bug
2. 修正在没有根目录权限时未正确监听root-dir，导致root-dir更新后也不会重载对象列表的bug
3. 通过向list方法传入forceRefresh属性的方式来控制多次请求listObjects接口引起列表中对象重复的问题
4. 增加listObjectsFail事件，用于外部对获取对象列表失败时进行处理
5. 去掉listLoading属性，不再通过判断listLoading来控制多次频繁请求导致列表中对象重复的问题
6. 优化了禁止上传/创建文件夹时的按钮样式
7. 移除refresh()方法来刷新对象列表的代码
8. 修正调用$message.error未传入字符串类型的错误信息，导致错误信息未正确显示的bug
