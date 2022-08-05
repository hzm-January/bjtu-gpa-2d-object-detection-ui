<template>
  <div class="hello">

    <!-- <h1>{{ msg }}</h1>
      <h2>Essential Links</h2> -->
      <!-- :on-change="imgUpload" -->
      <!-- :file-list="fileList" -->
      <el-upload
      ref="upload"
      action="#"
      list-type="picture-card"
      :limit=limit
      :accept="'image/*'"
      :file-list="fileList"
      :before-upload="onBeforeUpload"
      :http-request="imgUpload"
      :on-preview="handlePictureCardPreview"
      :on-exceed="handleExceed"
      :on-success="handleSuccess"
      :on-progress="handleProgress"
      :auto-upload="true">
      <i slot="default" class="el-icon-plus" ></i>
      <div slot="file" slot-scope="{file}">
        <img
        ref="img"
        class="el-upload-list__item-thumbnail"
        :src="file.url" alt=""
        >
        <!-- 图片中查看，下载，删除 -->
        <span class="el-upload-list__item-actions">
          <span
          class="el-upload-list__item-preview"
          @click="handlePictureCardPreview(file)"
          >
          <i class="el-icon-zoom-in"></i>
        </span>
        <span
        v-if="!disabled"
        class="el-upload-list__item-delete"
        @click="handleDownload(file)"
        >
        <i class="el-icon-download"></i>
      </span>
      <span
      v-if="!disabled"
      class="el-upload-list__item-delete"
      @click="handleRemove(file)"
      >
      <i class="el-icon-delete"></i>
    </span>
  </span>
</div>
</el-upload>
<el-dialog :visible.sync="dialogVisible">
  <img width="100%" :src="dialogImageUrl" alt="">
</el-dialog>
</div>
</template>

<script>
  export default {
    props:{


    },
    data() {
      return {
        id:0,
        limit: 10,
        fileList:[],
        dialogImageUrl: '',
        dialogVisible: false,
        showUploadFlag: true, //控制limit最大值之后 关闭上传按钮
        disabled: false,
      };
    },
    //监听上传图片的数组(为了处理修改时,自动渲染问题,和上传按钮消失问题);
    watch: {
      showUploadFlag(newVal, oldVal) {
        console.log('watch showUploadFlag')
        console.log(newVal+' '+oldVal);
      },
      fileList(newVal, oldVal) {
        console.log('watch fileList has worked...');
        console.log(newVal.length+' '+oldVal.length);
        // if (newName.length == this.limit) this.showUploadFlag = true;
        // else this.showUploadFlag = false;
      },
      // 监听图片上传成功事件
      // handleSuccess(response) {
      //   // 1.拼接得到一个图片信息对象 临时路径
      //   const picInfo = { pic: response.data.tmp_path };
      //   // 2.将图片信息对象，push到pics数组中
      //   this.addForm.pics.push(picInfo);
      // },

      // fileList:{

      //   handler:function(){
      //     console.log('is run');
      //   },
      //   deep: true,
      // },
    },
    methods: {
      onBeforeUpload(file){
        const isIMAGE = file.type === "image/jpg" ||'image/jpeg'||'image/gif'||'image/png';
        const isLt1M = file.size / 1024 / 1024 /2 < 1;
        console.log(isIMAGE);
        console.log(isLt1M);
        if (!isIMAGE) {
          // this.$message.error('上传文件只能是图片格式!');
          this.$notify.warning({
            title: "警告",
            message:
            "请上传格式为image/png, image/gif, image/jpg, image/jpeg的图片",
          });
        }
        if (!isLt1M) {
          // this.$message.error('上传文件大小不能超过 2MB!');
          this.$notify.warning({
            title: "警告",
            message: "图片大小必须小于2M",
          });
        }
        const i = this.fileList.findIndex((x) => x.name === file.name);
        console.log(i);
        if(i!=-1){
          this.$notify.warning({
            title:"警告",
            message:"不能上传同名图片"
          })
        }
        return isIMAGE && isLt1M&&i==-1;
      },
      handleSuccess(res, file){
        console.log('handle success:');
        console.log(file);
      },
      
      handleProgress(event, file, fileList){
        console.log('handle progress:');
        console.log(file);
      },
      handleRemove(file) {
        console.log(file);
        // 1.获取将要删除图片的临时路径
        // const filePath = file.response.data.tmp_path;
        // 2.从pics数组中，找到图片对应的索引值
        const i = this.fileList.findIndex((x) => x.name === file.name);
        // 3.调用splice方法，移除图片信息
        this.fileList.splice(i,1);
      }, 

      base64ImgtoFile (dataurl, filename = 'file') {
        //将base64格式分割：['data:image/png;base64','XXXX']
        const arr = dataurl.split(',')
        // .*？ 表示匹配任意字符到下一个符合条件的字符 刚好匹配到： 
       // image/png
        const mime = arr[0].match(/:(.*?);/)[1]  //image/png
        //[image,png] 获取图片类型后缀
        const suffix = mime.split('/')[1] //png
        const bstr = atob(arr[1])   //atob() 方法用于解码使用 base-64 编码的字符串
        let n = bstr.length
        const u8arr = new Uint8Array(n)
        while (n--) {
          u8arr[n] = bstr.charCodeAt(n)
        }
        return new File([u8arr], `${filename}.${suffix}`, {
          type: mime
        })
      },

      // 将base64转换为blob
      // dataURLtoFile(dataURI,img_name) {
      //   //将base64格式分割：['data:image/png;base64','XXXX']
      //   const arr = dataURI.split(',')
      //   // .*？ 表示匹配任意字符到下一个符合条件的字符 刚好匹配到： 
      //  // image/png
      //   const img_type = arr[0].match(/:(.*?);/)[1]  //image/png
      //   let binary = atob(dataURI.split(',')[1]);
      //   let array = [];
      //   for (let i = 0; i < binary.length; i++) {
      //       array.push(binary.charCodeAt(i));
      //   }
      //   // return new Blob([new Uint8Array(array)], {type: img_type});
      //   return new File([new Uint8Array(array)], img_name, {
      //     type: img_type
      //   })
      // },
      //将base64转换为文件对象
      dataURLtoFile(dataurl, filename) {
        var arr = dataurl.split(',');
        var mime = arr[0].match(/:(.*?);/)[1];
        var bstr = atob(arr[1]);
        var n = bstr.length; 
        var u8arr = new Uint8Array(n);
        while(n--){
            u8arr[n] = bstr.charCodeAt(n);
        }
        //转换成file对象
        return new File([u8arr], filename, {type:mime});
        //转换成成blob对象
        //return new Blob([u8arr],{type:mime});
      },
      handlePictureCardPreview(file) {
        // 请求目标检测结果
        const params = new FormData();
        params.append('id',this.id);
        params.append('name',file.name)
        let config = {
          headers:{'Content-Type':'multipart/form-data'},
          processData:false,
          contentType:false,
          async:false,
          // dataType:'jsonp',
        }; //添加请求头
        this.$axios.post('http://127.0.0.1:5000/annotation', params, config).then((response)=>{
          console.log('打印标注返回结果：');
          console.log(response.data);
          const status = response.data['status'];
          let url = file.url
          if (status==200){// 查询成功
            const image_file = this.dataURLtoFile(response.data['annotation'],response.data['name'])
            url = URL.createObjectURL(image_file);
          } 
          this.dialogImageUrl = url;
        }).catch((response)=>{
          console.log(response.data);
          this.dialogImageUrl = file.url;
        })
        
        this.dialogVisible = true;
      },
      // handleDownload(file) {
      //   console.log(file);
      // },
      // 自动发送图片
      imgUpload(e){
        console.log('upload  images');
        // console.log(e)
        // console.log(e.target)
        // console.log(e.file)
        console.log(' e.target: -- ');
        console.log(e.target)
        // console.log(e.target.files)
        let url = URL.createObjectURL(e.file);
        console.log(url)
        // console.log(e.file.raw)
        // console.log(" print file url: ")
        // console.log(file.url)
        const file = e.file;
        console.log('file name:'+file.name)
        
        // 定义 filereader对象
        let reader = new FileReader()
        reader.readAsDataURL(e.file)
        reader.onload = e=>{
         const img = e.target.result;
         console.log(e.target)
         console.log('ref:')
         // console.log(this.$refs.img)
         // console.log(this.$refs.img.file)
         // console.log(this.$refs.upload)
         // console.log(this.$refs.upload.getFile())
         this.fileList.push({name:file.name,url:url,data:img,id:++this.id});

         const params = new FormData();
         params.append("image", img);
         params.append('id',this.id);
         params.append('name',file.name)


         console.log('打印上传参数');
         console.log(params.get("image"));
          // uploadImg(params).then((res) => {
          // //这里返回的数据结构(根据自己返回结构进行修改)
          //   if (res.data.status === 1) {
          //     this.$message.success("上传成功");
          //     this.imgUrl = res.data;
          //     //调用父组件的方法来传递图片参数
          //     this.$emit("getUrl", this.imgUrl);
          //   } else this.$message.error("上传失败");
          // });
          let config = {
            headers:{'Content-Type':'multipart/form-data'},
            processData:false,
            contentType:false,
            async:false,
            // dataType:'jsonp',
          }; //添加请求头
          this.$axios.post('http://127.0.0.1:5000/image', params, config).then((response)=>{
            console.log(response.data);
            console.log(response)
          }).catch((response)=>{
            console.log(response.data);
          })

           // const image = new Image()
           // image.src=img
           
           // image.onload = _=>{
           //     let width = image.width
           //     let height= image.height
           //     console.log(image.url);
           //     // 然后就可以做需要的操作了
           // }
         }
      //   
      //   //判断用户上传最大数量限制,来让上传按钮消失
      //   if (this.fileList.length >= this.limit) this.showUploadFlag = false;


      //   //图片大小 M
      //   const size = file.size / 1024 / 1024 / 2;
      //   if (
      //     !(
      //       file.type === "image/png" ||
      //       file.type === "image/gif" ||
      //       file.type === "image/jpg" ||
      //       file.type === "image/jpeg"
      //       )
      //     ) {
      //     this.$notify.warning({
      //       title: "警告",
      //       message:
      //       "请上传格式为image/png, image/gif, image/jpg, image/jpeg的图片",
      //     });
      // } else if (size > 2) {
      //   this.$notify.warning({
      //     title: "警告",
      //     message: "图片大小必须小于2M",
      //   });
      // } else {
      //     // if (this.limit == 1) this.imgUrl = []; //此处判断为一张的时候需要清空数组
      

      //   }
    },
      //文件超出个数限制时的函数
      handleExceed(files, fileList) {
        console.log(' over limit')

        this.$message.info(`最多只允许上传${this.limit}张图片`);
        // this.showUploadFlag=false;
      },
      
    }
  }
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>

</style>
