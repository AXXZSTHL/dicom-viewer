<template>
  <div class="main">
    <!--    <div class="top-title">-->
    <!--      <div class="title-logo">-->
    <!--        <img src="../assets/logo.png">-->
    <!--      </div>-->
    <!--      <div class="title-name">Dicom Viewer Pro By Vf2e</div>-->
    <!--      <div class="patient-list">Patient List</div>-->
    <!--    </div>-->

    <div class="top-tool-bar">
      <div @click="hflip">
        <i class="el-icon-refresh-left"></i>
        <h4>左翻转</h4>
      </div>
      <div @click="vflip">
        <i class="el-icon-refresh-right"></i>
        <h4>右翻转</h4>
      </div>
      <div @click="lRotate">
        <i class="el-icon-sort-up"></i>
        <h4>左旋转</h4>
      </div>
      <div @click="rRotate">
        <i class="el-icon-sort-down"></i>
        <h4>右旋转</h4>
      </div>
      <div @click="reset">
        <i class="el-icon-refresh"></i>
        <h4>重置</h4>
      </div>
      <!--      <div @click="show(0)">-->
      <!--        <i class="el-icon-refresh"></i>-->
      <!--        <h4>Grow</h4>-->
      <!--      </div>-->
      <!--      <div @click="renderModel">-->
      <!--        <img src="../assets/1.png">-->
      <!--        <h4>3D</h4>-->
      <!--      </div>-->
      <div @click="getLength">
        <i class="el-icon-c-scale-to-original"></i>
        <h4>测距</h4>
      </div>
      <div @click="probe">
        <i class="el-icon-paperclip"></i>
        <h4>探针</h4>
      </div>
      <div @click="arrowAnnotation">
        <i class="el-icon-bottom-left"></i>
        <h4>箭头注释</h4>
      </div>
      <div @click="angle">
        <i class="el-icon-rank"></i>
        <h4>角度工具</h4>
      </div>
      <div @click="freeHand">
        <i class="el-icon-mouse"></i>
        <h4>鼠标工具</h4>
      </div>
      <div @click="ellipticalRoi">
        <i class="el-icon-help"></i>
        <h4>椭圆工具</h4>
      </div>
      <div @click="rectangleRoi">
        <i class="el-icon-full-screen"></i>
        <h4>矩形工具</h4>
      </div>
      <div @click="playVideo">
        <i :class="timer?'el-icon-video-pause':'el-icon-video-play'"></i>
        <h4>播放</h4>
      </div>

      <!-- <div class="block slider-bar">
        <span class="demonstration"></span>
        <el-slider v-model="value1" :format-tooltip="formatTooltip"></el-slider>
      </div>-->
    </div>

    <!-- <div
      class="image-canvas-wrapper"
      oncontextmenu="return false"
      unselectable="on"
      onselectstart="return false;"
      onmousedown="return false;"
    >-->
    <!--     <span id="loadProgress">Diocm加载:</span>-->

    <!-- <div ref="canvas" class="image-canvas" oncontextmenu="return false"></div> -->

    <!-- @cornerstoneimagerendered="onImageRendered" -->
    <div style="position: relative;">
      <div ref="canvas1" class="image-canvas" oncontextmenu="return false"></div>
      <!--      @click="showAxes"-->
      <div
        id="topleft"
        class="overlay"
        style="position:absolute;top:3vh;left: 21vw;color: white;"
      >姓名：{{ patientName }}
      </div>
      <div
        id="topright"
        class="overlay"
        style="position:absolute;top:3vh;right: 21vw;color: white;"
      >渲染时间：{{ renderTime }}
      </div>
      <div
        id="bottomright"
        class="overlay"
        style="position:absolute;bottom: 3vh;left: 21vw;color: white;"
      >缩放：{{ zoom }}
      </div>
      <div
        id="bottomleft"
        class="overlay"
        style="position:absolute;bottom: 3vh;right: 21vw;color: white;"
      >WW/WC:{{ ratio }}
      </div>
    </div>
    <!--    <div ref="canvas2" class="mask-canvas" oncontextmenu="return false"></div>-->

    <!-- <div
        ref="canvas2"
        class="image-canvas-right"
        oncontextmenu="return false"
        @cornerstoneimagerendered="onImageRendered"
    ></div>-->

    <!-- </div> -->

    <br>
    <div class="block slide-bar">
      <span class="demonstration"></span>
      <el-slider :min="0" :max="dcmlist.length" v-model="value1" :format-tooltip="formatTooltip"></el-slider>
    </div>
  </div>
</template>

<script>
//引入 cornerstone,dicomParser,cornerstoneWADOImageLoader
import * as cornerstone from "cornerstone-core";
// import * as dicomParser from "dicom-parser";
import JsZip from "jszip"

// 不建议 npm 安装 cornerstoneWADOImageLoader 如果你做了 会很头疼
import * as cornerstoneWADOImageLoader from "../../static/dist/cornerstoneWADOImageLoader.js";

// Cornerstone 工具外部依赖
import Hammer from "hammerjs";
import * as cornerstoneMath from "cornerstone-math";
import * as cornerstoneTools from "cornerstone-tools";

// Specify external dependencies
cornerstoneTools.external.Hammer = Hammer;
cornerstoneTools.external.cornerstone = cornerstone;
cornerstoneTools.external.cornerstoneMath = cornerstoneMath;
cornerstoneWADOImageLoader.external.cornerstoneMath = cornerstoneMath;

//指定要注册加载程序的基石实例
cornerstoneWADOImageLoader.external.cornerstone = cornerstone;

cornerstone.registerImageLoader("http", cornerstoneWADOImageLoader.loadImage);
cornerstone.registerImageLoader("https", cornerstoneWADOImageLoader.loadImage);

//配置 webWorker (必须配置)
//注意这里的路径问题  如果路径不对 cornerstoneWADOImageLoaderWebWorker 会报错 index.html Uncaught SyntaxError: Unexpected token <
var config = {
  webWorkerPath: "/static/dist/cornerstoneWADOImageLoaderWebWorker.js",
  taskConfiguration: {
    decodeTask: {
      codecsPath: "/static/dist/cornerstoneWADOImageLoaderCodecs.js"
    }
  }
};
cornerstoneWADOImageLoader.webWorkerManager.initialize(config);

export default {
  name: "Index",
  data() {
    return {
      baseUrl: "",
      exampleStudyImageIds: "",
      isInitLoad: true,
      isShow: true,
      patientName: '张三',
      renderTime: '',
      zoom: '',
      ratio: '',
      value1: 1,
      imageData: [],
      imageDataIndex: 0,
      val: '',
      num: '',
      result: [],
      currentGrayLevel: -1,
      low: 100,
      high: 100,
      currentLine: 0,
      basePath: 'wadouri:\\static\\dcm\\',
      dcmlist: [],
      timer: null,
      dcmNum: 0
    };
  },
  watch: {
    val(val) {
      this.num = this.val
      let _this = this
      this.show(this.num)

    }
  },
  mounted() {
    let value = `0F9AF173B57C43BCBB8145FB947EB4F2.dcm.pk
1ABDA6F967B84BA4ACC11EED8FF363E5.dcm.pk
1C37A74DE4E7481E82D1B77859970772.dcm.pk
2A722A5F1DC749B18176F14C228E0E7F.dcm.pk
02C18D67896E48AA9F34AC23D6D04B25.dcm.pk
2F16988996984ED18C033B3259437007.dcm.pk
3E9A3712FE4E418FA2D6A09E7E8BB5FD.dcm.pk
4F4C4534DFCB473C8403CB37BF18E35B.dcm.pk
5C462E9CF0DE42DF86A71D9E3E8DDC34.dcm.pk
5EC431393C544041A3BC77C2D36DD8A5.dcm.pk
5F25B1D9C43547398590696C9BD9788D.dcm.pk
6A97D82B1ACF4279B23CF7CBDE09D4C7.dcm.pk
6B0C375277F34307BBC93123EB4FAB64.dcm.pk
6B44F8D2170D487799985A203FB22B02.dcm.pk
6B94FDE20A614C48B52C2EBBF71A6C7D.dcm.pk
6C31860A6E864D04886DA2A1A85FE50F.dcm.pk
6D2C014A3C9A426F9B4ACFB377864E39.dcm.pk
6EB24CB86208457B9D6CE995C8BC425A.dcm.pk
7AF62D0331B4423E9169622D5A49E395.dcm.pk
7B221E69BD3249B39ACEE169E0FFB239.dcm.pk
07C7472FF14B4DE3B0214B29BB079B46.dcm.pk
7DFC3DCFEADD4F95B9BFCDC769E4CA09.dcm.pk
7FD43A8EEDB448AE8F93987B78126EEF.dcm.pk
8C18AF2EFAEC4C0489B981B5D8559795.dcm.pk
9A45FE4FC8AF450697B86621AAC39DC6.dcm.pk
9A6642C9FB2B4C5A90914ECBB0AD396C.dcm.pk
9F5A98F77DF54408922DA479F2BAFC12.dcm.pk
14D7895AE9D846EF9FD02384DD6CAAF7.dcm.pk
16F6840A73284A73AB113F7462BEF095.dcm.pk
18C3B1D46BA249EEB1F841A2608FE3C0.dcm.pk
23CD4F6A673A4159A410562C5A782CFB.dcm.pk
32A2D38EAAA548798FF40357D87FD44A.dcm.pk
34BDE4A2165441DDA9175F85A604B677.dcm.pk
41CBBDE68217453EA582A2C1A1DB4EC7.dcm.pk
43C8BCFA98BC41D88B530FC687CE5181.dcm.pk
047E3CDC48D841A084DB9AFBF1023C47.dcm.pk
53D31567EC674DC7A271F1770CA772D1.dcm.pk
57A64F4F995E49EB8F7821E07FDF27C5.dcm.pk
68B50B8060C04B9C8E8572C9F9F3EE41.dcm.pk
77DCF8C376B34ED285817AF404AC3D62.dcm.pk
80FB1CB4DB9843E48603C4146AEC6232.dcm.pk
082B1D8AB770414BBED1C461974DCF35.dcm.pk
151E2CFDC59446A6B8C046BE26EB5F0E.dcm.pk
369E104272EC4479A1F61F4A1F1D4C29.dcm.pk
491A4DCDBEB04ED6906CA96B555FFE90.dcm.pk
515F1D71BB3E44379216A3844B17D8BC.dcm.pk
684FB43F3B9F4D8DBDBEAB045786D57E.dcm.pk
693A630980EA4C9A94553B4852D8E3E7.dcm.pk
885D5D5B09474171BB00B36904BEFA62.dcm.pk
979F5565CBA44270B84CA282B860AF42.dcm.pk
4110CA3892224534A9FF6B85FF93A82D.dcm.pk
9228BFE3BEE5480186DCE4986080AE16.dcm.pk
43523E01F09F4C8F8DC71B1FC1C335A9.dcm.pk
66071DEDA48F48C4863294E99AA12177.dcm.pk
90660F9568CB40B9B62007D86686BCA6.dcm.pk
180308E4EAB447B0A25025C3306A42D9.dcm.pk
485446D8FF144163BB825B269FCA345A.dcm.pk
598337CE399B45208F8734C5130EA937.dcm.pk
835844DF0A7C402AB0D9FD7E04F756E8.dcm.pk
972387A21AE34C9FB6BC085BC4AB9837.dcm.pk
1921374BFD30444F9B85847730B2904B.dcm.pk
2764374EC80C4FBD936E19BA2C3D29E1.dcm.pk
4194618CA7D54119960BC3D981759BC6.dcm.pk
5585851A49C84E6593717894857F84F2.dcm.pk
23747734DCE64D86BB477B38702B0EBC.dcm.pk
27061945D50D431A8EB3455172DF8048.dcm.pk
61417859633E4BC2AAB72A97F75303B1.dcm.pk
A2F9EE92142A442A9C9D109F0407B0FD.dcm.pk
A7AA0B712196463E9B63CF03C9509FA0.dcm.pk
A8CF667513C94DDB8D8955E1B160E00B.dcm.pk
A9F926CCA5F44F37BBC7E1F80636AB8B.dcm.pk
A9961D3001DF4312AE29015EF775D937.dcm.pk
AC0BEFA20F6848AC902F71B4C1D0270C.dcm.pk
B99BC6371D4B49D3B9B77DEAB6643E19.dcm.pk
B275CB8927BC41F382090B42159418BC.dcm.pk
B2861A052D214B9AB7A7B5B689ED0391.dcm.pk
BDB75AE3A1164419B421F1B13810439A.dcm.pk
C0F76E10F4334A6DAFBD99CA2BCE997C.dcm.pk
C8CC35D951154B65BF28350FA4BF3C29.dcm.pk
C71FC54BD832471E9149BF8F7BA0EC25.dcm.pk
C509D92CD96F4BB5A2E18437D488449E.dcm.pk
C8013F9967614EAD9FBBE9A9B35538FF.dcm.pk
CA69B25C13304FD4A48BBA925AC61F21.dcm.pk
CACD7869CF4F4792BA0EEBA6285BEC67.dcm.pk
D3E3379685E64519A8C218875140D6A9.dcm.pk
D255326BE3964969A5BD2BFB30DFC15C.dcm.pk
D3890965C7F24E66B07A67A0956025D7.dcm.pk
DEC682053A694DAFADEE69070F28AB72.dcm.pk
E2D658F1CE1C4ED699525341DFA75694.dcm.pk
E4ED6603A44949D28B71878EA3ECD9A7.dcm.pk
E7DEEFD6F9EA47AF904D80385E50FF00.dcm.pk
E8C8CBE14B6D4DF8B8F1BB539665CACA.dcm.pk
E24F2319E47B4C03A93B409D7E301951.dcm.pk
E50C8097998D4048BF4053D742E5FEF1.dcm.pk
E62DA31C0FE640699B842EAD22EE5E00.dcm.pk
E552F92D31204C2D900C1B9572D8FFE7.dcm.pk
E0129543914848008A872534070B502C.dcm.pk
EC40EBF4AAF44C1CA7E1FD2950A4EAA0.dcm.pk
ED89A6E61D4D42DB8C7A5D7A5CD0215D.dcm.pk
EEC7197DDBC446FE88D43FED72C7674E.dcm.pk
EF4DE6B1875F41CCB45982DB0BB33BA1.dcm.pk
EFA14B943CEF4D048990F0FC7EA5DC17.dcm.pk
F2FEF51B82FC4E8C9254942686B770F6.dcm.pk
F5DC261D9E8545AD9AD9DBDF1F11E9C0.dcm.pk
F17A8171795C4B938387259474869678.dcm.pk
F51FF771893048D0B17B6E7D06C81230.dcm.pk
F71D60304D514B8DADEA0EC147970C0F.dcm.pk
FBA4342F0B334F9792831998324B4B72.dcm.pk
FDA6CCDBF7EE4C8DB1B4DF253CFC25FD.dcm.pk
FE8380410C7A42DAAC0349B3D6F0BC48.dcm.pk
1_0.dcm
2_1.dcm
3_2.dcm
4_3.dcm
5_4.dcm
6_5.dcm
7_6.dcm
8_7.dcm
9_8.dcm
10_9.dcm
1709081.2.840.113619.2.55.3.554365571.894.1637394514.454.16.dic`
    let dcmTList = value.split(/[(\r\n)\r\n]+/);
    dcmTList.forEach((item) => {
      this.dcmlist.push({name: item, data: null})
    })

    this.val = 0
    window.onresize = () => {
      this.show(this.val);
    }


    this.$refs.canvas1.addEventListener('cornerstoneimagerendered', this.onImageRendered);
    //
    // cornerstone.events.addEventListener('cornerstoneimageloaded', function(e) {
    //     const eventData = e.detail;
    //     document.getElementById('imageLoaded').textContent = `Last ImageId Loaded: ${eventData.image.imageId}`;
    // });

  },
  methods: {
    playVideo() {
      if (this.timer) {
        clearInterval(this.timer)
        this.timer = null
      } else {
        this.val = this.value1
        this.timer = setInterval(() => {
          // this.show(this.val)
          this.val += 1;
          this.value1 = this.val
          if (this.val > this.dcmlist.length) this.val = 0
        }, 50);
      }
    },

    formatTooltip(val) {
      this.val = val
      return val
    },

    onImageRendered(e) {
      const eventData = e.detail;
      cornerstone.setToPixelCoordinateSystem(eventData.enabledElement, eventData.canvasContext);

      this.renderTime = eventData.renderTimeInMs.toFixed(4) + " ms";
      this.zoom = eventData.viewport.scale.toFixed(2);
      this.ratio = Math.round(eventData.viewport.voi.windowWidth) + "/" + Math.round(eventData.viewport.voi.windowCenter);

      // e.detail.viewport.voi.windowWidth = 200;
      //  e.detail.viewport.voi.windowCenter = -90;
      // console.log(this.$refs.canvas1.offsetWidth, this.$refs.canvas1.offsetHeight)
    },

    showAxes(e) {
      this.result = []
      this.currentGrayLevel = -1
      this.currentLine = 0
      console.log("start: " + this.result.length)
      this.showAxesTemp(e)
      console.log("end : " + this.result.length)

      this.showMask(this.random())
    },

    random() {
      var b = 0;
      var a = Math.floor(Math.random() * 80);
      if (a == b) {
        return this.random();
      } else {
        b = a;
        return b;
      }
    },

    showAxesTemp(e) {
      let canvas = this.$refs.canvas1
      let x = e.clientX - canvas.getBoundingClientRect().left
      let y = e.clientY - canvas.getBoundingClientRect().top
      let index = x + y * 480
      this.currentGrayLevel = this.imageData[index]
      // console.log("currentGrayLevel : " + this.currentGrayLevel)
      this.result.push(index)

      //计算上边点
      this.showAxesTop(e, x, y - 1)
      //计算下边点
      this.showAxesBottom(e, x, y + 1)
      //计算左边点
      this.showAxesLeft(e, x - 1, y)
      //计算右边点
      this.showAxesRight(e, x + 1, y)
    },

    showAxesLeft(e, x, y) {
      if (x >= 0) {
        let index = x + y * 480
        let currentGrayLevelTemp = this.imageData[index]
        if (currentGrayLevelTemp >= this.currentGrayLevel - this.low && currentGrayLevelTemp <= this.currentGrayLevel + this.high) {
          this.result.push(index)
          this.showAxesTop(e, x, y - 1)
          this.showAxesBottom(e, x, y + 1)
          this.showAxesLeft(e, x - 1, y)
        } else {
          console.log("tempGrayLevel : " + currentGrayLevelTemp)
        }
      }
    },

    showAxesRight(e, x, y) {
      if (x < 480) {
        let index = x + y * 480
        let currentGrayLevelTemp = this.imageData[index]
        if (currentGrayLevelTemp >= this.currentGrayLevel - this.low && currentGrayLevelTemp <= this.currentGrayLevel + this.high) {
          this.result.push(index)
          this.showAxesTop(e, x, y - 1)
          this.showAxesBottom(e, x, y + 1)
          this.showAxesRight(e, x + 1, y)
        }
      }
    },

    showAxesTop(e, x, y) {
      if (y >= 0) {
        let index = x + y * 480
        let currentGrayLevelTemp = this.imageData[index]
        if (currentGrayLevelTemp >= this.currentGrayLevel - this.low && currentGrayLevelTemp <= this.currentGrayLevel + this.high) {
          this.result.push(index)
          this.showAxesTop(e, x, y - 1)
        }
      }
    },

    showAxesBottom(e, x, y) {
      if (y < 480) {
        let index = x + y * 480
        let currentGrayLevelTemp = this.imageData[index]
        if (currentGrayLevelTemp >= this.currentGrayLevel - this.low && currentGrayLevelTemp <= this.currentGrayLevel + this.high) {
          this.result.push(index)
          this.showAxesBottom(e, x, y + 1)
        }
      }
    },

    renderCanvas(e) {
      let canvas = this.$refs.canvas2
      var ctx = canvas.getContext("2d");
      console.log(ctx)
      var imgData = ctx.createImageData(100, 100);
      for (var i = 0; i < imgData.data.length; i += 4) {
        imgData.data[i + 0] = 255;
        imgData.data[i + 1] = 0;
        imgData.data[i + 2] = 0;
        imgData.data[i + 3] = 255;
      }
      ctx.putImageData(imgData, 10, 10);

      // let context = canvas.getContext('2d')
      let canvasX = e.clientX - canvas.getBoundingClientRect().left
      let canvasY = e.clientY - canvas.getBoundingClientRect().top
      console.log('X:' + (e.clientX - canvas.getBoundingClientRect().left) + ',Y:' + (e.clientY - canvas.getBoundingClientRect().top))
    },
    // getImageInfo(image){
    //   this.imageInfo.seriesNumber=image.data.string('x00200011');//图像序列号
    //   this.imageInfo.imageNum=image.data.string('x00200013');//图像位置
    //   this.imageInfo.imageDate=image.data.string("x00080021");//拍摄日期
    //   this.imageInfo.sliceThickness=image.data.string('x00180050');//层厚
    //   this.imageInfo.patientId=image.data.string('x00100020');//病理号
    //   // 判断窗宽窗位是否合法
    //   this.pixelR = image.data.uint16('x00280103');
    //   this.heightBit = image.data.uint16('x00280102') || '';
    //   // 病人基本信息
    //   this.patientName = image.data.string('x00100010');
    //   this.patientBirthDate = image.data.string('x00100030');
    //   this.patientID = image.data.string('x00100020');
    //   this.patientGender = image.data.string('x00100040');
    //   this.sID = image.data.string('x00200011');
    //   // 像素间距
    //   this.pixelSpacing = image.data.string('x00280030');
    //   this.imagePixelSpacing = image.data.string('x00181164') || '';
    //   this.rowPixelSpacing = image.rowPixelSpacing;
    //   // 放射放大系数
    //   this.magnification = Number(image.data.string('x00181114'));
    //   // 放射源到面板的距离
    //   this.sourceTOdetector = image.data.string('x00181110');
    //   // 放射源到病人的距离
    //   this.sourceTOpatient = image.data.string('x00181111');
    //   //this.modalityLUT = cornerstone.metaData.get('modalityLutModule', image.imageId).modalityLUTSequence;
    //   this.voiContent = cornerstone.metaData.get('voiLutModule', image.imageId);
    //   // 斜率截距
    //   this.rescaleIntercept = Number(image.data.string('x00281052'));
    //   this.rescaleSlope = Number(image.data.string('x00281053'));
    // },
    async show(num) {
      //清除缓存
      // cornerstone.imageCache.purgeCache();
      // cornerstoneWADOImageLoader.wadouri.dataSetCacheManager.purge();
      // cornerstoneWADOImageLoader.wadouri.fileManager.purge();

      const _this = this;
      console.time("sort");
      if (this.isShow === true) {
        // this.isShow = false;
        //  Load & Display
        let canvas = this.$refs.canvas1
        cornerstone.enable(canvas)

        if (this.dcmlist[num].name.indexOf('.pk') != -1) {
          if (!this.dcmlist[num].data) {
            let dcmres = await this.$axios({
              method: 'get',
              responseType: 'blob',
              url: `/static/dcm/${this.dcmlist[num].name}`
            })
            // 把blob格式文件转成FIle类型
            // 读取zip压缩文件里面的文件内容
            let zipfile = await JsZip.loadAsync(dcmres.data)
            for (let key in zipfile.files) {
              let blob = await zipfile.file(zipfile.files[key].name).async("blob")
              console.log(zipfile.files[key].name)
              let files = new window.File([blob], zipfile.files[key].name, {type: 'dic'})
              this.dcmlist[num].data = files
            }
          }
        } else {
          let res = await this.$axios({
            method: 'get',
            responseType: 'blob',
            url: `/static/dcm/${this.dcmlist[num].name}`
          })
          this.dcmlist[num].data = res.data
        }

        let imageId = cornerstoneWADOImageLoader.wadouri.fileManager.add(this.dcmlist[num].data);
        console.timeEnd('sort')
        // console.log(this.dcmlist[num].data)

        cornerstone.loadAndCacheImage(imageId).then(
          function (image) {
            // console.timeEnd('sort')
            image.imageId = _this.dcmlist[num].name
            // console.log(image)
            _this.patientName = image.data.string('x00100010');
            let imageData = image.getPixelData()
            // console.log(imageData.length)
            _this.imageData = imageData

            // image.maxPixelValue = 512;
            // image.minPixelValue = 0;

            image.windowWidth = image.width
            image.windowCenter = image.intercept
            // 设置元素视口
            var viewport = cornerstone.getDefaultViewportForImage(
              canvas,
              image
            );
            //cornerstoneTools API
            // console.log(cornerstoneTools)
            // 显示图像
            cornerstone.displayImage(canvas, image, viewport);
            cornerstoneTools.mouseInput.enable(canvas);
            cornerstoneTools.mouseWheelInput.enable(canvas);
            // cornerstoneTools.length.activate(canvas,1);
            // cornerstoneTools.probe.activate(canvas,1);
            cornerstoneTools.wwwc.activate(canvas, 1); // Left Click
            cornerstoneTools.pan.activate(canvas, 2); // Middle Click
            cornerstoneTools.zoom.activate(canvas, 4); // Right Click
            cornerstoneTools.zoomWheel.activate(canvas); // Mouse Wheel


            // 激活工具
            // _this.initCanvasTools(num);

          },
          function (err) {
            console.log('加载错误', err)
          }
        );
        // Dicom 加载 进度
        // cornerstone.events.addEventListener(
        //   "cornerstoneimageloadprogress",
        //   function (event) {
        //     const eventData = event.detail;
        //     const loadProgress = document.getElementById("loadProgress");
        //     loadProgress.textContent = `Dicom文件加载: ${
        //       eventData.percentComplete
        //       }%`;
        //   }
        // )
      }
    },

    showMask(num) {
      const _this = this;
      var url = this.basePath + this.dcmlist[num]
      // console.log(url)
      let canvas = this.$refs.canvas2
      cornerstone.enable(canvas)
      cornerstone.loadImage(url).then(
        function (image) {
          let imageMaskData = image.getPixelData()

          for (let i = 0; i < imageMaskData.length; i++) {
            imageMaskData[i] = 0
          }

          for (let i = 0; i < _this.result.length; i++) {
            imageMaskData[_this.result[i]] = 255
          }
          // 设置元素视口
          var viewport = cornerstone.getDefaultViewportForImage(
            canvas,
            image
          );
          // 显示图像
          cornerstone.displayImage(canvas, image, viewport);
          cornerstoneTools.mouseInput.enable(canvas);
          cornerstoneTools.mouseWheelInput.enable(canvas);
          cornerstoneTools.wwwc.activate(canvas, 1); // Left Click
          cornerstoneTools.pan.activate(canvas, 2); // Middle Click
          cornerstoneTools.zoom.activate(canvas, 4); // Right Click
          cornerstoneTools.zoomWheel.activate(canvas); // Mouse Wheel
        },
        function (err) {
          console.log('加载错误', err)
        }
      );
    },

    hflip(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.hflip = !viewport.hflip;
      cornerstone.setViewport(canvas, viewport);
    },

    vflip(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.vflip = !viewport.vflip;
      cornerstone.setViewport(canvas, viewport);
    },

    lRotate(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.rotation -= 90;
      cornerstone.setViewport(canvas, viewport);
    },

    rRotate(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      viewport.rotation += 90;
      cornerstone.setViewport(canvas, viewport);
    },

    getLength(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.length.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    probe(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.probe.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    arrowAnnotation(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.arrowAnnotate.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    angle(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.angle.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    freeHand(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.freehand.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    ellipticalRoi(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.ellipticalRoi.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    rectangleRoi(e) {
      let canvas = this.$refs.canvas1
      const viewport = cornerstone.getViewport(canvas);
      cornerstoneTools.rectangleRoi.activate(canvas, 1);
      cornerstone.setViewport(canvas, viewport);
    },

    reset(e) {
      let canvas = this.$refs.canvas1
      cornerstone.reset(canvas);
    },

    renderModel() {
      myWindow = window.open('http://www.baidu.com', '百度', 'width=800,height=600')
    },

    initCanvasTools(num) {
      let _self = this;
      let canvas = this.$refs.canvas1;
      this.isInitLoad = false;

      // 为 canvasStack 找到 imageIds

      // let allImageIds = [];
      // this.exampleStudyImageIds.forEach(function (imageId) {
      //   let imageUrl = "wadouri:" + _self.baseUrl + imageId;
      //   allImageIds.push(imageUrl);
      // });

      // var url = 'wadouri:\\static\\dcm\\' + num + '.DCM'

      // Create canvasStack
      // let canvasStack = {
      //   currentImageIdIndex: 0,
      //   imageIds: url
      // };

    },

    // Window Resize
    listenForWindowResize() {
      this.$nextTick(function () {
        window.addEventListener(
          "resize",
          this.debounce(this.onWindowResize, 100)
        );
      });
    },

    onWindowResize() {
      cornerstone.resize(this.$refs.canvas, true);
    },

    // Utility Methods
    debounce(func, wait, immediate) {
      var timeout;
      return function () {
        var context = this;
        var args = arguments;
        var later = function () {
          timeout = null;
          if (!immediate) func.apply(context, args);
        };
        var callNow = immediate && !timeout;
        clearTimeout(timeout);
        timeout = setTimeout(later, wait);
        if (callNow) func.apply(context, args);
      };
    }
  }
};
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style lang="css" scoped>

.main {
  width: 100vw;
  height: 100vh;
  background-color: #000;
}

.top-title {
  width: 100vw;
  height: 50px;
  background: black;
  position: absolute;
  left: 0;
  top: 0;
  display: flex;
  flex-direction: row;
}

.title-logo {
  position: absolute;
  top: 0;
  left: 0;

}

.top-tool-bar > div > i {
  /*width: 50px;*/
  height: 33px;
  line-height: 30px;
  font-size: 20px;
}

.top-tool-bar > div :hover {
  color: #b7b9b7;
}

.title-name {
  color: white;
  line-height: 40px;
  font-size: 18px;
  width: 240px;
  height: 40px;
  margin: 5px 16px 5px 50px;
  border-right: 1px solid white;
}

.patient-list {
  color: lightblue;
  line-height: 42px;
  font-size: 18px;
  width: 100px;
  height: 42px;
  margin: 4px 16px 4px 0px;
}

.top-tool-bar {
  width: 100vw;
  height: 50px;
  background: #2D2F33;
  position: absolute;
  left: 0;
  top: 0px;
  padding-top: 10px;
  padding-bottom: 10px;
  display: flex;
  /*flex-direction: row;*/
  justify-content: center;
}


.top-tool-bar > div {
  padding: 0 18px;
  color: #D1E4FF;
  cursor: pointer;
  opacity: .3;
}

.top-tool-bar > div:hover {
  color: #D1E4FF;
}

.top-tool-bar > h4 {
  color: white;
  font-size: 14px;
  line-height: 24px;
}

.image-canvas-wrapper {
  width: 100vw;
  height: 80vh;
  position: absolute;
  top: 150px;
  color: orange;
  display: flex;
}

.image-canvas {
  border: 1px dashed #b7b9b7 !important;
}

.image-canvas-right {
  width: 100vw;
  height: 80vh;
  background: lightgrey;
}

.slide-bar {
  width: 400px;
  height: 32px;
  margin: 20px auto;
}

.image-canvas {
  width: 60vw;
  height: 60vw;
  margin: 20px;
  display: inline-block;
}

.mask-canvas {
  width: 40vw;
  height: 40vw;
  margin: 20px;
  display: inline-block;
  border: 1px dashed #b7b9b7 !important;
}
</style>
