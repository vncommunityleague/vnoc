<template>
    <div class="staff">
        <div class="staff_main_content">
            <OpenTitle>
                {{ $t("open.staff.title") }}
            </OpenTitle>
            <div
                v-if="staffList"
                class="staff_main_content_staff_list"
            >
                <div
                    v-for="(staffRow, i) in staffList"
                    :key="i"
                    class="staff_row"
                >
                    <div class="staff_row__title">
                        {{ staffRow.role.toUpperCase() }}
                        <hr class="line--red line--no-space">
                    </div>
                    <div class="staff_row_members">
                        <a
                            v-for="(staffMember, j) in staffRow.users"
                            :key="j"
                            class="staff_row_members_card"
                            :style="{ 'cursor': 'pointer' }"
                            :href="staffMember.url"
                        >
                            <div
                                class="staff_row_members_card__headshot"
                                :style="{ 'backgroundImage': `url(${staffMember.avatar})` }"
                            />
                            <div class="staff_row_members_card__line"/>
                            <div class="staff_row_members_card_details">
                                <div class="staff_row_members_card_details__username">
                                    {{ staffMember.username }}
                                </div>
                                <div
                                    v-if="staffMember.country"
                                    class="staff_row_members_card_details__nationality"
                                    :style="{ 'backgroundImage': `url(https://osu.ppy.sh/images/flags/${staffMember.country}.png)` }"
                                />
                                <div
                                    v-else
                                    class="staff_row_members_card_details__not_logged_in"
                                >
                                    {{ $t("open.staff.notLoggedIn") }}
                                </div>
                            </div>
                        </a>
                    </div>
                </div>
            </div>
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component } from "vue-property-decorator";
import { namespace } from "vuex-class";

import OpenTitle from "../../Assets/components/open/OpenTitle.vue";
import { Tournament, TournamentRoleType } from "../../Interfaces/tournament";
import { StaffList } from "../../Interfaces/staff";

const openModule = namespace("open");

@Component({
    components: {
        OpenTitle,
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
export default class Staff extends Vue {
    @openModule.State tournament!: Tournament | null;
    @openModule.State staffList!: StaffList[] | null;

    async mounted () {
        await this.$store.dispatch("open/setStaffList", this.tournament?.ID);
    }

    getRoleTypeName (roleType: TournamentRoleType): string {
        return TournamentRoleType[roleType];
    }
}
</script>

<style lang="scss">
@import '@s-sass/_mixins';
@import '@s-sass/_variables';

$flex-gap: 25px;

.staff {
    &_main_content {
        align-self: center;
        position: relative;
        width: 65vw;
        padding: 35px;

        &_staff_list {
            padding: 20px 0;
            display: flex;
            flex-direction: column;
            justify-content: space-around;
            gap: 20px;
        }
    }

    &_row {
        display: flex;
        flex-direction: column;
        align-items: center;

        &__title {
            font-size: $font-base;
            font-weight: bold;
            text-overflow: ellipsis;
            overflow: hidden;
            width: 100%;
            color: $open-red;
            margin-bottom: 10px;
        }

        &_members {
            display: flex;
            align-items: center;
            width: 100%;
            gap: $flex-gap;
            flex-wrap: wrap;

            &_card {
                display: flex;
                flex-basis: calc(25% - $flex-gap * 0.75); // 4 cards per row, 3 gaps of 25px per row
                text-overflow: ellipsis;
                overflow: hidden;
                height: 75px;
                flex-direction: row;
                align-items: center;

                background-color: white;

                &:hover {
                    text-decoration: none;
                    box-shadow: 0px 0px 8px 4px $darker-gray;
                }

                &__headshot {
                    height: 100%;
                    width: 75px;

                    background-size: cover;
                    background-repeat: no-repeat;
                    background-position: center;
                }

                &__line {
                    width: 10px;
                    height: 100%;
                    background-color: $open-red;
                }

                &_details {
                    display: flex;
                    flex-direction: column;
                    padding: 0 10px;

                    overflow: hidden;
                    text-overflow: ellipsis;
                    white-space: nowrap;

                    &__username {
                        color: $open-dark;
                        font-weight: bold;
                        line-height: 29px;
                        letter-spacing: 0em;
                        text-align: left;

                        overflow: hidden;
                        text-overflow: ellipsis;
                        white-space: nowrap;
                    }

                    &__nationality {
                        width: 16px;
                        height: 11px;

                        background-size: 100%;
                    }

                    &__not_logged_in {
                        font-size: $font-xsm;
                        font-weight: bold;
                        font-stretch: condensed;
                        line-height: 16px;
                        letter-spacing: 0em;
                        text-align: left;
                        color: $open-red;
                    }
                }
            }
        }
    }
}

</style>