<template>
    <span :class="className">
        <font-awesome-icon :icon="icon" :class="color" />
    </span>
    <HeartbeatBar v-if="!display" size="mid" :monitor-id="monitorId" />
</template>

<script>
import { DOWN, MAINTENANCE, PENDING, UP } from "../util.ts";
import HeartbeatBar from "./HeartbeatBar.vue";

export default {
    components: {
        HeartbeatBar,
    },
    props: {
        /** Monitor this represents */
        monitor: {
            type: Object,
            default: null,
        },
        display: {
            type: Boolean,
            default: true,
        },
    },

    computed: {

        monitorId() {
            console.log(this.monitor.id);
            return this.monitor.id;
        },

        color() {
            console.log(this.lastHeartBeat.status);
            if (this.lastHeartBeat.status === MAINTENANCE) {
                return "maintenance status-icon";
            }

            if (this.lastHeartBeat.status === DOWN) {
                return "danger status-icon";
            }

            if (this.lastHeartBeat.status === UP) {
                return "ok status-icon";
            }

            if (this.lastHeartBeat.status === PENDING) {
                return "warning status-icon";
            }

            return "secondary status-icon";
        },

        icon() {
            if (this.lastHeartBeat.status === MAINTENANCE) {
                return "wrench";
            }

            if (this.lastHeartBeat.status === DOWN) {
                return "times";
            }

            if (this.lastHeartBeat.status === UP) {
                return "check";
            }

            if (this.lastHeartBeat.status === PENDING) {
                return "clock";
            }

            return "wrench";
        },

        lastHeartBeat() {
            console.log(this.monitor.id);
            console.log("lastHeartbeatList", this.$root.lastHeartbeatList);
            if (this.monitor.id in this.$root.lastHeartbeatList && this.$root.lastHeartbeatList[this.monitor.id]) {
                return this.$root.lastHeartbeatList[this.monitor.id];
            }

            return {
                status: -1,
            };
        },

        className() {
            return this.display ? "status " + this.color : "status status-hidden";
        },

    },
};
</script>

<style lang="scss" scoped>
@import "../assets/vars.scss";

.status-icon{
    width: 24px;
    height: 24px;
    border-radius: 50%;
}

.status-hidden{
    visibility: hidden;
}
.status {
    float: right;
    .maintenance {
        color: $maintenance;
        border: 2px solid $maintenance;
        padding: 5px 5px;
    }
    .ok {
        color: $primary;
        border: 2px solid $primary;
        padding: 5px 5px;
    }
    .warning {
        color: $warning;
        border: 2px solid $warning;
        padding: 5px 5px;
    }
    .danger {
        color: $danger;
        border: 2px solid $danger;
        padding: 5px 5px;
    }
}
</style>
