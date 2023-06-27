<template>
  <header class="nav">
    <div class="nav_title"><Icon type="md-done-all" />论文降重助手</div>
  </header>

  <main class="main">
    <div class="level-selector">
      <label>降重等级</label>
      <RadioGroup v-model="selectedLevel" type="button">
        <Radio v-for="(item, idx) in levelList" :key="idx" :label="item.index">{{ item.label }}</Radio>
      </RadioGroup>
      <span>降重规则：{{ levelList[selectedLevel].text }}</span>
    </div>

    <div class="work_area">
      <label class="work_block">
        <textarea v-model="text" placeholder="输入要转换的文本，然后点击下方「转换」按钮进行降重"></textarea>
        <button class="textarea_btn" :disabled="!text || running" @click="run">
          <template v-if="!running">转换</template>
          <Spin v-else fix></Spin>
        </button>
      </label>

      <label class="work_block result">
        <textarea v-model="result" :placeholder="resultTips" disabled></textarea>
        <button class="textarea_btn" :disabled="!result" @click="copy">复制</button>
      </label>
    </div>
  </main>
</template>

<script>
import md5 from 'blueimp-md5';
import $ from 'jquery';

const LS_KEY = 'selectedLevel';

export default {
  name: 'App',
  data() {
    return {
      running: false, // 是否转换中
      percent: 0,
      selectedLevel: localStorage.getItem(LS_KEY) * 1, // 选中的去重等级
      levelList: [{ // 去重等级列表
        label: '初级',
        text: '中 > 英 > 德 > 中',
        index: 0,
        value: ['zh en', 'en de', 'de zh']
      }, {
        label: '中级',
        text: '中 > 英 > 德 > 日 > 葡萄牙 > 中',
        index: 1,
        value: ['zh en', 'en de', 'de jp', 'jp pt', 'pt zh']
      }, {
        label: '高级',
        text: '中 > 英 > 德 > 日 > 葡萄牙 > 意大利 > 波兰 > 保加利亚 > 爱沙尼亚 > 中',
        index: 2,
        value: ['zh en', 'en de', 'de jp', 'jp pt', 'pt it', 'it pl', 'pl bul', 'bul est', 'est zh']
      }],
      text: '', // 待转换文本
      result: '' // 转换结果
    };
  },
  computed: {
    resultTips() {
      return this.running ? `转换中${this.percent}%...` : '转换成功后，这里会显示降重后的内容，可点击下方「复制」按钮一键复制';
    }
  },
  watch: {
    selectedLevel(newVal) {
      localStorage.setItem(LS_KEY, newVal);
    }
  },
  created() {
    // https://api.fanyi.baidu.com/api/trans/product/apichoose 在百度翻译上获取appid和key
    this.appid = '20230625001723789';
    this.key = 'HxLzi31TATpShy19p17f';
  },
  methods: {
    run() {
      if (this.running) return;

      this.running = true;
      this.result = '';

      const level = this.levelList[this.selectedLevel].value;
      let res = this.text;
      (async () => {
        this.percent = 0;
        for (let i = 0, len = level.length; i < len; i++) {
          const lang = level[i].split(' ');
          res = await this._translate(res, lang[0], lang[1]);
          this.percent = Math.ceil((i + 1) * 100 / len);
        }

        this.result = res;
        this.running = false;
        this.$Notice.success({
          title: '转换成功'
        });
      })();
    },
    copy() {
      this.$Copy({
        text: this.result,
        showTip: false,
        success: () => {
          this.$Notice.success({
            title: '复制成功'
          });
        },
        error: () => {
          this.$Notice.error({
            title: '复制失败'
          });
        }
      });
    },
    _translate(q, from, to) { // 调百度翻译API来翻译文本
      const salt = Math.floor(Math.random() * 10000000000);
      return new Promise((resolve, reject) => {
        $.ajax({
          type: 'POST',
          url: 'https://fanyi-api.baidu.com/api/trans/vip/translate',
          data: {
            q,
            from,
            to,
            appid: this.appid,
            salt,
            sign: md5(this.appid + q + salt + this.key)
          },
          dataType: 'jsonp',
          headers: {
            'Content-Type': 'application/x-www-form-urlencoded',
          },
          success: res => {
            resolve(res.trans_result[0].dst);
          },
          error: () => {
            reject();
          }
        });
      });
    }
  },
};
</script>

<style lang="less">
#app {
  font-family: Roboto, arial, sans-serif;
  -webkit-font-smoothing: antialiased;
  -moz-osx-font-smoothing: grayscale;
  color: #5f6368;
  font-size: 14px;

  button.textarea_btn {
    position: absolute;
    bottom: 4px;
    right: 16px;
    width: 40px;
    height: 40px;
    line-height: 40px;
    border-radius: 50%;
    border: 0;
    color: rgb(95, 99, 104);
    background: transparent;
    font-size: 12px;
    cursor: pointer;
    transition: background-color .15s linear;

    .ivu-spin-fix .ivu-spin-main {
      transform: translate(-50%, -15px);
    }

    &:hover:not(:disabled) {
      background: rgba(0, 0, 0, .04);
    }

    &:active:not(:disabled) {
      background: rgba(0, 0, 0, .12);
    }

    &:disabled {
      color: #bbb;
      cursor: default;
    }
  }
}

.nav {
  border-bottom: 1px solid rgba(0, 0, 0, .12);
  padding: 8px;

  .nav_title {
    height: 48px;
    padding: 0 56px;
    font-size: 22px;
    color: rgb(26, 115, 232);
    display: flex;
    gap: 8px;
    align-items: center;
  }
}

.main {
  width: auto;
  max-width: 1280px;
  margin: 0 auto;
  padding: 0 48px;

  .level-selector {
    height: 68px;
    display: flex;
    align-items: center;
    gap: 8px;

    label {
      font-weight: 500;
    }
  }

  .work_area {
    display: flex;
    flex-flow: row nowrap;
    gap: 8px;

    .work_block {
      flex: 1;
      position: relative;
      height: 300px;
      border: 1px solid rgba(0, 0, 0, .12);
      border-radius: 8px;
      cursor: text;

      &.result {
        background: #f5f5f5;
        border: 1px solid #f5f5f5;
        cursor: default;
      }

      textarea {
        width: 100%;
        height: calc(100% - 20px);
        box-sizing: border-box;
        padding: 12px 16px;
        resize: none;
        outline: none;
        color: rgb(60, 64, 67);
        line-height: 1.5em;
        background: transparent;
        border: 0;
      }
    }
  }
}
</style>
