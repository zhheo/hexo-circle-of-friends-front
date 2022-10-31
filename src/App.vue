<template>
  <div>
    <ManagePanel
      v-if="manage_panel_open"
      :Config="Config"
      @close_manage_panel="close_manage_panel"
    ></ManagePanel>
    <ArticleCard
      v-if="article_card_data.open"
      :article_card_data="article_card_data.data"
      :Config="Config"
      @close_article_card="close_article_card"
    ></ArticleCard>
    <div v-if="change_map[Config.sort_rule]">
      <div id="cf-container">
        <Header
          :Config="Config"
          :all_data="change_map[Config.sort_rule]"
          @watch_sort_rule="change_sort_rule"
          @show_article_card="open_article_card"
          @toggle_api_url="toggle_api_url"
          @open_manage_panel="open_manage_panel"
        >
        </Header>
        <ArticleBody
          :Config="Config"
          :all_data="change_map[Config.sort_rule]"
          @show_article_card="open_article_card"
          @open_manage_panel="open_manage_panel"
        ></ArticleBody>
      </div>
    </div>
    <span v-else>与主机通讯中……</span>
  </div>
</template>

<script>
import Header from "./components/Header.vue";
import ArticleBody from "./components/ArticleBody.vue";
import ArticleCard from "./components/ArticleCard.vue";
import ManagePanel from "./components/ManagePanel.vue";
import DefaultConfig from "./utils/Config";
import { ElMessage } from "element-plus";

function init_config(default_config) {
  if (typeof UserConfig !== "undefined") {
    for (let key in UserConfig) {
      if (default_config[key]) {
        default_config[key] = UserConfig[key];
      }
    }
  }
  return default_config;
}

export default {
  name: "App",
  data() {
    return {
      // 配置
      Config: DefaultConfig,
      // 当前api
      current_api: null,
      // 排序规则
      change_map: {
        updated: null,
        created: null,
      },
      // 文章卡片
      article_card_data: {
        open: false,
        data: null,
      },
      // 管理面板状态
      manage_panel_open: false,
    };
  },
  methods: {
    // 加载文章数据
    get_data(base_url) {
      // 本地加载
      let CreatedData = JSON.parse(
        sessionStorage.getItem(base_url + "CreatedData")
      );
      let UpdatedData = JSON.parse(
        sessionStorage.getItem(base_url + "UpdatedData")
      );
      this.change_map["created"] = CreatedData;
      this.change_map["updated"] = UpdatedData;
      if (CreatedData === null) {
        // 没有缓存，第一次加载created
        this.$axios
          .get(base_url + "all?rule=created")
          .then((response) => {
            sessionStorage.setItem(
              base_url + "CreatedData",
              JSON.stringify(response.data)
            );
            this.change_map["created"] = response.data;
          })
          .catch((error) => {
            ElMessage({
              message: error.message,
              type: "error",
            });
          });
      }
      if (UpdatedData === null) {
        // 没有缓存，第一次加载updated
        this.$axios
          .get(base_url + "all?rule=updated")
          .then((response) => {
            sessionStorage.setItem(
              base_url + "UpdatedData",
              JSON.stringify(response.data)
            );
            this.change_map["updated"] = response.data;
          })
          .catch((error) => {
            ElMessage({
              message: error.message,
              type: "error",
            });
          });
      }
    },
    // 切换排序规则
    change_sort_rule(rule) {
      // 监听事件，修改排序规则rule
      this.Config.sort_rule = rule;
    },
    // 打开文章卡片
    open_article_card(link) {
      let url;
      // 监听事件，获取link对应的文章卡片展示
      // 选择私有库还是公共库api
      let current_base_api =
        this.current_api === "private"
          ? this.Config.private_api_url
          : this.Config.public_api_url;
      if (link !== "") {
        url = current_base_api + "post?num=5&link=" + link;
      } else {
        url = current_base_api + "post?num=5";
      }
      this.$axios.get(url).then((response) => {
        if ("statistical_data" in response.data) {
          this.article_card_data.data = response.data;
          this.article_card_data.open = true;
        } else {
          ElMessage({
            message:
              "未获取到文章卡片òᆺó\n如果持续出现此错误，检查数据库是否正常",
            type: "error",
          });
        }
      });
    },
    // 关闭文章卡片
    close_article_card() {
      this.article_card_data.open = false;
    },
    toggle_api_url() {
      if (this.current_api === "private") {
        this.current_api = "public";
        this.get_data(this.Config.public_api_url);
      } else {
        this.current_api = "private";
        this.get_data(this.Config.private_api_url);
      }
    },
    // 打开管理面板
    open_manage_panel() {
      this.manage_panel_open = true;
    },
    // 关闭管理面板
    close_manage_panel() {
      this.manage_panel_open = false;
    },
  },
  created() {
    this.Config = init_config(this.Config);
    this.current_api = "private";
    this.get_data(this.Config.private_api_url);
  },
  components: {
    Header,
    ArticleBody,
    ArticleCard,
    ManagePanel,
  },
};
</script>

<style>

:root{
  --heo-white: #fff;
  --heo-white-op: rgba(255,255,255,0.2);
  --heo-black: #000;
  --heo-black-op: rgba(0,0,0,0.2);
  --heo-none: #00000000;
  --heo-gray: #999999;
  --heo-gray-op: #9999992b;
  --heo-vip: #e5a80d;
  --heo-main: var(--heo-theme);
  --heo-main-op: var(--heo-theme-op);
  --heo-main-op-deep: var(--heo-theme-op-deep);
  --heo-main-none: var(--heo-theme-none);
  --heo-shadow-theme: 0 8px 12px -3px var(--heo-theme-op);
  --heo-shadow-blackdeep: 0 2px 16px -3px rgba(0, 0, 0,.15);
  --heo-shadow-main: 0 8px 12px -3px var(--heo-main-op);
  --heo-shadow-blue: 0 8px 12px -3px rgba(40, 109, 234,.20);
  --heo-shadow-white: 0 8px 12px -3px rgba(255, 255, 255,.20);
  --heo-shadow-black: 0 0 12px 4px rgba(0, 0, 0,.05);
  --heo-shadow-yellow: 0px 38px 77px -26px rgba(255, 201, 62,.12);
  --heo-shadow-red: 0 8px 12px -3px #ee7d7936;
  --heo-shadow-green: 0 8px 12px -3px #87ee7936;
  --heo-logo-color: linear-gradient(215deg,#4584ff 0%,#cf0db9 100%);
  --heo-snackbar-time: 5s;
  --style-border: 1px solid var(--heo-card-border);
  --style-border-always: 1px solid var(--heo-card-border);
  --style-border-hover: 1px solid var(--heo-main);
  --style-border-hover-always: 1px solid var(--heo-main);
  --style-border-dashed: 1px dashed var(--heo-theme-op);

  /* 默认 */
  --heo-theme: #425AEF;
  --heo-theme-op: #4259ef23;
  --heo-theme-op-deep: #4259efdd;
  --heo-theme-none: #4259ef01;
  --heo-blue: #425AEF;
  --heo-red: #D8213C;
  --heo-pink: #FF7C7C;
  --heo-green: #28a63f;
  --heo-yellow: #c28b00;
  --heo-yellow-op: #d99c001a;
  --heo-orange: #e38100;
  --heo-fontcolor: #363636;
  --heo-background: #f7f9fe;
  --heo-reverse: #000;
  --heo-maskbg: rgba(255, 255, 255, 0.6);
  --heo-maskbgdeep: rgba(255, 255, 255, 0.85);
  --heo-hovertext: var(--heo-main);
  --heo-ahoverbg: #F7F7FA;
  --heo-lighttext: var(--heo-main);
  --heo-secondtext: rgba(60, 60, 67, 0.6);
  --heo-scrollbar: rgba(60, 60, 67, 0.4);
  --heo-card-btn-bg: #edf0f7;
  --heo-post-blockquote-bg: #fafcff;
  --heo-post-tabs-bg: #f2f5f8;
  --heo-secondbg: #f1f3f8;
  --heo-shadow-nav:0 5px 12px -5px rgba(102, 68, 68, 0.05);
  --heo-card-bg: #fff;
  --heo-card-bg-op: var(--heo-black-op);
  --heo-card-bg-none: rgba(255, 255, 255, 0);
  --heo-shadow-lightblack:0 5px 12px -5px rgba(102, 68, 68, 0.00);
  --heo-shadow-light2black:0 5px 12px -5px rgba(102, 68, 68, 0.00);
  --heo-card-border: #e3e8f7;
  --heo-shadow-border: 0 8px 16px -4px #2c2d300c;
  --style-border-forever: 2px solid var(--heo-main);

}

::selection {
  background: var(--heo-fontcolor);
  color: var(--heo-background);
}

[data-theme=light] {
  --heo-theme: #425AEF;
  --heo-theme-op: #4259ef23;
  --heo-theme-op-deep: #4259efdd;
  --heo-theme-none: #4259ef01;
  --heo-blue: #425AEF;
  --heo-red: #D8213C;
  --heo-pink: #FF7C7C;
  --heo-green: #28a63f;
  --heo-yellow: #c28b00;
  --heo-yellow-op: #d99c001a;
  --heo-orange: #e38100;
  --heo-fontcolor: #363636;
  --heo-background: #f7f9fe;
  --heo-reverse: #000;
  --heo-maskbg: rgba(255, 255, 255, 0.6);
  --heo-maskbgdeep: rgba(255, 255, 255, 0.85);
  --heo-hovertext: var(--heo-main);
  --heo-ahoverbg: #F7F7FA;
  --heo-lighttext: var(--heo-main);
  --heo-secondtext: rgba(60, 60, 67, 0.6);
  --heo-scrollbar: rgba(60, 60, 67, 0.4);
  --heo-card-btn-bg: #edf0f7;
  --heo-post-blockquote-bg: #fafcff;
  --heo-post-tabs-bg: #f2f5f8;
  --heo-secondbg: #f1f3f8;
  --heo-shadow-nav:0 5px 12px -5px rgba(102, 68, 68, 0.05);
  --heo-card-bg: #fff;
  --heo-card-bg-op: var(--heo-black-op);
  --heo-card-bg-none: rgba(255, 255, 255, 0);
  --heo-shadow-lightblack:0 5px 12px -5px rgba(102, 68, 68, 0.00);
  --heo-shadow-light2black:0 5px 12px -5px rgba(102, 68, 68, 0.00);
  --heo-card-border: #e3e8f7;
  --heo-shadow-border: 0 8px 16px -4px #2c2d300c;
  --style-border-forever: 2px solid var(--heo-main);
}

[data-theme=dark] {
  --heo-theme: #0084FF;
  --heo-theme-op: #0084FF23;
  --heo-theme-op-deep: #0084ffdd;
  --heo-theme-none: #0084FF00;
  --heo-blue: #0084FF;
  --heo-red: #FF3842;
  --heo-pink: #FF7C7C;
  --heo-green: #57bd6a;
  --heo-yellow: #ffc93e;
  --heo-yellow-op: #ffc93e30;
  --heo-orange: #ff953e;
  --heo-fontcolor: #F7F7FA;
  --heo-background: #18171d;
  --heo-reverse: #fff;
  --heo-maskbg: rgba(0,0,0, 0.6);
  --heo-maskbgdeep: rgba(0,0,0, 0.85);
  --heo-hovertext: #0A84FF;
  --heo-ahoverbg: #fff;
  --heo-lighttext: #f2b94b;
  --heo-secondtext: #a1a2b8;
  --heo-scrollbar: rgba(200, 200, 223, 0.4);
  --heo-card-btn-bg: #30343f;
  --heo-post-blockquote-bg: #000;
  --heo-post-tabs-bg: #121212;
  --heo-secondbg: #30343f;
  --heo-shadow-nav:0 5px 20px 0px rgba(28, 28, 28, 0.4);
  --heo-card-bg: #1d1e22;
  --heo-card-bg-op: var(--heo-white-op);
  --heo-card-bg-none: #1d1b2600;
  --heo-shadow-lightblack:0 5px 12px -5px rgba(102, 68, 68, 0.0);
  --heo-shadow-light2black:0 5px 12px -5px rgba(102, 68, 68, 0.0);
  --heo-card-border: #282829;
  --heo-shadow-border: 0 8px 16px -4px #00000050;
  --style-border-forever: 2px solid var(--heo-lighttext);
}

/* 排序按钮 */
#cf-change {
  font-size: 14px;
  display: block;
  padding: 12px 0 4px;
  width: 100%;
  text-align: center;
}

/* 主容器 */
#cf-container {
  width: 100%;
  height: auto;
  margin: auto;
  background: var(--heo-background);
  max-width: 1500px;
  border-radius: 12px;
}

#cf-container a {
  text-decoration: none;
  line-height: 1.3;
  color: var(--heo-fontcolor);
  transition: 0.3s;
}

#cf-container a:hover {
  color: var(--heo-main);
}

.cf-time-updated,
.cf-time-created {
  display: inline-block;
  text-align: left;
  white-space: nowrap;
}

.cf-time-updated i.fas,
.cf-time-created i.far {
  padding-right: 8px;
}

/* 底部 */
#cf-footer {
  margin-right: 8px;
  margin-bottom: 8px;
  text-align: right;
  font-size: 13px;
}

.cf-data-lastupdated {
  font-size: 13px;
  text-align: right;
  display: block;
}

#cf-footer-info {
  display: flex;
}

#cf-footer-info div {
  margin-left: 0.5rem;
}

/* 颜色 */
.cf-article .cf-article-title:hover {
  color: var(--heo-main) !important;
}

#cf-state,
#cf-more {
  background: var(--heo-card-bg);
  color: var(--heo-fontcolor);
}

#cf-change,
.cf-time-updated,
.cf-time-created,
.cf-article-floor {
  color: var(--heo-secondtext);
}

.cf-article-author,
.cf-article a.cf-article-title,
.cf-article:hover .cf-article-floor {
  color: var(--heo-fontcolor);
}

.cf-article {
  background: var(--heo-card-bg);
}

#cf-change span:hover {
  color: var(--heo-main);
  cursor: pointer;
}

.cf-overshow p a:hover {
  color: var(--heo-main) !important;
}

.cf-overshow p span {
  color: var(--heo-secondtext);
}

.cf-setting-btn {
  color: var(--heo-fontcolor);
  cursor: pointer;
  transition: 0.3s;
}

.cf-setting-btn:hover {
  color: var(--heo-main);
}

/* 暗色主题 */
.dark-theme #cf-overlay,
.theme-dark #cf-overlay {
  background-color: rgba(59, 61, 66, 0.42);
}

.dark-theme .cf-overshow,
.theme-dark .cf-overshow {
  background: #292a2d;
}

.dark-theme .cf-overshow p a,
.theme-dark .cf-overshow p a {
  color: var(--heo-fontcolor);
}

.dark-theme .cf-overshow .cf-overshow-content,
.theme-dark .cf-overshow .cf-overshow-content {
  background: #eaeaea;
}

.dark-theme #cf-state,
.dark-theme #cf-more,
.theme-dark #cf-state,
.theme-dark #cf-more {
  background: var(--lmm-dack-background);
  color: var(--lmm-dark-fontcolor);
}

.dark-theme #cf-change,
.dark-theme .cf-time-updated,
.dark-theme .cf-time-created,
.dark-theme .cf-article-floor,
.theme-dark #cf-change,
.theme-dark .cf-time-updated,
.theme-dark .cf-time-created,
.theme-dark .cf-article-floor {
  color: var(--lmm-dark-floorcolor);
}

.dark-theme .cf-article-author,
.dark-theme .cf-article a.cf-article-title,
.theme-dark .cf-article-author,
.theme-dark .cf-article a.cf-article-title {
  color: var(--lmm-dark-fontcolor);
}

.dark-theme .cf-article,
.theme-dark .cf-article {
  background: var(--lmm-dack-background);
}

.dark-theme .cf-article:hover .cf-article-floor,
.dark-theme .cf-article:hover .cf-time-created,
.dark-theme .cf-article:hover .cf-time-updated,
.dark-theme .cf-overshow p span,
.theme-dark .cf-article:hover .cf-article-floor,
.theme-dark .cf-article:hover .cf-time-created,
.theme-dark .cf-article:hover .cf-time-updated,
.theme-dark .cf-overshow p span {
  color: var(--lmm-dark-fontcolor);
}

/* 移动端适配 */
@media screen and (max-width: 400px) {
  #cf-state {
    font-size: 14px;
  }

  .cf-article-time i {
    display: none;
  }
}

@media screen and (max-width: 300px) {
  #cf-state,
  .cf-article-time {
    display: none;
  }
}
</style>
