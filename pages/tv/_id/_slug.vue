<template>
    <div :key="$route.params.slug" class="bg-streaming text-light min-vh-100 pb-5">
        
        <!-- 1. MODERN CINEMATIC HERO + SINOPSIS & COMPACT PLAY BUTTON -->
        <div class="hero-container position-relative mb-4" :style="{ backgroundImage: `url(${backdrop})` }">
            <div class="hero-gradient-overlay position-absolute w-100 h-100"></div>
            
            <!-- Frosted Glass Synopsis Card -->
            <div class="container h-100 position-relative z-2 d-flex align-items-center justify-content-center pt-5 pb-3">
                <div class="col-lg-9 col-xl-8">
                    <div class="glass-synopsis-card p-4 p-md-5 rounded-4 shadow-lg border border-glass">
                        <div class="d-flex align-items-center gap-2 mb-2">
                            <span class="badge badge-neon-cyan text-uppercase fw-bold text-xs tracking-wider">Series Info</span>
                            <span class="text-muted text-xs">•</span>
                            <span class="text-muted text-xs fw-medium">{{ year }}</span>
                        </div>
                        <h1 class="hero-title mb-2 text-tracking-tight">{{ item.name }}</h1>
                        <p class="hero-overview leading-relaxed text-justify mb-4 opacity-85">
                            {{ item.overview || 'No overview available for this show.' }}
                        </p>
                        
                        <!-- TOMBOL WATCH NOW UTAMA MERAPAT KE ATAS -->
                        <div class="d-inline-block">
                            <button @click="goToWatchPage" class="btn btn-neon-action text-xs fw-bold text-uppercase py-2.5 px-4 rounded-3 d-flex align-items-center gap-2">
                                <svg viewBox="0 0 24 24" width="14" height="14" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                                WATCH S{{ activeSeason }}E{{ activeEpisode }} NOW
                            </button>
                        </div>
                    </div>
                </div>
            </div>
        </div>

        <!-- MAIN CONTAINER -->
        <div class="container">
            <div class="row justify-content-center">
                <div class="col-lg-9 col-xl-8">

                    <!-- 2. PREMIUM CONTENT SELECTOR BOX -->
                    <div class="content-box-premium p-4 rounded-4 mb-4 border border-glass-dim shadow-sm">
                        
                        <!-- KOTAK WRAPPER SEASON DENGAN ANTI-MELAR ABSOLUT -->
                        <div class="jw-season-box-container p-3 rounded-3 mb-4 border border-glass-dim overflow-hidden">
                            <div class="d-flex align-items-center justify-content-between mb-3">
                                <h2 class="box-section-title text-uppercase mb-0">Select Season</h2>
                                <span class="text-muted text-xs opacity-50 fw-medium">Swipe to scroll</span>
                            </div>

                            <!-- Struktur Pengunci Grid Presisi -->
                            <div class="season-slider-layout">
                                <!-- Tombol Kiri -->
                                <button @click="scrollSeasons('left')" class="slider-glide-btn btn-left" aria-label="Scroll left">
                                    <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2.5" fill="none"><polyline points="15 18 9 12 15 6"></polyline></svg>
                                </button>

                                <!-- Jalur Scroll Utama (Dikunci Total) -->
                                <div ref="seasonContainer" class="season-slider-wrapper no-scrollbar">
                                    <div class="season-scroll-content">
                                        <button 
                                            v-for="s in item.seasons" 
                                            :key="s.id"
                                            @click="changeSeason(s.season_number)"
                                            class="season-glide-tab"
                                            :class="{ 'active': activeSeason === s.season_number }"
                                        >
                                            Season {{ s.season_number }}
                                        </button>
                                    </div>
                                </div>

                                <!-- Tombol Kanan -->
                                <button @click="scrollSeasons('right')" class="slider-glide-btn btn-right" aria-label="Scroll right">
                                    <svg viewBox="0 0 24 24" width="16" height="16" stroke="currentColor" stroke-width="2.5" fill="none"><polyline points="9 18 15 12 9 6"></polyline></svg>
                                </button>
                            </div>
                        </div>

                        <!-- 3. VERTICAL LIST EPISODE -->
                        <div class="d-flex align-items-center justify-content-between mb-3 px-1">
                            <h2 class="box-section-title text-uppercase mb-0">Episodes List</h2>
                            <span class="text-muted text-xs fw-semibold opacity-50">{{ episodesList.length }} Episodes</span>
                        </div>

                        <div v-if="episodesList.length > 0" class="premium-episodes-stack">
                            <div 
                                v-for="ep in episodesList" 
                                :key="ep.id"
                                @click="changeEpisode(ep.episode_number)"
                                class="episode-premium-row d-flex align-items-center justify-content-between"
                                :class="{ 'active': activeEpisode === ep.episode_number }"
                            >
                                <div class="d-flex align-items-center overflow-hidden gap-3 w-100">
                                    <div class="ep-badge-circle flex-shrink-0">
                                        {{ ep.episode_number }}
                                    </div>
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
                const scrollAmount = 140; 
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
   1. AMBIENT HERO BANNER SECTION 
   ========================================================= */
.hero-container { 
    min-height: 400px; 
    background-size: cover; 
    background-position: center 15%;
    display: flex;
    align-items: center;
}
.hero-gradient-overlay { 
    background: linear-gradient(180deg, rgba(8,9,12,0.1) 0%, rgba(8,9,12,0.6) 60%, #08090c 100%); 
}
.glass-synopsis-card {
    background-color: rgba(13, 17, 23, 0.45);
    backdrop-filter: blur(20px);
    -webkit-backdrop-filter: blur(20px);
}
.border-glass { border: 1px solid rgba(255, 255, 255, 0.06) !important; }
.border-glass-dim { border: 1px solid rgba(255, 255, 255, 0.04) !important; }

/* Typo Premium */
.hero-title { font-size: 2.25rem; font-weight: 800; letter-spacing: -0.03em; color: #ffffff; }
.hero-overview { font-size: 0.95rem; color: #cbd5e1; line-height: 1.6; }
.text-xs { font-size: 0.75rem; }
.text-tracking-tight { letter-spacing: -0.02em; }
.tracking-wider { letter-spacing: 0.08em; }
.badge-neon-cyan { background-color: rgba(0, 242, 254, 0.1); color: #00f2fe; padding: 5px 10px; border-radius: 6px; }

/* Button Play */
.btn-neon-action {
    background: linear-gradient(135deg, #00f2fe 0%, #4facfe 100%);
    color: #08090c;
    border: none;
    box-shadow: 0 4px 15px rgba(0, 242, 254, 0.2);
    transition: all 0.3s cubic-bezier(0.25, 0.8, 0.25, 1);
}
.btn-neon-action:hover {
    transform: translateY(-1px);
    box-shadow: 0 6px 20px rgba(0, 242, 254, 0.4);
    opacity: 0.95;
}

/* =========================================================
   2. ANTI-MELAR SEASON SLIDER SYSTEM (PERBAIKAN TOTAL MUTLAK)
   ========================================================= */
.content-box-premium {
    background-color: #0d1117;
    box-shadow: 0 10px 30px rgba(0, 0, 0, 0.2);
}
.box-section-title { font-size: 0.8rem; color: #64748b; font-weight: 700; letter-spacing: 0.1em; }

/* Wadah Utama */
.jw-season-box-container {
    background-color: rgba(255, 255, 255, 0.015);
    width: 100%;
}

/* Mengunci layout baris menggunakan CSS Grid murni */
.season-slider-layout {
    display: grid;
    grid-template-columns: auto 1fr auto;
    align-items: center;
    width: 100%;
}

/* Memaksa area scroll tetap pada ukurannya dan membuang elemen meluber */
.season-slider-wrapper {
    overflow-x: auto;
    overflow-y: hidden;
    width: 100%;
    display: block; /* Menghilangkan efek flexbox yang merusak layout mobile */
    -webkit-overflow-scrolling: touch; /* Mengaktifkan smooth swipe gesture di iPhone/Android */
}

/* Konten internal horizontal */
.season-scroll-content {
    display: inline-flex;
    gap: 8px;
    padding: 2px 0;
    white-space: nowrap; /* Mencegah tombol turun ke bawah */
}

/* Menyembunyikan Scrollbar standard di semua browser */
.no-scrollbar::-webkit-scrollbar { display: none !important; }
.no-scrollbar { -ms-overflow-style: none !important; scrollbar-width: none !important; }

/* Tombol Season Modis */
.season-glide-tab {
    display: inline-block;
    background: transparent;
    border: none;
    color: #64748b;
    font-size: 0.88rem;
    font-weight: 600;
    padding: 8px 16px;
    border-radius: 12px;
    cursor: pointer;
    transition: all 0.2s ease;
}
.season-glide-tab:hover { color: #ffffff; background-color: rgba(255, 255, 255, 0.03); }
.season-glide-tab.active {
    color: #00f2fe;
    background-color: rgba(0, 242, 254, 0.08);
    font-weight: 700;
}

/* Tombol Panah Navigasi */
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
.slider-glide-btn:hover { color: #ffffff; background-color: rgba(255, 255, 255, 0.08); }

/* Pengondisian ketat untuk HP */
@media (max-width: 576px) {
    .season-glide-tab {
        font-size: 0.82rem;
        padding: 7px 12px;
    }
    .slider-glide-btn {
        width: 28px;
        height: 28px;
    }
}

/* =========================================================
   3. VERTICAL LIST EPISODE
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
    transform: translateX(4px); 
}

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
}
.ep-premium-title { font-size: 0.9rem; color: #cbd5e1; transition: color 0.2s ease; }
.play-glow-icon { color: #ffffff; opacity: 0.15; display: flex; align-items: center; transition: all 0.2s ease; }

.episode-premium-row.active {
    background-color: rgba(0, 242, 254, 0.03);
    border-color: rgba(0, 242, 254, 0.15);
}
.episode-premium-row.active .ep-badge-circle { background-color: #00f2fe; color: #08090c; }
.episode-premium-row.active .ep-premium-title { color: #ffffff; font-weight: 600; }
.episode-premium-row.active .play-glow-icon { color: #00f2fe; opacity: 1; transform: scale(1.1); }
</style>
