<template>
  <el-container>

    <div class="work_place">
      <div class="block">
        <canvas id="myCanvas" v-model="canvas" @click="drawShapes" @dblclick="selectDialog" width="800px" height="500px"></canvas>
      </div>
    </div>
    <div style="position: fixed; top: 0px; left: 0px; width: 18%; height: 680px; z-index:-10000;background-color: #e9e9e9;border: 1px solid rgba(156, 162, 148, 0.75);border-top: none" >
      <div style="position: absolute; left: 20px; height: 70px; top: 72px ">
        图片预览
      </div>

      <happy-scroll class="happy_scroll" color="rgba(51,51,51,0.2)" hide-vertical="false" style="top: 107px">
        <div v-for="(item,index) of imgList" :key="item">
          <div class="upload_warp_img_div_top">
            <div class="upload_warp_img_div_text" style="font-size: 12px;">
              {{item.filename}}
            </div>
          </div>
          <img style="width: 100%; height: auto;" :src="item.url">
        </div>
      </happy-scroll>

    </div>


    <div style="display: none">
      <el-button @click="dialogVisible=true"></el-button>
      <el-dialog :visible.sync="dialogVisible" width="30%"></el-dialog>
    </div>
    <!--classification类型标注弹出框-->
    <el-dialog title="下列描述中最符合图片的是？" :visible.sync="classificationDialogVisible">
    <el-select v-model="classificationValue" >
      <el-option
        v-for="item in classificationOptions"
        :key="item.classificationValue"
        :label="item.label"
        :value="item.classificationValue">
      </el-option>
    </el-select>
      <div>
      <el-table
        :data="classificationInfoList"
        style="width: 100%">
        <el-table-column
          prop="fileName"
          label="图片名"
          width="380">
        </el-table-column>
        <el-table-column
          prop="select"
          label="选择"
          width="180">
        </el-table-column>
      </el-table>
      </div>
      <div slot="footer" class="dialog-footer">
        <el-button  icon="el-icon-close" @click="classificationDialogVisible = false">取消</el-button>
        <el-button type="primary"icon="el-icon-circle-plus-outline" @click="saveClassificationInfo">确定</el-button>
      </div>
    </el-dialog>
    <!--classification类型标注弹出框结束-->
    <!--caption标注弹出框-->
    <el-dialog title="整体标注" :visible.sync="captionDialogVisible">
      <el-form :model="form">
        <el-form-item label="整体标注内容" :label-width="formLabelWidth">
          <el-input v-model="captionDialogInfo" auto-complete="off"></el-input>
        </el-form-item>
      </el-form>
      <div slot="footer" class="dialog-footer">
        <el-button @click="captionDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveCaptionInfo">确 定</el-button>
      </div>
    </el-dialog>
    <!--caption标注弹出框结束-->
    <!--attribute标注弹出框-->
    <el-dialog title="属性标注" :visible.sync="attributeDialogVisible">
      <el-table
        :data="attributeTableData"
        style="width: 100%"
        border="true"
      stripe="true"
      >
        <el-table-column
          label="属性"
          width="250">
          <template slot-scope="scope">
            <span>{{ scope.row.attributeValue }}</span>
          </template>
        </el-table-column>
        <el-table-column
          label="内容"
          width="330">
          <template slot-scope="scope">
            <el-input v-model="scope.row.label" placeholder="请输入内容"></el-input>
          </template>
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button @click="attributeDialogVisible = false">取 消</el-button>
        <el-button type="primary" @click="saveAttributeInfo">确 定</el-button>
      </div>
    </el-dialog>
    <!--attribute弹出框结束-->

    <!--双击图片弹出已提交标注文本开始-->
    <el-dialog title="已提交标注列表" :visible.sync="committedCommentsVisible">
      <el-table
        :data="committedTableData"
        style="width: 100%"
        stripe="true"
        border="true">
        <el-table-column
          label="序号"
          width="120"
          prop="index">
        </el-table-column>
        <el-table-column
          label="类型"
          width="120"
          prop="commentType">
        </el-table-column>
        <el-table-column
          label="标注内容"
          width="320"
          prop="message">
        </el-table-column>
      </el-table>
      <div slot="footer" class="dialog-footer">
        <el-button  icon="el-icon-close" @click="committedCommentsVisible = false">取消</el-button>
      </div>
    </el-dialog>
    <!--双击文本弹出已提交标注文本结束-->
    <div id="toolbar" v-drag style="position: fixed; right: 300px; top: 100px; width: 25px; height: 38px;">
      <div style="position:relative;">
        <img src="../assets/toolbox.png" style="position: absolute; width: 50px; top: -3px; right: -48px">
        <el-tooltip class="item" effect="dark" v-bind:content=tooltip placement="bottom">
          <el-button @click="showToolBar" type="text" v-bind:icon="btnText" size="mini"
                     style="position: absolute; right:0px;width: 20px; height: 36px; font-size: 17px; color: #e90c05;">
          </el-button>
        </el-tooltip>
      </div>
      <div v-show="isShow" style="position:absolute;right: 20px; width: 323px; background-color: #43a0ff; height: 36px; border-radius: 3px">
        <div style="position: relative;" >
          <el-button-group style="padding: 2px; position: absolute; right: 0px;">

            <el-tooltip class="item" effect="dark" content="添加标注信息" placement="bottom">
              <el-button type="primary" icon="el-icon-edit" style="font-size: 14px;" size="mini" @click="selectDialog"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="重新标注当前图片" placement="bottom-end">
              <el-button type="primary" icon="el-icon-ali-xiangpica" style="font-size: 14px;" size="mini" @click="reLabelImg"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="保存图片并提交" placement="bottom">
              <el-button type="primary" icon="el-icon-upload" style="font-size: 14px;" size="mini" @click="commitImg" ></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="加载任务数据" placement="bottom-start">
              <el-button type="primary" icon="el-icon-download" size="mini" style="font-size: 14px;" @click="downloadSource" ></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="上一张" placement="bottom-start">
              <el-button type="primary" icon="el-icon-caret-left" style="font-size: 14px;" size="mini" @click="prevImg"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="下一张" placement="bottom">
              <el-button type="primary" icon="el-icon-caret-right" style="font-size: 14px;" size="mini" @click="nextImg"></el-button>
            </el-tooltip>
            <el-tooltip class="item" effect="dark" content="标注信息" placement="bottom">
              <el-button type="primary" icon="el-icon-info" style="font-size: 14px;" size="mini" @click="selectDialog"></el-button>
            </el-tooltip>
          </el-button-group>
        </div>
      </div>
    </div>
  </el-container>
</template>

<script>
import ElMain from 'element-ui/packages/main/src/main'
import ElAside from 'element-ui/packages/aside/src/main'

export default {
  components: {
    ElAside,
    ElMain
  },
  name: 'Worker',
  data () {
    return {
      imgList: [{
        url: '',
        filename: ''
      }],

      colorList: ['#ff0000', '#FFA500', '#FFEB0D', '#69FF66', '#007FFF', '#BCB4FF'],
      colorPos: 0,
      imgPos: 0,


      captionInfoList:[
        /*{
        filename:"",
        caption:""
      }
      */],//存储当前数据集的全部整体标注内容
      classificationInfoList:[
        {
        fileName:"",
        select:""
      }
      ],//存储当前数据集每张图片的分类结果
      attributeInfoList:[
        /*{
          fileName:"",
          info:{}
        }*/],//存储当前数据集每张图片的属性结果，每个元素都是一个List

      tooltip: '收起',
      btnText: 'el-icon-arrow-right',
      isShow: true,
      toolbar_info: {
        down: false,
        dx: 0,
        dy: 0,
        sx: 0,
        sy: 0
      },
      missionType: '', // 当前任务类型
      missionID: '', // 当前任务ID号
      classificationDialogVisible: false, // classification类型标注时的弹出框
      classificationOptions: [
        /*{ // classification类型标注的选项列表
        classificationValue: '选项1',
        label: '黄金糕'
      }, {
        classificationValue: '选项2',
        label: '双皮奶'
      }
       */],
      classificationValue: "", // classificationOptions的选择结果(号码)
      classificationLabel: "",//classificationOptions的选择结果(文本)
      captionDialogVisible: false, // caption弹出框是否可见
      captionDialogInfo: '', // caption标注的内容
      attributeDialogVisible: false, // attribute弹出框是否可见
      attributeTableData: [
        /*
        {
          attributeValue: '',
          label: ''
        }
        */
      ],
      pointList:[],//存储点集

    }
  },
  created () {
    this.missionType = localStorage.getItem("missionType");
    this.missionID = localStorage.getItem('missionID')
  },
  watch: {
    imgList: {
      handler (oldValue, newValue) {
        this.draw()

      },
      deep: true
    }
  },
  directives: {
    drag (el, bindings) {
      el.onmousedown = function (e) {
        var disx = e.pageX - el.offsetLeft
        var disy = e.pageY - el.offsetTop
        document.onmousemove = function (e) {
          el.style.left = e.pageX - disx + 'px'
          el.style.top = e.pageY - disy + 'px'
        }
        document.onmouseup = function () {
          document.onmousemove = document.onmouseup = null
        }
      }
    }
  },
  methods: {
    // 以下方法用于处理classfication类型标注/caption类型标注/Arribute类型标注，用于实验后期可以删除
    saveClassificationInfo () {
     // console.log(JSON.stringify(classificationInfoList))

      var _this = this;
      for(var i=0;i<_this.classificationInfoList.length;i++){
        if(_this.classificationInfoList[i].fileName == _this.imgList[_this.imgPos].filename){
          _this.classificationInfoList[i].select = _this.classificationValue ;
          console.log(_this.classificationInfoList[i].select)
        }
      }

      this.classificationDialogVisible = false
    },
    saveCaptionInfo (){
      var _this = this;
      for(var i=0;i<_this.captionInfoList.length;i++){
        if(_this.captionInfoList[i].fileName === _this.imgList[_this.imgPos].filename){
          _this.captionInfoList[i].caption = _this.captionDialogInfo;
        }
      }
      //console.log(this.captionDialogInfo);
      this.captionDialogVisible = false;
    },
    saveAttributeInfo () {
      //console.log("close")
      /*
      for (var i = 0; i < this.attributeTableData.length; i++) {
        console.log(this.attributeTableData[i].label)
      }
      */
      this.attributeDialogVisible = false;
    },
    // 选择弹出框类型
    selectDialog () {
      //console.log(this.missionType)
      switch (this.missionType) {
        case 'Caption': {
          this.captionDialogVisible = true
          break
        }
        case 'Classification': {
          this.classificationDialogVisible = true
          break
        }
        case 'Attribute': {
          this.attributeDialogVisible = true
          break
        }
        case 'Detection': {
          var _this = this
          _this.$notify({
            title: '标注信息',
            message: '当前是方框标注',
            type: 'info',
            duration: 2000,
            position: 'top-left'
          })
          break
        }
        case 'Segmentation': {
          var _this = this
          _this.$notify({
            title: '标注信息',
            message: '当前是区域标注',
            type: 'info',
            duration: 2000,
            position: 'top-left'
          })
          break
        }
        default: {}
      }
    },
    showToolBar () {
      console.log(123)
      this.isShow = !this.isShow
      if (this.isShow) {
        this.tooltip = '收起'
        this.btnText = 'el-icon-arrow-left'
      } else {
        this.tooltip = '展开'
        this.btnText = 'el-icon-arrow-right'
      }
    },
    // 重新标注当前图片
    reLabelImg () {
      this.tableData = []

      var _this = this;
      if(this.missionType=="Caption") {
        //重设caption标注
        _this.captionDialogInfo = "";
        for (var i = 0; i < _this.captionInfoList.length; i++) {
          if (_this.captionInfoList[i].fileName === _this.imgList[_this.imgPos].filename) {
            _this.captionInfoList[i].caption = _this.captionDialogInfo;
          }
        }
        _this.commitImg();
      }
      if(this.missionType=="Classification") {
        //重设classification标注
        _this.classificationValue = 0;
        for (var i = 0; i < _this.classificationInfoList.length; i++) {
          if (_this.classificationInfoList[i].fileName === _this.imgList[_this.imgPos].filename) {
            _this.classificationInfoList[i].select = _this.classificationValue;
            //console.log("relabel")
          }
        }
        _this.commitImg()
      }
      if(this.missionType=="Attribute"){
        for(var i=0;i<_this.attributeTableData.length;i++){
          _this.attributeTableData[i].label = "";
        }
        for (var i = 0; i < _this.attributeInfoList.length; i++) {
          if (_this.attributeInfoList[i].fileName === _this.imgList[_this.imgPos].filename) {
            _this.attributeInfoList[i].info = _this.attributeTableData;
            //console.log("relabel")
          }
        }
        _this.commitImg();
      }
      if(this.missionType=="Detection"||this.missionType=="Segmentation") {
        this.colorPos = 0
        // _this.imgDataClear();
        var xmlhttp = new XMLHttpRequest()
        var _this = this
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            var newImg = JSON.parse(xmlhttp.responseText)
            _this.imgList[_this.imgPos].url = newImg.url
            _this.draw()
          }
        }
        let formData3 = new FormData()
        formData3.append('missionid', localStorage.getItem("missionID"))
        formData3.append('imgname', this.imgList[this.imgPos].filename)
        // console.log("12345678");
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/image/checkimg', true)
        xmlhttp.send(formData3)

      }
    },
    // 下载数据集
    downloadSource () {

      var xmlhttp1 = new XMLHttpRequest()
      var xmlhttp = new XMLHttpRequest()
      var xmlhttp0 = new XMLHttpRequest();
      var _this = this
      if(this.missionType=="Segmentation"||this.missionType=="Detection"){
        xmlhttp1.onreadystatechange = function () {
          if (xmlhttp1.readyState == 4 && xmlhttp1.status == 200) {
            //console.log("123")
            // 加入imgList
            _this.imgPos = 0
            _this.imgList = []
            var dataSet = JSON.parse(xmlhttp1.responseText)
            for (var i = 0; i < dataSet.length; i++) {
              //var dataurl = dataSet[i].url
              //var name = dataSet[i].location.split('/')[1]
              var dataurl = dataSet[i].url;
              var name = dataSet[i].fileName;
              _this.imgList.push({url: dataurl, filename: name})
            }
          }
        }
        let formData = new FormData()
        formData.append('missionid', localStorage.getItem("missionID"))
        formData.append('username',localStorage.getItem("username"))
        console.log(localStorage.getItem("missionID"))
        xmlhttp1.open('POST', 'http://106.14.188.84:8080/counts/label/getlocallabel', true)
        xmlhttp1.send(formData)
      }
      else {
        xmlhttp1.onreadystatechange = function () {
          if (xmlhttp1.readyState == 4 && xmlhttp1.status == 200) {
            //console.log("123")
            // 加入imgList
            _this.imgPos = 0
            _this.imgList = []
            var dataSet = JSON.parse(xmlhttp1.responseText)
            for (var i = 0; i < dataSet.length; i++) {
              //var dataurl = dataSet[i].url
              //var name = dataSet[i].location.split('/')[1]
              var dataurl = dataSet[i].base64;
              var name = dataSet[i].fileName;
              _this.imgList.push({url: dataurl, filename: name})
            }
          }
        }
        let formData3 = new FormData()
        formData3.append('missionid', localStorage.getItem("missionID"))
        console.log(localStorage.getItem("missionID"))
        xmlhttp1.open('POST', 'http://106.14.188.84:8080/counts/image/originmission', true)
        xmlhttp1.send(formData3)
      }
      //判断任务类型下载任务数据
      switch (this.missionType) {
        case 'Caption': {
         //下载整体标注结果集
          xmlhttp.onreadystatechange = function () {
            if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
              // 下载所有的整体标注结果
              //console.log("download")
              _this.captionInfoList = JSON.parse(xmlhttp.responseText);
              //console.log(JSON.parse(xmlhttp.responseText));
              //console.log(_this.captionInfoList)
              _this.setCurrentImgCaptionInfo();
            }
          }
          let formDataCap = new FormData()
          formDataCap.append('userid', localStorage.getItem('username'))
          formDataCap.append('missionid', localStorage.getItem("missionID"))
          xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/label/get/captionlabel', true)
          xmlhttp.send(formDataCap);
          break
        }
        case 'Classification': {
         //下载分类标注选项
          xmlhttp.onreadystatechange = function () {
            if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
              console.log(JSON.parse(xmlhttp.responseText).selects)
              // 选项列表放入选项数组
             var selects = JSON.parse(xmlhttp.responseText).selects;
             for(var i=0;i<selects.length;i++){
               _this.classificationOptions.push({
                 classificationValue: i+1,
                 label: selects[i]
               })
             }
             console.log("00"+JSON.stringify(_this.classificationOptions))
            }
          }
          let formDataCla = new FormData()
          formDataCla.append('id', localStorage.getItem("missionID"))
          //console.log(localStorage.getItem("missionID")+"mission")
          xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/mission/findmission/id', true)
          xmlhttp.send(formDataCla);


          //下载分类标注结果
          xmlhttp0.onreadystatechange = function () {
            if (xmlhttp0.readyState == 4 && xmlhttp0.status == 200) {
              // 加入下载结果
              _this.classificationInfoList = JSON.parse(xmlhttp0.responseText);
              console.log("999"+JSON.stringify(_this.classificationInfoList));
              _this.setCurrentImgClassificationValue();
            }
          }
          let formDataClaRes = new FormData()
          formDataClaRes.append('missionid', localStorage.getItem("missionID"))
          formDataClaRes.append('userid',localStorage.getItem("username"));
          xmlhttp0.open('POST', 'http://106.14.188.84:8080/counts/label/get/classificationlabel', true)
          xmlhttp0.send(formDataClaRes);
          break
        }
        case 'Attribute': {
          var selects = [];
          var result = [];
         //下载属性标注选项
          xmlhttp.onreadystatechange = function () {
            if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
              // 选项列表放入选项数组
               selects = JSON.parse(xmlhttp.responseText).selects;
               console.log(selects)
            }
          }
          let formDataAtt = new FormData()
          formDataAtt.append('id', localStorage.getItem("missionID"))
          xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/mission/findmission/id', true)
          xmlhttp.send(formDataAtt);


          //下载属性标注结果并解析
          xmlhttp0.onreadystatechange = function () {
            if (xmlhttp0.readyState == 4 && xmlhttp0.status == 200) {
              // 加入下载结果
               result = JSON.parse(xmlhttp0.responseText);
               console.log(result);
              //把结果加入infoList
              for(var i=0;i<result.length;i++){
                //console.log("00000")
                var filename = result[i].fileName;
                var info = [];
                for(var j=0;j<selects.length;j++){
                  if(result[i].attributes!=null) {
                    info.push({
                      attributeValue: selects[j],
                      label: result[i].attributes[j],
                    })
                  }
                  else{
                    //console.log("123456")
                    info.push({
                      attributeValue: selects[j],
                      label: "",
                    })
                  }
                  //console.log(info);
                }

                _this.attributeInfoList.push({
                  fileName: filename,
                  info: info
                })
              }
              console.log(_this.attributeInfoList);
              _this.setCurrentImgAttributeTable();
            }
          }
          let formDataAttRes = new FormData()
          formDataAttRes.append('missionid', localStorage.getItem("missionID"))
          formDataAttRes.append('userid',localStorage.getItem("username"));
          xmlhttp0.open('POST', 'http://106.14.188.84:8080/counts/label/get/attributelabel', true)
          xmlhttp0.send(formDataAttRes);
          break
        }

       // default: {}
      }
    },
    //以下三个方法用于设置当前图片的标注结果
    setCurrentImgCaptionInfo(){
      this.captionDialogInfo = "";
      console.log(this.captionInfoList)
      //console.log(_this.imgList[_this.imgPos].filename)
      var _this = this;
      for(var i=0;i<_this.captionInfoList.length;i++){
        if(_this.captionInfoList[i].fileName == _this.imgList[_this.imgPos].filename){
          _this.captionDialogInfo = _this.captionInfoList[i].caption
        }
      }
    },
    setCurrentImgClassificationValue(){
      var _this = this;
      for(var i=0;i<_this.classificationInfoList.length;i++){
        if(_this.classificationInfoList[i].fileName === _this.imgList[_this.imgPos].filename){
          _this.classificationValue = _this.classificationInfoList[i].select;
        }
      }


      //console.log(_this.classificationInfoList)

    },
    setCurrentImgAttributeTable(){
      var _this = this;
      for(var i=0;i<_this.attributeInfoList.length;i++){
        if(_this.attributeInfoList[i].fileName == _this.imgList[_this.imgPos].filename){
          _this.attributeTableData = _this.attributeInfoList[i].info
        }
      }
    },
    setCurrentImgInfo(){
      //console.log(localStorage.getItem("missionType"))
      switch(localStorage.getItem("missionType")){
        case "Caption":{
          console.log("uuuuu")
          this.setCurrentImgCaptionInfo();
          break;
        }
        case "Classification":{
          this.setCurrentImgClassificationValue();
          break;
        }
        case "Attribute":{
          this.setCurrentImgAttributeTable();
          break;
        }
      }
    },


    // 提交图片
    commitImg () {
      var xmlhttp = new XMLHttpRequest()
      var xmlhttp1 = new XMLHttpRequest()
      var _this = this
      var userName = localStorage.getItem("username");
      var missionID = localStorage.getItem("missionID")


      if(this.missionType=="Segmentation"||this.missionType=="Detection"){
        // 准备数据
        var c = document.getElementById('myCanvas')
        var dataURL = c.toDataURL()
        var _this = this;
        var filename = _this.imgList[_this.imgPos].filename;
        var data = {
          userName: localStorage.getItem('username'),
          missionID: localStorage.getItem('missionID'),
          fileName: filename,
          url: dataURL,
          dots: _this.pointList,
        }
        // 替换img图片
        this.imgList[this.imgPos].url = dataURL;
        // 发送数据


        xmlhttp1.onreadystatechange = function () {
          if (xmlhttp1.readyState == 4 && xmlhttp1.status == 200) {
            if (JSON.parse(xmlhttp1.responseText).result == true) {
              _this.$notify({
                title: '提交成功',
                message: '图片已上传！',
                type: 'success',
                duration: 2000,
                position: 'top-left'
              })
            } else {
              _this.$notify({
                title: '提交失败',
                message: '图片上传失败，请重试！',
                type: 'warning',
                duration: 2000,
                position: 'top-left'
              })
            }
          }
        }
        xmlhttp1.open('POST', 'http://106.14.188.84:8080/counts/label/savecanvas', true)
        xmlhttp1.setRequestHeader('Content-type', 'application/json; charset=utf-8')
        xmlhttp1.send(JSON.stringify(data))
      }
      if(this.missionType=="Caption"){
        var captionLabel = {
          fileName: _this.imgList[_this.imgPos].filename,
          caption: _this.captionDialogInfo
        }
        var data = {
          userName: userName,
          missionID: missionID,
          captionLabel: captionLabel,
       }
        console.log(data);
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            if (JSON.parse(xmlhttp.responseText).result == true) {
              //console.log("commitsucc")
              _this.$notify({
                title: '提交成功',
                message: '图片已上传！',
                type: 'success',
                duration: 2000,
                position: 'top-left'
              })
            } else {
              _this.$notify({
                title: '提交失败',
                message: '图片上传失败，请重试！',
                type: 'warning',
                duration: 2000,
                position: 'top-left'
              })
            }
          }
        }
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/label/add/captionlabel', true)
        xmlhttp.setRequestHeader('Content-type', 'application/json; charset=utf-8')
        xmlhttp.send(JSON.stringify(data));
      }
      if(this.missionType=="Classification"){

        var classificationLabel = {
          fileName: _this.imgList[_this.imgPos].filename,
          select: _this.classificationValue

        }
        console.log(",,"+JSON.stringify(classificationLabel))
        var data = {
          userName: userName,
          missionID: missionID,
          classificationLabel: classificationLabel,
        }
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            if (JSON.parse(xmlhttp.responseText).result == true) {
              _this.$notify({
                title: '提交成功',
                message: '图片已上传！',
                type: 'success',
                duration: 2000,
                position: 'top-left'
              })
            } else {
              _this.$notify({
                title: '提交失败',
                message: '图片上传失败，请重试！',
                type: 'warning',
                duration: 2000,
                position: 'top-left'
              })
            }
          }
        }
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/label/add/classificationlabel', true)
        xmlhttp.setRequestHeader('Content-type', 'application/json; charset=utf-8')
        xmlhttp.send(JSON.stringify(data))
      }
      if(this.missionType=="Attribute"){
        //获得属性值List
        var attributes = [];
        for(var i=0;i<_this.attributeTableData.length;i++){
          attributes.push(_this.attributeTableData[i].label);
        }
        var attributeLabel = {
          fileName: _this.imgList[_this.imgPos].filename,
          attributes: attributes
        }
        var data = {
          userName: userName,
          missionID: missionID,
          attributeLabel: attributeLabel,
        }
        xmlhttp.onreadystatechange = function () {
          if (xmlhttp.readyState == 4 && xmlhttp.status == 200) {
            if (JSON.parse(xmlhttp.responseText).result == true) {
              _this.$notify({
                title: '提交成功',
                message: '图片已上传！',
                type: 'success',
                duration: 2000,
                position: 'top-left'
              })
            } else {
              _this.$notify({
                title: '提交失败',
                message: '图片上传失败，请重试！',
                type: 'warning',
                duration: 2000,
                position: 'top-left'
              })
            }
          }
        }
        xmlhttp.open('POST', 'http://106.14.188.84:8080/counts/label/add/attributelabel', true)
        xmlhttp.setRequestHeader('Content-type', 'application/json; charset=utf-8')
        xmlhttp.send(JSON.stringify(data))
      }




    },

    // 获取图片location属性
    getLocation () {
      var location = this.missionID + this.imgList[this.imgPos].filename
      return location
    },

    // 以下方法用于绘画
    drawShapes () {
      // 判断标注类型并开始作画
      if (this.missionType == 'Detection') {
        this.drawRec()
      }
      if (this.missionType == 'Segmentation') {
        this.drawBrokenLine()
      } else {}
    },
    // 以下方法用于加载图片到画布上
    draw () {
      var _this = this;
      var cvs = document.getElementById('myCanvas')
      var imgObj = new Image() // 创建image对象
      var img = this.imgList[this.imgPos].url
      imgObj.src = img
      // 待图片加载完后，将其显示在canvas上
      imgObj.onload = function () {
        var ctx = cvs.getContext('2d')
        // 压缩图片
        var rate = 800 / imgObj.width
        cvs.height = imgObj.height * rate
        // ctx.drawImage(this, 0, 0);//this即是imgObj,保持图片的原始大小：470*480
        ctx.drawImage(this, 0, 0, 800, cvs.height)// 改变图片的大小到800*500
        //设置当前图片标注
      }



      _this.setCurrentImgInfo();

    },
    // 以下两个方法分别用于画折线和矩形
    drawBrokenLine () {
      var canvas = document.getElementById('myCanvas')
      var ctx = canvas.getContext('2d')
      var _this = this
      if (this.missionType == 'Segmentation') {
        canvas.onmousedown = function (ev) {
          var x = ev.offsetX
          var y = ev.offsetY
          _this.pointList.push({x:x,y:y});
          console.log(_this.pointList)
          ctx.beginPath()
          ctx.moveTo(x, y)
          canvas.onmousemove = function (ev) {
            var targetX = ev.offsetX
            var targetY = ev.offsetY
            _this.pointList.push({x:targetX,y:targetY});
            console.log(_this.pointList)
            ctx.lineWidth = 1
            ctx.strokeStyle = 'rgba(52, 136, 255, 1)'
            ctx.lineTo(targetX, targetY)
            ctx.stroke()
          }
          canvas.onmouseup = function (ev) {
            canvas.onmousemove = null
            canvas.onmouseup = null
            ctx.lineWidth = 2
            ctx.strokeStyle = 'rgba(52, 136, 255, 1)'
            ctx.lineTo(x, y)
            ctx.stroke()
            ctx.fillStyle = 'rgba(52, 136, 255, 0.5)'
            ctx.fill()
            // ctx.closePath();
          }
        }
      }
    },
    drawRec () {
      //var pointList = new Array()
      var pos = 0
      //var flag = this.checkList.indexOf('矩形画框')
      var canvas = document.getElementById('myCanvas')
      var ctx = canvas.getContext('2d')
      var x2;
      var y2;
      var _this = this;
      if (this.missionType == 'Detection') {
        var lineColor = this.colorList[this.colorPos]
        canvas.onmousedown = function (ev) {
          var x1 = ev.offsetX
          var y1 = ev.offsetY
          console.log(x1)
          ctx.beginPath()
          ctx.moveTo(x1, y1)
          _this.pointList.push({x:x1,y:y1});
          console.log(_this.pointList)
          //pointList[pos] = '(' + x1 + ',' + y1 + '),'
          pos = pos + 1

          canvas.onmousemove = function (ev) {
            x2 = ev.offsetX
            y2 = ev.offsetY
            //pointList[pos] = '(' + x2 + ',' + y2 + '),'
            pos = pos + 1
            ctx.strokeStyle = '#000000'
            ctx.rect(x1, y1, x2 - x1, y2 - y1)
            // ctx.strokeRect(x1,y1,x2-x1,y2-y1);
            // ctx.save();
          }
          canvas.onmouseup = function (ev) {
            ctx.lineWidth = 2
            ctx.strokeStyle = 'rgba(52, 136, 255, 1)'
            ctx.fillStyle = 'rgba(52, 136, 255, 0.5)'
            ctx.fill()
            _this.pointList.push({x:x2,y:y2});
            console.log(_this.pointList);

            ctx.strokeRect(x1, y1, x2 - x1, y2 - y1)
            c.onmousemove = null
            c.onmouseup = null
            ctx.closePath()
          }
        }
      }
    },
    // 以下方法用于清空数据缓存
    imgDataClear () {

      var c = document.getElementById('myCanvas')
      c.click()
      var cxt = c.getContent('2d')
      cxt.clearRect(0, 0, 800, 5000)
    },
    // 以下三个方法用来切换图片
    prevImg () {
      // console.log(this.imgPos);
      if (this.imgPos > 0 && this.imgList.length != 0) {
        this.imgPos--
        this.draw()
      }
      this.imgDataClear()
    },
    nextImg () {
      // console.log(this.imgPos);
      if (this.imgPos < this.imgList.length - 1) {
        this.imgPos++
        this.draw()
      }
      this.imgDataClear()
    },


  }
}
</script>

<style scoped>

  .toolBar {
    position: absolute;
    right: 0px;
    top: 57px;
    width: 15%;
    height: 580px;
    background-color: #e9e9e9;
    border: 1px solid rgba(156, 162, 148, 0.75);
  }

  .work_place {
    position: absolute;
    top: 90px;
    width: 60%;
    left: 20.5%;
  }

  .carousel img {
    width: 90%;
    position: absolute;
    top: 0px;
    left: 5%;
  }

  .labelType {
    position: absolute;
    top: 0px;
    left: 5%;
    width: 90%;
    height: 200px;

  }

  .labelType .labelType_text {
    position: absolute;
    top: 20px;
    left: 10px;
  }

  .labelType .labelType_opera {
    position: absolute;
    top: 50px;
    left: 5px;
    width: 150px;

  }

  #table_btn{
    position: absolute;
    top:200px;
    left: 20px;
  }

  .changeImg{
    position: absolute;
    top: 335px;
    left: 5%;
    height: 90px;
    width: 90%;

  }

  .changeImg .changeImg_text {
    position: absolute;
    top: 20px;
    left: 10px;
  }

  .changeImg .changeImg_opera{
    position: absolute;
    top: 50px;
    left: 10px;
  }

  .operationToImg {
    position: absolute;
    top: 440px;
    left: 5%;
    height: 90px;
    width: 90%;

  }

  .operationToImg .operationToImg_opera{
    position: absolute;
    top: 50px;
    left: 10px;
  }

  .otherOperation {
    position: absolute;
    top: 40%;
    left: 5%;
    width: 90%;
    height: 90px;

  }

  .otherOperation .otherOperation_text {
    position: absolute;
    top: 20px;
    left: 10px;
  }

  .operationToImg .operationToImg_text {
    position: absolute;
    top: 20px;
    left: 10px;
  }

  .otherOperation .otherOperation_opera {
    position: absolute;
    top: 50px;
    left: 10px;
  }

  .happy_scroll{
    position: absolute;
    top: 30px;
    left: 15%;
    width:70%;
    height: 90%;
  }

  .upload_warp_img_div_del {
    position: absolute;
    top: 0px;
    width: 20px;
    left: 0px;
  }

  .upload_warp_img_div_del img {
    width: 20px;
  }

  .cb {
    margin-bottom: 5px;

  }

  #cb1 {
    left: 5px;
  }

</style>
