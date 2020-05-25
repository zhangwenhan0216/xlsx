<template>
  <div>
    <el-upload
      class="upload-demo"
      action
      accept='.xlsx,.xls'
      :on-change='handle'
      :auto-upload='false'
    >
      <el-button
        size="small"
        type="primary"
      >点击上传</el-button>
    </el-upload>
    <input
      id="myfiles"
      multiple
      type="file"
      class="visually-hidden"
    >
    <label for="myfiles">Select some files</label>
    <div id="dropbox"></div>
    <div id="preview"></div>

    //图片缩略图
    <input
      type="file"
      id="fileElem"
      style="display:none"
      @change="handleImage"
    >
    <a
      href="#"
      id="fileSelect"
    >选择上传文件</a>
    <div id="fileList">
      <p>没有选择文件</p>
    </div>
  </div>
</template>

<script>
import xlsx from "xlsx";
export default {
  data() {
    return {
      character: {
        name: {
          text: "姓名",
          type: "string"
        },
        phone: {
          text: "号码",
          type: "string"
        }
      }
    };
  },
  mounted() {
    document.querySelector("#myfiles").onchange = this.pullfiles;

    // 拖放
    let dropbox;
    dropbox = document.getElementById("dropbox");
    dropbox.addEventListener("dragenter", this.dragenter, false);
    dropbox.addEventListener("dragover", this.dragover, false);
    dropbox.addEventListener("drop", this.drop, false);

    //缩略图
    let fileSelect = document.getElementById("fileSelect"),
      fileElem = document.getElementById("fileElem"),
      fileList = document.getElementById("fileList");

    fileSelect.addEventListener('click', (e)=>{
      if(fileElem){
        fileElem.click()
      }
      e.preventDefault()
    }, false)
  },
  methods: {
    handleImage(){
      var fileInput = document.querySelector("#fileElem");
      var files = fileInput.files;
      if(!files.length){
        fileList.innerHTML = '<p>no files selected</p>'
      }else{
        fileList.innerHTML = '';
        let list = document.createElement('ul');
        fileList.appendChild(list);
        for(let i=0;i<files.length;i++){
          let li = document.createElement('li');
          list.appendChild(li)

          let img = document.createElement('img');
          img.src = window.URL.createObjectURL(files[i]);
          img.height = 60;
          img.onload = function(){
            window.URL.revokeObjectURL(this.src);
          }
          li.appendChild(img);
          let info = document.createElement('span');
          info.innerHTML = files[i].name + ':' + files[i].size + 'bytes';
          li.appendChild(info)
        }
      }
    },
    dragenter(e) {
      e.stopPropagation();
      e.preventDefault();
    },
    dragover(e) {
      e.stopPropagation();
      e.preventDefault();
    },
    drop(e) {
      e.stopPropagation();
      e.preventDefault();

      let dt = e.dataTransfer;
      let files = dt.files;

      this.handleFiles(files);
      this.handlePDF(files);
    },
    handleFiles(files) {
      for (let i = 0; i < files.length; i++) {
        let file = files[i];
        console.log(file.type);
        let imageType = /^image\//;
        if (!imageType.test(file.type)) {
          continue;
        }
        let img = document.createElement("img");
        img.classList.add("obj");
        img.file = file;
        document.getElementById("preview").appendChild(img);
        let reader = new FileReader();
        reader.onload = (function(aImg) {
          return function(e) {
            aImg.src = e.target.result;
          };
        })(img);
        reader.readAsDataURL(file);
      }
    },
    sendFiles(){
      let imgs = document.querySelectorAll('.obj');
      for(let i= 0;i<imgs.length;i++){
        new FileUpload(imgs[i], imgs[i].file)
      }
    },
    FileUpload(img, file){
      var reader = new FileReader();  
  this.ctrl = createThrobber(img);
  var xhr = new XMLHttpRequest();
  this.xhr = xhr;
  
  var self = this;
  this.xhr.upload.addEventListener("progress", function(e) {
        if (e.lengthComputable) {
          var percentage = Math.round((e.loaded * 100) / e.total);
          self.ctrl.update(percentage);
        }
      }, false);
  
  xhr.upload.addEventListener("load", function(e){
          self.ctrl.update(100);
          var canvas = self.ctrl.ctx.canvas;
          canvas.parentNode.removeChild(canvas);
      }, false);
  xhr.open("POST", "http://demos.hacks.mozilla.org/paul/demos/resources/webservices/devnull.php");
  xhr.overrideMimeType('text/plain; charset=x-user-defined-binary');
  reader.onload = function(evt) {
    xhr.send(evt.target.result);
  };
  reader.readAsBinaryString(file);
    },
    handlePDF(files) {
      for (let i = 0; i < files.length; i++) {
        let file = files[i];
        console.log(file.type);
        let pdfType = /\/pdf$/;
        if (!pdfType.test(file.type)) {
          continue;
        }
        let node = document.createElement("div");
        node.classList.add("obj");
        node.file = file;
        document.getElementById("preview").appendChild(node);
        let reader = new FileReader();
        reader.onload = (function(aPDF) {
          return function(e) {
            aPDF.src = e.target.result;
          };
        })(node);
        // 开始读取指定的Blob中的内容。一旦完成，result属性中将包含一个data: URL格式的Base64字符串以表示所读取文件的内容。
        reader.readAsDataURL(file);
      }
    },
    pullfiles() {
      var fileInput = document.querySelector("#myfiles");
      var files = fileInput.files;
      // 获取所选文件数量
      var fl = files.length;
      var i = 0;

      while (i < fl) {
        // localize file var in the loop
        var file = files[i];
        console.log(file.name);
        i++;
      }
    },
    async handle(ev) {
      let file = ev.raw;
      if (!file) return;
      let data = await this.readFile(file);
      let workbook = xlsx.read(data, { type: "binary" }),
        worksheet = workbook.Sheets[workbook.SheetNames];
      data = xlsx.utils.sheet_to_json(worksheet);
      let arr = [];
      data.forEach(item => {
        let obj = {};
        for (let key in this.character) {
          if (!this.character.hasOwnProperty(key)) break;
          let value = this.character[key],
            text = value.text,
            type = value.type;
          value = item[text];
          value === "string" ? String(value) : null;
          obj[key] = value;
        }
        arr.push(obj);
      });
      console.log(arr);
    },
    readFile(file) {
      return new Promise(resolve => {
        let reader = new FileReader();
        // 开始读取指定的Blob中的内容。一旦完成，result属性中将包含所读取文件的原始二进制数据。
        reader.readAsBinaryString(file); //将文件转化为二进制
        reader.onload = ev => {
          resolve(ev.target.result);
        };
      });
    }
  }
};
</script>

<style lang="less">
.visually-hidden {
  position: absolute !important;
  height: 1px;
  width: 1px;
  overflow: hidden;
  clip: rect(1px, 1px, 1px, 1px);
}
input.visually-hidden:focus + label {
  outline: thin dotted;
}
input.visually-hidden:focus-within + label {
  outline: thin dotted;
}
#dropbox {
  width: 500px;
  height: 500px;
  background-color: #ccc;
}
</style>