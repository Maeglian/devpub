<template>
  <div :class="classObject">
    <BaseNavbar
      v-if="!daySelected && !tagSelected && !searchQuery"
      className="Articles-Nav"
      :navItems="navItems"
      :activeNavIndex="activeNavIndex"
      @set-nav-value="selectActiveNavIndex"
    />
    <div
      :class="[
        forModeration || myPosts
          ? 'Articles-Content Articles-Content--noborder'
          : 'Articles-Content'
      ]"
    >
      <div class="ServerInfo" v-if="articlesAreErrored">
        Sorry, some error happened :(
      </div>
      <template v-else>
        <div v-if="daySelected" class="Title Articles-Title">
          Публикации {{ formatedDate }}
        </div>
        <div v-if="tagSelected" class="Title Articles-Title">
          Публикации по тэгу #{{ tagSelected.toUpperCase() }}
        </div>
        <div v-if="searchQuery" class="Title Articles-Title">
          Поиск по "{{ searchQuery }}"
        </div>
        <BaseArticle
          v-for="item in articles"
          :key="item.id"
          :className="'Articles-ArticlePreview'"
          :isPreview="true"
          :forModeration="forModeration"
          :myPosts="myPosts"
          :id="item.id"
          :time="item.time"
          :author="item.user.name"
          :title="item.title"
          :text="item.announce"
          :likeCount="item.likeCount"
          :dislikeCount="item.dislikeCount"
          :commentCount="item.commentCount"
          :viewCount="item.viewCount"
          @moderated="onModerated"
        />
        <div v-if="moreArticles && !articlesAreLoading" class="Articles-Button">
          <BaseButton
            :className="'Button--mode_add-load'"
            :onClickButton="onLoadMore"
          >
            Ещё публикации ({{ moreArticles }})
          </BaseButton>
        </div>
      </template>
    </div>
  </div>
</template>

<script>
import { mapGetters, mapMutations, mapActions } from "vuex";
const BaseNavbar = () =>
  import(/* webpackChunkName: "baseNavbar" */ "@/components/BaseNavbar.vue");
const BaseArticle = () =>
  import(/* webpackChunkName: "baseArticle" */ "@/components/BaseArticle.vue");
const BaseButton = () =>
  import(/* webpackChunkName: "baseButton" */ "@/components/BaseButton.vue");

export default {
  components: {
    BaseNavbar,
    BaseArticle,
    BaseButton
  },

  props: {
    className: {
      type: String,
      required: false
    },
    navItems: {
      type: Array,
      required: true
    },
    forModeration: {
      type: Boolean,
      required: false,
      default: false
    },
    myPosts: {
      type: Boolean,
      required: false,
      default: false
    }
  },

  data() {
    return {
      activeNavIndex: 0,
      articlesNumber: 10,
      offset: 0
    };
  },

  computed: {
    ...mapGetters([
      "articles",
      "articlesCount",
      "articlesAreLoading",
      "articlesAreErrored"
    ]),

    classObject() {
      let str = "Articles";

      if (this.className) {
        str += ` ${this.className}`;
      }

      if (this.myPosts || this.forModeration) {
        str += " Wrapper";
      }

      return str;
    },

    tagSelected() {
      return this.$route.params.tag;
    },

    daySelected() {
      return this.$route.params.date;
    },

    searchQuery() {
      return this.$route.params.search;
    },

    moreArticles() {
      let dif = this.articlesCount - this.offset - this.articlesNumber;
      return dif > 0 ? dif : 0;
    },

    formatedDate() {
      if (this.daySelected) {
        return new Date(this.daySelected).toLocaleString("ru-RU", {
          year: "numeric",
          month: "2-digit",
          day: "2-digit"
        });
      }

      return false;
    }
  },

  watch: {
    $route() {
      this.activeNavIndex = this.navItems.findIndex(
        item => item.value === this.$route.params.pathMatch
      );
      this.clearArticles();
      this.offset = 0;
      this.selectMethod();
    }
  },

  methods: {
    ...mapMutations(["clearArticles", "clearSearchQuery", "clearSelectedDay"]),
    ...mapActions(["getArticles", "moderateArticle"]),

    selectActiveNavIndex(value) {
      this.activeNavIndex = value;
    },

    selectMethod() {
      let query;

      if (this.forModeration) query = this.makeQuery("status", "/moderation");
      else if (this.myPosts) query = this.makeQuery("status", "/my");
      else if (this.tagSelected) query = this.makeQuery("tag", "/byTag");
      else if (this.daySelected) query = this.makeQuery("date", "/byDate");
      else if (this.searchQuery) query = this.makeQuery("query", "/search");
      else query = this.makeQuery("mode");

      this.getArticles(query);
    },

    getValue() {
      if (this.tagSelected) return this.tagSelected;
      if (this.daySelected) return this.daySelected;
      if (this.searchQuery) return this.searchQuery;
      return this.navItems[this.activeNavIndex].value;
    },

    makeQuery(prop, url = "") {
      const value = this.getValue();
      return `${url}?offset=${this.offset}&limit=${this.articlesNumber}&${prop}=${value}`;
    },

    onLoadMore() {
      if (this.articlesCount > this.offset + this.articlesNumber) {
        this.offset += this.articlesNumber;
        this.selectMethod();
      }
    },

    onModerated(post) {
      const payload = {
        post_id: post.id,
        decision: post.status
      };

      this.moderateArticle(payload);
    }
  },

  mounted() {
    this.activeNavIndex = this.navItems.findIndex(
      item => item.value === this.$route.params.pathMatch
    );
    this.clearArticles();
    this.offset = 0;
    this.selectMethod();
  },

  metaInfo() {
    let value;

    if (this.tagSelected) value = `Публикации по тэгу #${this.tagSelected}`;
    else if (this.forModeration) value = "Модерирование публикаций";
    else if (this.myPosts) value = "Мои публикации";
    else if (this.postByDate) value = `Публикации за ${this.formatedDate}`;
    else if (this.searchQuery) value = `Искать "${this.searchQuery}"`;
    else return "DevPub - рассказы разработчиков";

    return {
      title: `${value} | DevPub - рассказы разработчиков`
    };
  }
};
</script>

<style lang="scss">
.Articles {
  .Nav {
    margin-bottom: 25px;

    @media (max-width: $screen-mobile) {
      display: grid;
      grid-template-columns: 1fr 1fr;
      justify-items: center;
      align-items: center;
      margin-bottom: 10px;
    }

    &-Item {
      @media (max-width: $screen-tablet) {
        margin-right: 10px;
      }

      @media (max-width: $screen-mobile) {
        margin-bottom: 5px;
      }
    }
  }

  &-Title {
    margin-top: 5px;
    margin-bottom: 40px;

    @media (max-width: $screen-tablet) {
      margin-top: 25px;
    }
  }

  &-Content {
    height: 100%;
    padding: 10px 7px 50px 0;
    border-right: 1px solid var(--color-layout-primary);

    @media (max-width: $screen-tablet) {
      padding-top: 0;
      padding-right: 0;
      border-right: none;
    }

    &--noborder {
      border: none;
    }
  }

  &-ArticlePreview {
    margin-bottom: 20px;
  }

  &-Button {
    text-align: center;
  }
}
</style>
