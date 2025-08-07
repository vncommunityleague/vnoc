<template>
    <div class="players_list">
        <!--        <SubHeader-->
        <!--            :selections="[-->
        <!--                { text: $t('open.players.playerList'), value: 'list' },-->
        <!--                { text: $t('open.players.teamManagement'), value: 'management' },-->
        <!--            ]"-->
        <!--            :current-page="page"-->
        <!--            @update:page="page = $event"-->
        <!--        />-->
        <div
            v-if="page === 'list'"
            class="players_list__main_content"
        >
            <OpenTitle>
                {{ $t("open.players.playerList") }}
                <template #right>
                    <OpenFilter>
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
                        </template>
                    </OpenFilter>
                    <SearchBar
                        :placeholder="`${$t('open.players.searchPlaceholder')}`"
                        style="margin-bottom: 10px;"
                        @update:search="searchValue = $event"
                    />
                </template>
            </OpenTitle>
            <div
                v-if="filteredplayers.length !== 0"
                class="players_list__main_content_list"
            >
                <OpenCardTeam
                    v-for="team in filteredplayers"
                    :key="team.ID"
                    :team="team"
                    registered
                />
            </div>
            <div
                v-else-if="loading"
                class="players_list__main_content"
            >
                {{ $t("open.status.loading") }}...
            </div>
            <div
                v-else
                class="players_list__main_content"
            >
                {{ $t("open.players.noRegisteredPlayers") }}
            </div>
        </div>
        <div
            v-else-if="page === 'management' && loggedInUser?.discord.userID"
            class="players_list__main_content"
        >
            <OpenTitle>
                {{ $t("open.players.teamManagement") }}
                <template #right>
                    <ContentButton
                        class="content_button--red"
                        :link="'team/create'"
                    >
                        {{ $t("open.create.title") }}
                    </ContentButton>
                </template>
            </OpenTitle>
            <div
                v-if="filteredplayers.length !== 0"
                class="players_list__main_content_list"
            >
                <OpenCardTeam
                    v-for="team in filteredplayers"
                    :key="team.ID"
                    :team="team"
                    :registered="playerList && playerList.some(t => t.ID === team.ID)"
                />
            </div>
            <div
                v-else
                class="players_list__main_content"
            >
                {{ $t("open.players.noCreatedplayers") }}
            </div>
        </div>
        <div
            v-else-if="page === 'management'"
            class="players_list__main_content"
        >
            {{ $t("open.players.loginManagement") }}
        </div>
        <div
            v-else-if="loading"
            class="players_list__main_content"
        >
            {{ $t("open.status.loading") }}...
        </div>
        <div
            v-else
            class="players_list__main_content"
        >
            {{ $t("open.players.error") }}...
        </div>
    </div>
</template>

<script lang="ts">
import { Mixins, Component } from "vue-property-decorator";
import { State, namespace } from "vuex-class";
import { ExtendedPublicationContext } from "centrifuge";
import CentrifugeMixin from "../../Assets/mixins/centrifuge";

import { Tournament } from "../../Interfaces/tournament";
import { Team, TeamList } from "../../Interfaces/team";
import { UserInfo } from "../../Interfaces/user";

import OpenFilter from "../../Assets/components/open/OpenFilter.vue";
import SearchBar from "../../Assets/components/SearchBar.vue";
import OpenTitle from "../../Assets/components/open/OpenTitle.vue";
import ContentButton from "../../Assets/components/open/ContentButton.vue";
import OpenCardTeam from "../../Assets/components/open/OpenCardTeam.vue";
import SubHeader from "../../Assets/components/open/SubHeader.vue";

const openModule = namespace("open");

@Component({
    components: {
        OpenFilter,
        SearchBar,
        OpenTitle,
        ContentButton,
        OpenCardTeam,
        SubHeader,
    },
    head () {
        return {
            title: this.$store.state.open.title,
            meta: [
                {
                    hid: "description",
                    name: "description",
                    content: this.$store.state.open.tournament?.description || "",
                },

                { hid: "og:site_name", property: "og:site_name", content: this.$store.state.open.title },
                { hid: "og:title", property: "og:title", content: this.$store.state.open.title },
                { hid: "og:url", property: "og:url", content: `https://vnoc.hoaq.works${this.$route.path}` },
                {
                    hid: "og:description",
                    property: "og:description",
                    content: this.$store.state.open.tournament?.description || "",
                },
                { hid: "og:image", property: "og:image", content: require("../../Assets/img/site/open/banner.png") },

                { name: "twitter:title", content: this.$store.state.open.title },
                { name: "twitter:description", content: this.$store.state.open.tournament?.description || "" },
                { name: "twitter:image", content: require("../../Assets/img/site/open/banner.png") },
                { name: "twitter:image:src", content: require("../../Assets/img/site/open/banner.png") },
            ],
            link: [ { rel: "canonical", hid: "canonical", href: `https://vnoc.hoaq.works${this.$route.path}` } ],
        };
    },
})
export default class Players extends Mixins(CentrifugeMixin) {

    @State loggedInUser!: null | UserInfo;
    @openModule.State tournament!: Tournament | null;
    @openModule.State myplayers!: Team[] | null;
    @openModule.State("teamList") playerList!: TeamList[] | null;

    loading = true;
    showUnregistered = false;
    sortDir: "ASC" | "DESC" = "ASC";
    // sorts = ["RANK", "BWS AVG", "A-Z", "ID", "TEAM SIZE"] as const;
    sorts = [ "RANK", "A-Z", "ID" ] as const;
    sortFunctions: Record<typeof this.sorts[number], (a: TeamList, b: TeamList) => number> = {
        "RANK": (a, b) => a.rank - b.rank,
        // "BWS AVG": (a, b) => a.BWS - b.BWS,
        "A-Z": (a, b) => a.name.localeCompare(b.name),
        "ID": (a, b) => a.ID - b.ID,
        // "TEAM SIZE": (a, b) => a.members.length - b.members.length,
    };
    currentSort: typeof this.sorts[number] = "RANK";
    searchValue = "";
    page: "list" | "management" = "list";
    unregisteredPlayers: TeamList[] | null = null;

    get filteredplayers () {
        if (this.page === "management")
            return this.myplayers ?? [];
        let players = [ ...(this.playerList ?? []) ];
        if (this.showUnregistered && this.unregisteredPlayers)
            players = [ ...players, ...this.unregisteredPlayers ];
        if (!this.searchValue) // consider asc/desc
            return players.sort((a, b) => this.sortDir === "ASC" ? this.sortFunctions[this.currentSort](a, b) : this.sortFunctions[this.currentSort](b, a));

        return players.filter(team =>
            team.name.toLowerCase().includes(this.searchValue.toLowerCase()) ||
            team.members.some(member => member.username.toLowerCase().includes(this.searchValue.toLowerCase())) ||
            team.ID.toString().includes(this.searchValue.toLowerCase()) ||
            team.members.some(member => member.osuID.toLowerCase().includes(this.searchValue.toLowerCase()))
        ).sort((a, b) => this.sortFunctions[this.currentSort](a, b) * (this.sortDir === "ASC" ? 1 : - 1));
    }

    async mounted () {
        if (this.$route.query.s === "my")
            this.page = "management";
        this.loading = true;
        if (this.tournament) {
            await this.$store.dispatch("open/setTeamList", this.tournament.ID);
            const { data } = await this.$axios.get<{
                teams: TeamList[]
            }>(`/api/tournament/${this.tournament.ID}/unregisteredTeams`);
            if (!data.success) {
                alert("Failed to get unregistered players, check console for more information");
                console.error(data.error);
                return;
            }
            this.unregisteredPlayers = data.teams;
        }
        this.loading = false;

        if (this.tournament)
            await this.initCentrifuge(`teams:${this.tournament.ID}`);
    }

    handleData (ctx: ExtendedPublicationContext) {
        if (ctx.data.type === "teamRegistered")
            this.$store.commit("open/addTeamList", ctx.data.team);
    }
}
</script>

<style lang="scss">
@import '@s-sass/_variables';

.players_list {

    &__button {
        min-width: 150px;
        text-transform: uppercase;
        padding: 0;
    }

    &__main_content {
        align-self: center;
        position: relative;
        width: 65vw;
        padding: 35px;

        &_list {
            display: flex;
            flex-wrap: wrap;
            gap: 20px;
            justify-content: flex-start;
            margin-top: 25px;
        }
    }
}
</style>
