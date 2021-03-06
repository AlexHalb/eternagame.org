<template>
  <EternaPage>
    <div class="overview">
      <h2 class="overview-title">
        Eterna is a citizen science game that recruits players to design RNA molecules for
        biomedical research.
      </h2>
      <h3 class="overview-subtitle text-muted">
        Players and researchers have created original software programs to aid in complex RNA
        design, which are now available for licensing.
      </h3>
    </div>

    <Gallery :sm="12" :md="6">
      <SoftwareCard v-for="project in projects" :key="project.nid" :project="project" />
    </Gallery>
  </EternaPage>
</template>

<script lang="ts">
  import { Component, Prop, Vue, Mixins } from 'vue-property-decorator';
  import { RouteCallback, Route } from 'vue-router';
  import axios from 'axios';
  import DropdownSidebarPanel, { Option } from '@/components/Sidebar/DropdownSidebarPanel.vue';
  import EternaPage from '@/components/PageLayout/EternaPage.vue';
  // Assets need to be imported; can't hardcode '@/assets/...'
  import EternaBrainLogo from '@/assets/software/logo_eternabrain.svg';
  import EternaBrainThumbnail from '@/assets/software/thumb_eternabrain_paper.png';
  import NemoLogo from '@/assets/software/logo_nemo.svg';
  import NemoThumbnail from '@/assets/software/thumb_nemo_paper.png';
  import EternaBotLogo from '@/assets/software/logo_eternabot.svg';
  import EternaBotThumbnail from '@/assets/software/thumb_eternabot_paper.jpg';
  import SentRNALogo from '@/assets/software/logo_sentrna.svg';
  import SentRNAThumbnail from '@/assets/software/thumb_sentrna_paper.png';
  import RNAMakeLogo from '@/assets/software/logo_rnamake.svg';
  import RNAMakeThumbnail from '@/assets/software/thumb_rnamake_paper.png';

  import SoftwareCard from './SoftwareCard.vue';

  export interface SoftwareProject {
    name: string;
    nid: string;
    logo: string;
    description: string;
    thumbnail: string;
    paper_name: string;
    paper_url: string;
    paper_journal: string;
    paper_date: string;
    // HTML string for license. If present, we'll use the license approval flow.
    license_terms?: string;
    // Otherwise, we allow user to download the content immediately.
    download_tooltip?: string;
    download_url?: string;
    help_tooltip?: string;
    help_url?: string;
  }

  const ROUTE = '/get/?type=software_packages';

  @Component({
    components: {
      EternaPage,
      SoftwareCard,
    },
  })
  export default class SoftwareExplore extends Vue {
    projects: SoftwareProject[] = [];

    async mounted() {
      const response = await axios.get(ROUTE);
      this.projects = response.data.data;
    }
  }
</script>

<style scoped lang="scss">
  .overview {
    max-width: 820px;
    margin: 2rem;
    margin-bottom: 4rem;
  }

  .overview-title {
    font-size: 1.9rem;
    line-height: 2.5rem;
  }

  .overview-subtitle {
    font-size: 1.6rem;
    line-height: 2.2rem;
    font-weight: normal;
  }

  h2 {
    line-height: 4rem;
    font-size: 2rem;
    font-weight: bold;
  }
</style>
