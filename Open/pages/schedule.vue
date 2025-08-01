<template>
    <div class="schedule">
        <SubHeader
            :selections="[
                { text: $t('open.schedule.nav.schedule'), value: 'schedule' },
                { text: $t('open.schedule.nav.brackets'), value: 'brackets' },
            ]"
            :current-page="page"
            @update:page="changePage"
        />
        <div class="schedule_main_content">
            <OpenTitle>
                {{ $t(`open.schedule.nav.${page}`) }} - <span class="schedule_main_content__abbreviation">{{ selectedStage?.abbreviation.toUpperCase() || '' }}</span>
                <template #right>
                    <OpenFilter v-if="page === 'schedule'">
                        <template #view>
                            <div
                                :class="{ 'open_filter__selected': view === 'ALL' }"
                                @click="view = 'ALL'"
                            >
                                {{ $t("open.schedule.all") }}
                            </div>
                            <div 
                                :class="{ 'open_filter__selected': view === 'UPCOMING' }"
                                @click="view = 'UPCOMING'"
                            >
                                {{ $t("open.schedule.upcoming") }}
                            </div>
                            <div 
                                :class="{ 'open_filter__selected': view === 'ONGOING' }"
                                @click="view = 'ONGOING'"
                            >
                                {{ $t("open.schedule.ongoing") }}
                            </div>
                            <div 
                                :class="{ 'open_filter__selected': view === 'PAST' }"
                                @click="view = 'PAST'"
                            >
                                {{ $t("open.schedule.past") }}
                            </div>
                            <div class="open_filter__separator" />
                            <div
                                style="word-spacing: 1000px;"
                                :style="{ fontWeight: hidePotentials ? 'bold' : 'normal' }"
                                @click="hidePotentials = !hidePotentials"
                            >
                                {{ $t("open.schedule.hidePotentials") }}
                            </div>
                            <div class="open_filter__separator" />
                            <div
                                v-if="loggedInUser"
                                :style="{ fontWeight: myMatches ? 'bold' : 'normal' }"
                                @click="myMatches = !myMatches"
                            >
                                {{ $t("open.schedule.myMatches") }}
                            </div>
                            <div
                                v-if="loggedInUser?.staff.staff"
                                :style="{ fontWeight: myStaff ? 'bold' : 'normal' }"
                                @click="myStaff = !myStaff"
                            >
                                {{ $t("open.schedule.myMatchesStaff") }}
                            </div>
                        </template>
                        <template #sort>
                            <div
                                v-for="sort in sorts"
                                :key="sort"
                                :class="{ 'open_filter__selected': currentSort === sort }"
                                @click="currentSort === sort ? (sortDir = sortDir === 'ASC' ? 'DESC' : 'ASC') : (currentSort = sort)"
                            >
                                <div
                                    v-if="currentSort === sort"
                                    class="open_filter__arrows"
                                >
                                    <div :class="{ 'open_filter__arrows--selected': sortDir === 'ASC' }">
                                        ▲
                                    </div>
                                    <div :class="{ 'open_filter__arrows--selected': sortDir === 'DESC' }">
                                        ▼
                                    </div>
                                </div>
                                {{ $t(`open.components.filter.sorts.${sort}`) }}
                            </div>
                            <div class="open_filter__separator" />
                            <div 
                                class="schedule__matchID_filter"
                                style="cursor: default;"
                            >
                                {{ $t("open.schedule.matchID") }}
                                <div
                                    style="text-align: right;"
                                    @click="selectedMatchIDs = visibleMatchIDs.reduce((acc, id) => ({ ...acc, [id]: !selectedMatchIDs[id] }), {})"
                                >
                                    {{ $t("open.schedule.invertSelection") }}
                                </div>
                                <div
                                    class="schedule__matchID_filter_container"
                                    style="cursor: default;"
                                >
                                    <div
                                        v-for="matchID in visibleMatchIDs"
                                        :key="matchID"
                                        class="schedule__matchID_filter__selection"
                                        :class="{ 'schedule__matchID_filter__selection--selected': selectedMatchIDs[matchID] }"
                                        @click="selectedMatchIDs[matchID] = !selectedMatchIDs[matchID]"
                                    >
                                        {{ matchID }}
                                    </div>
                                </div>
                            </div>
                        </template>
                    </OpenFilter>
                    <OpenFilter v-else-if="page === 'brackets' && selectedStage?.stageType === 3">
                        <template #view>
                            <div style="cursor: default;">
                                {{ $t("open.schedule.group") }}
                                <div
                                    class="schedule__matchID_filter_container"
                                    style="cursor: default;"
                                >
                                    <div
                                        v-for="matchID in visibleMatchIDs"
                                        :key="matchID"
                                        class="schedule__matchID_filter__selection"
                                        :class="{ 'schedule__matchID_filter__selection--selected': currGroup === matchID }"
                                        @click="currGroup = matchID"
                                    >
                                        {{ matchID }}
                                    </div>
                                </div>
                            </div>
                        </template>
                    </OpenFilter>
                    <StageSelector
                        :not-beginning="selectedStage?.ID !== stageList[0]?.ID"
                        :not-end="selectedStage?.ID !== stageList[stageList.length - 1]?.ID"
                        @prev="index--"
                        @next="index++"
                    >
                        <template #text>
                            {{ $t("open.components.stageSelector") }}
                        </template>

                        <template #stage>
                            {{ selectedStage?.abbreviation.toUpperCase() || '' }}
                        </template>
                    </StageSelector>
                    <SearchBar
                        v-if="page === 'schedule'"
                        :placeholder="`${$t('open.teams.searchPlaceholder')}`"
                        @update:search="searchValue = $event"
                    />
                    <!-- TODO: NOT MAKE THIS A STATIC LINK LOL -->
                    <ContentButton
                        class="content_button--red"
                        :link="'https://docs.google.com/spreadsheets/d/1h2akeQIgphGPXFe0995p5ujangNYdKotvzp2QvBESWE'"
                        :img-src="require('../../Assets/img/site/open/mappool/sheets-ico.svg')"
                        external
                    >
                        {{ $t('open.qualifiers.mappool.sheets') }}
                    </ContentButton>
                </template>
            </OpenTitle>
            <div
                v-if="loading"
            >
                {{ $t('open.status.loading') }}...
            </div>
            <div
                v-if="page === 'schedule'"
                class="schedule_main_content_matches"
            >
                <ScheduleMatchBox
                    v-for="matchup in filteredMatchups"
                    :key="matchup.ID"
                    :matchup="matchup"
                    :tbddate="(matchupList?.filter(m => m.potentialFor === matchup.matchID)?.map(m => m.date.getUTCDate()).filter((date, i, arr) => arr.indexOf(date) === i) || []).length > 1"
                    :tbdtime="(matchupList?.filter(m => m.potentialFor === matchup.matchID)?.map(m => m.date.getTime()).filter((date, i, arr) => arr.indexOf(date) === i) || []).length > 1"
                    @update:matchup="updateMatchup()"
                />
            </div>
            <div
                v-else-if="page === 'brackets'"
                ref="bracketContainer"
                class="schedule_main_content_brackets"
            >
                <div class="schedule_main_content_brackets__scrollbar_container">
                    <div
                        ref="bracketScroll"
                        class="schedule_main_content_brackets__scrollbar"
                        draggable
                        @dragstart="scrollStart"
                        @drag="scrollBracket"
                        @dragend="reAddScroll"
                    />
                </div>
                <RoundRobinView
                    v-if="selectedStage?.stageType === 3"
                    :matchups="matchupList"
                    :current="currGroup"
                    @change="currGroup = $event"
                />
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, Watch } from "vue-property-decorator";
import { State, namespace } from "vuex-class";

import OpenTitle from "../../Assets/components/open/OpenTitle.vue";
import StageSelector from "../../Assets/components/open/StageSelector.vue";
import ScheduleMatchBox from "../../Assets/components/open/ScheduleMatchBox.vue";
import ContentButton from "../../Assets/components/open/ContentButton.vue";
import OpenFilter from "../../Assets/components/open/OpenFilter.vue";
import SearchBar from "../../Assets/components/SearchBar.vue";
import SubHeader from "../../Assets/components/open/SubHeader.vue";
import RoundRobinView from "../../Assets/components/open/RoundRobinView.vue";

import { Tournament } from "../../Interfaces/tournament";
import { Stage, StageType } from "../../Interfaces/stage";
import { MatchupList, matchIDAlphanumericSort } from "../../Interfaces/matchup";
import { UserInfo } from "../../Interfaces/user";

const openModule = namespace("open");

@Component({
    components: {
        StageSelector,
        OpenTitle,
        ScheduleMatchBox,
        ContentButton,
        OpenFilter,
        SearchBar,
        SubHeader,
        RoundRobinView,
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
                {hid: "og:image",property: "og:image", content: require("../../Assets/img/site/open/banner.png")},
                
                {name: "twitter:title", content: this.$store.state.open.title},
                {name: "twitter:description", content: this.$store.state.open.tournament?.description || ""},
                {name: "twitter:image", content: require("../../Assets/img/site/open/banner.png")},
                {name: "twitter:image:src", content: require("../../Assets/img/site/open/banner.png")},
            ],
            link: [{rel: "canonical", hid: "canonical", href: `https://vnoc.hoaq.works${this.$route.path}`}],
        };
    },
})
export default class Schedule extends Vue {
    
    @State loggedInUser!: UserInfo | null;
    @openModule.State tournament!: Tournament | null;
    @openModule.State matchupList!: MatchupList[] | null;

    loading = false;
    stageList: Stage[] = [];
    index = 0;
    searchValue = "";
    page: "schedule" | "brackets" = "schedule";
    view: "ALL" | "UPCOMING" | "ONGOING" | "PAST" = "ALL";
    hidePotentials = false;
    myMatches = false;
    myStaff = false;
    sortDir: "ASC" | "DESC" = "ASC";
    sorts = ["DATETIME", "MATCHID", "RANK AVERAGE", "BWS AVERAGE", "RANK DIFF", "BWS DIFF"] as const;
    sortFunctions: Record<typeof this.sorts[number], (a: MatchupList, b: MatchupList) => number> = {
        DATETIME: (a, b) => a.date.getTime() - b.date.getTime(),
        MATCHID: matchIDAlphanumericSort,
        "RANK AVERAGE": (a, b) => !a.teams || !b.teams ? 0 : a.teams.reduce((acc, team) => acc + team.rank, 0) / (a.teams.length || 1) - b.teams.reduce((acc, team) => acc + team.rank, 0) / (b.teams.length || 1),
        "BWS AVERAGE": (a, b) => !a.teams || !b.teams ? 0 : a.teams.reduce((acc, team) => acc + team.BWS, 0) / (a.teams.length || 1) - b.teams.reduce((acc, team) => acc + team.BWS, 0) / (b.teams.length || 1),
        "RANK DIFF": (a, b) => !a.teams || !b.teams ? 0 : (Math.max(...a.teams.map(team => team.rank)) - Math.min(...a.teams.map(team => team.rank))) - (Math.max(...b.teams.map(team => team.rank)) - Math.min(...b.teams.map(team => team.rank))),
        "BWS DIFF": (a, b) => !a.teams || !b.teams ? 0 : (Math.max(...a.teams.map(team => team.BWS)) - Math.min(...a.teams.map(team => team.BWS)) - (Math.max(...b.teams.map(team => team.BWS)) - Math.min(...b.teams.map(team => team.BWS)))),
    };
    currentSort: typeof this.sorts[number] = "DATETIME";
    
    get selectedStage (): Stage | null {
        return this.stageList[this.index] || null;
    }

    get filteredMatchups () {
        if (!this.matchupList) return [];
        return this.matchupList.filter(matchup => {
            if (matchup.matchID && this.visibleMatchIDs.length > 0 && !this.selectedMatchIDs[matchup.matchID[0]]) return false;
            if (this.myMatches && !matchup.teams?.some(team => team.members.some(player => player.osuID === this.loggedInUser?.osu.userID))) return false;
            if (this.myStaff && (matchup.referee?.ID !== this.loggedInUser?.ID && matchup.streamer?.ID !== this.loggedInUser?.ID && !matchup.commentators?.some(comm => comm.ID === this.loggedInUser?.ID))) return false;
            if (this.hidePotentials && matchup.potentialFor) return false;
            if (this.searchValue && !(
                matchup.matchID.toLowerCase().includes(this.searchValue.toLowerCase()) || 
                matchup.ID.toString().includes(this.searchValue) ||
                matchup.teams?.some(team => team.ID.toString().includes(this.searchValue)) ||
                matchup.teams?.some(team => team.name.toLowerCase().includes(this.searchValue.toLowerCase())) ||
                matchup.teams?.some(team => team.members.some(player => player.osuID.includes(this.searchValue))) ||
                matchup.teams?.some(team => team.members.some(player => player.username.toLowerCase().includes(this.searchValue.toLowerCase())))
            ))
                return false;
            if (this.view === "ALL") return true;
            if (this.view === "UPCOMING") return matchup.date.getTime() > Date.now();
            if (this.view === "ONGOING") return matchup.date.getTime() < Date.now() && !matchup.mp && !matchup.forfeit;
            if (this.view === "PAST") return matchup.date.getTime() < Date.now();
            return false;
        }).sort((a, b) => this.sortFunctions[this.currentSort](a, b) * (this.sortDir === "ASC" ? 1 : -1));
    }

    visibleMatchIDs: string[] = [];
    selectedMatchIDs: Record<string, boolean> = {};
    currGroup = "";

    @Watch("selectedStage")
    async stageMatchups () {
        if (!this.selectedStage) {
            this.$store.commit("open/setMatchups", []);
            return;
        }
        
        this.loading = true;
        this.$store.commit("open/setMatchups", []);

        await this.$store.dispatch("open/setMatchups", this.selectedStage?.ID);
        const matchupIDSet = new Set<string>();
        for (const matchup of this.matchupList ?? []) {
            if (matchup.matchID && isNaN(parseInt(matchup.matchID)))
                matchupIDSet.add(matchup.matchID[0]);
        }
        this.visibleMatchIDs = Array.from(matchupIDSet).sort();
        this.selectedMatchIDs = this.visibleMatchIDs.reduce((acc, id) => ({ ...acc, [id]: true }), {});
        if (this.visibleMatchIDs.length)
            this.currGroup = this.visibleMatchIDs[0];
        this.loading = false;
    }

    changePage (page: "schedule" | "brackets") {
        if (page === this.page)
            return;
        if (page === "brackets" && !this.selectedStage)
            return;
        this.page = page;
    }

    mounted () {
        this.stageList = this.tournament?.stages.filter(stage => stage.stageType !== StageType.Qualifiers) ?? [];
        this.index = this.stageList.findIndex(stage => stage.timespan.end.getTime() > Date.now());
        if (this.index === -1)
            this.index = this.stageList.length - 1;
    }

    updated () {
        const bracketContainer = this.$refs.bracketContainer;
        const bracketScroll = this.$refs.bracketScroll;
        if (bracketContainer instanceof HTMLElement && bracketScroll instanceof HTMLElement) {
            bracketContainer.onscroll = null;
            if (bracketContainer.scrollWidth === bracketContainer.clientWidth)
                bracketScroll.style.display = "none";
            else
                bracketScroll.style.display = "block";
            if (!bracketContainer.children[1])
                return;
            const childWidth = bracketContainer.children[1].scrollWidth;
            bracketScroll.parentElement!.style.width = `${childWidth}px`;
            bracketContainer.onscroll = () => {
                bracketScroll.style.left = `${Math.max(0, Math.min(bracketContainer.scrollLeft / bracketContainer.scrollWidth * childWidth, childWidth))}px`;
            };
        }
    }

    scrollStart (event: Event) {
        if (!(event instanceof DragEvent) || !event.dataTransfer)
            return;
        event.dataTransfer.setData("text", "scroll");
        event.dataTransfer.effectAllowed = "move";
        event.dataTransfer.dropEffect = "move";
        const bracketContainer = this.$refs.bracketContainer;
        if (bracketContainer instanceof HTMLElement)
            bracketContainer.onscroll = null;
    }

    scrollBracket (event: Event) {
        if (!(event instanceof DragEvent))
            return;
        
        event.dataTransfer!.dropEffect = "move";
        const bracketContainer = this.$refs.bracketContainer;
        const bracketScroll = this.$refs.bracketScroll;
        if (bracketContainer instanceof HTMLElement && bracketScroll instanceof HTMLElement) {
            const rect = bracketContainer.getBoundingClientRect();
            const left = Math.min(Math.max(0, event.x - rect.x - 0.05 * rect.width), 0.95 * rect.width);
            bracketScroll.style.left = `${left}px`;
            bracketContainer.scrollLeft = left / rect.width * (bracketContainer.scrollWidth - bracketContainer.clientWidth);
        }
    }

    reAddScroll () {
        const bracketContainer = this.$refs.bracketContainer;
        if (bracketContainer instanceof HTMLElement)
            bracketContainer.onscroll = () => {
                const bracketScroll = this.$refs.bracketScroll;
                if (bracketScroll instanceof HTMLElement)
                    bracketScroll.style.left = `${bracketContainer.scrollLeft / (bracketContainer.scrollWidth - bracketContainer.clientWidth) * bracketContainer.clientWidth}px`;
            };
    }

    async updateMatchup () {
        await this.$store.dispatch("open/setMatchups", this.selectedStage?.ID);
    }
}

</script>

<style lang="scss">
@import '@s-sass/_variables';

.schedule {

    &_main_content {
        display: flex;
        align-self: center;
        justify-content: center;
        flex-direction: column;
        width: 75vw;
        position: relative;
        padding: 35px;

        &__abbreviation {
            color: $open-red;
        }

        @media screen and (max-width: $breakpoint-xl) {
            width: 100vw;
        }

        &_matches {
            display: flex;
            flex-direction: column;
            margin-top: 20px;
            gap: 20px;
        }

        &_brackets {
            margin-top: 20px;
            overflow-x: auto;
            scrollbar-width: none;
            position: relative;

            &__scrollbar {
                position: absolute;
                border-radius: 5px;
                background-color: #272727;
                width: 5%;
                height: 10px;
                transition: none;
                box-shadow: 0 4px 4px rgba(0, 0, 0, 0.25);

                &_container {
                    width: 100%;
                    position: relative;
                    z-index: 3;
                }
            }
        }
    }

    &__matchID_filter {
        display: flex;
        flex-direction: column;
        gap: inherit;
        width: 100%;
        align-items: flex-end;

        &_container {
            cursor: default;
            width: 100%;
            display: flex;
            flex-wrap: wrap;
            gap: 2px;
        }

        &__selection {
            aspect-ratio: 1/1;
            height: fit-content;
            padding: 3px;
            border: 1px solid $open-dark;
            border-radius: 2px;
            font-size: $font-xsm;

            &--selected {
                background-color: $open-dark;
                color: $open-red;
            }
        }
    }
}
</style>