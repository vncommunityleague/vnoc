<template>
    <div class="index">
        <img 
            class="index__background" 
            src="../../Assets/img/site/open/background.png"
            alt=""
        > 
        <div class="index__content">
            <div
                class="index__banner"
                @click="increaseCounter">
                <img src="../../Assets/img/site/open/banner.png">
            </div>
            <div class="index_portal">
                <div class="index_portal__section">
                    <div class="index_schedule">
                        <div class="index_schedule--xl">
                            {{ $t("open.home.timeline") }}
                        </div>
                        <hr class="line--red line--no-space">
                        <ul class="index_schedule__content">
                            <li
                                v-for="time in tournament?.schedule || []"
                                :key="time.event"
                                class="index_schedule__group"
                            >
                                <span class="index_schedule__event"> {{ time.event }} </span>
                                <span class="index_schedule__line"/>
                                <span class="index_schedule__time">
                                    {{
                                        new Date(time.start || "").toLocaleString("en-US", optionsRange)
                                    }} - {{ new Date(time.end || "").toLocaleString("en-US", optionsRange) }}
                                </span>
                            </li>
                        </ul>
                    </div>
                </div>
                <!--- MIDDLE: middle-->
                <div class="index_portal__section">
                    <OpenFrontPageButton
                        :link="'/qualifiers'"
                    >
                        <template #title>
                            {{ $t("open.home.button.qualifiersMappool") }}
                        </template>
                        {{ $t("open.home.button.mappoolDownload") }}
                    </OpenFrontPageButton>
                    <!--                    <OpenFrontPageButton-->
                    <!--                        disabled-->
                    <!--                    />-->
                    <div class="index_portal__text-content">
                        {{ $t("open.home.presentedBy") }}
                        <hr class="line--red line--no-space">
                        <div class="index_portal__image index_portal__image--row">
                            <a
                                href="http://wooting.io"
                                target="_blank"
                            >
                                <img src="../../Assets/img/partners/wooting.png">
                            </a>
                        </div>
                    </div>
                </div>
                <!-- RIGHT: register your team-->
                <!--                <div class="index_portal__section">-->
                <!--                    <OpenFrontPageButton-->
                <!--                        :link="loggedInUser ? loggedInUser.discord.username ? '/team/create' : '/api/login/discord?site=vnoc&redirect=/team/create' : '/api/login/osu?site=vnoc&redirect=/'"-->
                <!--                        :external="loggedInUser?.discord.username ? false : true"-->
                <!--                    >-->
                <!--                        <template #title>-->
                <!--                            {{ $t("open.home.button.register") }}-->
                <!--                        </template>-->
                <!--                        {{-->
                <!--                            $t("open.home.button.registrationsEnd", { time: new Date(tournament?.registrations.end || "").toLocaleString("en-US", options) })-->
                <!--                        }}-->
                <!--                    </OpenFrontPageButton>-->
                <!--                </div>-->
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { State, namespace } from "vuex-class";

import { Tournament } from "../../Interfaces/tournament";
import { UserInfo } from "../../Interfaces/user";

import OpenFrontPageButton from "../../Assets/components/open/OpenFrontPageButton.vue";

const openModule = namespace("open");

@Component({
    components: {
        OpenFrontPageButton,
    },
    head () {
        return {
            title: this.$store.state.open.title,
            meta: [
                { hid: "description", name: "description", content: "Giải đấu osu! standard hàng đầu Việt Nam." || "" },

                { hid: "og:site_name", property: "og:site_name", content: "Vietnam osu! Championship 2025" },
                { hid: "og:title", property: "og:title", content: "Vietnam osu! Championship 2025" },
                { hid: "og:url", property: "og:url", content: `https://vnoc.hoaq.works${this.$route.path}` },
                {
                    hid: "og:description",
                    property: "og:description",
                    content: "Vietnam osu! Championship 2025 là giải đấu hàng đầu về osu! standard dành riêng cho cộng đồng người chơi tại Việt Nam.",
                },
                { hid: "og:image", property: "og:image", content: require("../../Assets/img/site/open/banner.png") },

                { name: "twitter:title", content: "Vietnam osu! Championship 2025" },
                {
                    name: "twitter:description",
                    content: "Vietnam osu! Championship 2025 là giải đấu hàng đầu về osu! standard dành riêng cho cộng đồng người chơi tại Việt Nam." || "",
                },
                { name: "twitter:image", content: require("../../Assets/img/site/open/banner.png") },
                { name: "twitter:image:src", content: require("../../Assets/img/site/open/banner.png") },
            ],
            link: [ { rel: "canonical", hid: "canonical", href: `https://vnoc.hoaq.works` } ],
        };
    },
})
export default class Default extends Vue {

    options: Intl.DateTimeFormatOptions = {
        month: "long", // Full month name (e.g., "July")
        day: "numeric", // Day of the month (e.g., "30")
        hour: "2-digit", // Two-digit hour (e.g., "23")
        hour12: false, // 24-hour format hour
        minute: "2-digit", // Two-digit minute (e.g., "59")
        timeZone: "UTC", // Set the time zone to UTC
        timeZoneName: "short", // Abbreviated time zone name (e.g., "UTC")
    };

    optionsRange: Intl.DateTimeFormatOptions = {
        month: "short", // Full month name (e.g., "July")
        day: "numeric", // Day of the month (e.g., "01")
    };

    @openModule.State tournament!: Tournament | null;

    @State loggedInUser!: UserInfo | null;

    @State counter = 0;

    get avatarURL (): string {
        return this.loggedInUser?.osu.avatar ?? "";
    }

    async mounted () {
        await this.$store.dispatch("setInitialData", "vnoc");
    }

    increaseCounter () {
        this.counter ++;

        if (this.counter < 6) {
            return;
        }

        window.open("https://www.twitch.tv/vncommunityleague/schedule", "_blank");
        this.counter = 0;
    }
}
</script>

<style lang="scss">
@import '@s-sass/_variables';

.index {
    //background: $dark;
    position: relative;
    overflow: hidden;
    text-align: center;
    display: flex;
    align-items: center;
    justify-content: center;
    height: 100%;

    &__background {
        position: absolute;
        top: 0;
        left: 0;
        object-fit: cover;
        height: 100%;
        width: 100%;
        mask-image: linear-gradient(180deg, #131313 0%, rgba(19, 19, 19, 0.075) 90%, rgba(19, 19, 19, 0) 100%);
    }

    &__content {
        position: relative;
        display: flex;
        flex-direction: column;
    }

    &__banner {
        display: flex;
        align-self: center;
        flex-direction: column;
        margin-top: 50px;
        gap: 50px;
    }

    &_portal {
        width: 80vw;
        display: flex;
        flex-direction: row;
        justify-content: center;
        column-gap: 12px;

        &__section {
            width: 25vw;
        }

        &__image {
            &--row {
                display: flex;
                justify-content: center;
                padding: 25px 50px;
            }

            & img {
                height: 38px;
                width: auto;
            }
        }

        &__text-content {
            margin: 50px 0;
        }

    }

    &_schedule {
        margin: 30px 0px;
        text-align: start;
        width: 75%;

        &--xl {
            color: $white;
            padding: 5px 0px;
            text-align: start;
            font-size: $font-xl;
            letter-spacing: 0.23em;
        }

        &__content {
            padding: 0;
            display: flex;
            flex-direction: column;
            justify-content: space-between;
            align-items: stretch;
        }

        &__group {
            display: flex;
            justify-content: space-between;
            align-items: center;
            padding: 10px 0;
        }

        &__event {
            text-align: left;
            padding-right: 0.2em;
        }

        &__line {
            flex-grow: 1;
            height: 1rem;
            background-image: url("data:image/svg+xml;base64,PHN2ZyB4bWxucz0iaHR0cDovL3d3dy53My5vcmcvMjAwMC9zdmciIHdpZHRoPSI0IiBoZWlnaHQ9IjEiPgogIDxsaW5lIHgxPSIyIiB5MT0iMSIgeDI9IjQiIHkyPSIxIiBzdHlsZT0ic3Ryb2tlOiM2OTY5Njk7IHN0cm9rZS13aWR0aDoxIiAvPgo8L3N2Zz4K");
            background-repeat: repeat-x;
            margin: 0 0.2em;
            background-position: bottom;
        }

        &__time {
            text-align: right;
            white-space: nowrap;
            padding-left: 0.2em;
        }
    }
}
</style>