# oss-file-manager 
![npm](https://img.shields.io/npm/dt/oss-file-manager.svg)
![NPM](https://img.shields.io/npm/l/oss-file-manager)
![npm](https://img.shields.io/npm/v/oss-file-manager)

> 基于 Vue、ElementUI、OSS BrowserJS、OSS STS 的OSS管理组件

[View on GitHub](https://github.com/BunnyShing/oss-file-manager) 

## 特性
1. 类似 OSS Browser/Windows文件管理器 所见即所得的用户体验
2. 自带ElementUI界面，开箱即用，无需额外配置
3. 支持文件/文件夹拖曳上传
4. 文件默认分片上传，实时显示文件上传进度
5. 支持重命名文件夹
6. 支持删除文件夹及下级文件/文件夹

## 安装

``` bash
npm i oss-file-manager
```

## 引入
```javascript
import Vue from 'vue'
import OssFileManager from 'oss-file-manager'

Vue.use(OssFileManager)
```

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
    root-dir="指定根目录"
    >
</oss-file-manager>
```
> 参数access-key-id与access-key-secret可传入OSS长期密钥，但强烈建议通过OSS STS临时访问凭证来调用。
> 暴露OSS长期密钥到前端存在巨大安全风险，详见：https://help.aliyun.com/document_detail/100624.htm

## Attributes
| 参数 | 说明 | 类型 | 可选值 | 默认值	|
| :---- | :---- | :----: | :----: | :----: |
region | OSS区域 | String | 详见OSS说明文档 | oss-cn-shenzhen
access-key-id | STS临时Access Id/长期Access Id | String | - | -
access-key-secret | STS临时Access Secret/长期Access Secret | String| - | -
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

## Methods
| 方法名 | 说明 | 参数 |
| :---- | :---- | :---- |
uploadSuccess | 上传成功回调 | (res: OSS API 返回上传结果对象)
uploadFail | 上传失败回调 | (err: 错误信息)
removeSuccess | 删除成功回调 | (path: 对象OSS路径)
removeFail | 删除失败回调 | (err: 错误信息)
renameSuccess | 重命名成功回调 | (newPath: 对象新OSS路径,oldPath: 对象旧OSS路径)
renameFail | 重命名失败回调 | (err: 错误信息)
