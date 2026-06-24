<template>
    <div>
        <!-- COVERS & WATCH NOW BUTTON -->
        <div class="player-wrapper bg-dark text-center position-relative rounded-top overflow-hidden" style="min-height: 450px; background-size: cover; background-position: center;" :style="{ backgroundImage: `url(${backdrop})` }">
            <div class="watch-overlay d-flex flex-column justify-content-center align-items-center position-absolute w-100 h-100" style="background: rgba(0, 0, 0, 0.6); z-index: 10;">
                <h2 class="text-light mb-1 fw-bold">{{ item.name }}</h2>
                <p class="text-muted mb-4">Season {{ activeSeason }} - Episode {{ activeEpisode }}</p>
                
                <button @click="goToWatchPage" class="btn btn-danger btn-lg fw-bold px-5 py-3 rounded-pill shadow-lg d-flex align-items-center gap-2 text-white">
                    <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="currentColor" class="me-1"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                    WATCH NOW
                </button>
            </div>
        </div>

        <!-- DETAIL INFO SECTION -->
        <div class="container box-info mt-4">
            <div class="row justify-content-center">
                <div class="col-lg-11">
                    <div class="card">
                        <div class="card-body pt-5 p-lg-4">
                            <div class="row">
                                <div class="col-lg-3 d-none d-lg-block">
                                    <aside>
                                        <img :src="poster(item.poster_path)" :alt="item.name" class="img-fluid rounded mb-4">
                                        <div class="mb-3 d-flex justify-content-around">
                                            <div v-for="(item, index) in votes" :key="index" style="color: #f1c830">
                                                <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon></svg>
                                            </div>
                                            <div v-for="(item, index) in unvotes" :key="'un'+index">
                                                <svg viewBox="0 0 24 24" width="24" height="24" stroke="currentColor" stroke-width="2" fill="none" stroke-linecap="round" stroke-linejoin="round"><polygon points="12 2 15.09 8.26 22 9.27 17 14.14 18.18 21.02 12 17.77 5.82 21.02 7 14.14 2 9.27 8.91 8.26 12 2"></polygon></svg>
                                            </div>
                                        </div>
                                        
                                        <table class="table table-borderless">
                                            <tbody>
                                                <tr>
                                                    <td>{{ $t('Genres') }}</td>
                                                    <td class="text-muted small"><span v-for="(it, index) in item.genres" :key="index">{{ it.name }}, </span></td>
                                                </tr>
                                                <tr>
                                                    <td>{{ $t('Runtime') }}</td>
                                                    <td class="text-muted small">{{ this._.head(item.episode_run_time) || 0 }} min</td>
                                                </tr>
                                            </tbody>
                                        </table>
                                    </aside>
                                </div>
                                <div class="col-lg-9">
                                    <div class="d-flex justify-content-center justify-content-md-between align-items-center mb-4 flex-column-reverse flex-md-row">
                                        <div class="title">
                                            <h1 class="text-light h3">{{ item.name }} <span class="text-muted fs-4">({{ year }})</span></h1>
                                        </div>
                                        <div class="dl mb-3 mb-md-0 text-center">
                                            <ButtonDownload />
                                        </div>
                                    </div>
                                    <p class="text-muted">{{ item.overview }}</p>

                                    <!-- SEASONS & EPISODES SELECTION -->
                                    <Seasons :number="item.number_of_seasons" :seasons="item.seasons" :title="slug(item.name)" class="mb-4" @changeSeason="activeSeason = $event; activeEpisode = 1;" />
                                    <Episodes :tvId="id" :seasonNumber="activeSeason" @changeEpisode="activeEpisode = $event" />

                                    <Casts :id="id" :type="'tv'" class="mb-4"  />
                                    <Recommendations :id="id" :type="'tv'"  />
                                </div>
                            </div>
                        </div>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script>
const mopie = require('~/mopie')

export default {
    name: 'tv-id-slug',
    head() {
        return {
            title: (this.item.name || 'Loading') + ' - ' + this.$i18n.t('Stream Free Movies & TV Shows'),
            meta: [{ hid: 'description', name: 'description', content: (this.item.overview || '') }]
        }
    },
    async fetch() {
        let params = { api_key: mopie.API_KEY, include_adult: false, language: this.$i18n.locale }
        this.item = await this.$axios.$get(`tv/${this.id}`, { params: params })
    },
    data() {
      return {
        item: [],
        activeSeason: 1,      
        activeEpisode: 1      
      }
    },
    // PERBAIKAN: Deteksi otomatis saat halaman pertama kali dibuka berdasarkan isi URL
    mounted() {
        const slugParam = this.$route.params.slug || '';
        // Regex untuk mencari pola akhir angka "-8-18" di akhir URL
        const match = slugParam.match(/-(\d+)-(\d+)$/);
        if (match) {
            this.activeSeason = parseInt(match[1]);
            this.activeEpisode = parseInt(match[2]);
        }
    },
    computed: {
        id() {
            const routeId = this.$route.params.id;
            if (routeId) {
                const match = routeId.match(/\d+/);
                return match ? match[0] : routeId;
            }
            return '';
        },
        backdrop() {
            if (this.item && this.item.backdrop_path) { return mopie.IMAGE_BACKDROP + this.item.backdrop_path }
            return ''
        },
        year() {
            if (this.item.first_air_date) { return this.item.first_air_date.split('-')[0]; }
            return '';
        },
        votes() {
            if (this.item.vote_average) { return Math.round(this.item.vote_average) }
            return 0;
        },
        unvotes() {
            if (this.votes) {
                var unvote = 10 - this.votes
                return [...Array(unvote).keys()];
            }
            return [...Array(10).keys()];
        }
    },
    methods: {
        poster(poster) {
            if (poster == null) { return '/images/no-poster.png' }
            return mopie.IMAGE_POSTER + poster
        },
        slug(txt = '') {
            return txt.toLowerCase().replace(/ /g, '-').replace(/[^\w-]+/g, '')
        },
        // Meneruskan season dan episode hasil deteksi URL ke halaman watch
        goToWatchPage() {
            this.$router.push({
                path: `/tv/${this.id}/watch`,
                query: {
                    s: this.activeSeason,
                    e: this.activeEpisode
                }
            })
        }
    }
}
</script>
