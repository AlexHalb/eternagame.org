<template>
  <div class="custom-input-group">
    <input
      type="text"
      :placeholder="placeholder || $t('search:search')"
      class="local-search"
      :value="searchValue"
      @input="onSearch"
      ref="puzzleSearch"
    />
    <input
      type="text"
      placeholder="Author"
      class="local-search"
      :value="authorValue"
      @input="onSearch"
      ref="authorSearch"
      v-if="showAuthor"
    >
    <span id="puzzle-search-icon">
      <img src="@/assets/sidebar/search.svg" />
    </span>
    <span id="author-search-icon" v-if="showAuthor">
      <img src="@/assets/people.svg" />
    </span>
  </div>
</template>

<script lang="ts">
  import { Component, Prop, Vue, Ref } from 'vue-property-decorator';
  import { mixins } from 'vue-class-component';
  import SidebarPanel from '@/components/Sidebar/SidebarPanel.vue';
  import SidebarPanelMixin from '@/mixins/SidebarPanel';
  // @ts-ignore
  import debounce from 'lodash.debounce';

  import icon from '@/assets/Filter.svg';

  @Component({
    components: {
      SidebarPanel,
    },
  })
  export default class SearchPanel extends mixins(SidebarPanelMixin) {
    @Prop() readonly placeholder?: string;

    @Prop({ default: false }) showAuthor!: boolean;

    private search: string = '';

    get searchValue() {
      return this.extractSearch(this.search || this.$route.query.search as string);
    }

    extractSearch(str: string) {
      return str?.replace(/:+\S+$/, '').trimStart();
    }

    get authorValue() {
      return this.extractAuthor(this.search || this.$route.query.search as string);
    }

    extractAuthor(str: string) {
      return str && str.match(/:\S+$/) && str.match(/:\S+$/)![0]
      ? str.match(/:+\S+$/)![0].replace(/^:/, '')
      : '';
    }

    replaceRoute() {
      if (this.authorSearch.value.trim() && this.showAuthor) {
        if (!this.$route.query.filters) this.$route.query.filters = 'player';
        else if (!(this.$route.query.filters as string).includes('player')) this.$route.query.filters += ',player';
      } else if (this.showAuthor) {
        this.$route.query.filters = (this.$route.query.filters as string)
          .split(',')
          .filter(e => e !== 'player')
          .join(',');
      }
      this.$router.replace({
        name: this.$route.name!,
        query: {
          ...this.$route.query,
          search: `${this.puzzleSearch.value}${this.authorSearch.value && this.showAuthor
            ? `:${this.authorSearch.value }`: ''}`,
        },
      });
    }

    debouncedReplaceRoute = debounce(this.replaceRoute, 200);

    onSearch() {
      this.debouncedReplaceRoute();
    }

    @Ref() readonly puzzleSearch !: HTMLInputElement;

    @Ref() readonly authorSearch !: HTMLInputElement;
  }
</script>

<style scoped lang="scss">
  input {
    margin-bottom: 10px;
  }
  .custom-input-group {
    position: relative;
  }

  .custom-input-group {
    #puzzle-search-icon {
      position: absolute;
      top: 7.375px;
      right: 11.25px;
    }
    #author-search-icon {
      position: absolute;
      top: 51.750px;
      right: 11.25px;
    }
    #author-search-icon > img {
      height: 25.33px;
    }
  }
</style>
