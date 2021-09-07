<template>
  <div id="app">
    <div class="mater-upload-container">
      <Simple
        ref="upload"
        :headers="headers"
        :before-upload="beforeUpload"
        :accept="accepts"
        :upload-arguments="uploadArgumentsObj"
        :limit="limit"
        :on-exceed="fileLimitFn"
        :base-url="baseUrl"
        :chunk-size="chunkSize"
        @success="success"
      >
        <div slot="tip" class="upload-tip">
          <i class="el-icon-info"></i>:
          只能上传：{{ acceptDesc[uploadType] }}
        </div>
      </Simple>
    </div>
  </div>
</template>
<script>
import Simple from './simple';
export default {
  name: 'MaterUpload',
  components: { Simple },
  data: () => ({
    accepts: 'image/png',
    acceptsObj: {
      video: ['*'],
      image: [
        'image/png',
        'image/gif',
        'image/jpeg',
        'image/jpg',
        'image/bmp',
        '.'
      ], // 火狐的accept只支持【.png,.jpg】这种形式，为了兼容，使用 .
      audio: ['audio/mp3', 'audio/mpeg'],
      ppt: [
        'application/vnd.ms-powerpoint',
        'application/vnd.openxmlformats-officedocument.presentationml.presentation',
        '.ppt',
        '.pptx'
      ],
      excel: [
        'application/vnd.ms-excel',
        'application/vnd.openxmlformats-officedocument.spreadsheetml.sheet'
      ]
    },
    acceptDesc: {
      video: 'mp4',
      image: 'png,gif,jpeg,jpg,bmp',
      audio: 'mp3',
      ppt: 'ppt',
      excel: 'xls,xlsx'
    },
    // 临时自测使用
    uploadArguments: {
      type: 'video'
    },
    limit: 20,
    chunkSize: 10 * 1024 * 1024,
    share: 1 // 是否共享 0私有  1共享
  }),
  computed: {
    headers() {
      return {
        Authorization: 'jJJfk5ssHIW_0Q6ivrO5dP23-PrDxoiKN52oiQHR2JrUyKfvsJ3e9xrOM-c8cNmTWxt3IhMJYZpYVJCVzLnb-eQ_Mi516SeHt8ueyDZWqp2PIptX0VdqJPIuoACHamaDHjToUINmO9UeAp-sZMpVPynZlIelMTA5vPhUSpjASovBhok_aHATcEIulaVMC6qyAFAT841XgOhuGNHWRX_z4U6LwYJi2EGzDaSHqfRs0FykfqIuMb3zyp2b8nDrPj-u5-ooMkFeIJlDi9ABCRYEZKOSb1QDJmITfVw-eeWiGru3tmLmvmGtLe6XVh3Qpf5QRN5x6id7U0dWU1NU42hsPhPV2fDFp5o8QcITfOdlmE1crrwezdnWmQ1zc2tjNZ7r3P1yNg3ietoEr8t_cFg65iGg4BoCjk337LSJ85wvJkvl6BoHWeE9aBVqLhCCOJ_SYsBNdXxkAbdBf4xZzBENPNv0O1cRIvL-ohI2KplLA89N_WT5syfZeBXpLj7svW_d-jq_p5W1dBPdW5uqbm4WjtkDfGdLZWhcPfnERSZDDNrDtBgRBONKZmpxYT7Q68rCqGBW_42NR3E9vpXiHTfpD2L9mrL09N3-PE0_NRvz_0_vZH33VOCEIWLdhWD2GwAA__8',
        // 'Paasport-App-Id': '1400397007335464961',
        'Paasport-App-Id': '1258518235020660736',
        // 'Paasport-Device-Id': '102755df-682e-49b7-be56-d00fbd35bb71-abc123'
        'Paasport-Device-Id': '81bec2c5-872f-48b1-965c-3e834df0ac47'
      };
    },
    baseUrl() {
      // return 'https://passport-gw.zielhome.com';
      return 'http://wksw.com:9091';
    },
    uploadType() {
      return this.uploadArguments.type;
    },
    uploadArgumentsObj() {
      return { ...this.uploadArguments, share: this.share };
    }
  },

  created() {
    if (this.uploadType) {
      this.accepts = this.acceptsObj[this.uploadType].join(','); // 设置文件类型
    } else {
      this.$message('存在类型不正确的文件');
    }
  },
  methods: {
    beforeUpload(file) {
      console.log('beforeAvatarUpload -> file', file);
      console.log(this.acceptsObj[this.uploadType].indexOf('*'));
      if (this.acceptsObj[this.uploadType].indexOf(file.type) === -1 && this.acceptsObj[this.uploadType].indexOf('*') !== 0) {
        this.$notify({
          message: '只能上传' + this.acceptDesc[this.uploadType],
          type: 'warning',
          offset: 50
        });
        return false;
      }
      if (!file.size) {
        setTimeout(() => {
          this.$notify({
            message: '不能上传大小为0的文件',
            type: 'warning',
            offset: 50
          });
        }, 500);
        return false;
      }
      return this.propertyRestrictions(file);
    },
    // 文件个数限制钩子
    fileLimitFn(files) {
      this.$message.warning(
        `最多支持选择${this.limit}个文件`
      );
    },
    // 清空文件，暂未使用
    clearFiles() {
      this.$refs.upload.clearFiles();
    },
    success() {
      this.$message('上传成功');
    },
    // 属性限制
    async propertyRestrictions(file) {
      return new Promise(async (resolve, reject) => {
        if (this.uploadType === 'image') {
          const isVerifyResolution = await this.verifyResolution(file);
          console.log('propertyRestrictions -> isVerifyResolution', isVerifyResolution);
          if (!isVerifyResolution) {
            this.$message(this.$t('messageTips.notAbove4k'));
            reject();
          }
        }
        resolve(true);
      });
    },
    // 分辨率校验
    verifyResolution(file, maxWidth = 3840, maxHeight = 2160) {
      return new Promise((resolve, reject) => {
        const reader = new FileReader();
        reader.readAsDataURL(file);
        reader.onload = function () {
          if (reader.readyState === 2) {
            const img = new Image();
            img.src = reader.result;
            img.onload = function () {
              const width = this.width;
              const height = this.height;
              const bool = width > maxWidth || height > maxHeight;
              if (bool) {
                resolve(false);
              }
              resolve(true);
            };
          }
        };
      });
    }
  }
};
</script>
<style scoped lang="scss">
.mater-upload-container {
  >>> .simple-upload-container {
    border: none;
    max-height: 500px;
  }
}
</style>
