<template>
  <div>
    <Card id="html2canvas">
      <Tabs :animated="false">
        <TabPane label="工具类" id="printjs">
          <Divider orientation="left">打印 - Print.js</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/crabbly/Print.js"
            target="_blank"
            class="href-text"
          >https://github.com/crabbly/Print.js</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a
            class="href-text"
            href="http://printjs.crabbly.com"
            target="_blank"
          >http://printjs.crabbly.com</a>
          <br>

          <Button @click="printHtml" class="example-btn">打印网页示例</Button>
          <Button @click="printJson" class="example-btn">打印Json数据示例</Button>

          <Divider orientation="left">截屏 - html2canvas</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/niklasvh/html2canvas"
            target="_blank"
            class="href-text"
          >https://github.com/niklasvh/html2canvas</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a
            class="href-text"
            href="https://html2canvas.hertzen.com"
            target="_blank"
          >https://html2canvas.hertzen.com</a>
          <br>

          <Button @click="html2canvas" class="example-btn">截取网页示例</Button>
          <Button @click="html2canvas2" class="example-btn">截取指定内容示例</Button>

          <Divider orientation="left">打印 + 截屏 实现网页样式不丢失打印</Divider>
          <Button @click="html2canvas3" class="example-btn">打印原样式网页图片示例</Button>

          <Divider orientation="left">复制命令 - vue-clipboard2</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/Inndy/vue-clipboard2"
            target="_blank"
            class="href-text"
          >https://github.com/Inndy/vue-clipboard2</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a
            class="href-text"
            href="https://github.com/Inndy/vue-clipboard2"
            target="_blank"
          >https://github.com/Inndy/vue-clipboard2</a>
          <br>
          <Button
            v-clipboard:copy="content"
            v-clipboard:success="onCopy"
            v-clipboard:error="onError"
            class="example-btn"
          >复制文本示例</Button>
          <Input
            type="textarea"
            :autosize="{minRows: 2,maxRows: 5}"
            style="width:300px"
            placeholder="粘贴测试"
          />

          <Divider orientation="left">轻量级时间转换工具 - date-fns</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/date-fns/date-fns"
            target="_blank"
            class="href-text"
          >https://github.com/date-fns/date-fns</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a class="href-text" href="https://date-fns.org" target="_blank">https://date-fns.org</a>
          <br>

          <span class="href-text">示例：{{time}}</span>

          <Divider orientation="left">图片懒加载 - vue-lazyload</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/hilongjw/vue-lazyload"
            target="_blank"
            class="href-text"
          >https://github.com/hilongjw/vue-lazyload</a>
          <br>
          <span class="href-text">官网：</span>
          <a
            class="href-text"
            href="http://hilongjw.github.io/vue-lazyload"
            target="_blank"
          >http://hilongjw.github.io/vue-lazyload</a>
          <br>
        </TabPane>

        <TabPane label="组件类">
          <Divider orientation="left">省市县级联组件 - iView Area</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/iview/iview-area"
            target="_blank"
            class="href-text"
          >https://github.com/iview/iview-area</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a
            class="href-text"
            href="https://iview.github.io/iview-area"
            target="_blank"
          >https://iview.github.io/iview-area</a>
          <br>

          <al-selector v-model="resArr" class="example-btn" style="width: 800px"/>
          {{resArr}}
          <al-cascader v-model="resArr2" class="example-btn" style="width: 400px"/>
          {{resArr2}}

          <Divider orientation="left">图片裁剪 - vue-cropper</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/xyxiao001/vue-cropper"
            target="_blank"
            class="href-text"
          >https://github.com/xyxiao001/vue-cropper</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a
            class="href-text"
            href="http://xyxiao.cn/vue-cropper/example"
            target="_blank"
          >http://xyxiao.cn/vue-cropper/example</a>
          <br>
          <br>

          <vueCropper
            style="height:300px;width:500px"
            ref="cropper"
            :img="option.img"
            :outputType="option.outputType"
            autoCrop
            @realTime="realTime"
          ></vueCropper>

          <br>
          <div :style="previewStyle">
            <div :style="previews.div">
              <img :src="previews.url" :style="previews.img">
            </div>
          </div>
          <br>
          <Alert type="warning" show-icon style="width:500px">base64上传，上传接口传入参数base64即可</Alert>
          <Button
            type="primary"
            @click="upload"
            :loading="uploadLoading"
            icon="ios-cloud-upload-outline"
          >上传裁剪后的图片</Button>

          <Divider orientation="left">弹幕视频播放器 - DPlayer</Divider>
          <span class="href-text">Github：</span>
          <a
            href="https://github.com/MoePlayer/DPlayer"
            target="_blank"
            class="href-text"
          >https://github.com/MoePlayer/DPlayer</a>
          <br>
          <span class="href-text">官方文档：</span>
          <a class="href-text" href="http://dplayer.js.org" target="_blank">http://dplayer.js.org</a>
          <br>
          <span class="href-text">自己搭建弹幕服务：</span>
          <a
            class="href-text"
            href="https://github.com/MoePlayer/DPlayer-node"
            target="_blank"
          >https://github.com/MoePlayer/DPlayer-node</a>
          <br>
          <br>

          <div id="dplayer" style="width:600px;height:400px"></div>
        </TabPane>
      </Tabs>
    </Card>

    <Modal title="截屏预览" v-model="modalVisible" :width="1000">
      <img id="render" :src="imgUrl" width="100%" height="500px">
      <div slot="footer">
        <Button type="primary" @click="printPic" v-if="showPrintImage">打印图片</Button>
        <Button @click="modalVisible=false">关闭</Button>
      </div>
    </Modal>
  </div>
</template>

<script>
// 组件内使用
import { VueCropper } from "vue-cropper";
import printJS from "print-js";
import html2canvas from "html2canvas";
import { format } from "date-fns";
import "dplayer/dist/DPlayer.min.css";
import DPlayer from "dplayer";
import { base64Upload } from "@/api/index.js";
export default {
  name: "xboot-library",
  components: {
    VueCropper
  },
  data() {
    return {
      resArr: [],
      resArr2: [],
      modalVisible: false,
      imgUrl: "",
      showPrintImage: false,
      content: "测试文本内容",
      time: "",
      data: [
        {
          id: "1",
          name: "XBoot",
          createTime: "2018-08-08 00:08:00",
          updateTime: "2018-08-08 00:08:00"
        },
        {
          id: "2",
          name: "Exrick",
          createTime: "2018-08-08 00:08:00",
          updateTime: "2018-08-08 00:08:00"
        }
      ],
      option: {
        img: "https://i.loli.net/2019/03/12/5c87521fb8ae9.jpeg",
        outputType: "png"
      },
      previews: "",
      previewStyle: {},
      uploadLoading: false
    };
  },
  methods: {
    init() {
      this.time = format(new Date(), "YYYY-MM-DD");
      this.initVideo();
    },
    printHtml() {
      printJS("printjs", "html");
    },
    printJson() {
      printJS({
        printable: this.data,
        header: "用户数据",
        gridStyle: "border: 1px solid lightgray;text-align:center",
        properties: ["id", "name", "createTime", "updateTime"],
        type: "json"
      });
    },
    html2canvas() {
      let that = this;
      html2canvas(document.body).then(function(canvas) {
        that.imgUrl = canvas.toDataURL("image/jpeg");
      });
      this.modalVisible = true;
      this.showPrintImage = false;
    },
    html2canvas2() {
      let that = this;
      html2canvas(document.getElementById("html2canvas")).then(function(
        canvas
      ) {
        that.imgUrl = canvas.toDataURL("image/jpeg");
      });
      this.modalVisible = true;
      this.showPrintImage = false;
    },
    html2canvas3() {
      let that = this;
      html2canvas(document.body).then(function(canvas) {
        that.imgUrl = canvas.toDataURL("image/jpeg");
      });
      this.modalVisible = true;
      this.showPrintImage = true;
    },
    printPic() {
      printJS({
        printable: "render",
        type: "html",
        header: "图片数据"
      });
    },
    onCopy() {
      this.$Message.success("复制成功");
    },
    onError() {
      this.$Message.warning("复制失败，请手动复制");
    },
    handleRadioClick(v) {
      this.selectCount = this.$refs.treeTable.getCheckedProp().length;
    },
    click(v) {
      this.$Message.info("点击数据ID为：" + v.id);
    },
    initVideo(v) {
      const dp = new DPlayer({
        container: document.getElementById("dplayer"),
        screenshot: true,
        video: {
          url:
            "https://cloud.video.taobao.com//play/u/95292294/p/1/e/6/t/1/210877258798.mp4"
        },
        danmaku: {
          id: "弹幕ID210877258798",
          api: "https://api.prprpr.me/dplayer/"
        }
      });
    },
    // 实时预览
    realTime(data) {
      let preview = data,
        h = 0.5;
      this.previewStyle = {
        width: preview.w + "px",
        height: preview.h + "px",
        overflow: "hidden",
        margin: "0",
        zoom: h
      };
      this.previews = data;
    },
    upload() {
      if (!this.$route.meta.permTypes.includes("upload")) {
        this.$Message.error("您此处没有上传权限");
        return;
      }
      // 获取截图的base64 数据
      this.$refs.cropper.getCropData(data => {
        this.uploadLoading = true;
        base64Upload({ base64: data }).then(res => {
          this.uploadLoading = false;
          if (res.success) {
            this.$Message.success("上传成功");
          }
        });
      });
    }
  },
  mounted() {
    this.init();
  }
};
</script>

<style lang="less">
.href-text {
  font-size: 12px;
}
.example-btn {
  margin: 10px 0;
  display: block;
}
</style>