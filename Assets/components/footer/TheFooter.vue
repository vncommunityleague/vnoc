<template>
    <div 
        class="footer"
        :class="`footer--${viewTheme} footer--${site}`"
    >
        <slot />
        
        <language-switcher />

        <theme-switcher 
            v-if="site !== 'open'"
        />
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { State } from "vuex-class";

@Component({
    components: {},
})
export default class TheFooter extends Vue {
    @State site!: string;
    @State viewTheme!: "light";

    isSmall = false;

    mounted () {
        if (process.client) {
            this.isSmall = window.innerWidth < 576;
            window.addEventListener("resize", () => {
                this.isSmall = window.innerWidth < 576;
            });
        }
    }
}
</script>

<style lang="scss">
@import '@s-sass/_variables';
@import '@s-sass/_mixins';

.footer {
    &--light {
        background-color: #ffa1a1;
        color: #0f0606;
    }
    &--dark {
        background-color: $dark;
        color: white;
    }
    &--vnoc {
        color: #0f0606;
        background-color: #0f0606;
    }
    bottom: 0;

    @include breakpoint(mobile) {
        font-size: $font-lg;
    }
    font-size: $font-xl;
    @include breakpoint(laptop) {
        font-size: $font-xxl;
    }

    width: 100%;
    height: 50px;
    display: flex;
}
</style>