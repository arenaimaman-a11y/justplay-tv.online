<template>
    <div class="watch-page text-white pb-5">
        <div class="hero-backdrop" :style="backdropStyle"></div>

        <div class="container position-relative wrapper-content py-4">
            <div class="mb-4">
                <button @click="$router.push(`/tv/${$route.params.id}`)" class="btn-back d-inline-flex align-items-center gap-2">
                    <svg viewBox="0 0 24 24" width="18" height="18" stroke="currentColor" stroke-width="2.5" fill="none" stroke-linecap="round" stroke-linejoin="round"><line x1="19" y1="12" x2="5" y2="12"></line><polyline points="12 19 5 12 12 5"></polyline></svg>
                    Back to Details
                </button>
            </div>

            <div class="movie-header mb-4">
                <h1 class="fw-extrabold text-white display-6 title-glow mb-2">{{ item.name || 'Loading Video...' }}</h1>
                <div class="d-flex align-items-center gap-3 text-white-50 small flex-wrap">
                    <span class="badge-custom">NOW STREAMING</span>
                    <span class="meta-item d-flex align-items-center gap-1">
                        <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none"><path d="M4 15s1-1 4-1 5 2 8 2 4-1 4-1V3s-1 1-4 1-5-2-8-2-4 1-4 1z"></path><line x1="4" y1="22" x2="4" y2="15"></line></svg>
                        Season {{ season }}
                    </span>
                    <span class="meta-item d-flex align-items-center gap-1">
                        <svg viewBox="0 0 24 24" width="14" height="14" stroke="currentColor" stroke-width="2" fill="none"><circle cx="12" cy="12" r="10"></circle><polygon points="12 6 12 12 16 14"></polygon></svg>
                        Episode {{ episode }}
                    </span>
                </div>
            </div>

            <div class="player-glow-wrapper mb-5">
                <div class="player-container ratio ratio-16x9 shadow-2xl rounded-2xl overflow-hidden position-relative">
                    
                    <div v-if="!isPlayed" class="video-ad-overlay d-flex flex-column align-items-center justify-content-center" @click="handlePlayClick">
                        <div class="play-btn-circle d-flex align-items-center justify-content-center mb-3">
                            <svg viewBox="0 0 24 24" width="40" height="40" fill="currentColor"><polygon points="5 3 19 12 5 21 5 3"></polygon></svg>
                        </div>
                        <span class="fw-bold tracking-wide text-uppercase small opacity-90">Click to Play Video</span>
                    </div>

                    <iframe 
                        :key="`player-${id}-${season}-${episode}`"
                        :src="videoUrl" 
                        allowfullscreen
                        scrolling="no"
                        allow="autoplay; encrypted-media">
                    </iframe>
                </div>
            </div>

            <div class="row justify-content-center">
                <div class="col-lg-12">
                    <div class="modern-card p-4 p-md-5 rounded-2xl">
                        <div class="d-flex align-items-center gap-2 mb-4">
                            <div class="indicator-bar"></div>
                            <h3 class="text-light h5 fw-bold m-0">Sponsored Content</h3>
                        </div>
                        
                        <div class="ad-container-wrapper d-flex flex-column align-items-center justify-content-center p-3 text-center">
                            <span class="ad-notice mb-3">ADVERTISEMENT</span>
                            
                            <div class="ad-content-box" ref="adBox">
                                <div id="container-dd5a3c4937ebe8b47da808bcd5e1d283"></div>
                            </div>
                        </div>
                    </div>

                    <div class="text-center mt-5 opacity-50">
                        <p class="small m-0">If the video is buffering, you can try switching servers inside the player or refresh the page.</p>
                    </div>

                </div>
            </div>
        </div>
    </div>
</template>

<script>
const mopie = require('~/mopie')

export default {
    name: 'tv-watch-page',
    watchQuery: ['s', 'e'],
    data() {
        return {
            item: [],
            isPlayed: false
        }
    },
    watch: {
        '$route.query': {
            handler() {
                this.isPlayed = false;
            },
            deep: true
        }
    },
    async fetch() {
        let params = {
            api_key: mopie.API_KEY,
            include_adult: false,
            language: this.$i18n.locale,
        }
        this.item = await this.$axios.$get(`tv/${this.id}`, { params: params })
    },
    mounted() {
        // Hanya memuat Popunder dan Iklan Banner bawaan Anda
        this.initAdsterraPopunder();
        this.initNewAdWidget(); 
    },
    computed: {
        id() {
            const routeId = this.$route.params.id || '';
            const match = routeId.match(/^\d+/);
            return match ? match[0] : routeId;
        },
        season() {
            if (this.$route.query && this.$route.query.s !== undefined && this.$route.query.s !== null && this.$route.query.s !== '') {
                return parseInt(this.$route.query.s);
            }
            const fullPath = this.$route.fullPath || '';
            const match = fullPath.match(/-(\d+)-(\d+)/);
            return match ? parseInt(match[1]) : 1;
        },
        episode() {
            if (this.$route.query && this.$route.query.e !== undefined && this.$route.query.e !== null && this.$route.query.e !== '') {
                return parseInt(this.$route.query.e);
            }
            const fullPath = this.$route.fullPath || '';
            const match = fullPath.match(/-(\d+)-(\d+)/);
            return match ? parseInt(match[2]) : 1;
        },
        videoUrl() {
            return `https://vidsrc.me/embed/tv?tmdb=${this.id}&s=${this.season}&e=${this.episode}`
        },
        backdropStyle() {
            if (this.item && this.item.backdrop_path) {
                return {
                    backgroundImage: `linear-gradient(to bottom, rgba(8,9,12,0.7) 0%, #08090c 100%), url(${mopie.IMAGE_BACKDROP}${this.item.backdrop_path})`
                }
            }
            return { backgroundColor: '#08090c' }
        }
    },
    methods: {
        slug(txt = '') {
            return txt.toLowerCase().replace(/ /g, '-').replace(/[^\w-]+/g, '')
        },
        handlePlayClick() {
            if (process.client) {
                window.open('https://twigcrucialpal.com/qhexrkev?key=8f5d9e9efc0679706823f58257516b31', '_blank');
            }
            this.isPlayed = true;
        },
        initNewAdWidget() {
            if (process.client) {
                const oldScript = document.getElementById('adsterra-new-widget');
                if (oldScript) oldScript.remove();

                const script = document.createElement('script');
                script.id = 'adsterra-new-widget';
                script.type = 'text/javascript';
                script.async = true;
                script.setAttribute('data-cfasync', 'false');
                script.src = 'https://twigcrucialpal.com/dd5a3c4937ebe8b47da808bcd5e1d283/invoke.js';
                
                document.head.appendChild(script);
            }
        },
        initAdsterraPopunder() {
            if (process.client) {
                const oldScript = document.getElementById('adsterra-popunder');
                if (oldScript) oldScript.remove();

                window.atOptions = {
                    'key' : 'GANTI_DENGAN_KODE_KEY_ADSTERRA_ANDA',
                    'format' : 'iframe',
                    'height' : 1,
                    'width' : 1,
                    'params' : {}
                };

                const script = document.createElement('script');
                script.id = 'adsterra-popunder';
                script.type = 'text/javascript';
                script.src = `//www.highperformanceformat.com/GANTI_DENGAN_KODE_KEY_ADSTERRA_ANDA/invoke.js`;
                
                document.head.appendChild(script);
            }
        }
    }
}
</script>

<style scoped>
/* AMBIENCE DESIGN */
.watch-page {
    background-color: #08090c; 
    font-family: 'Inter', -apple-system, BlinkMacSystemFont, sans-serif;
    overflow-x: hidden;
    position: relative;
}

/* OVERLAY PLAY BUTTON STYLE */
.video-ad-overlay {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 100%;
    background: rgba(0, 0, 0, 0.75);
    backdrop-filter: blur(4px);
    z-index: 10;
    cursor: pointer;
    transition: background 0.3s ease;
}
.video-ad-overlay:hover {
    background: rgba(0, 0, 0, 0.6);
}
.play-btn-circle {
    width: 80px;
    height: 80px;
    background: linear-gradient(135deg, #00f2fe 0%, #4facfe 100%); 
    border-radius: 50%;
    color: #08090c; 
    box-shadow: 0 0 30px rgba(0, 242, 254, 0.4); 
    transition: transform 0.3s cubic-bezier(0.175, 0.885, 0.32, 1.275);
}
.video-ad-overlay:hover .play-btn-circle {
    transform: scale(1.15);
    opacity: 0.95;
    box-shadow: 0 0 40px rgba(0, 242, 254, 0.6);
}
.tracking-wide {
    letter-spacing: 2px;
    text-shadow: 0 2px 4px rgba(0,0,0,0.8);
}

/* BACKDROP BLUR EFFECT */
.hero-backdrop {
    position: absolute;
    top: 0;
    left: 0;
    width: 100%;
    height: 550px;
    background-size: cover;
    background-position: center top;
    filter: blur(40px);
    opacity: 0.35;
    z-index: 1;
    pointer-events: none;
}

.wrapper-content {
    z-index: 2;
}

/* MODERN BUTTON BACK */
.btn-back {
    background: rgba(255, 255, 255, 0.06);
    border: 1px solid rgba(255, 255, 255, 0.1);
    color: #e5e5e5;
    padding: 10px 22px;
    font-size: 0.875rem;
    font-weight: 600;
    border-radius: 50px;
    transition: all 0.3s cubic-bezier(0.4, 0, 0.2, 1);
    backdrop-filter: blur(8px);
}
.btn-back:hover {
    background: rgba(255, 255, 255, 0.15);
    border-color: rgba(0, 242, 254, 0.3); 
    color: #ffffff;
    transform: translateX(-3px);
}

/* TYPOGRAPHY */
.fw-extrabold {
    font-weight: 800;
}
.title-glow {
    text-shadow: 0 2px 10px rgba(0, 0, 0, 0.5);
    letter-spacing: -0.5px;
}
.badge-custom {
    background: linear-gradient(135deg, #00f2fe 0%, #4facfe 100%); 
    color: #08090c; 
    font-size: 0.7rem;
    font-weight: 800;
    padding: 5px 12px;
    border-radius: 4px;
    letter-spacing: 1px;
}
.meta-item {
    font-weight: 500;
    letter-spacing: 0.3px;
}

/* PREMIUM PLAYER LAYOUT WITH GLOW EFFECT */
.player-glow-wrapper {
    position: relative;
}
.player-glow-wrapper::before {
    content: '';
    position: absolute;
    top: 5%;
    left: 2%;
    width: 96%;
    height: 90%;
    background: rgba(0, 242, 254, 0.12); 
    filter: blur(60px);
    z-index: -1;
    border-radius: 20px;
    pointer-events: none;
}
.player-container {
    background: #000000;
    border: 1px solid rgba(255, 255, 255, 0.08);
    box-shadow: 0 25px 50px -12px rgba(0, 0, 0, 0.7);
}

/* THE CONTAINER CARD FOR SEASONS/EPISODES */
.modern-card {
    background: rgba(13, 17, 23, 0.75) !important; 
    backdrop-filter: blur(16px);
    -webkit-backdrop-filter: blur(16px);
    border: 1px solid rgba(255, 255, 255, 0.05) !important;
    box-shadow: 0 20px 40px rgba(0, 0, 0, 0.4);
}
.indicator-bar {
    width: 4px;
    height: 22px;
    background: #00f2fe; 
    border-radius: 10px;
}

/* ADSTERRA STYLING FOR NEAT DISPLAY */
.ad-container-wrapper {
    background: rgba(0, 0, 0, 0.3);
    border-radius: 12px;
    border: 1px dashed rgba(255, 255, 255, 0.12);
    min-height: 150px;
    width: 100%;
}
.ad-notice {
    font-size: 0.65rem;
    font-weight: 700;
    color: rgba(255, 255, 255, 0.3);
    letter-spacing: 2px;
}
.ad-content-box {
    width: 100%;
    display: flex;
    justify-content: center;
    align-items: center;
}

/* DEEP OVERRIDE FOR TEMPLATE COMPONENTS STYLE */
::v-deep .card {
    background-color: transparent !important;
    border: none !important;
}

::v-deep select, 
::v-deep .btn, 
::v-deep button:not(.btn-back) {
    border-radius: 8px !important;
    transition: all 0.2s ease;
}
</style>
