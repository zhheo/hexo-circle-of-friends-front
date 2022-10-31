<template>
  <div class="cf-article-group">
    <div v-for="(data, index) in (all_data['article_data'].slice(0, current_arcitle_num))" :key="index"
      class="cf-article-item">
      <div class="cf-article">
        <a class="cf-article-title" :href="data.link" target="_blank" rel="noopener nofollow"
          :data-title="data.title">{{ data.title }}</a>
        <div class="cf-article-avatar no-lightbox flink-item-icon">
          <!-- 添加no-lightbox解决渲染问题 -->
          <img class="cf-img-avatar avatar no-lightbox" :src="data.avatar" alt="avatar" @error="loadDefaultImg($event)">
          <span class="cf-article-author" @click="open_article_card(data.link)">{{ data.author }}</span>
          <span class="cf-article-time">
            <span v-if="Config.sort_rule === 'created'" class="cf-time-created">
              <i class="far fa-calendar-alt"></i>{{ data.created }}
            </span>
            <span v-if="Config.sort_rule === 'updated'" class="cf-time-updated">
              <i class="fas fa-history">更新于</i>{{ data.updated }}
            </span>
          </span>
        </div>
      </div>
    </div>
    <div id="cf-footer">
      <div id="cf-more" class="cf-new-add" @click="loadMoreArticle()">
        <small v-if="is_ended">一切皆有尽头！</small>
        <i v-else class="fas fa-angle-double-down"></i>
      </div>
      <div id="cf-footer" class="cf-new-add">
        <span id="cf-version-up" onclick="checkVersion()"></span>
        <div>Powered by
          <a href="https://github.com/Rock-Candy-Tea/hexo-circle-of-friends" target="_blank">FriendCircle</a>
        </div>
        <div>Design by
          <a href="https://zhheo.com/" target="_blank">Heo</a>
        </div>
        <div id="cf-footer-info">
          <div class="cf-data-friends">
            <span class="cf-label">订阅</span>
            <span class="cf-message">{{ all_data.statistical_data.friends_num }}</span>
          </div>
          <div class="cf-data-active">
            <span class="cf-label">活跃</span>
            <span class="cf-message">{{ all_data.statistical_data.active_num }}</span>
          </div>
          <div class="cf-data-article">
            <span class="cf-label">日志</span>
            <span class="cf-message">{{ all_data.statistical_data.article_num }}</span>
          </div>
        </div>
        <div style="display: flex;">
          <span class="cf-data-lastupdated">更新于：{{ all_data['statistical_data']['last_updated_time'] }}</span>
          <span class="cf-setting-btn" style="margin-left: 8px;" @click="open_manage_panel()">设置</span>
        </div>
      </div>
      <div id="cf-overlay" class="cf-new-add" onclick="closeShow()"></div>
      <div id="cf-overshow" class="cf-new-add"></div>
    </div>
  </div>
</template>

<script>


export default {
  name: 'ArticleBody',
  emits: ['show_article_card','open_manage_panel'],
  data() {
    return {
      current_arcitle_num: this.all_data['statistical_data']['article_num'] > 20 ? 20 : this.all_data['statistical_data']['article_num'],
      is_ended: this.all_data['statistical_data']['article_num'] <= 20,
    }
  },
  methods: {
    loadMoreArticle() {
      this.current_arcitle_num += this.Config.page_turning_number;
      if (this.current_arcitle_num >= this.all_data['statistical_data']['article_num']) this.is_ended = true;
      console.log(this.is_ended)
    },
    loadDefaultImg(event) {
      // 当图片出错时，用默认图片替换
      event.target.setAttribute('src', this.Config.error_img)
    },
    open_article_card(link) {
      this.$emit('show_article_card', link);
    },
    // 打开管理面板
    open_manage_panel(){
      this.$emit('open_manage_panel')
    }
  },

  props: ['Config', 'all_data']
}
</script>

<style scoped>
/* 文章卡片样式开始 */
.cf-article-group {
  display: flex;
  flex-direction: row;
  flex-wrap: wrap;
  justify-content: flex-start;
}

.cf-article-item {
  width: calc(100% / 4);
}

@media screen and (max-width: 1200px) {
  .cf-article-item {
  width: calc(100% / 3);
  }
}

@media screen and (max-width: 768px) {
  .cf-article-item {
  width: calc(100% / 2);
  }
}

@media screen and (max-width: 375px) {
  .cf-article-item {
  width: calc(100%);
  }
}

.cf-article {
  margin: 8px;
  border-radius: 8px;
  font-weight: bolder;
  overflow: hidden;
  transition: all ease-out .3s;
  position: relative;
  padding: 1rem;
  border: var(--style-border);
  height: 160px;
  display: flex;
  align-content: space-between;
  flex-direction: column;
  justify-content: space-between;
  box-shadow: var(--heo-shadow-border);
}

.cf-article:hover {
  transition: transform .3s;
}

.cf-article-avatar {
  line-height: 1;
  display: flex;
  flex-direction: row;
  align-items: center;
  justify-content: flex-start;
  position: relative;
  z-index: 1;
}

.cf-img-avatar {
  align-self: center;
  text-align: center;
  display: inline-block !important;
  position: absolute;
  width: 40%;
  border-radius: 50%;
  margin: 0 0 -4px !important;
  background: #fff;
  opacity: 0.1;
  z-index: 0;
  right: -20px;
  bottom: -20px;
  transition: 0.3s;
}

.cf-article-author {
  line-height: 1;
  font-size: 14px;
  font-weight: bold;
  align-self: center;
  text-align: center;
  cursor: pointer;
  white-space: nowrap;
  overflow: hidden;
  z-index: 1;
  color: var(--heo-fontcolor);
  font-size: .7rem;
  background-color: var(--heo-gray-op);
  padding: 8px;
  border-radius: 20px;
  display: flex;
  align-items: center;
  transition: 0.3s;
}

.cf-article-author:hover {
  background: var(--heo-main);
  color: var(--heo-white);
}

.cf-article-floor {
  position: absolute;
  top: 0;
  right: 0.5rem;
  font-style: italic;
  font-size: 3rem;
  line-height: 1.5rem;
  z-index: 1;
  font-weight: 400;
  transition: 0.3s;
}

.cf-article-title {
  font-weight: 500;
  position: relative;
  z-index: 2;
  width: 100%;
  display: block;
  letter-spacing: 1.5px;
  font-size: 18px;
  align-self: start;
  text-align: left;
  line-height: 40px;
  padding: 0;
  margin-bottom: 10px;
  transition: 0.3s;
  -webkit-line-clamp: 4;
  overflow: hidden;
  text-overflow: ellipsis;
  display: -webkit-box !important;
  -webkit-box-orient: vertical;
}

.cf-article-time {
  font-size: 12px;
  text-align: right;
  float: right;
  font-weight: 400;
  transition: 0.3s;
  margin-left: auto;
  z-index: 1;
}

/* 文章卡片样式结束 */

/* 更多按钮样式开始 */
#cf-footer {
  width: 100%;
  margin-top: 0.5rem;
  display: flex;
  flex-direction: column;
  align-items: flex-end;
  color: var(--heo-secondtext);
}

#cf-more {
  width: 40%;
  max-width: 810px;
  height: 30px;
  margin: auto;
  border-radius: 12px;
  font-weight: bolder;
  text-align: center;
  display: flex;
  flex-direction: column;
  justify-content: space-around;
  cursor: pointer;
  transition: 0.3s;
  border: var(--style-border);
  box-shadow: var(--heo-shadow-border);
}

#cf-more:hover {
  width: 60%;
  background: var(--heo-main);
  color: var(--heo-white);
}

#cf-more i.fas::before {
  content: "∞";
}

/* 更多按钮样式结束 */
</style>
