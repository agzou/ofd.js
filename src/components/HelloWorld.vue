<template>
  <el-container style="width:100vw; height: 100vh;">
    <el-header style="display: flex; height: 40px; border: 1px solid #e8e8e8; align-items: center;padding-left: 90px">
      <div class="upload-icon" @click="uploadFile">
        <div class="upload-icon">打开OFD</div>
        <font-awesome-icon icon="cloud-upload-alt"/>
        <input type="file" ref="file" class="hidden" accept=".ofd"
               @change="fileChanged">
      </div>

      <div style="display: flex" v-if="ofdObj">
        <div class="scale-icon" style="margin-left: 10px" @click="plus">
          <font-awesome-icon icon="search-plus"/>
        </div>

        <div class="scale-icon" @click="minus">
          <font-awesome-icon icon="search-minus" />
        </div>
        <div class="scale-icon">
          <font-awesome-icon icon="step-backward" @click="firstPage"/>
        </div>

        <div class="scale-icon" style="font-size: 18px" @click="prePage">
          <font-awesome-icon icon="caret-left"/>
        </div>

        <div class="scale-icon">
          {{pageIndex}}/{{pageCount}}
        </div>

        <div class="scale-icon" style="font-size: 18px" @click="nextPage">
          <font-awesome-icon icon="caret-right"/>
        </div>

        <div class="scale-icon" @click="lastPage">
          <font-awesome-icon icon="step-forward"/>
        </div>
      </div>

    </el-header>
    <el-main style="height: auto;background: #808080;;padding: 0">
      <div
          style="position: fixed;width: 88px;height: 100%;background: white;border: 1px solid #e8e8e8;align-items: center;display: flex;flex-direction: column">
        <div class="text-icon" @click="demo(1)">
          <p>电子发票</p>
        </div>

        <div class="text-icon" @click="demo(2)">
          <p>电子公文</p>
        </div>

        <div class="text-icon" @click="demo(3)">
          <p>骑缝章</p>
        </div>

        <div class="text-icon" @click="demo(4)">
          <p>多页文档</p>
        </div>
      </div>
      <div
          style="padding-top: 20px;margin-left:88px;display: flex;flex-direction: column;align-items: center;justify-content: center;background: #808080;overflow: hidden"
          id="content" ref="contentDiv" @mousewheel="scrool">
      </div>
    </el-main>
    <div class="SealContainer" id="sealInfoDiv" hidden="hidden" ref="sealInfoDiv">
      <div class="SealContainer mask" @click="closeSealInfoDialog"></div>
      <div class="SealContainer-layout">
        <div class="SealContainer-content">
          <p class="content-title">签章信息</p>
          <div class="subcontent">
            <span class="title">签章人</span>
            <span class="value" id="spSigner">Signer</span>
          </div>
          <div class="subcontent">
            <span class="title">签章提供者</span>
            <span class="value" id="spProvider">Provider</span>
          </div>
          <div class="subcontent">
            <span class="title">原文摘要值</span>
            <span class="value" id="spHashedValue" @click="showMore('原文摘要值', 'spHashedValue')"
                  style="cursor: pointer">HashedValue</span>
          </div>
          <div class="subcontent">
            <span class="title">签名值</span>
            <span class="value" id="spSignedValue" @click="showMore('签名值', 'spSignedValue')"
                  style="cursor: pointer">SignedValue</span>
          </div>
          <div class="subcontent">
            <span class="title">签名算法</span>
            <span class="value" id="spSignMethod">SignMethod</span>
          </div>
          <div class="subcontent">
            <span class="title">版本号</span>
            <span class="value" id="spVersion">Version</span>
          </div>
          <div class="subcontent">
            <span class="title">验签结果</span>
            <span class="value" id="VerifyRet">VerifyRet</span>
          </div>

          <p class="content-title">印章信息</p>
          <div class="subcontent">
            <span class="title">印章标识</span>
            <span class="value" id="spSealID">SealID</span>
          </div>
          <div class="subcontent">
            <span class="title">印章名称</span>
            <span class="value" id="spSealName">SealName</span>
          </div>
          <div class="subcontent">
            <span class="title">印章类型</span>
            <span class="value" id="spSealType">SealType</span>
          </div>
          <div class="subcontent">
            <span class="title">有效时间</span>
            <span class="value" id="spSealAuthTime">从NotBefore到NotAfter</span>
          </div>
          <div class="subcontent">
            <span class="title">制章日期</span>
            <span class="value" id="spSealMakeTime">MakeTime</span>
          </div>
          <div class="subcontent">
            <span class="title">印章版本</span>
            <span class="value" id="spSealVersion">Version</span>
          </div>
        </div>
        <input style="position:absolute;right:1%;top:1%;" type="button" name="" id="" value="X"
               @click="closeSealInfoDialog()"/>
      </div>

    </div>


    <el-dialog :title="title" :visible.sync="dialogFormVisible">
      <span style="text-align: left">
        {{value}}
      </span>
      <div slot="footer" class="dialog-footer">
        <el-button type="primary" @click="dialogFormVisible = false">确 定</el-button>
      </div>
    </el-dialog>
  </el-container>
</template>

<script>

import {parseOfdDocument, renderOfd, renderOfdByScale} from "@/utils/ofd/ofd";
import * as JSZipUtils from "jszip-utils";
import {getPageScal, setPageScal} from "@/utils/ofd/ofd_util";

export default {
  name: 'HelloWorld',
  data() {
    return {
      pageIndex: 1,
      pageCount: 0,
      scale: 0,
      title: null,
      value: null,
      dialogFormVisible: false,
      ofdObj: null,
      screenWidth: document.body.clientWidth - 88,
    }
  },

  created() {
    this.file = null;
  },

  mounted() {
    let that = this;
    this.$refs.contentDiv.addEventListener('scroll', this.scrool);
    window.onresize = () => {
      return (() => {
        that.screenWidth = (document.body.clientWidth - 88);
        const divs = renderOfd(that.screenWidth, that.ofdObj);
        that.displayOfdDiv(divs);
      })()
    }
  },

  methods: {
    scrool() {
      let scrolled = this.$refs.contentDiv.firstElementChild.getBoundingClientRect()?.top - 60;
      let top = 0
      let index = 0;
      for (let i=0;i<this.$refs.contentDiv.childElementCount; i ++) {
        top += (Math.abs(this.$refs.contentDiv.children.item(i)?.style.height.replace('px','')) + Math.abs(this.$refs.contentDiv.children.item(i)?.style.marginBottom.replace('px','')));
        if (Math.abs(scrolled) < top) {
          index = i;
          break;
        }
      }
      this.pageIndex = index+1;
    },

    closeSealInfoDialog() {
      this.$refs.sealInfoDiv.setAttribute('style', 'display: none');
    },

    showMore(title, id) {
      this.dialogFormVisible = true;
      this.value = document.getElementById(id).innerText;
      this.title = title;
    },

    plus() {
      setPageScal(++this.scale);
      const divs = renderOfdByScale(this.ofdObj);
      this.displayOfdDiv(divs);
    },

    minus() {
      setPageScal(--this.scale);
      const divs = renderOfdByScale(this.ofdObj);
      this.displayOfdDiv(divs);
    },

    prePage() {
      let contentDiv = document.getElementById('content');
      let ele = contentDiv.children.item(this.pageIndex-2);
      ele?.scrollIntoView(true);
      ele?this.pageIndex=this.pageIndex-1:'';
    },

    firstPage() {
      let contentDiv = document.getElementById('content');
      let ele = contentDiv.firstElementChild;
      ele?.scrollIntoView(true);
      ele?this.pageIndex=1:'';
    },

    nextPage() {
      let contentDiv = document.getElementById('content');
      let ele = contentDiv.children.item(this.pageIndex);
      ele?.scrollIntoView(true);
      ele?++this.pageIndex:'';
    },

    lastPage() {
      let contentDiv = document.getElementById('content');
      let ele = contentDiv.lastElementChild;
      ele?.scrollIntoView(true);
      ele?this.pageIndex=contentDiv.childElementCount:'';
    },

    demo(value) {
      let ofdFile = null;
      switch (value) {
        case 1:
          ofdFile = 'https://51shouzu.xyz/999.ofd';
          break;
        case 2:
          ofdFile = 'https://51shouzu.xyz/n.ofd';
          break;
        case 3:
          ofdFile = 'https://51shouzu.xyz/h.ofd';
          break;
        case 4:
          ofdFile = 'https://51shouzu.xyz/2.ofd';
          break;
      }
      let that = this;
      JSZipUtils.getBinaryContent(ofdFile, function (err, data) {
        if (err) {
          throw err; // or handle err
        }
        that.getOfdDocumentObj(data, that.screenWidth);
      });

    },

    uploadFile() {
      this.file = null;
      this.$refs.file.click();
    },
    fileChanged() {
      this.file = this.$refs.file.files[0];
      let ext = this.file.name.replace(/.+\./, "");
      if (["ofd"].indexOf(ext) === -1) {
        // this.$toast('error', "仅支持png、jpg、jpeg的图片类型");
        return;
      }
      if (this.file.size > 5 * 1024 * 1024) {
        // this.$toast('error', "文件大小需 < 5M");
        return;
      }
      this.getOfdDocumentObj(this.file, this.screenWidth);
      this.$refs.file.value = null;
    },


    getOfdDocumentObj(file, screenWidth) {
      let that = this;
      parseOfdDocument({
        ofd: file,
        success(res) {
          that.ofdObj = res;
          that.pageCount = res.pages.length;
          const divs = renderOfd(screenWidth, res);
          that.displayOfdDiv(divs);
        },
        fail(error) {
          console.log(error)
        }
      });
    },

    displayOfdDiv(divs) {
      this.scale = getPageScal();
      let contentDiv = document.getElementById('content');
      contentDiv.innerHTML = '';
      for (const div of divs) {
        contentDiv.appendChild(div)
      }
    }

  }
}
</script>

<!-- Add "scoped" attribute to limit CSS to this component only -->
<style scoped>
.upload-icon {
  display: flex;
  cursor: pointer;
  justify-content: center;
  align-items: center;
  height: 28px;
  padding-left: 10px;
  padding-right: 10px;
  background-color: rgb(59, 95, 232);
  border-radius: 1px;
  border-color: #5867dd;
  font-weight: 500;
  font-size: 12px;
  color: white;
  margin: 1px;
}

.scale-icon {
  display: flex;
  cursor: pointer;
  justify-content: center;
  align-items: center;
  width: 33px;
  height: 28px;
  background-color: white;
  border-radius: 1px;
  font-weight: 500;
  font-size: 12px;
  color: #333333;
  text-align: center;
  padding: 2px;

}
.scale-icon :active {
  color: rgb(59, 95, 232);
}
.scale-icon :hover {
  color: rgb(59, 95, 232);
}

.text-icon {
  display: flex;
  cursor: pointer;
  justify-content: center;
  align-items: center;
  height: 28px;
  width: 90%;
  background-color: rgb(59, 95, 232);
  border-radius: 1px;
  border-color: #5867dd;
  font-weight: 500;
  font-size: 10px;
  color: white;
  margin-top: 20px;

}

.hidden {
  display: none !important;
}

.SealContainer {
  position: fixed;
  left: 0;
  top: 0;
  width: 100vw;
  height: 100vh;
}

.SealContainer .mask {
  background: #000000;
  opacity: 0.3;
}

.content-title {
  font-size: 16px;
  text-align: center;
  border-bottom: 1px solid rgb(59, 95, 232);
  color: rgb(59, 95, 232);
  margin-top: 10px;
}


.SealContainer-content {
  position: relative;
  width: 100%;
  height: 100%;
  overflow-y: auto;
  background: white;
  display: flex;
  flex-direction: column;
  padding: 10px;
  align-items: center;
}

.SealContainer-layout {
  position: relative;
  width: 60%;
  height: 80vh;
  overflow-y: auto;
  background: white;
  z-index: 100;
  display: flex;
  flex-direction: column;
  padding: 10px;
  align-items: center;
}

.subcontent {
  width: 80%;
  display: flex;
  flex-direction: column;
  text-align: left;
  margin-bottom: 10px;
  font-family: simsun;
}

@media (max-width: 767px) {
  .SealContainer-layout {
    position: relative;
    width: 90%;
    height: 90vh;
    overflow-y: auto;
    background: white;
    z-index: 100;
    display: flex;
    flex-direction: column;
    padding: 10px;
    align-items: center;
  }

  .subcontent {
    width: 95%;
    display: flex;
    flex-direction: column;
    text-align: left;
    margin-bottom: 10px;
    font-family: simsun;
  }
}

.subcontent .title {
  font-weight: 600;
}

.subcontent .value {
  font-weight: 400;
  -webkit-line-clamp: 1;
  display: -webkit-box;
  -webkit-box-orient: vertical;
  overflow: hidden;
}
</style>
