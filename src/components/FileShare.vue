<template>
  <a-row type="flex">
    <a-col :flex="5"></a-col>
    <a-col :flex="2">
      <a-divider/>
      <a-row>
        <a-col span="1"></a-col>
        <a-col span="22">
          <a-card title="温馨提示">
            <h5 style="color: darkgreen">欢迎使用临时网盘系统.</h5>
            <h5 style="color: brown">请勿上传隐私文件!</h5>
            <h3>文件有效期为 <a style="color: red"> {{ limitFileLife }} </a> 小时！</h3>
            <h3>文件大小限制 <a style="color: red"> {{ limitFileSize }} </a> MB！</h3>
          </a-card>
        </a-col>
        <a-col span="1"></a-col>
      </a-row>
      <br/>
      <div style="padding-inline: 4%;">
        <a-upload-dragger :progress="progress" name="file" :before-upload="beforeUpload" :showUploadList="true"
                          :capture="null" :multiple="false" action="/file/upload" @change="handleChange">
          <p class="ant-upload-drag-icon">
            <inbox-outlined></inbox-outlined>
          </p>
          <p style="color: mediumpurple" class="ant-upload-text">点击或拖拽文件到这里进行上传</p>
        </a-upload-dragger>
      </div>

      <br/>
      <div  style="padding-inline: 4%;margin: auto">
        <a-input v-model:value="shareCode" allowClear placeholder="请输入文件提取码" show-count :maxlength="10"
                 style="vertical-align:middle;width: calc(95% - 120px);height: 40px;"/>
        <a-divider type="vertical"/>
        <a-button @click="download" type="primary" shape="round" style="vertical-align:middle;height: 40px;">
          <template #icon>
            <DownloadOutlined/>
          </template>
          提取文件
        </a-button>
      </div>
      <a-divider/>
    </a-col>
    <a-col :flex="5"></a-col>
  </a-row>
  <a-modal v-model:visible="showResult" okText="确定" cancelText="取消" @ok="handleOkShowResult">
    <a-result :status="uploadStatus" :title="uploadResult" :sub-title="subTitle">
      <template #extra>
        <a-button @click="copy" type="primary" shape="round">
          <template #icon>
            <CopyOutlined/>
          </template>
          复制下载链接
        </a-button>
      </template>
    </a-result>
  </a-modal>
  <a-modal v-model:visible="showConfirm" title="确定下载吗？" okText="确定" cancelText="取消" @ok="handleOkShowConfirm">
    <p>文件名: {{ fileName }}</p>
    <p>文件大小: {{ fileSize }}MB</p>
    <p>文件md5: {{ fileMd5 }}</p>
    <p>文件上传时间: {{ uploadDate }}</p>
  </a-modal>
</template>

<script>
import {CopyOutlined, DownloadOutlined, InboxOutlined} from '@ant-design/icons-vue';
import {message} from 'ant-design-vue';
import {defineComponent} from 'vue';
import useClipboard from 'vue-clipboard3';
import axios from 'axios';

const {toClipboard} = useClipboard();

export default defineComponent({
  components: {
    InboxOutlined, DownloadOutlined, CopyOutlined
  },
  mounted() {
    axios.get("/file/config").then((res) => {
      this.limitFileLife = res.data.limitFileLife
      this.limitFileSize = res.data.limitFileSize
    });
  },
  data() {
    const beforeUpload = file => {
      const sizeLimit = file.size / 1024 / 1024 < this.limitFileSize;
      if (!sizeLimit) {
        message.error('文件大小不可超过 ' + this.limitFileSize + ' MB !');
      }
      return sizeLimit;
    };
    const progress = {
      strokeColor: {
        '0%': '#108ee9',
        '100%': '#87d068',
      },
      strokeWidth: 3,
      format: percent => `${parseFloat(percent.toFixed(2))}%`,
      class: 'test',
    };
    return {
      downLoadUrl: '',
      limitFileSize: 10,
      limitFileLife: 10,
      fileName: '',
      fileSize: 0,
      fileMd5: '',
      uploadDate: '',
      subTitle: '',
      shareCode: '',
      showResult: false,
      showConfirm: false,
      uploadResult: '',
      uploadStatus: 'info',
      handleChange: e => {
        if (e.file.status === 'done') {
          this.showResult = true
          this.uploadStatus = e.file.response.status
          this.downLoadUrl = window.location.href + "file/download/" + e.file.response.fileObj.shareCode
          this.subTitle = "下载链接：" + this.downLoadUrl

          if (e.file.response.success) {
            this.uploadResult = e.file.response.message + "提取码：" + e.file.response.fileObj.shareCode
          } else {
            this.uploadResult = e.file.response.message + "提取码：" + e.file.response.fileObj.shareCode
          }
        }
      },
      beforeUpload,
      progress,
    };
  },
  methods: {
    handleOkShowResult() {
      this.showResult = false
    },
    handleOkShowConfirm() {
      this.showConfirm = false
      message.info("即将开始下载！文件大小为：" + this.fileSize + " MB！", 10)

      //准备下载文件
      window.location.href = "/file/download/" + this.shareCode
    },
    download() {
      if (this.shareCode.length < 1) {
        message.warning("提取码不可为空！")
        return
      }
      axios.get("/file/exist/" + this.shareCode).then((res) => {
        if (res.data.success) {
          this.showConfirm = true
          this.fileSize = parseFloat(res.data.fileObj.fileSize / 1024 / 1024).toFixed(2)
          this.fileName = res.data.fileObj.fileName
          this.fileMd5 = res.data.fileObj.fileMd5
          this.uploadDate = res.data.fileObj.uploadDate
        } else {
          message.warning(res.data.message)
        }
      });
    },
    copy() {
      try {
        toClipboard(this.downLoadUrl)
        message.success('复制成功！')
      } catch (e) {
        message.error('复制失败！')
      }
    },
  }
});
</script>

