<template>
    <div>
        <div class="bg-dark offcanvas offcanvas-end" :class="{show: isShow}">
            <div class="offcanvas-header">
                <button type="button" class="btn-close text-reset" @click="close"></button>
            </div>
            <div class="offcanvas-body">
                <div class="">
                Some text as placeholder. In real life you can have the elements you have chosen. Like, text, images, lists, etc.
                </div>
            </div>
        </div>
        <div class="offcanvas-backdrop fade show" v-if="isShow"></div>
    </div>
</template>

<style scoped>
.offcanvas.offcanvas-end {
    visibility: hidden;
    transition: all .2s ease-in-out;
}
.offcanvas.offcanvas-end.show {
    visibility: visible;
}
.offcanvas-header .btn-close {
    filter: invert(1);
}
</style>

<script>
export default {
    data() {
        return {
            isShow: false
        }
    },
    mounted() {
        this.$nuxt.$on('showSignIn', () => {
            this.isShow = true;
        })
    },
    methods: {
        toggleShow() {
            this.isShow = !this.isShow
        },
        close() {
            this.isShow = false
        }
    }
}
</script>