<template>
  <main class="MainPage Wrapper">
    <Articles
      :navItems="navItems"
      :className="'MainPage-Articles'"
      :tagSelected="tagSelected"
      :postByDate="postByDate"
    />
    <Tags className="MainPage-Tags" @select-tag="onClickTag" />
  </main>
</template>

<script>
import { mapGetters } from "vuex";
const Articles = () =>
  import(/* webpackChunkName: "articles" */ "@/components/TheArticles.vue");
const Tags = () =>
  import(/* webpackChunkName: "tags" */ "@/components/TheTags.vue");

export default {
  name: "mainPage",

  components: {
    Articles,
    Tags
  },

  data() {
    return {
      navItems: [
        {
          name: "Новые",
          value: "recent"
        },
        {
          name: "Самые обсуждаемые",
          value: "popular"
        },
        {
          name: "Лучшие",
          value: "best"
        },
        {
          name: "Старые",
          value: "early"
        }
      ],
      tagSelected: ""
    };
  },

  computed: {
    ...mapGetters(["blogInfo"]),

    postByDate() {
      return this.$route.params.date ? this.$route.params.date : "";
    }
  },

  methods: {
    onClickTag(value) {
      this.tagSelected = value;

      if (this.$route.path !== "/") this.$router.push("/");
    }
  },

  mounted() {
    if (this.$route.params.tag) this.tagSelected = this.$route.params.tag;
  },

  metaInfo() {
    return {
      title: this.blogInfo
        ? `${this.blogInfo.title} - ${this.blogInfo.subtitle}`
        : ""
    };
  }
};
</script>

<style lang="scss">
.MainPage {
  display: grid;
  grid-template-columns: 7fr 3fr;
  grid-template-rows: repeat(auto-fit, 100%);
  flex-grow: 1;

  @media (max-width: $screen-tablet) {
    grid-template-columns: 1fr;
  }

  &-Tags {
    @media (max-width: $screen-tablet) {
      display: none;
    }
  }
}

.MainPage.Wrapper {
  margin-bottom: 0;
}
</style>
