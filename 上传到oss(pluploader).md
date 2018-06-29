## 上传到oss(pluploader)



```
    this.$nextTick(function() {
      const that = this;
      that.plupload = new plupload.Uploader({
        file_data_name: "file",
        browse_button: "addupload",
        multi_selection: true, //允许多个上传
        runtimes: "html5,flash,html4",
        // 记得改路径
        flash_swf_url: "https://cdn.bootcss.com/plupload/2.3.6/Moxie.swf",
        url: "http://oss.aliyuncs.com",
        filters: {
          mime_types: [
            // { title: "压缩文件", extensions: "zip,rar" },
            {
              title: "Image files",
              extensions: "jpg,jpeg,bmp,png,gif",
              mimeTypes: "image/*"
            },
            {
              title: "Video files",
              extensions: "mp4,avi,flv,wmv,mov",
              mimeTypes: "video/*"
            },
          ],
          max_file_size: "1gb", //最大上传
          prevent_duplicates: false //不允许选取重复文件
        },
        init: {
          //  添加文件
          FilesAdded: function(up, files) {
            var arr = [];
            that.idArr = [];
            files.map(function(v, i) {
              var obj = {};
              obj["name"] = v.name;
              arr.push(obj);
            });
            for (var i = 0; i < files.length; i++) {
              var obj = {};
              obj["fileId"] = files[i].id;
              obj["open"] = true;
              obj["barwidth"] = 0;
              obj["status"] = false;
              obj["material-library"] = false
              that.idArr.push(files[i].id);
              that.infor.push(obj);
            }
            that.$esellApi.send(
              "/media/callback/batch/add.shtml",
              arr,
              res => {
                if (res.message == "OK") {
                  for (let i = 0; i < res.payload.length; i++) {
                    that.payloadInfor[that.idArr[i]] = res.payload[i];
                  }
                  that.plupload.start();
                }
              },
              res => {
                that.message("error", res.message)
              }
            );
          },
          // 上传的进度条
          UploadProgress: function(up, file) {
            if (that.inforIndex < 0) {
              for (let i = 0; i < that.infor.length; i++) {
                if (that.infor[i].fileId == file.id) {
                  that.inforIndex = i
                }
              }
            }
            that.infor[that.inforIndex].barwidth = file.percent;
          },
          // 准备开始上传开始到阿里云的方法
          BeforeUpload: function(up, file) {
            that.plupload.setOption({
              url: that.payloadInfor[file.id].host,
              multipart_params: {
                key: "" + that.payloadInfor[file.id].key,
                policy: that.payloadInfor[file.id].policy,
                OSSAccessKeyId: that.payloadInfor[file.id]["access-id"],
                success_action_status: "200", //让服务端返回200,不然，默认会返回204
                callback: that.payloadInfor[file.id].callback,
                signature: that.payloadInfor[file.id].sign
              }
            });
          },
          // 单个上传成功
          FileUploaded: function(up, file, info) {
            console.log(info)
            if (info.status == "200") {
              var info = JSON.parse(info.response);
              that.infor[that.inforIndex].open = false;
              that.infor[that.inforIndex].duration = info.payload["duration"];
              that.infor[that.inforIndex].url = info.payload["coverUrl"];
              that.infor[that.inforIndex].id = info.payload["id"];
              that.infor[that.inforIndex]["name"] = that.getFileName(that.payloadInfor[file.id].name);
              that.infor[that.inforIndex]["status"] = true;
              if (file.type.indexOf("video") >= 0) {
                if (info.payload["coverUrl"]) {
                  that.infor[that.inforIndex]["cover-url"] = info.payload["coverUrl"];
                } else {
                  that.infor[that.inforIndex]["cover-url"] = 'http://file1.yixinfa.cn/dev/180313/fdfa2242540744efa64aed672e838765.jpg'
                }
                that.infor[that.inforIndex]["type"] = "2";
              } else {
                that.infor[that.inforIndex]["type"] = "1";
              }
            }
            that.inforIndex = -1;
          }
        },
        // 全部上传成功
        UploadComplete: function(up, files) {
        },
        Error: function(up, err) {
          if (err.code == -600) {
            that.message("error", "选择的文件太大");
          } else if (err.code == -601) {
            that.message("error", "文件类型不符合要求");
          } else if (err.code == -602) {
            that.message("error", "不允许有重复文件");
          } else {
            that.message("error", "服务器错误，请刷新页面");
          }
        }
      });
      // 实例化（初始）
      that.plupload.init();
```

