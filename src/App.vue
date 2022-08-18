<template>
  <div id="app">
    {{msg}}
    <!--<oss-file-manager-->
      <!--@uploadSuccess="上传成功回调"-->
      <!--@removeSuccess="删除成功回调"-->
      <!--@renameSuccess="重命名成功回调"-->
      <!--access-key-id="OSS STS返回的临时access id"-->
      <!--access-key-secret="OSS STS返回的临时access secret"-->
      <!--sts-token="OSS STS返回的token"-->
      <!--bucket="OSS Bucket"-->
      <!--root-dir="指定根目录"-->
    <!--&gt;-->
    <!--</oss-file-manager>-->
  </div>
</template>

<script>
import OssFileManager from './components/OssFileManager'
export default {
  name: 'app',
  components:{OssFileManager},
  data () {
    return {
      msg: 'OSS文件管理器Demo'
    }
  },
  methods:{
  }
}
</script>

<style lang="scss">
</style>
