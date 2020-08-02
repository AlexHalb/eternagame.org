<template>
  <div class="custom-input-group">
    <input
      type="text"
      :placeholder="placeholder || $t('search:search')"
      class="local-search"
      v-model="searchValue"
      @input="onSearch"
      ref="puzzleSearch"
    />
    <input
      type="text"
      placeholder="Author"
      class="local-search"
      v-model="authorValue"
      @input="onSearch"
      ref="authorSearch"
      v-if="showAuthor"
    >
    <b-form-select :options="engineOptions" v-model="folder" @input="onSearch"/>
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
  
    @Prop({ default: false }) showEngineDropdown!: boolean;

    private search: string = '';

    searchValue = this.extractSearch(this.search || this.$route.query.search as string);

    extractSearch(str: string) {
      return str
        ?.replace(/:\S+$/, '')
        .replace(/^\[.+\]/, '')
        .trimStart();
    }

    authorValue = this.extractAuthor(this.search || this.$route.query.search as string);

    extractAuthor(str: string) { // Gets the author from the search
      return str && str.match(/:\S+$/) && str.match(/:\S+$/)![0]
      ? str.match(/:\S+$/)![0].replace(/^:/, '')
      : '';
    }

    folder: string | null = this.extractEngine(this.search || this.$route.query.search as string);

    extractEngine(str: string) { // Gets the engine from the search
      return str && str.match(/\[.+]/) && str.match(/\[.+\]/)![0] || null;
    }

    replaceRoute() {
      // Add the player puzzles filter if not already (fixes bug where searching by player only displays results if player puzzles are checked)
      if (this.authorValue.trim() && this.showAuthor) {
        if (!this.$route.query.filters) this.$route.query.filters = 'player';
        else if (!(this.$route.query.filters as string).includes('player')) this.$route.query.filters += ',player';
      } else if (this.showAuthor) {
        this.$route.query.filters = (this.$route.query.filters as string) || ''
          .split(',')
          .filter(e => e !== 'player')
          .join(',');
      }

      const searchFolder = this.folder ? `[${this.folder}]`.replace(/\[{2,}/, '').replace(/\]{2,}/, '') : '';
      const searchAuthor = this.authorValue && this.showAuthor? `:${this.authorValue }`: '';
      this.$router.replace({
        name: this.$route.name!,
        query: {
          ...this.$route.query,
          search: `${searchFolder}${this.searchValue}${searchAuthor}`,
        },
      });
    }

    onSearch = debounce(this.replaceRoute, 200);

    engineOptions = [
      { 'text': 'No engine selected', value: null},
      { 'text': 'Vienna', value: 'VRNA_1'},
      { 'text': 'Vienna2', value: 'VRNA_2'},
      { 'text': 'NuPACK', value: 'NuPACK'},
      { 'text': 'LinearFoldV', value: 'LFV'},
      { 'text': 'LinearFoldC', value: 'LFC'},
      { 'text': 'Contrafold', value: 'CONTRA'},
      { 'text': 'LinearFoldE', value: 'LFE'},
      { 'text': 'EternaFold', value: 'EFOLD'},
    ];
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
