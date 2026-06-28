<template>
    <div :key="$route.params.slug" class="bg-streaming text-light min-vh-100 pb-5">
        
        <!-- 1. MODERN CINEMATIC HERO + DYNAMIC OVERLAY -->
        <div class="hero-container position-relative mb-5" :style="{ backgroundImage: `url(${backdrop})` }">
            <div class="hero-gradient-overlay position-absolute w-100 h-100"></div>
            
            <!-- Frosted Glass Synopsis Card (Fluid Layout) -->
            <div class="container h-100 position-relative z-2 d-flex align-items-center justify-content-center pt-5 pb-4">
                <div class="col-lg-9 col-xl-8">
                    <div class="glass-synopsis-card p-4 p-md-5 rounded-4 shadow-lg border border-glass">
                        <div class="d-flex align-items-center gap-2 mb-3">
                            <span class="badge badge-neon-cyan text-uppercase fw-bold text-xs tracking-wider">Series Info</span>
                            <span class="text-muted text-xs">•</span>
                            <span class="text-muted text-xs fw-medium">{{ year }}</span>
                        </div>
                        <h1 class="hero-title mb-3 text-tracking-tight">{{ item.name }}</h1>
                        <p class="hero-overview leading-relaxed text-justify mb-0">
                            {{ item.overview || 'No overview available for this show.' }}
                        </p>
                    </div>
                </div>
            </div>
        </div>

        <!-- MAIN CONTAINER -->
        <div class="container mt-n2">
            <div class="row justify-content-center">
                <div class="col-lg-9 col-xl-8">
                    
                    <!-- 2. ACTION CONTROLS (WATCH NOW & DOWNLOAD WITH SHARP MICRO-GLOW) -->
                    <div class="d-flex flex-wrap align-items-center justify-content-between gap-3 mb-4 px-2">
                        <button @click="goToWatchPage" class="btn btn-neon-action text-xs fw-bold text-uppercase py-3 px-4 rounded-3 flex-grow-1 flex-sm-grow-0 d-flex align-items-center justify-content-center gap-2">
                            <svg viewBox="0 0 24 24" width="16" height="16" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                            WATCH S{{ activeSeason }}E{{ activeEpisode }} NOW
                        </button>
                        <div class="flex-grow-1 flex-sm-grow-0 d-flex justify-content-end align-items-center btn-download-wrapper">
                            <ButtonDownload />
                        </div>
                    </div>

                    <!-- 3. PREMIUM CONTENT SELECTOR BOX -->
                    <div class="content-box-premium p-4 rounded-4 mb-4 border border-glass-dim shadow-sm">
                        <div class="d-flex align-items-center justify-content-between mb-4">
                            <h2 class="box-section-title text-uppercase mb-0">Seasons &amp; Episodes</h2>
                            <span class="text-muted text-xs fw-semibold opacity-50">{{ episodesList.length }} Episodes Available</span>
                        </div>
                        
                        <!-- SLIDER SEASON DENGAN TOMBOL GLIDE MODERN -->
                        <div class="w-100 position-relative mb-4 pb-2 border-bottom border-glass-dim d-flex align-items-center">
                            <button @click="scrollSeasons('left')" class="slider-glide-btn me-2" aria-label="Scroll left">
                                <svg viewBox="0 0 24 24" width="18" height="18" stroke="currentColor" stroke-width="2.5" fill="none"><polyline points="15 18 9 12 15 6"></polyline></svg>
                            </button>

                            <div ref="seasonContainer" class="season-slider-wrapper d-flex align-items-center overflow-x-auto no-scrollbar flex-grow-1">
                                <button 
                                    v-for="s in item.seasons" 
                                    :key="s.id"
                                    @click="changeSeason(s.season_number)"
                                    class="season-glide-tab text-nowrap"
                                    :class="{ 'active': activeSeason === s.season_number }"
                                >
                                    Season {{ s.season_number }}
                                </button>
                            </div>

                            <button @click="scrollSeasons('right')" class="slider-glide-btn ms-2" aria-label="Scroll right">
                                <svg viewBox="0 0 24 24" width="18" height="18" stroke="currentColor" stroke-width="2.5" fill="none"><polyline points="9 18 15 12 9 6"></polyline></svg>
                            </button>
                        </div>

                        <!-- VERTICAL LIST EPISODE DENGAN GAYA MODERN APPS -->
                        <div v-if="episodesList.length > 0" class="premium-episodes-stack">
                            <div 
                                v-for="ep in episodesList" 
                                :key="ep.id"
                                @click="changeEpisode(ep.episode_number)"
                                class="episode-premium-row d-flex align-items-center justify-content-between"
                                :class="{ 'active': activeEpisode === ep.episode_number }"
                            >
                                <div class="d-flex align-items-center overflow-hidden gap-3 w-100">
                                    <!-- Bulatan Nomor Episode Minimalis -->
                                    <div class="ep-badge-circle flex-shrink-0">
                                        {{ ep.episode_number }}
                                    </div>
                                    <!-- Judul Episode -->
                                    <span class="ep-premium-title text-truncate fw-medium">{{ ep.name }}</span>
                                </div>
                                
                                <div class="d-flex align-items-center gap-3 ms-3 flex-shrink-0">
                                    <span v-if="ep.vote_average" class="text-muted text-xs d-none d-sm-inline opacity-60 fw-semibold">
                                        {{ ep.vote_average.toFixed(1) }} ★
                                    </span>
                                    <div class="play-glow-icon">
                                        <svg viewBox="0 0 24 24" width="12" height="12" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                                    </div>
                                </div>
                            </div>
                        </div>
                    </div>

                    <!-- EXTRA FOOTER CAROUSELS -->
                    <div class="border-top border-glass-dim pt-4 mt-4">
                        <Casts :id="id" :type="'tv'" class="mb-5" />
                        <Recommendations :id="id" :type="'tv'" />
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
    key(route) { return route.params.slug },
    head() {
        return {
            title: (this.item.name || 'Loading') + ' - ' + this.$i18n.t('Stream Free Movies & TV Shows'),
            meta: [{ hid: 'description', name: 'description', content: (this.item.overview || '') }]
        }
    },
    async fetch() {
        let params = { api_key: mopie.API_KEY, include_adult: false, language: this.$i18n.locale }
        this.item = await this.$axios.$get(`tv/${this.id}`, { params: params })
        await this.fetchEpisodes();
    },
    data() {
      return {
        item: [],
        episodesList: [], 
        activeSeason: 1,      
        activeEpisode: 1      
      }
    },
    created() { this.syncDataFromUrl(); },
    watch: {
        '$route.params.slug': {
            handler() {
                this.syncDataFromUrl();
                this.fetchEpisodes();
            }
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
        }
    },
    methods: {
        slug(txt = '') { return txt.toLowerCase().replace(/ /g, '-').replace(/[^\w-]+/g, '') },
        syncDataFromUrl() {
            const slugStr = this.$route.params.slug || '';
            const match = slugStr.match(/-(\d+)-(\d+)$/);
            if (match) {
                this.activeSeason = parseInt(match[1]);
                this.activeEpisode = parseInt(match[2]);
            } else {
                this.activeSeason = 1;
                this.activeEpisode = 1;
            }
        },
        async fetchEpisodes() {
            try {
                let params = { api_key: mopie.API_KEY, language: this.$i18n.locale }
                const res = await this.$axios.$get(`tv/${this.id}/season/${this.activeSeason}`, { params: params });
                this.episodesList = res.episodes || [];
            } catch (e) { this.episodesList = []; }
        },
        getBaseSlug() {
            const currentSlug = this.$route.params.slug || '';
            return currentSlug.replace(/-(\d+)-(\d+)$/, '') || this.slug(this.item.name);
        },
        changeSeason(seasonNumber) {
            const baseSlug = this.getBaseSlug();
            const s = parseInt(seasonNumber) || 1;
            const e = 1; 
            this.$router.push({ path: `/tv/${this.id}/${baseSlug}-${s}-${e}` });
        },
        changeEpisode(episodeNumber) {
            const baseSlug = this.getBaseSlug();
            const s = this.activeSeason;
            const e = parseInt(episodeNumber) || 1;
            this.$router.push({ path: `/tv/${this.id}/${baseSlug}-${s}-${e}` });
        },
        goToWatchPage() {
            this.$router.push({
                path: `/tv/${this.id}/watch`,
                query: { s: this.activeSeason, e: this.activeEpisode }
            })
        },
        scrollSeasons(direction) {
            const container = this.$refs.seasonContainer;
            if (container) {
                const scrollAmount = 200; 
                container.scrollBy({ left: direction === 'left' ? -scrollAmount : scrollAmount, behavior: 'smooth' });
            }
        }
    }
}
</script>

<style scoped>
/* Core Dark Theme Layout */
.bg-streaming { background-color: #08090c !important; font-family: system-ui, -apple-system, sans-serif; }
.z-2 { z-index: 2; }

/* =========================================================
   1. AMBIENT HERO BANNER SECTION (SOFT & DYNAMIC)
   ========================================================= */
.hero-container { 
    min-height: 420px; 
    background-size: cover; 
    background-position: center 15%;
    display: flex;
    align-items: center;
}
.hero-gradient-overlay { 
    background: linear-gradient(180deg, rgba(8,9,12,0.2) 0%, rgba(8,9,12,0.7) 60%, #08090c 100%); 
}
.glass-synopsis-card {
    background-color: rgba(13, 17, 23, 0.45);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
    transition: transform 0.4s ease, border-color 0.4s ease;
}
.glass-synopsis-card:hover {
    border-color: rgba(0, 242, 254, 0.2) !important;
}
.border-glass { border: 1px solid rgba(255, 255, 255, 0.06) !important; }
.border-glass-dim { border: 1px solid rgba(255, 255, 255, 0.04) !important; }

/* Typo Premium */
.hero-title { font-size: 2.25rem; font-weight: 800; letter-spacing: -0.03em; color: #ffffff; }
.hero-overview { font-size: 0.95rem; color: #cbd5e1; line-height: 1.7; opacity: 0.85; }
.text-xs { font-size: 0.75rem; }
.text-tracking-tight { letter-spacing: -0.02em; }
.tracking-wider { letter-spacing: 0.08em; }

/* Badges */
.badge-neon-cyan { background-color: rgba(0, 242, 254, 0.1); color: #00f2fe; padding: 5px 10px; border-radius: 6px; }

/* =========================================================
   2. ACTION CONTROLS (BUTTON RE-DESIGN)
   ========================================================= */
.btn-neon-action {
    background: linear-gradient(135deg, #00f2fe 0%, #4facfe 100%);
    color: #08090c;
    border: none;
    box-shadow: 0 4px 20px rgba(0, 242, 254, 0.25);
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
    letter-spacing: 0.05em;
}
.btn-neon-action:hover {
    transform: translateY(-2px);
    box-shadow: 0 8px 25px rgba(0, 242, 254, 0.45);
    opacity: 0.95;
}
/* Memastikan tombol download ikut beradaptasi mengikuti lengkungan modern */
.btn-download-wrapper ::v-deep button,
.btn-download-wrapper ::v-deep .btn {
    border-radius: 10px !important;
    padding: 11px 20px !important;
}

/* =========================================================
   3. BOX SELECTOR & SEASON SLIDER (DYNAMIC GLIDE)
   ========================================================= */
.content-box-premium {
    background-color: #0d1117;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}
.box-section-title { font-size: 0.8rem; color: #64748b; font-weight: 700; letter-spacing: 0.1em; }

.season-slider-wrapper { scroll-behavior: smooth; gap: 8px; }
.no-scrollbar::-webkit-scrollbar { display: none; }
.no-scrollbar { -ms-overflow-style: none; scrollbar-width: none; }

.season-glide-tab {
    background: transparent;
    border: none;
    color: #64748b;
    font-size: 0.9rem;
    font-weight: 600;
    padding: 10px 20px;
    border-radius: 20px;
    cursor: pointer;
    transition: all 0.25s cubic-bezier(0.4, 0, 0.2, 1);
}
.season-glide-tab:hover { color: #ffffff; background-color: rgba(255, 255, 255, 0.03); }
.season-glide-tab.active {
    color: #00f2fe;
    background-color: rgba(0, 242, 254, 0.08);
    font-weight: 700;
}

.slider-glide-btn {
    background: rgba(255, 255, 255, 0.02);
    border: 1px solid rgba(255, 255, 255, 0.04);
    color: #475569;
    width: 32px;
    height: 32px;
    border-radius: 50%;
    cursor: pointer;
    display: flex;
    align-items: center;
    justify-content: center;
    transition: all 0.2s ease;
}
.slider-glide-btn:hover { color: #ffffff; background-color: rgba(255, 255, 255, 0.08); border-color: rgba(255, 255, 255, 0.1); }

/* =========================================================
   4. LIst EPISODE DENGAN ANIMASI DAN INTERAKSI HALUS
   ========================================================= */
.premium-episodes-stack {
    display: flex;
    flex-direction: column;
    gap: 6px;
    max-height: 400px; 
    overflow-y: auto;
    padding-right: 4px;
}
.premium-episodes-stack::-webkit-scrollbar { width: 4px; }
.premium-episodes-stack::-webkit-scrollbar-track { background: transparent; }
.premium-episodes-stack::-webkit-scrollbar-thumb { background: rgba(255, 255, 255, 0.1); border-radius: 10px; }

.episode-premium-row {
    display: flex;
    align-items: center;
    padding: 12px 16px;
    background-color: rgba(255, 255, 255, 0.01);
    border: 1px solid transparent;
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.2s cubic-bezier(0.4, 0, 0.2, 1);
}
.episode-premium-row:hover {
    background-color: rgba(255, 255, 255, 0.03);
    transform: translateX(4px); /* Efek geser kanan dinamis saat di-hover */
}

/* Lingkaran nomor episode */
.ep-badge-circle {
    width: 28px;
    height: 28px;
    background-color: rgba(255, 255, 255, 0.04);
    color: #94a3b8;
    border-radius: 50%;
    display: flex;
    align-items: center;
    justify-content: center;
    font-size: 0.75rem;
    font-weight: 700;
    transition: all 0.2s ease;
}
.ep-premium-title { font-size: 0.9rem; color: #cbd5e1; transition: color 0.2s ease; }
.play-glow-icon { color: #ffffff; opacity: 0.15; display: flex; align-items: center; transition: all 0.2s ease; }

/* State Saat Episode Dipilih (Active State) */
.episode-premium-row.active {
    background-color: rgba(0, 242, 254, 0.03);
    border-color: rgba(0, 242, 254, 0.15);
}
.episode-premium-row.active .ep-badge-circle {
    background-color: #00f2fe;
    color: #08090c;
}
.episode-premium-row.active .ep-premium-title {
    color: #ffffff;
    font-weight: 600;
}
.episode-premium-row.active .play-glow-icon {
    color: #00f2fe;
    opacity: 1;
    transform: scale(1.1);
}
.episode-premium-row:hover .play-glow-icon { opacity: 0.6; }
</style>
