<template>
  <div class="stt-page">
    <header class="hero">
      <div class="hero__inner">
        <div class="brand-mark">
          <img src="/sttlink-logo.jpg" alt="STTlink">
        </div>
        <div class="hero__copy">
          <h1>STTlink</h1>
          <p>简单、快速的订阅链接转换工具</p>
        </div>
        <button class="project-link" type="button" @click="goToSttlink" aria-label="打开 STTlink 官网">
          <img class="project-link__logo" src="/sttlink-logo.jpg" alt="STTlink">
          <span>STTlink</span>
        </button>
      </div>
    </header>

    <main class="page-shell">
    <el-row>
      <el-col>
        <el-card class="converter-card" shadow="never">
          <div slot="header" class="card-heading">
            <div>
              <h2>订阅转换</h2>
              <p>填写订阅信息，生成适用于不同客户端的配置链接</p>
            </div>
            <span class="version-badge">{{ backendVersion || 'Backend' }}</span>
          </div>
          <el-container>
            <el-form class="converter-form" :model="form" label-width="140px" label-position="left">
              <el-form-item label="模式设置:">
                <el-radio v-model="advanced" label="1">基础模式</el-radio>
                <el-radio v-model="advanced" label="2">进阶模式</el-radio>
              </el-form-item>
              <el-form-item label="订阅链接:">
                <el-input v-model="form.sourceSubUrl" type="textarea" rows="3"
                  placeholder="支持订阅或ss/ssr/vmess链接，多个链接每行一个或用 | 分隔" @blur="saveSubUrl" />
              </el-form-item>
              <el-form-item label="客户端:">
                <el-select v-model="form.clientType" style="width: 100%">
                  <el-option v-for="(v, k) in options.clientTypes" :key="k" :label="k" :value="v"></el-option>
                </el-select>
              </el-form-item>

              <el-form-item label="节点过滤:">
                <el-select v-model="nodeRegion" style="width: 100%">
                  <el-option v-for="item in regionOptions" :key="item.value"
                    :label="item.label" :value="item.value"></el-option>
                </el-select>
              </el-form-item>

              <div v-if="advanced === '2'">
                <el-form-item label="Backend 定制:">
                  <el-input style="width: 100%" :value="currentBackend" disabled>
                    <el-button slot="append" icon="el-icon-lock">已锁定</el-button>
                  </el-input>
                </el-form-item>
                <el-form-item label="远程配置:">
                  <div class="input-action-row">
                    <el-select v-model="form.remoteConfig" allow-create filterable placeholder="请选择远程配置">
                      <el-option-group v-for="group in options.remoteConfig" :key="group.label" :label="group.label">
                        <el-option v-for="item in group.options" :key="item.value" :label="item.label"
                          :value="item.value"></el-option>
                      </el-option-group>
                    </el-select>
                    <el-button @click="gotoRemoteConfig">配置示例</el-button>
                  </div>
                </el-form-item>
                <el-form-item label="过滤指定节点:">
                  <el-input v-model="form.includeRemarks" placeholder="节点名包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="移除指定节点:">
                  <el-input v-model="form.excludeRemarks" placeholder="节点名不包含的关键字，支持正则" />
                </el-form-item>
                <el-form-item label="订阅文件名:">
                  <el-input v-model="form.filename" placeholder="返回的订阅文件名" />
                </el-form-item>

                <el-form-item v-for="(param, i) in customParams" :key="i">
                  <el-input slot="label" v-model="param.name" placeholder="自定义参数名">
                    <div slot="suffix" style="width: 10px;">:</div>
                  </el-input>
                  <el-input v-model="param.value" placeholder="自定义参数内容">
                      <el-button slot="suffix" type="text" icon="el-icon-delete" style="margin-right: 5px" @click="customParams.splice(i, 1)"/>
                  </el-input>
                </el-form-item>

                <el-form-item label="输出格式:">
                  <el-checkbox v-model="form.nodeList">输出为 Node List</el-checkbox>
                </el-form-item>

                <section class="option-panel">
                  <h4>基础选项</h4>
                  <div class="option-grid">
                    <el-checkbox v-model="form.emoji">Emoji</el-checkbox>
                    <el-checkbox v-model="form.scv">跳过证书验证</el-checkbox>
                    <el-checkbox v-model="form.udp" @change="needUdp = true">启用 UDP</el-checkbox>
                    <el-checkbox v-model="form.appendType">节点类型</el-checkbox>
                    <el-checkbox v-model="form.sort">排序节点</el-checkbox>
                    <el-checkbox v-model="form.fdn">过滤非法节点</el-checkbox>
                    <el-checkbox v-model="form.expand">规则展开</el-checkbox>
                  </div>
                </section>

                <section class="option-panel">
                  <h4>定制功能</h4>
                  <div class="option-grid option-grid--custom">
                    <el-checkbox v-model="form.tpl.surge.doh">Surge.DoH</el-checkbox>
                    <el-checkbox v-model="form.tpl.clash.doh">Clash.DoH</el-checkbox>
                    <el-checkbox v-model="form.insert">网易云</el-checkbox>
                    <el-button size="small" icon="el-icon-plus" @click="addCustomParam">添加参数</el-button>
                    <el-link type="primary" :href="subDocAdvanced" target="_blank">文档</el-link>
                  </div>
                </section>
              </div>

              <div style="margin-top: 50px"></div>

              <div class="result-heading">
                <span class="result-heading__icon"><i class="el-icon-magic-stick"></i></span>
                <div>
                  <h3>生成结果</h3>
                  <p>生成的定制订阅链接和官方短链接</p>
                </div>
              </div>

              <el-form-item label="定制订阅:">
                <el-input class="copy-content" disabled v-model="customSubUrl">
                  <el-button slot="append" v-clipboard:copy="customSubUrl" v-clipboard:success="onCopy" ref="copy-btn"
                    icon="el-icon-document-copy">复制</el-button>
                </el-input>
              </el-form-item>
              <el-form-item label="订阅短链:">
                <el-input class="copy-content" disabled v-model="curtomShortSubUrl">
                  <el-button slot="append" v-clipboard:copy="curtomShortSubUrl" v-clipboard:success="onCopy"
                    ref="copy-btn" icon="el-icon-document-copy">复制</el-button>
                </el-input>
              </el-form-item>

              <!-- 操作按钮组 -->
              <el-form-item label-width="0px" style="margin-top: 40px; text-align: center">
                <el-button
                  :style="buttonStyle"
                  type="danger"
                  @click="makeUrlClick"
                  :disabled="!canGenerateUrl">
                  生成订阅链接
                </el-button>
                <el-button
                  :style="buttonStyle"
                  type="danger"
                  @click="makeShortUrlClick"
                  :loading="loading"
                  :disabled="!canGenerateShortUrl">
                  生成短链接
                </el-button>
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  :style="buttonStyle"
                  type="primary"
                  @click="dialogUploadConfigVisible = true"
                  icon="el-icon-upload"
                  :loading="loading">
                  上传配置
                </el-button>
                <el-button
                  :style="buttonStyle"
                  type="primary"
                  @click="clashInstall"
                  icon="el-icon-connection"
                  :disabled="!canImportClash">
                  一键导入 Clash
                </el-button>
              </el-form-item>

              <el-form-item label-width="0px" style="text-align: center">
                <el-button
                  :style="{ width: '290px' }"
                  type="primary"
                  @click="dialogLoadConfigVisible = true"
                  icon="el-icon-copy-document"
                  :loading="loading">
                  从 URL 解析
                </el-button>
              </el-form-item>
            </el-form>
          </el-container>
        </el-card>
      </el-col>
    </el-row>
    </main>

    <!-- 配置上传对话框 -->
    <ConfigUploadDialog
      :visible="dialogUploadConfigVisible"
      :upload-config="uploadConfig"
      :loading="loading"
      @cancel="handleUploadCancel"
      @confirm="handleConfigUpload"
    />

    <!-- URL解析对话框 -->
    <UrlParseDialog
      :visible="dialogLoadConfigVisible"
      :load-config="loadConfig"
      :loading="loading"
      @cancel="handleLoadCancel"
      @confirm="handleUrlParse"
    />
  </div>
</template>

<script>
// 导入配置
import { CONSTANTS } from '@/config/constants';
import { CLIENT_TYPES } from '@/config/client-types';
import { REMOTE_CONFIGS } from '@/config/remote-configs';

// 导入Composables
import { useSubscriptionForm, addCustomParam, saveSubUrl as saveSubscriptionUrl } from '@/composables/useSubscriptionForm';
import { useSubscription } from '@/composables/useSubscription';
import { useUrlParser } from '@/composables/useUrlParser';

// 导入工具函数
import { getLocalStorageItem } from '@/utils/storage';

// 导入服务
import { BackendService } from '@/services/backendService';
import { ShortUrlService } from '@/services/shortUrlService';
import { ConfigUploadService } from '@/services/configUploadService';

// 导入组件
import ConfigUploadDialog from '@/components/ConfigUploadDialog.vue';
import UrlParseDialog from '@/components/UrlParseDialog.vue';

export default {
  name: 'Subconverter',
  components: {
    ConfigUploadDialog,
    UrlParseDialog
  },
  data() {
    const subscriptionForm = useSubscriptionForm();

    return {
      // 配置选项
      options: {
        clientTypes: CLIENT_TYPES,
        backendOptions: [{ value: "http://127.0.0.1:25500/sub?" }],
        remoteConfig: REMOTE_CONFIGS
      },

      // 状态
      backendVersion: "",
      loading: false,
      curtomShortSubUrl: "",
      dialogUploadConfigVisible: false,
      loadConfig: "",
      dialogLoadConfigVisible: false,
      uploadConfig: "",
      subDocAdvanced: CONSTANTS.DOC_ADVANCED,
      nodeRegion: 'all',
      regionOptions: [
        { label: '全部节点', value: 'all' },
        { label: '亚洲节点', value: 'asia' },
        { label: '美洲节点', value: 'america' },
        { label: '欧洲节点', value: 'europe' },
        { label: '非洲节点', value: 'africa' },
        { label: '常用节点 (HK/TW/SG/JP/US)', value: 'popular' },
        { label: '游戏节点', value: 'game' }
      ],

      // 是否为 PC 端
      isPC: true,

      // 合并表单状态
      ...subscriptionForm
    };
  },
  computed: {
    // 按钮统一样式
    buttonStyle() {
      return { width: '140px' };
    },

    canGenerateShortUrl() {
      return this.customSubUrl.length > 0 && !this.loading;
    },

    canGenerateUrl() {
      return this.form.sourceSubUrl.length > 0 && this.form.clientType;
    },

    canImportClash() {
      return this.customSubUrl.length > 0;
    },

    processedSubUrl() {
      return this.form.sourceSubUrl.replace(/(\n|\r|\n\r)/g, "|");
    },

    currentBackend() {
      return CONSTANTS.DEFAULT_BACKEND;
    },

    conversionForm() {
      const filters = {
        all: '',
        asia: '香港|HK|台湾|TW|新加坡|SG|日本|JP|韩国|KR|印度|IN|泰国|TH',
        america: '美国|US|加拿大|CA|巴西|BR|阿根廷|AR|墨西哥|MX',
        europe: '英国|UK|GB|德国|DE|法国|FR|荷兰|NL|俄罗斯|RU|欧洲|EU',
        africa: '南非|ZA|埃及|EG|非洲|Africa',
        popular: '香港|HK|台湾|TW|新加坡|SG|日本|JP|美国|US',
        game: '游戏|Game|Gaming|低延迟'
      };
      const regionFilter = filters[this.nodeRegion];
      const manualFilter = this.form.includeRemarks;
      const includeRemarks = regionFilter && manualFilter
        ? `(?=.*(?:${regionFilter}))(?=.*(?:${manualFilter}))`
        : regionFilter || manualFilter;

      return { ...this.form, includeRemarks };
    }
  },
  created() {
    document.title = "STTlink 订阅转换";
    this.isPC = this.$getOS().isPc;

    // 获取 url cache
    if (import.meta.env.VITE_USE_STORAGE === 'true') {
      const cachedUrl = getLocalStorageItem('sourceSubUrl');
      if (cachedUrl) {
        this.form.sourceSubUrl = cachedUrl;
      }
    }
  },
  mounted() {
    this.form.clientType = CONSTANTS.DEFAULT_CLIENT_TYPE;
    this.getBackendVersion();
    
    // 延迟加载隐私提示，避免阻塞页面初始化
    this.notifyTimer = setTimeout(() => {
      this.notify();
    }, 1000);
  },
  beforeDestroy() {
    clearTimeout(this.notifyTimer);
  },
  methods: {
    onCopy() {
      this.$message.success("Copied!");
    },

    goToSttlink() {
      window.open('https://sttgo.cc/', '_blank');
    },

    gotoGayhub() {
      window.open(CONSTANTS.BACKEND_RELEASE);
    },

    gotoRemoteConfig() {
      window.open(CONSTANTS.REMOTE_CONFIG_SAMPLE);
    },

    clashInstall() {
      if (this.customSubUrl === "") {
        this.$message.error("请先填写必填项，生成订阅链接");
        return false;
      }

      const url = "clash://install-config?url=";
      window.open(
        url +
        encodeURIComponent(
          this.curtomShortSubUrl !== ""
            ? this.curtomShortSubUrl
            : this.customSubUrl
        )
      );
    },

    makeUrlClick() {
      const url = this.makeUrl(this.conversionForm, this.advanced, this.processedSubUrl, this.currentBackend, this.customParams, this.needUdp);
      if (url) {
        this.customSubUrl = url;
        this.$copyText(this.customSubUrl);
        this.$message.success("定制订阅已复制到剪贴板");
      } else {
        this.$message.error("订阅链接与客户端为必填项");
      }
    },

    makeShortUrlClick() {
      if (this.customSubUrl === "") {
        this.$message.warning("请先生成订阅链接，再获取对应短链接");
        return false;
      }

      this.loading = true;

      ShortUrlService.generateShortUrl(this.$axios, this.customSubUrl)
        .then(shortUrl => {
          this.curtomShortSubUrl = shortUrl;
          this.$copyText(shortUrl);
          this.$message.success("短链接已复制到剪贴板");
        })
        .catch(error => {
          this.$message.error("短链接获取失败：" + error.message);
        })
        .finally(() => {
          this.loading = false;
        });
    },

    confirmUploadConfig() {
      if (this.uploadConfig === "") {
        this.$message.warning("远程配置不能为空");
        return false;
      }

      this.loading = true;

      ConfigUploadService.uploadConfig(this.$axios, this.uploadConfig)
        .then(res => {
          const result = ConfigUploadService.handleUploadSuccess(res, this.$copyText, this.$message);
          if (result.success) {
            // 自动填充至『表单-远程配置』
            this.form.remoteConfig = result.url;
            this.$copyText(this.form.remoteConfig);
            this.dialogUploadConfigVisible = false;
            this.uploadConfig = "";
          }
        })
        .catch(error => {
          this.$message.error("远程配置上传失败: " + error.message);
        })
        .finally(() => {
          this.loading = false;
        });
    },

    handleUploadCancel() {
      this.uploadConfig = "";
      this.dialogUploadConfigVisible = false;
    },

    handleConfigUpload(configContent) {
      this.uploadConfig = configContent;
      this.confirmUploadConfig();
    },

    handleLoadCancel() {
      this.loadConfig = "";
      this.dialogLoadConfigVisible = false;
    },

    handleUrlParse(url) {
      this.loadConfig = url;
      this.confirmLoadConfig();
    },

    confirmLoadConfig() {
      this.loading = true;

      this.parseUrl(
        this.loadConfig,
        this.form,
        this.customParams,
        () => {
          this.dialogLoadConfigVisible = false;
          this.loadConfig = "";
          this.$message.success("长/短链接已成功解析为订阅信息");
        },
        (error) => {
          this.$message.error(error);
        }
      ).then(() => {
        this.loading = false;
      }).catch(() => {
        this.loading = false;
      });
    },

    backendSearch(queryString, cb) {
      const results = this.backendSearchSuggestions(queryString, this.options.backendOptions);
      cb(results);
    },

    backendSearchSuggestions(queryString, backends) {
      if (queryString) {
        return backends.filter(backend => {
          return backend.value.toLowerCase().indexOf(queryString.toLowerCase()) === 0;
        });
      }
      return backends;
    },

    async getBackendVersion() {
      this.backendVersion = await BackendService.getBackendVersion(this.$axios);
    },

    notify() {
      const h = this.$createElement;

      this.$notify({
        title: "隐私提示",
        type: "warning",
        message: h(
          "i",
          { style: "color: teal" },
          "本服务由 STTlink 官方搭建，各种订阅链接（短链接服务除外）生成纯前端实现，无隐私问题，请放心使用。"
        )
      });
    },

    // 表单相关方法
    saveSubUrl() {
      saveSubscriptionUrl(this.form);
    },

    addCustomParam() {
      addCustomParam(this.customParams);
    },

    // 使用 composables
    ...useSubscription(),
    ...useUrlParser()
  }
};
</script>

<style scoped>
.stt-page {
  min-height: 100vh;
  background:
    radial-gradient(circle at 10% 10%, rgba(99, 102, 241, .12), transparent 28%),
    radial-gradient(circle at 90% 30%, rgba(14, 165, 233, .1), transparent 30%),
    #f5f7fb;
  color: #172033;
  padding-bottom: 56px;
}

.hero {
  position: relative;
  overflow: hidden;
  background: linear-gradient(125deg, #172554 0%, #3730a3 48%, #2563eb 100%);
  color: #fff;
  padding: 34px 24px 86px;
}

.hero::after {
  content: '';
  position: absolute;
  width: 360px;
  height: 360px;
  right: -90px;
  top: -230px;
  border: 1px solid rgba(255, 255, 255, .2);
  border-radius: 50%;
  box-shadow: 0 0 0 44px rgba(255, 255, 255, .04), 0 0 0 88px rgba(255, 255, 255, .03);
}

.hero__inner {
  position: relative;
  z-index: 1;
  max-width: 960px;
  margin: 0 auto;
  display: flex;
  align-items: center;
  gap: 16px;
}

.brand-mark {
  width: 52px;
  height: 52px;
  display: grid;
  place-items: center;
  border-radius: 15px;
  background: rgba(255, 255, 255, .16);
  border: 1px solid rgba(255, 255, 255, .26);
  box-shadow: inset 0 1px rgba(255, 255, 255, .2);
  overflow: hidden;
}

.brand-mark img {
  width: 100%;
  height: 100%;
  display: block;
  object-fit: cover;
}

.hero__copy h1,
.hero__copy p,
.card-heading h2,
.card-heading p,
.result-heading h3,
.result-heading p {
  margin: 0;
}

.hero__copy h1 {
  font-size: 30px;
  line-height: 1.15;
  letter-spacing: -.5px;
}

.hero__copy p {
  margin-top: 5px;
  color: rgba(255, 255, 255, .72);
  font-size: 14px;
}

.project-link {
  margin-left: auto;
  display: inline-flex;
  align-items: center;
  gap: 8px;
  padding: 9px 14px;
  color: #fff;
  background: rgba(255, 255, 255, .12);
  border: 1px solid rgba(255, 255, 255, .22);
  border-radius: 10px;
  cursor: pointer;
}

.project-link__logo {
  width: 24px;
  height: 24px;
  display: block;
  object-fit: cover;
  border-radius: 7px;
}

.page-shell {
  position: relative;
  z-index: 2;
  max-width: 960px;
  margin: -54px auto 0;
  padding: 0 20px;
}

.converter-card {
  border: 0;
  border-radius: 18px;
  box-shadow: 0 18px 50px rgba(30, 41, 59, .13);
}

.card-heading {
  display: flex;
  align-items: center;
  justify-content: space-between;
  gap: 20px;
}

.card-heading h2 {
  color: #172033;
  font-size: 21px;
}

.card-heading p,
.result-heading p {
  margin-top: 6px;
  color: #8490a5;
  font-size: 13px;
}

.version-badge {
  flex: 0 0 auto;
  padding: 6px 10px;
  color: #2563eb;
  background: #eff6ff;
  border: 1px solid #dbeafe;
  border-radius: 999px;
  font-size: 12px;
}

.converter-form {
  width: 100%;
}

.input-action-row {
  display: grid;
  grid-template-columns: minmax(0, 1fr) auto;
  gap: 10px;
}

.input-action-row .el-select {
  width: 100%;
}

.option-panel {
  margin: 20px 0 0 140px;
  padding: 18px 20px;
  background: #f8faff;
  border: 1px solid #e8edf6;
  border-radius: 12px;
}

.option-panel h4 {
  margin: 0 0 16px;
  color: #334155;
  font-size: 14px;
}

.option-grid {
  display: grid;
  grid-template-columns: repeat(4, minmax(0, 1fr));
  gap: 16px 12px;
}

.option-grid--custom {
  grid-template-columns: repeat(5, auto);
  align-items: center;
  justify-content: start;
}

::v-deep .option-grid .el-checkbox {
  margin-right: 0;
}

.result-heading {
  display: flex;
  align-items: center;
  gap: 12px;
  margin: 42px 0 24px;
  padding-top: 24px;
  border-top: 1px solid #edf0f5;
}

.result-heading__icon {
  width: 40px;
  height: 40px;
  display: grid;
  place-items: center;
  color: #fff;
  background: linear-gradient(135deg, #4f46e5, #2563eb);
  border-radius: 12px;
}

.result-heading h3 {
  font-size: 17px;
}

::v-deep .el-card__header {
  padding: 22px 28px;
  border-color: #edf0f5;
}

::v-deep .el-card__body {
  padding: 30px 28px 34px;
}

::v-deep .el-form-item__label {
  color: #3e4a5f;
  font-weight: 600;
}

::v-deep .el-input__inner,
::v-deep .el-textarea__inner {
  border-color: #dce2eb;
  border-radius: 9px;
  transition: border-color .2s, box-shadow .2s;
}

::v-deep .el-input__inner:focus,
::v-deep .el-textarea__inner:focus {
  border-color: #4f46e5;
  box-shadow: 0 0 0 3px rgba(79, 70, 229, .1);
}

::v-deep .el-button {
  border-radius: 9px;
}

::v-deep .el-button--danger {
  border-color: #4f46e5;
  background: linear-gradient(135deg, #4f46e5, #2563eb);
}

::v-deep .el-button--primary {
  border-color: #2563eb;
  background: #2563eb;
}

@media (max-width: 700px) {
  .hero {
    padding: 26px 18px 74px;
  }

  .hero__copy h1 {
    font-size: 24px;
  }

  .project-link span {
    display: none;
  }

  .page-shell {
    padding: 0 12px;
  }

  .card-heading {
    align-items: flex-start;
  }

  .version-badge {
    max-width: 120px;
    overflow: hidden;
    text-overflow: ellipsis;
    white-space: nowrap;
  }

  ::v-deep .el-card__header,
  ::v-deep .el-card__body {
    padding-left: 18px;
    padding-right: 18px;
  }

  ::v-deep .el-form-item__label {
    width: 100% !important;
    float: none;
    line-height: 28px;
  }

  ::v-deep .el-form-item__content {
    margin-left: 0 !important;
  }

  .option-panel {
    margin-left: 0;
  }

  .option-grid,
  .option-grid--custom {
    grid-template-columns: repeat(2, minmax(0, 1fr));
  }

  .input-action-row {
    grid-template-columns: 1fr;
  }

  ::v-deep .el-form-item[style*="text-align: center"] .el-button {
    width: 100% !important;
    margin: 0 0 10px;
  }
}
</style>
