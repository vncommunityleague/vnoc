<template>
    <div class="qualifier">
        <div 
            v-if="qualifierData"
            class="qualifier__wrapper"
        >
            <div class="qualifier__main_content">
                <OpenTitle>
                    QUALIFIERS
                    <template #right>
                        <ContentButton
                            v-if="qualifierData.mp"
                            class="content_button--red content_button--red_sm"
                            :link="`https://osu.ppy.sh/community/matches/${qualifierData.mp}`"
                            external
                        >
                            osu! MP LINK
                        </ContentButton>
                        <!-- <ContentButton 
                            class="content_button--header_button content_button--disabled"
                            @click.native="togglePopup()"
                        >
                            JOIN
                        </ContentButton> -->
                        <BaseModal
                            v-if="isOpen"
                            @click.native="togglePopup()"
                        >
                            <span>You cannot create/join a qualifier until you have {{ tournament?.minTeamSize === tournament?.maxTeamSize ? tournament?.minTeamSize : tournament?.minTeamSize + " to " + tournament?.maxTeamSize }} players!</span>
                            <span>Press anywhere to close</span>
                        </BaseModal>
                    </template>
                </OpenTitle>
                <div class="qualifier__info_bar">
                    <div class="qualifier__info_bar_group">
                        <div class="qualifier__info_bar_group__title">
                            REFEREES: 
                        </div>
                        <div class="qualifier__info_bar_group__data">
                            {{ qualifierData.referee?.username || "N/A" }}
                        </div>
                    </div>
                    <div class="qualifier__info_bar_group">
                        <div class="qualifier__info_bar_group__title">
                            TEAM: 
                        </div>
                        <div 
                            class="qualifier__info_bar_group"
                            style="flex-direction: column;"
                        >
                            <NuxtLink
                                v-for="team in qualifierData.teams"
                                :key="team.ID"
                                class="qualifier__info_bar_group__data"
                                :to="`/team/${team.ID}`"
                            >
                                {{ team.name || "N/A" }}
                            </NuxtLink>
                        </div>
                    </div>
                    <div class="qualifier__info_bar_time qualifier__info_bar_group__title">
                        {{ qualifierData.date.toLocaleString('en-US', optionsUTC) }} ({{ qualifierData.date.toLocaleString('en-US', options) }})
                    </div>
                </div>
                <div class="qualifier__switch">
                    <ContentButton 
                        class="content_button--header_button content_button--red_sm"
                        :class="{
                            'content_button--red': scoreView === 'players',
                            'content_button--red_outline': scoreView !== 'players',
                        }"
                        @click.native="scoreView = 'players'"
                    >
                        {{ $t('open.qualifiers.scores.players') }}
                    </ContentButton>
                    <ContentButton 
                        class="content_button--header_button content_button--red_sm"
                        :class="{
                            'content_button--red': scoreView === 'teams',
                            'content_button--red_outline': scoreView !== 'teams',
                        }"
                        @click.native="scoreView = 'teams'"
                    >
                        {{ $t('open.qualifiers.scores.teams') }}
                    </ContentButton>
                    <ContentButton
                        class="content_button--red content_button--font_sm"
                        @click.native="placementLock = !placementLock"
                    >
                        {{ placementLock ? $t('open.qualifiers.scores.lockedPlacement') : $t('open.qualifiers.scores.unlockedPlacement') }}
                    </ContentButton>
                </div>
                <ScoresView
                    v-if="mappools?.[0]?.isPublic"
                    :view="scoreView"
                    :placement-lock="false"
                    :pool="mappools[0]"
                    default="zScore"
                />
            </div>
        </div>
        <div 
            v-else-if="loading"
            class="qualifier__wrapper"
        >
            <div class="qualifier__main_content">
                <OpenTitle>
                    LOADING...
                </OpenTitle>
            </div>
        </div>
        <div 
            v-else
            class="qualifier__wrapper"
        >
            <div class="qualifier__main_content">
                <OpenTitle>
                    NO QUALIFIER FOUND
                </OpenTitle>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { namespace } from "vuex-class";

import { Qualifier as QualifierInterface } from "../../../Interfaces/qualifier";
import { Tournament } from "../../../Interfaces/tournament";
import { Stage, StageType } from "../../../Interfaces/stage";
import { Mappool } from "../../../Interfaces/mappool";
import { Team } from "../../../Interfaces/team";

import ContentButton from "../../../Assets/components/open/ContentButton.vue";
import ScoresView from "../../../Assets/components/open/ScoresView.vue";
import OpenTitle from "../../../Assets/components/open/OpenTitle.vue";
import BaseModal from "../../../Assets/components/BaseModal.vue";

const openModule = namespace("open");

@Component({
    components: {
        ContentButton,
        ScoresView,
        OpenTitle,
        BaseModal,
    },
    head () {
        return {
            title: this.$store.state.open.title,
            meta: [
                {hid: "description", name: "description", content: this.$store.state.open.tournament?.description || ""},

                {hid: "og:site_name", property: "og:site_name", content: this.$store.state.open.title},
                {hid: "og:title", property: "og:title", content: this.$store.state.open.title},
                {hid: "og:url", property: "og:url", content: `https://vnoc.hoaq.works${this.$route.path}`},
                {hid: "og:description", property: "og:description", content: this.$store.state.open.tournament?.description || ""},
                {hid: "og:image",property: "og:image", content: require("../../../Assets/img/site/open/banner.png")},
                
                {name: "twitter:title", content: this.$store.state.open.title},
                {name: "twitter:description", content: this.$store.state.open.tournament?.description || ""},
                {name: "twitter:image", content: require("../../../Assets/img/site/open/banner.png")},
                {name: "twitter:image:src", content: require("../../../Assets/img/site/open/banner.png")},
            ],
            link: [{rel: "canonical", hid: "canonical", href: `https://vnoc.hoaq.works${this.$route.path}`}],
        };
    },
    validate ({ params }) {
        return !params.id || !isNaN(parseInt(params.id));
    },
})
export default class Qualifier extends Vue {

    scoreView: "teams" | "players"  = "players";
    placementLock = false;

    @openModule.State tournament!: Tournament | null;
    @openModule.State myTeams!: Team[] | null;
    @openModule.State mappools!: Mappool[] | null;

    loading = false;
    qualifierData: QualifierInterface | null = null;

    optionsUTC: Intl.DateTimeFormatOptions = {
        month: "long", // Full month name (e.g., "July")
        day: "numeric", // Day of the month (e.g., "30")
        hour: "2-digit", // Two-digit hour (e.g., "23")
        hour12: false, // 24-hour format hour
        minute: "2-digit", // Two-digit minute (e.g., "59")
        timeZone: "UTC", // Set the time zone to UTC
        timeZoneName: "short", // Abbreviated time zone name (e.g., "UTC")
    };
    options: Intl.DateTimeFormatOptions = {
        month: "long", // Full month name (e.g., "July")
        day: "numeric", // Day of the month (e.g., "30")
        hour: "2-digit", // Two-digit hour (e.g., "23")
        hour12: false, // 24-hour format hour
        minute: "2-digit", // Two-digit minute (e.g., "59")
        timeZoneName: "short", // Abbreviated time zone name (e.g., "UTC")
    };

    get qualifiersStage (): Stage | null {
        return this.tournament?.stages.find(s => s.stageType === StageType.Qualifiers) ?? null;
    }

    async getQualifier (): Promise<QualifierInterface | null> {
        this.loading = true;
        let ID = 0;
        if (!this.$route.params.id) {
            if (!this.myTeams || !this.myTeams.some(t => t.qualifier?.ID)) {
                this.loading = false;
                return null;
            }
            
            ID = this.myTeams.find(t => t.qualifier?.ID)?.qualifier?.ID ?? 0;
        } else
            ID = parseInt(this.$route.params.id);

        const { data } = await this.$axios.get<{ qualifierData: QualifierInterface }>(`/api/qualifier/${ID}`);
        this.loading = false;
        return !data.success ? null : data.qualifierData;
    }

    async mounted () {
        this.qualifierData = await this.getQualifier();
        if (this.qualifierData) {
            await this.$store.dispatch("open/setMappools", this.qualifiersStage?.ID);
            this.$store.commit("open/setScores", this.qualifierData.scores);
            this.qualifierData.date = new Date(this.qualifierData.date);
        }
    }

    isOpen = false;
    togglePopup () {
        this.isOpen = !this.isOpen;
    }

}
</script>

<style lang="scss">
@import '@s-sass/_variables';

.qualifier {
    height: 100%;

    &__wrapper {
        display: flex;
        justify-content: center;
    }

    &__main_content {
        position: relative;
        width: 65vw;
        padding: 35px;
    }

    &__info_bar {
        margin-top: 20px;
        padding: 5px 10px 5px 10px;
        background: $open-dark;
        width: 100%;
        display: flex;
        flex-direction: row;

        &_group {
            display: flex;
            margin-right: 25px;

            &__title {
                font-family: $font-ggsans;
                font-weight: 700;
                font-size: $font-xl;
                margin-right: 0.5rem;
            }

            &__data {
                font-family: $font-ggsans;
                font-weight: 700;
                font-size: $font-xl;
                color: $open-red;

                &:hover {
                    text-decoration: none;
                }
            }
        }

        &_time {
            flex-grow: 1;
            text-align: right;
        }
    }

    &__switch {
        display: flex;
        justify-content: center;
        cursor: pointer;

        &_item {
            color: $gray;
            background-color: $open-dark;
            margin: 20px 0px 20px 0px;
            padding: 10px 35px 10px 35px;

            &--active {
                color: $open-red;
            }
        }
    }
    
    &__scores {
        overflow: hidden;
    }
}
</style>