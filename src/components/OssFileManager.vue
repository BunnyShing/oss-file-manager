<template>
  <div v-loading="loading">
    <div style="display: flex;margin:0.5rem 0;">
      <el-button v-if="rootDir !== prefix && prefix.length > rootDir.length" icon="el-icon-back" size="mini" style="padding: 0 0.5rem;" @click="prefix = breadcrumb.slice(0,-1).join('/') + '/'"></el-button>
      <el-button icon="el-icon-refresh" size="mini" style="padding: 0 0.5rem;" @click="list(true)"></el-button>
      <div style="padding: 0.2rem 0rem;margin: 0 0.5rem;">当前目录路径：</div>
      <el-breadcrumb style="font-size: 1rem;width: 65%;padding: 0.2rem 0.5rem;border:1px solid rgb(0 0 0 / 20%);border-radius: 0.2rem;" separator-class="el-icon-arrow-right">
        <el-breadcrumb-item v-for="(item,index) in breadcrumb" :key="index" v-show="index >= rootDirArr.length - 1 || !hiddenRoot">
          <span v-if="index === breadcrumb.length - 1 || index < rootDirArr.length - 1" >{{item}}</span>
          <a v-else @click="prefix = breadcrumb.slice(0,index + 1).join('/') + '/'" >{{item}}</a>
        </el-breadcrumb-item>
      </el-breadcrumb>
      <el-popover title="上传列表" trigger="hover">
        <el-table :data="uploadingList" >
          <el-table-column label="文件名称" width="350">
            <template slot-scope="{row}">
              {{ row.fileName }}
            </template>
          </el-table-column>
          <el-table-column label="上传进度" width="100">
            <template slot-scope="{row}">
              <el-progress type="circle" :percentage="row.progress" width="50"></el-progress>
            </template>
          </el-table-column>
        </el-table>
        <el-button slot="reference" size="mini" type="primary" style="margin: 0 0.5rem;">{{uploadingList.length > 0 ? '正在上传：' + uploadingList.length : '上传列表'}}</el-button>
      </el-popover>
    </div>
    <div style="display: flex;">
      <el-button v-show="allowUploadFolder" size="mini" @click="createDir">创建文件夹</el-button>
      <el-upload ref="uploadFolder" :http-request="handleUpload" action multiple :show-file-list="false">
        <el-button v-show="allowUploadFolder" size="mini" @click="uploadFolder" style="margin: 0 0.5rem;">上传文件夹</el-button>
      </el-upload>
      <el-upload ref="uploadFile" :http-request="handleUpload" action multiple :show-file-list="false">
        <el-button size="mini">上传文件</el-button>
      </el-upload>
    </div>
    <el-table style="min-height: 100vh" :data="objects" id="draggle-area" v-loading="dragCover" element-loading-background="rgba(0, 0, 0, 0.5)" element-loading-text="松开上传" element-loading-spinner="el-icon-upload2">
      <el-table-column label="名称">
        <template slot-scope="{row}">
          <el-link v-if="row.size === ''" @click="prefix = row.name">
            {{ row.name.replace(prefix,'').replace('/','') }}
          </el-link>
          <el-link v-else target="_blank" :href="ossDomain + row.name">{{ row.name.replace(prefix,'') }}</el-link>
        </template>
      </el-table-column>
      <el-table-column label="类型/大小" width="150">
        <template slot-scope="{row}">
          {{ row.size ? parseFloat(row.size / 1024).toFixed(2) + 'KB' : '文件夹'}}
        </template>
      </el-table-column>
      <el-table-column label="最后修改时间" width="180">
        <template slot-scope="{row}">
          {{ row.lastModified ? parseTime(new Date(row.lastModified)) : ''}}
        </template>
      </el-table-column>
      <el-table-column label="操作" align="center" width="100">
        <template slot-scope="{row}">
          <el-dropdown>
            <div>
              <el-button icon="el-icon-edit-outline" size="mini"></el-button>
            </div>
            <el-dropdown-menu slot="dropdown" style="text-align: center">
              <el-dropdown-item>
                <el-button  @click="rename(row.name)" size="mini">重命名</el-button>
              </el-dropdown-item>
              <el-dropdown-item>
                <el-button type="danger"  @click="remove([row.name])" size="mini">删除</el-button>
              </el-dropdown-item>
            </el-dropdown-menu>
          </el-dropdown>
        </template>
      </el-table-column>
    </el-table>
    <div v-show="nextMarker" style="text-align: center;margin: 1rem auto;">
      <el-link><span @click="list">点击加载更多</span></el-link>
    </div>
  </div>
</template>

<script>
  import OSS from 'ali-oss'

  export default {
    name: 'OssFileManager',
    props:{
      // 区域
      region:{
        type: String,
        default: 'oss-cn-shenzhen',
      },
      // STS接口返回的临时accessKeyId
      accessKeyId:{
        type: String,
        default: '',
        required: true
      },
      // STS接口返回的临时accessKeySecret
      accessKeySecret:{
        type: String,
        default: '',
        required: true
      },
      // STS接口返回的securityToken
      stsToken:{
        type: String,
        default: '',
        required: true
      },
      // Bucket
      bucket:{
        type: String,
        default: '',
        required: true
      },
      // 指定根目录
      rootDir:{
        type: String,
        default: '/',
      },
      // 自定义OSS域名
      customOssDomain:{
        type: String,
        default: '',
      },
      // OSS域名是否启用HTTPS
      https:{
        type: Boolean,
        default: true,
      },
      // 是否隐藏根目录
      hiddenRoot: {
        type: Boolean,
        default: true,
      },
      // 是否允许创建/上传文件夹
      allowUploadFolder: {
        type: Boolean,
        default: true,
      },
      // 自定义上传前置函数，返回false或reject将终止上传
      beforeUpload:{
        type: Function,
      },
      // 上传时的请求头，透传给OSS Client 详见：https://help.aliyun.com/document_detail/383952.html
      headers: {
        type: Object,
      },
      // 异步回调配置，透传给OSS Client 详见：https://help.aliyun.com/document_detail/383952.html
      callback: {
        type: Object,
      }
    },
    data() {
      return {
        dragCover: false,
        loading: false,
        uploadingList: [],
        prefix: this.rootDir,
        nextMarker: null,
        objects: []
      }
    },
    computed:{
      ossDomain(){
        let domain = this.customOssDomain !== '' ? this.customOssDomain : (this.bucket + '.' + this.region + '.aliyuncs.com/')
        return domain.startsWith('http') ? domain : ((this.https === true ? 'https://' : 'http://') + domain);
      },
      ossClient(){
        return new OSS({
          region: this.region,
          accessKeyId: this.accessKeyId,
          accessKeySecret: this.accessKeySecret,
          stsToken: this.stsToken,
          bucket: this.bucket
        });
      },
      breadcrumb(){
        return this.prefix.split('/').filter(s => s && s.trim())
      },
      rootDirArr(){
        return this.rootDir.split('/').filter(s => s && s.trim())
      }
    },
    mounted(){
      // 1.文件第一次进入拖动区时，触发 dragging 事件
      // 2.文件在拖动区来回拖拽时，不断触发 dragging 事件
      // 3.文件已经在拖动区，并松开鼠标时，触发 dragFinish 事件
      // 4.文件在拖动区来回拖拽时，不断触发 dragCancel 事件
      let dropBox = document.getElementById('draggle-area');
      dropBox.addEventListener("drop",this.dragFinish,false)
      dropBox.addEventListener("dragleave",this.dragCancel,false)
      dropBox.addEventListener("dragenter",this.dragging,false)
      dropBox.addEventListener("dragover",this.dragging,false)
    },
    watch:{
      accessKeyId(newVal){
        if (newVal) this.list(true)
      },
      rootDir(newVal){
        this.prefix = newVal
      },
      prefix(){
        this.list(true)
      }
    },
    methods:{
      async list(forceRefresh){
        this.loading = true
        let that = this
        await this.ossClient && this.ossClient.list({
          'prefix': this.prefix === '/' || this.prefix === '' ? null : this.prefix,
          'delimiter': '/',
          'marker': forceRefresh === true ? null : this.nextMarker
        })
          .then(result => {
            if (forceRefresh) {
              that.objects = []
            }
            that.nextMarker = result.nextMarker
            if (result.prefixes && result.prefixes.length > 0) {
              result.prefixes.map(v => that.objects.push({ name: v, size: '', lastModified: '' }))
            }
            if (result.objects && result.objects.length > 0) {
              result.objects
                .filter(k => k.size > 0) // 空目录会以objects形式返回，所以这里必须过滤掉对象大小为0的数据，否则会展示空目录对象
                .map(v => that.objects.push(v))
            }
            that.objects.sort((a, b) => {
              let [ta, tb] = [new Date(a.lastModified), new Date(b.lastModified)]
              return ta < tb ? 1 : ta > tb ? -1 : 0
            })
          })
          .catch(err => {
            if(this.$listeners['listObjectsFail']){
              this.$emit('listObjectsFail',err);
            }else{
              this.$message.error(err.toString())
            }
          })
          .finally(() => this.loading = false)
      },
      dragging(e){
        this.dragCover = true
        this.preventDefault(e)
      },
      dragCancel(e){
        this.dragCover = false
        this.preventDefault(e)
      },
      dragFinish(e){
        this.dragCover = false
        this.preventDefault(e)
        let that = this
        let items = e.dataTransfer.items;
        if(items && items.length && items[0].webkitGetAsEntry != null) {
          addFilesItems(items);
        }
        function addFilesItems(items) {
          let ret = [];
          for (let i = 0; i <  items.length; i++) {
            let item = items[i];
            let entry;
            if(item.webkitGetAsEntry && (entry = item.webkitGetAsEntry())) {
              // 上传文件
              if( entry.isFile) {
                // 写入上传文件的逻辑，调用上传接口
                entry.file(fileData => {
                  that.multipartUpload(fileData)
                })
              }else if(entry.isDirectory) {
                // 上传文件夹
                if (!that.allowUploadFolder) {
                  that.$message.error("文件夹[" + entry.name + "]上传失败，不允许上传文件夹")
                } else {
                  ret.push(addFilesFormDirectory(entry, entry.name));
                }
              }
            }
          }
        }

        function addFilesFormDirectory(directory, path){
          directory.createReader().readEntries(entries => {
            entries.forEach(entry => {
              // 判断是否是文件
              if (entry.isFile) {
                entry.file(fileData => {
                  // 用fullPath的值来代替webkitRelativePath
                  fileData.fullPath = path + "/" + fileData.name;
                  that.multipartUpload(fileData)
                })
              }else if (entry.isDirectory) {
                // 如果还是文件夹,则递归处理
                addFilesFormDirectory(entry, path + "/" + entry.name);
              }
            })
          })
        }
      },
      preventDefault(e){
        e.stopPropagation();
        e.preventDefault();
      },
      handleUpload(file){
        file.file.fullPath = file.file.webkitRelativePath || ''
        this.multipartUpload(file.file)
      },
      async multipartUpload(file){
        if(this.beforeUpload && typeof this.beforeUpload === 'function'){
          try {
            let beforeUploadResult = await this.beforeUpload(file)
            if(beforeUploadResult === false) return;
          }catch (err) {
            return;
          }
        }
        let name = this.prefix + (file.fullPath && file.fullPath !== '' ? file.fullPath : file.name);
        this.$message.info(this.hiddenRootDir(name) + ' 已加入上传列表')
        let options = {
          // 获取分片上传进度、断点和返回值。
          progress: (p, cpt) => {
            if (p >= 1) {
              this.$message.success(cpt ? (this.hiddenRootDir(cpt.name) + ' 已上传完成') : '已上传完成')
            }
            if (cpt) {
              let index = this.uploadingList.findIndex(v => v.uploadId === cpt.uploadId)
              if (p >= 1) {
                this.uploadingList.splice(index, 1)
              } else {
                let data = {
                  progress: parseFloat(p * 100).toFixed(),
                  fileName: this.hiddenRootDir(cpt.name),
                  uploadId: cpt.uploadId
                }
                if (index >= 0) {
                  this.uploadingList.splice(index, 1, data)
                } else {
                  this.uploadingList.push(data)
                }
              }
            }
          },
          headers: this.headers,
          callback: this.callback,
          parallel: 10,// 设置并发上传的分片数量。
          partSize: 1024 * 1024 * 5,// 设置分片大小。默认值为1 MB，最小值为100 KB。
          mime: file.type,
        };
        try {
          // 分片上传。
          let res = await this.ossClient.multipartUpload(name, file, options);
          this.$emit('uploadSuccess', res)
          this.list(true)
        } catch (err) {
          if(this.$listeners['uploadFail']){
            this.$emit('uploadFail',err);
          }else{
            this.$message.error(err.toString())
          }
        }
      },
      remove(objectNameArr){
        this.$confirm('此操作将永久删除选中文件/文件夹, 是否继续?', '提示')
          .then(async () => {
            this.loading = true
            let objectsTarget = [];
            for (let item of objectNameArr){
              // 判断对象是否以斜杠结尾，如果是，则代表是一个目录，就需要把该目录开头的OSS对象都查出来一并删除
              if(item.endsWith('/')){
                objectsTarget.push(...await this.getDirObjects(item,null))
              }
              objectsTarget.push(item)
            }
            await this.ossClient.deleteMulti(objectsTarget)
            let deletedObjectArr = [...new Set(objectsTarget)]
            deletedObjectArr.map(path => this.$emit('removeSuccess', path))
            this.list(true)
            this.$message.success('删除成功')
          })
          .catch(err => {
            if(err === 'cancel'){
              this.$message.info('取消删除')
            }else{
              if(this.$listeners['removeFail']){
                this.$emit('removeFail',err);
              }else{
                this.$message.error(err.toString())
              }
            }
          })
          .finally(() => this.loading = false)
      },
      rename(objectName){
        this.$prompt('如对大文件夹重命名是耗时操作，确认后请耐心等候', '重命名', {
          inputValue: objectName.split('/').filter(s => s && s.trim()).slice(-1)[0],
          inputPattern: /^[^/:*?"<>|]+$/,
          inputErrorMessage: '非法文件/文件夹名称'
        }).then(async ({ value }) => {
          this.loading = true
          let copyTarget = [ objectName ]
          if(objectName.endsWith('/')){
            copyTarget = await this.getDirObjects(objectName,null)
          }
          for(let oldPath of copyTarget){
            let objectNameArr = objectName.split('/').filter(s => s && s.trim());
            objectNameArr[objectNameArr.length - 1] = value
            let newPath = '';
            if(objectName.endsWith('/')){
              newPath = oldPath.replace(objectName,objectNameArr.join('/') + '/')
            }else{
              newPath = objectNameArr.join('/')
            }
            let copyResult = await this.ossClient.copy(newPath, oldPath)
            if(copyResult.res.status === 200){
              await this.ossClient.delete(oldPath)
              this.$emit('renameSuccess',newPath,oldPath);
            }
          }
          this.list(true)
        }).catch(err => {
          if(err === 'cancel'){
            this.$message.info('取消重命名')
          }else{
            if(this.$listeners['renameFail']){
              this.$emit('renameFail',err);
            }else{
              this.$message.error(err.toString())
            }
          }
        }).finally(() => this.loading = false)
      },
      // 递归获取指定目录下的全部对象
      async getDirObjects(objectName,nextMarker){
        let objects = []
        let res = await this.ossClient.list({ 'prefix': objectName, 'max-keys': 1000, 'marker': nextMarker })
        objects.push(...res.objects.map(v => {return v.name}))
        if(res.nextMarker){
          objects.push(...await this.getDirObjects(objectName,res.nextMarker))
        }
        return objects;
      },
      createDir(){
        this.$prompt('请输入文件夹名称', '新建文件夹', {
          inputPattern: /^[^/:*?"<>|]+$/,
          inputErrorMessage: '非法文件夹名称'
        }).then(async ({ value }) => {
          try {
            await this.ossClient.put(this.prefix + value + '/', Buffer.from(''));
            this.$message.success("创建文件夹成功");
            this.list(true)
          } catch (err) {
            this.$message.error(err.toString())
          }
        }).catch(err => {
          if(err === 'cancel'){
            this.$message.info('取消创建文件夹')
          }else{
            this.$message.error(err.toString())
          }
        });
      },
      hiddenRootDir(path){
        if(this.hiddenRoot){
          return path.replace(this.rootDirArr.slice(-2) + '/','')
        }else{
          return path
        }
      },
      uploadFolder(){
        this.$nextTick(() => this.$refs.uploadFolder.$children[0].$refs.input.webkitdirectory = true)
      },
      /**
       * Parse the time to string
       * @param {(Object|string|number)} time
       * @param {string} format
       * @returns {string | null}
       */
      parseTime(time, format = '{y}-{m}-{d} {h}:{i}:{s}') {
        if (arguments.length === 0 || !time) return null
        let date = time
        if (typeof time !== 'object') {
          if ((typeof time === 'string')) {
            if (/^[0-9]+$/.test(time)) {
              time = parseInt(time) // support "1548221490638"
            } else {
              time = time.replace(new RegExp(/-/gm), '/') // support safari[https://stackoverflow.com/questions/4310953/invalid-date-in-safari]
            }
          }
          if ((typeof time === 'number') && (time.toString().length === 10)) time = time * 1000
          date = new Date(time)
        }
        const formatObj = {
          y: date.getFullYear(),
          m: date.getMonth() + 1,
          d: date.getDate(),
          h: date.getHours(),
          i: date.getMinutes(),
          s: date.getSeconds(),
          a: date.getDay()
        }
        return format.replace(/{([ymdhisa])+}/g, (result, key) => {
          const value = formatObj[key]
          // Note: getDay() returns 0 on Sunday
          return (key === 'a') ? ['日', '一', '二', '三', '四', '五', '六'][value] : value.toString().padStart(2, '0')
        })
      }
    }
  }
</script>
