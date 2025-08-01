<template>
    <div 
        class="header"
        :class="`header--${viewTheme}`"
    >
        <slot />

        <div 
            v-if="loggedInUser"
            class="header-login"
            :class="`header-login--${site}`"
        >
            <div class="header-login__welcome-container">
                <img 
                    :src="avatarURL"
                    class="header-login__avatar"
                >
                <div
                    v-if="isSmall"
                    class="header-login__welcome"
                    :class="`header-login__welcome--${site}`"
                >
                    {{ loggedInUser.discord.username ? loggedInUser.discord.username : loggedInUser.osu.username }}
                </div>
                <div 
                    v-else
                    class="header-login__welcome"
                    :class="`header-login__welcome--${viewTheme} header-login__welcome--${site}`"
                >
                    {{ $t('header.welcomeBack') }}
                </div>
                <div
                    v-if="loggedInUser.discord.username && !isSmall"
                    class="header-login__username"
                    :class="`header-login__username--${site}`"
                >
                    {{ loggedInUser.osu.username }} 
                    <span 
                        class="header-login__line" 
                        :class="`header-login__line--${site}`"
                    >|</span>
                    {{ loggedInUser.discord.username }}
                </div>
                <div
                    v-else-if="loggedInUser.osu.username && !isSmall"
                    class="header-login__username"
                    :class="`header-login__username--${site}`"
                >
                    {{ loggedInUser.osu.username }}
                </div>
                <div>
                    <div  
                        class="header-login__welcome header-login__menu"
                        :class="`header-login__welcome--${viewTheme} header-login__welcome--${site}`"
                        @click="openMenu = !openMenu"
                    >
                        <div
                            class="hamburger"
                            :class="`hamburger--${viewTheme} hamburger--${site}`"
                        >
                            ≡
                            <div
                                class="notif" 
                                :class="`notif--${viewTheme} notif--${site}`"
                                :style="{ opacity: notifSync ? 1 : 0 }"
                            />
                        </div>
                        {{ $t('header.menu') }}
                    </div>
                    <div
                        v-if="openMenu"
                        class="header-login__menu_dropdown"
                        :class="`header-login__menu_dropdown--${viewTheme} header-login__menu_dropdown--${site}`"
                        @click="openMenu = false"
                    >
                        <slot name="menu" />
                        <a
                            v-if="!loggedInUser.discord || !loggedInUser.discord.userID"
                            :href="`/api/login/discord?site=${site}&redirect=${$route.fullPath}`"
                        >
                            <MenuItem>{{ $t('header.login', { site: "discord" }) }}</MenuItem>
                        </a>
                        <a 
                            v-else
                            :href="`/api/login/discord?site=${site}&redirect=${$route.fullPath}`"
                        >
                            <MenuItem>{{ $t('header.changeDiscord') }}</MenuItem>
                        </a>
                        <a
                            href="/api/logout"
                        >
                            <MenuItem>{{ $t('header.logout') }}</MenuItem>
                        </a>
                    </div>
                </div>
            </div>
        </div>
        <div
            v-else
            class="header-login"
            :class="`header-login--${site}`"
        >
            <a
                class="header-login__link"
                :class="`header-login__link--${viewTheme} header-login__link--${site}`"
                :href="`/api/login/osu?site=${site}&redirect=${$route.fullPath}`"
            >
                {{ site !== "open" ? $t('header.login', { site: "osu!" }) : $t('header.login', { site: "osu!" }).toString().split("osu!").map(p => p === "osu!" ? p : p.toUpperCase()).join('osu!') }}
            </a>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, PropSync } from "vue-property-decorator";
import { State } from "vuex-class";

import MenuItem from "./MenuItem.vue";

import { UserInfo } from "../../../Interfaces/user";

@Component({
    components: {
        MenuItem,
    },
})
export default class TheHeader extends Vue {

    @PropSync("notif", { type: Boolean, default : false }) notifSync!: boolean;

    @State site!: string;
    @State loggedInUser!: null | UserInfo;
    @State viewTheme!: "light" | "dark";

    get avatarURL (): string  {
        return this.loggedInUser?.osu?.avatar ?? "";
    }

    openMenu = false;
    isSmall = false;

    mounted () {
        if (process.client) {
            this.isSmall = window.innerWidth < 992;
            window.addEventListener("resize", () => this.isSmall = window.innerWidth < 992);
        }
    }

}
</script>

<style lang="scss">
@import '@s-sass/_mixins';
@import '@s-sass/_variables';

.header {
     &--light {
        background-color: #1f0d0c;
        color: #0f0606;
    }
    &--dark {
        background-color: $dark;
    }
    position: relative;
    top: 0;

    width: 100%;
    height: 90px;
    @include breakpoint(mobile) {
        height: 55px;
    }
    display: flex;
    flex-direction: row;
    justify-content: space-between;
}

.header-login {
    position: relative;
    display: flex;
    flex-direction: column;
    align-items: flex-end;
    justify-content: center;
    margin-left: auto;
    
    @include breakpoint(mobile) {
        margin-right: 10px;
    }
    margin-right: 15px;
    @include breakpoint(laptop) {
        margin-right: 30px;
    }
    
    &--open {
        color: $open-red;
    }

    &__welcome-container {
        display: flex;
        flex-direction: column;
        flex-wrap: wrap;

        justify-content: center;
        align-content: flex-end;

        height: 100%;
    }

    &__avatar {
		border-radius: 100%;
        max-height: 100%;

        flex-basis: 100%;

        padding: 10px 0;

        margin: 0 5px;
        @include breakpoint(tablet) {
            margin: 0 10px;
        }
	}
    
    &__link {
        display: flex;
        align-items: center;

        font-weight: bold;
        &--light {
            color: #ffa1a1;
        }
        &--dark {
            color: white;
        }

        &--open {
            color: $open-red;
        }
    }

    &__line {
        &--corsace {
            color: $pink;
        }
        &--mca-ayim {
            color: $blue;
        }
        padding: 0 2.5px;
        @include breakpoint(laptop) {
            padding: 0 5px;
        }
    }

    &__welcome {
        &--light {
            color: #ffa1a1;
            & > a {
                color: #ffa1a1;
            }
        }
        &--dark {
            color: #6f6f6f;
            & > a {
                color: #6f6f6f;
            }
        }
        &--open {
            color: $open-red;
            font-stretch: condensed;
        }
    }

    &__menu {
        cursor: pointer;
        position: relative;
        display: flex;
        width: fit-content;
        gap: 5px;
        align-items: center;
        flex-direction: column;
        @include breakpoint(laptop) {
            flex-direction: row;
        }

        @include breakpoint(mobile) {
            font-size: $font-sm;
        }
        font-size: 0.8rem;
        @include breakpoint(laptop) {
            font-size: $font-base;
        }

        &_dropdown {
            position: absolute;
            padding: 10px;
            display: flex;
            flex-direction: column;
            gap: 10px;
            z-index: 1;

            background-color: white;
            box-shadow: 0px 4px 4px 0px rgba(0,0,0,0.25);
            font-stretch: condensed;
            & > a {
                color: black;
            }
            &--dark {
                background-color: $dark;
                & > a {
                    color: white;
                }
            }
            &--open {
                background-color: white;
                & > a {
                    color: $open-red;
                }
            }
        }
    }

    &__username {
        color: #ffffff;
        text-transform: uppercase;
        text-overflow: ellipsis;
        &--open {
            font-weight: bold;
        }
        @include breakpoint(mobile) {
            font-size: $font-sm;
        }
    }
}

.hamburger {
    position: relative;
    color: #ffa1a1;
    font-size: 1.5rem;
    font-weight: bold;
    &--dark {
        color: #6f6f6f;
    }

    &--open {
        color: #ffa1a1;
    }
}

.notif {
    position: absolute;
    height: 8px;
    width: 8px;
    border-radius: 100%;
    top: 0.5rem;
    right: -0.25rem;
    background-color: black;
    &--dark {
        background-color: white;
    }

    &--open {
        background-color: $open-red;
    }
}
</style>