<template>
  <div>
    <div style="position: absolute; left: 50%; top: 68px; margin-left: -40px">
      发布任务
    </div>
    <div class="form_div">
      <el-form class="form" :model="ruleForm" :rules="rules" ref="ruleForm" label-width="100px">
        <el-form-item label="任务标题" prop="topic" required>
          <el-input v-model="ruleForm.topic" ></el-input>
        </el-form-item>
        <el-form-item label="任务内容" prop="contents" required>
          <el-input type="textarea" :rows="2" placeholder="输入任务的具体要求" v-model="ruleForm.contents"></el-input>
        </el-form-item>
        <el-form-item label="标注方式" prop="type" required>
          <el-select @change="labelChange" style="position: absolute; left: 0px;" v-model="ruleForm.type" placeholder="选择标注方式">
            <div v-for="item in options" :key="item.value">
              <el-option :label="item.label" :value="item.value"></el-option>
            </div>
          </el-select>
          <el-button @click="labelDetailsShow" class="label_info_button" icon="el-icon-info" type="text"></el-button>
        </el-form-item>
        <el-form-item label="任务时间" required>
          <el-col :span="11">
            <el-form-item prop="dateStart" required>
              <el-date-picker value-format="yyyy-MM-dd" type="date" placeholder="起始日期" v-model="ruleForm.dateStart" style="width: 100%;"></el-date-picker>
            </el-form-item>
          </el-col>
          <el-col class="line" :span="2">-</el-col>
          <el-col :span="11">
            <el-form-item prop="dateEnd" required>
              <el-date-picker value-format="yyyy-MM-dd" type="date" placeholder="截止日期" v-model="ruleForm.dateEnd" style="width: 100%;"></el-date-picker>
            </el-form-item>
          </el-col>
        </el-form-item>
        <el-form-item label="标注人数">
          <el-input-number size="medium" style="position: absolute; left: 0px;" v-model="ruleForm.workerNumber" :min="20" :max="100"></el-input-number>
        </el-form-item>
        <el-form-item label="积 分">
          <el-input-number size="medium" style="position: absolute; left: 0px;" v-model="ruleForm.workerPoints" :min="5" :max="20"></el-input-number>
          <span style="position:absolute; left: 230px;">总积分：{{ruleForm.workerPoints*ruleForm.workerNumber}}</span>
        </el-form-item>
        <el-form-item label="上传数据" required>
          <div class="project_open">
            <div class="upload_warp_left" @click="fileClick">
              <img style="position: absolute; left: 55px; top: 20px; width: 90px;" src="../assets/upload.png">
            </div>
            <div class="upload_wrap" @drop="drop($event)" @dragenter="dragenter($event)" @dragover="dragover($event)">
              <div style="position: absolute; left: 45px; top: 50px;">将文件拖拽到此处</div>
            </div>
            <div class="upload_wrap_text">
              共{{imgList.length}}张图片，{{bytesToSize(this.size)}}
            </div>
            <el-button style="position: absolute; top:130px; right: 5px;" type="text" @click="imgClear()">清除全部图片</el-button>
            <input @change="fileChange($event)" type="file" id="upload_file" multiple style="display: none"/>
          </div>
        </el-form-item>
        <el-form-item style="position: absolute; right: 20px; top: 580px; height:18px">
          <el-button size="medium" @click="handleCancelButton" type="danger" plain>取消</el-button>
          <el-button size="medium"  type="primary" plain @click="submitForm('ruleForm')">提交</el-button>
          <el-button size="medium" type="warning" plain @click="resetForm('ruleForm')">重置</el-button>
        </el-form-item>
        <div style="position: absolute; top: 155px; left: 360px;">
          <el-button type="text"  v-show="isClassBtnShow" @click="handleClassBtn">添加分类</el-button>
          <el-button type="text" v-show="isAttrBtnShow" @click="handleAttrBtn">添加属性</el-button>
        </div>
      </el-form>
      <router-view></router-view>
    </div>
    <div style="position: absolute; top: 57px; left: 0px; width: 20%; height: 680px; border-bottom-left-radius: 1px; border-color: grey" >
      <div style="position: absolute; left: 50%; margin-left: -29px; height: 50px;">
        图片预览
      </div>
      <happy-scroll class="happy_scroll" color="rgba(51,51,51,0.2)" hide-vertical="false">
        <div v-for="(item,index) of imgList" :key="item">
          <div class="upload_warp_img_div_top">
            <div class="upload_warp_img_div_text">
              {{item.file.name}}
            </div>
            <img src="../assets/del.png" class="upload_warp_img_div_del" @click="fileDel(index)">
          </div>
          <img style="width: 100%;" :src="item.file.src">
        </div>
      </happy-scroll>
    </div>
    <el-dialog title="填写所有的分类" :visible.sync="classDialogVisible" width="600px">
      <el-form style="height: 220px; ">
        <div v-for="item in classArray" :key="item" >
          <el-input style="width: 120px; float:left; margin: 5px" v-model="item.value"></el-input>
        </div>
        <el-button @click="addClass" type="text" icon="el-icon-circle-plus-outline" style="float: left; font-size: 24px" size="medium"></el-button>
        <el-button @click="classDialog_sureBtn" type="primary" style="font-size: 12px;position: absolute; right: 40px; bottom: 10px;">确定</el-button>
        <span style="color: red; position: absolute;left: 30px; bottom: 10px">最多可以填写10个分类</span>
      </el-form>
    </el-dialog>
    <el-dialog title="填写所有的属性" :visible.sync="attrDialogVisible" width="600px">
      <el-form style="height: 220px; ">
        <div v-for="item in attrArray" :key="item" >
          <el-input style="width: 120px; float:left; margin: 5px" v-model="item.value"></el-input>
        </div>
        <el-button @click="addAttribute" type="text" icon="el-icon-circle-plus-outline" style="float: left; font-size: 24px" size="medium"></el-button>
        <el-button @click="attrDialog_sureBtn" type="primary" style="font-size: 12px;position: absolute; right: 40px; bottom: 10px;">确定</el-button>
        <span  style="color: red; position: absolute;left: 30px; bottom: 10px">最多可以填写10个属性</span>
      </el-form>
    </el-dialog>
    <div v-show="isLabelDetailsShow" style="position: absolute; right: 10px; top: 70px; width: 300px; height: 340px; color: grey;">
      <span style="font-size: 20px;position:absolute; top: 5px; left: 110px;">标注详情</span>
      <span style="font-size: 16px; position: absolute; top: 40px; left: 12px; color: #4CAF50">整体描述</span>
      <p style="font-size: 14px; position: absolute; top: 50px; left: 12px; text-align: left">针对图像中具体情景及标注需求，为每张图像生成中文描述语句。</p>
      <span style="font-size: 16px; position: absolute; top: 105px; left: 12px;color: #4CAF50">方框标注</span>
      <p style="font-size: 14px; position: absolute; top: 115px; left: 12px; text-align: left">根据标注需求，对图像中的目标物体进行画框</p>
      <span style="font-size: 16px; position: absolute; top: 150px; left: 12px;color: #4CAF50">区域标注</span>
      <p style="font-size: 14px; position: absolute; top: 160px; left: 12px; text-align: left">根据标注需求，使⽤连续的折线标注图像中目标物体的轮廓，返回所有轮廓组成点的坐标。</p>
      <span style="font-size: 16px; position: absolute; top: 215px; left: 12px;color: #4CAF50">图像分类</span>
      <p style="font-size: 14px; position: absolute; top: 225px; left: 12px; text-align: left">根据图像内容，选择给出选项中合适的一项。</p>
      <span style="font-size: 16px; position: absolute; top: 260px; left: 12px;color: #4CAF50">属性标注</span>
      <p style="font-size: 14px; position: absolute; top: 270px; left: 12px; text-align: left">根据图像内容和给定的属性，填写属性值。</p>
    </div>
  </div>
</template>

<script>
var fs = require('fs')
import JSZip from 'jszip'
import JSZipUtils from 'jszip-utils'
export default {
  name: 'NewProject',
  data () {
    var validateTopic = (rule, value, callback) => {
      var len = 0;
      for (var i=0; i<value.length; i++) {
        if (value.charCodeAt(i)>127 || value.charCodeAt(i)==94) {
          len += 2;
        } else {
          len ++;
        }
      }
      if(len>20){
        callback(new Error('项目标题的长度不能超过10个汉字或20个字母'))
      }
      if(len==0){
        callback(new Error('标题不能为空'))
      }
    }
    return {
      rules: {
        topic: [
          {validator: validateTopic, trigger: 'blur'}
        ],
      },
      isLabelDetailsShow: false,
      isClassBtnShow: false,
      isAttrBtnShow: false,
      isWarnShow: false,
      classDialogVisible: false,
      attrDialogVisible: false,
      classNumber: 4,
      classValue: '',
      selectsArray: [],
      classArray: [{
        value: ''
      }],
      attrNumber: 4,
      attrValue: '',
      attrArray: [{
        value: ''
      }],
      imgList: [],
      imgFileList: [],
      size: 0,
      ruleForm: {
        topic: '',
        contents: '',
        type: '',
        dateStart: '',
        dateEnd: '',
        workerNumber: 20,
        workerPoints: 5
      },
      options: [
        {
          value: 'Caption',
          label: '整体标注',
          info: '描述整张图片里的内容'
        },
        {
          value: 'Detection',
          label: '方框描述',
          info: '把图片中指定类型的元素用矩形标注出来'
        },
        {
          value: 'Segmentation',
          label: '区域标注',
          info: '描画图片中指定类型的元素的轮廓'
        },
        {
          value: 'Classification',
          label: '图像分类',
          info: '对图片进行分类'
        },
        {
          value: 'Attribute',
          label: '属性描述',
          info: ' 标注图片中出现元素的属性'
        }
      ],
      value: ''
    }
  },
  created () {
    for (var i = 1; i < this.classNumber; i++) {
      var m = {
        value: this.classValue
      }
      this.classArray.push(m)
    }
    for (var i = 1; i < this.attrNumber; i++) {
      var m = {
        value: this.attrValue
      }
      this.attrArray.push(m)
    }
  },
  methods: {
    labelDetailsShow () {
      this.isLabelDetailsShow = !this.isLabelDetailsShow
      //  var _this = this;
      // setTimeout(function () {
      // _this.isLabelDetailsShow = false;
      // }, 4000);
    },
    handleClassBtn () {
      this.classDialogVisible = true
    },
    handleAttrBtn () {
      this.attrDialogVisible = true
    },
    labelChange () {
      console.log(this.ruleForm.type)
      if (this.ruleForm.type == 'Classification') {
        this.isClassBtnShow = true
        this.isAttrBtnShow = false
        this.classDialogVisible = true
      } else if (this.ruleForm.type == 'Attribute') {
        this.isClassBtnShow = false
        this.isAttrBtnShow = true
        this.attrDialogVisible = true
      } else {
        this.isClassBtnShow = false
        this.isAttrBtnShow = false
        this.classDialogVisible = false
        this.attrDialogVisible = false
      }
    },
    addClass () {
      if (this.classNumber < 10) {
        this.classNumber++
        this.classArray.push({value: ''})
      } else {
        this.isWarnShow = true
      }
    },
    addAttribute () {
      if (this.attrNumber < 10) {
        this.attrNumber++
        this.attrArray.push({value: ''})
      } else {
        this.isWarnShow = true
      }
    },
    attrDialog_sureBtn () {
      this.attrDialogVisible = false
    },
    classDialog_sureBtn () {
      this.classDialogVisible = false
    },

    openSucc (text) {
      this.$notify({
        title: '成功',
        message: text,
        type: 'success',
        duration: 2000,
        position: 'top-left'
      })
    },
    openInfo (text) {
      this.$notify({
        title: '消息',
        message: text,
        type: 'info',
        duration: 2000,
        position: 'top-left'
      })
    },
    openWarn (text) {
      this.$notify({
        title: '警告',
        message: text,
        type: 'warning',
        duration: 2000,
        position: 'top-left'
      })
    },
    LTrim (str) {
      return str.replace(/^\s*/g, '')
    },
    // 去右空格
    RTrim (str) {
      return str.replace(/\s*$/g, '')
    },
    handleCancelButton () {
      var path = '/' + localStorage.getItem('username')
      this.$router.push({path: path + '/requester'})
    },
    submitForm (formName) {
      if (this.ruleForm.type == 'Classification') {
        for (var i = 0; i < this.classArray.length; i++) {
          if (this.LTrim(this.RTrim(this.classArray[i].value)) != '') {
            this.selectsArray.push(this.LTrim(this.RTrim(this.classArray[i].value)))
          }
        }
      } else if (this.ruleForm.type == 'Attribute') {
        for (var i = 0; i < this.attrArray.length; i++) {
          if (this.LTrim(this.RTrim(this.attrArray[i].value)) != '') {
            this.selectsArray.push(this.LTrim(this.RTrim(this.attrArray[i].value)))
          }
        }
      }
      var isValid = true
      this.$refs[formName].validate((valid) => {
        if (valid) {
        } else {
          console.log('error submit!!')
          isValid = false
          return false
        }
      })
      if (this.imgFileList.length == 0) {
        this.openWarn('您尚未上传任何图片。')
        isValid = false
      }
      if (isValid == true) {
        var xmlhttp = new XMLHttpRequest()
        var _this = this
        var username = localStorage.getItem('username')
        var missionID = 0
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            missionID = xmlhttp.responseText
            console.log(missionID)
            _this.uploadImg(missionID)
            _this.openSucc('Submit successfully')
            _this.resetForm()
          }
        }
        let formData = new FormData()
        var mission = {
          requestorID: username,
          missionName: _this.ruleForm.topic,
          description: _this.ruleForm.contents,
          begin: _this.ruleForm.dateStart,
          end: _this.ruleForm.dateEnd,
          type: _this.ruleForm.type,
          points: _this.ruleForm.workerPoints,
          maxNumber: _this.ruleForm.workerNumber,
          selects: []
        }
        if (mission.type == 'Classification' || mission.type == 'Attribute') {
          mission.selects =  _this.selectsArray
        }
        console.log('ms '+ mission)
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/image/addmission', true)
        xmlhttp.setRequestHeader('Content-type', 'application/json; charset=utf-8')
        xmlhttp.send(JSON.stringify(mission))
      }
    },

    uploadImg (missionID) {
      var xmlhttp = new XMLHttpRequest()
      var _this = this
      if (missionID != 0) {
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            console.log(missionID)
          }
        }
        let formData = new FormData()
        var str = '' + missionID
        formData.append('mission', str)
        for (var i = 0; i < this.imgFileList.length; i++) {
          formData.append('file', _this.imgFileList[i])
        }
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/image/uploadImg', true)
        xmlhttp.send(formData)
      }
    },
    resetForm (formName) {
      console.log(this.ruleForm.dateStart)
      this.ruleForm.workerNumber = 20
      this.ruleForm.workerPoints = 5
      this.isClassBtnShow = false
      this.isAttrBtnShow = false
      this.$refs[formName].resetFields()
      this.imgList = []
      this.imgFileList = []
      this.size = 0
      this.openSucc('重置成功')
    },

    imgClear () {
      if (this.imgList.length == 0) {
        this.openInfo('您尚未上传任何图片')
        return
      }
      this.$confirm('确定要清除所有的图片吗？', 'Confirm', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.imgList = []
        this.imgFileList = []
        this.size = 0
        this.openSucc('图片清除成功')
      })
    },

    fileClick () {
      document.getElementById('upload_file').click()
    },

    fileChange (el) {
      if (!el.target.files[0].size) return
      this.fileList(el.target)
      el.target.value = ''
    },
    fileList (fileList) {
      let files = fileList.files
      for (let i = 0; i < files.length; i++) {
        // 判断是否为文件夹
        if (files[i].type != '') {
          this.fileAdd(files[i])
        } else {
          // 文件夹处理
          this.folders(fileList.items[i])
        }
      }
    },
    // 文件夹处理
    folders (files) {
      let _this = this
      // 判断是否为原生file
      if (files.kind) {
        files = files.webkitGetAsEntry()
      }
      files.createReader().readEntries(function (file) {
        for (let i = 0; i < file.length; i++) {
          if (file[i].isFile) {
            _this.foldersAdd(file[i])
          } else {
            _this.folders(file[i])
          }
        }
      })
    },
    foldersAdd (entry) {
      let _this = this
      entry.file(function (file) {
        _this.fileAdd(file)
      })
    },

    fileDel (index) {
      this.$confirm('确认要删除该图片吗？', 'Confirm', {
        confirmButtonText: '确定',
        cancelButtonText: '取消',
        type: 'warning'
      }).then(() => {
        this.size = this.size - this.imgList[index].file.size// 总大小
        this.imgList.splice(index, 1)
        this.imgFileList.splice(index, 1)
        this.openSucc('删除成功')
      })
    },
    fileAdd (file) {
      if (this.limit !== undefined) this.limit--
      if (this.limit !== undefined && this.limit < 0) return
      // 判断是否为图片文件
      var _this = this;
      console.log('file '+file.type)
      if (file.type.indexOf('image')==-1) {
        //如果是压缩包
        //console.log('name '+file)
        if(file.type.indexOf('x-zip-compressed')!=-1){
          var zip = new JSZip()
          zip.loadAsync(file).then(function (file) {
            var files = file.files;
            console.log(files)
            var imgArr = []
            var image = {};
            for(var i in files){
              console.log(i)
              imgArr.push(i)
            }
            for(var j =0;j<imgArr.length;j++){
              var fileName = imgArr[j]
              var buffer = zip.file(imgArr[j]).async("arraybuffer")
              var str = _this._arrayBufferToBase64(buffer)
              var pIndex = fileName.indexOf('.');
              var type = fileName.substr(pIndex + 1);
              var re = 'data:image/' + type + ';base64,';
              image[fileName] = re+str;
            }
            console.log(image)
          })
        }
      } else {
        this.size = this.size + file.size
        this.imgFileList.push(file)
        let reader = new FileReader()
        let image = new Image()
        let _this = this
        reader.readAsDataURL(file)
        reader.onload = function () {
          file.src = this.result
          image.onload = function () {
            let width = image.width
            let height = image.height
            file.width = width
            file.height = height
            _this.imgList.push({
              file
            })
          }
          image.src = file.src
        }
      }
    },
    _arrayBufferToBase64(buffer) {
      var binary = '';
      var bytes = new Uint8Array(buffer);
      var len = bytes.byteLength;
      for (var i = 0; i < len; i++) {
        binary += String.fromCharCode(bytes[i]);
      }
      return window.btoa(binary);
    },
    bytesToSize (bytes) {
      if (bytes === 0) return '0 B'
      let k = 1000, // or 1024
        sizes = ['B', 'KB', 'MB', 'GB', 'TB', 'PB', 'EB', 'ZB', 'YB'],
        i = Math.floor(Math.log(bytes) / Math.log(k))
      return (bytes / Math.pow(k, i)).toPrecision(3) + ' ' + sizes[i]
    },
    dragenter (el) {
      el.stopPropagation()
      el.preventDefault()
    },
    dragover (el) {
      el.stopPropagation()
      el.preventDefault()
    },
    drop (el) {
      el.stopPropagation()
      el.preventDefault()
      this.fileList(el.dataTransfer)
    }
  }
}
</script>

<style scoped>
  .label_info_button {
    color: #9c9c9c;
    position: absolute;
    left: 240px;
    top: 0px;
  }
  .label_info_button:hover{
    color:grey;
  }
  .form_div{
    position: absolute;
    width: 60%;
    left: 50%;
    margin-left: -22%;
    top: 100px;
  }
  .form{
    position: absolute;
    width: 500px;
  }
  .upload_wrap{
    position: absolute;
    top: 0px;
    width: 195px;
    left: 195px;
    height: 130px;
    border: 1px dashed #999;
    border-radius: 4px;
    color: #999;
  }
  .upload_wrap_text {
    position: absolute;
    left: 5px;
    top:130px;
    font-size: 14px;
  }

  .upload_warp_left {
    position: absolute;
    top: 0px;
    width: 195px;
    height: 130px;
    left: 0px;
    border: 1px dashed #999;
    border-radius: 4px;
    border-right: none;
    cursor: pointer;
  }
  .upload_warp_img_div_del {
    width: 18px;
    float: left;
  }

  .happy_scroll{
    position: absolute;
    top: 30px;
    left: 0px;
    width:100%;
  }
  .upload_warp_img_div_top {
    top: 0;
    width: 100%;
    height: 30px;
    background-color: rgba(101, 83, 108, 0.4);
    line-height: 28px;
    text-align: left;
    color: #fff;
    font-size: 12px;
    text-indent: 10px;
  }
  .upload_warp_img_div_text {
    white-space: nowrap;
    width: 92%;
    overflow: hidden;
    text-overflow: ellipsis;
    float: left;
  }
</style>
