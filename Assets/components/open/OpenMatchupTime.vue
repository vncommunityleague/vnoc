<template>
    <div class="open_matchup_time">
        <div class="open_matchup_time__time">
            {{ tbdSync ? "TBD" : formatTime(dateSync) }}
        </div>
        <div
            v-if="!tbdSync"
            class="open_matchup_time__timezone"
        >
            {{ timezoneSync }}
        </div>
    </div>
</template>

<script lang="ts">
import { Vue, Component, PropSync } from "vue-property-decorator";

@Component
export default class OpenMatchupTime extends Vue {
    @PropSync("date", { type: Date, required: false, default: null }) dateSync!: Date;
    @PropSync("timezone", { type: String, default: "UTC" }) timezoneSync!: string;
    @PropSync("tbd", { type: Boolean, default: false }) tbdSync!: boolean;

    formatTime (date: Date | null): string {
        if (!date) return "";
        const utc = date.getTime() + (date.getTimezoneOffset() * 60000);
        const tzDate = new Date(utc + 7 * 3600000);
        const hours = tzDate.getHours();
        const minutes = tzDate.getMinutes();
        return `${hours < 10 ? "0" : ""}${hours}:${minutes < 10 ? "0" : ""}${minutes}`;
    }
}
</script>

<style lang="scss">
@import '@s-sass/_mixins';
@import '@s-sass/_variables';

.open_matchup_time {

    display: flex;
    flex-direction: column;
    justify-content: center;
    align-items: center;
    white-space: nowrap;
    font-family: $font-univers;

    &__time {
        font-size: calc(1.25 * $font-xxxl);
        font-weight: 700;
        font-stretch: condensed;
        letter-spacing: 0em;
        text-align: center;
    }

    &__timezone {
        font-size: $font-sm;
        font-weight: 600;
        color: $open-red;
    }
}
</style>