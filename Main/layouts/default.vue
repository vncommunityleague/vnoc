<template>
    <div class="layout layout--corsace">
        <the-header
            class="main__header"
        >
            <a 
                class="header-nav"
                href="/"
            >
                <div 
                    class="header-nav__brand-container"
                    :class="`header-nav__brand-container--${viewTheme}`"
                >
                    <div 
                        class="header-nav__brand" 
                        :class="`header-nav__brand--${viewTheme}`"
                    />
                </div>
                <img
                    class="header-nav__brand-name"
                    :class="`header-nav__brand-name--${viewTheme}`"
                    src="../../Assets/img/site/mca-ayim/corsace_text.png"
                    alt=""
                >
            </a>
        </the-header>

        <nuxt 
            class="main" 
            :class="`main--${viewTheme}`"
        />
        
        <the-footer class="main__footer">
            <div 
                class="socials"
                :class="`socials--${viewTheme}`"
            >
                <a
                    href="https://twitter.com/corsace_"
                    target="_blank"
                >
                    <img
                        class="socials__icon"
                        src="../../Assets/img/social/twitter.png"
                        alt=""
                    >
                </a>
                <a
                    href="https://discord.gg/Z6vEMsr"
                    target="_blank"
                >
                    <img
                        class="socials__icon"
                        src="../../Assets/img/social/discord.png"
                        alt=""
                    >
                </a>
                <a
                    href="https://www.twitch.tv/corsace"
                    target="_blank"
                >
                    <img
                        class="socials__icon"
                        src="../../Assets/img/social/twitch.png"
                        alt=""
                    >
                </a>
                <a
                    href="https://youtube.com/corsace"
                    target="_blank"
                >
                    <img
                        class="socials__icon"
                        src="../../Assets/img/social/youtube.png"
                        alt=""
                    >
                </a>
                <a
                    href="https://github.com/corsace/corsace"
                    target="_blank"
                >
                    <img
                        class="socials__icon"
                        src="../../Assets/img/social/github.png"
                        alt=""
                    >
                </a>
            </div>

            <div 
                class="kaetwo"
                :class="`kaetwo--${viewTheme}`"
            >
                powered by
                <a
                    class="kaetwo__powered"
                    href="https://twitter.com/kkaetwo"
                >
                    <img
                        class="kaetwo__image" 
                        :class="`kaetwo__image--${viewTheme}`"
                        src="../../Assets/img/partners/kaetwo.png"
                    >
                </a>
            </div>
        </the-footer>
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { State } from "vuex-class";

import TheHeader from "../../Assets/components/header/TheHeader.vue";
import TheFooter from "../../Assets/components/footer/TheFooter.vue";

@Component({
    components: {
        TheHeader,
        TheFooter,
    },
    middleware: "index",
})
export default class Default extends Vue {

    @State viewTheme!: "light";

    async mounted () {
        await this.$store.dispatch("setViewTheme", "light");
    }
    
}
</script>

<style lang="scss">
@import '@s-sass/_mixins';
@import '@s-sass/_variables';

.main {
    &__header {
        border-bottom: 1px solid $pink;
    }

    &__footer {
        border-top: 1px solid $pink;
    }

    &--light {
        background-color: $white;
    }
    &--dark {
        background-color: $darker-gray;
    }
}

.header-nav {
    display: flex;
    align-items: center;

    &__brand {
        &-container {
            width: 90px;
            height: 90px;
            @include breakpoint(mobile) {
                width: 55px;
                height: 55px;
            }
            &--light {
                background-image: linear-gradient(to top, #d93044, rgba(231, 95, 120, 0.82));
            }
            &--dark {
                background-image: linear-gradient(to top, rgba(244, 182, 193, 0.82), #e98792);
            }
        }

        &-name {
            @include breakpoint(mobile) {
                width: 100px;
            }
            &--light {
                filter: invert(1);
            }
        }

        width: 100%;
        height: 100%;
        background-size: contain;
        background-image: url('../../Assets/img/corsace.png');
        &--light {
            filter: invert(1);
        }
    }
}

.socials {
    height: 100%;
    display: flex;
    align-items: center;
    margin-left: 10px;

    &__icon {
        margin-right: 10px;
        width: auto;
        height: 30px;
    }
    &--light {
        filter: invert(1);
    }
    &--dark {
        color: $gray;
    }
}

.kaetwo {
    &--light {
        color: $dark;
    }
    &--dark {
        color: $gray;
    }
    font-size: 1rem;
    display: flex;
    align-items: center;
    justify-content: center;
    padding: 10px 0;
    margin: auto;

    &__powered {
        display: flex;
        align-items: center;
        justify-content: center;
    }

    &__image {
        height: 20px;
        padding: 0 5px;
        &--light {
            filter: invert(1);
        }
        &--dark {
            color: $gray;
        }
    }
}
</style>
