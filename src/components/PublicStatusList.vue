<template>
    <!-- Group List -->
    <Draggable
        v-model="$root.publicGroupList"
        :disabled="!editMode"
        item-key="id"
        :animation="100"
    >
        <template #item="group">
            <div class="mb-5" data-testid="group">
                <!-- Group Title -->
                <h2 class="group-title">
                    <font-awesome-icon v-if="editMode && showGroupDrag" icon="arrows-alt-v" class="action drag me-3" />
                    <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeGroup(group.index)" />
                    <Editable v-model="group.element.name" :contenteditable="editMode" tag="span" data-testid="group-name" />
                </h2>
                <!-- three columns -->
                <div class="mt-4 position-relative">
                    <div v-if="group.element.monitorList.length === 0" class="text-center no-monitor-msg">
                        {{ $t("No Monitors") }}
                    </div>

                    <!-- Monitor List -->
                    <!-- animation is not working, no idea why -->
                    <Draggable
                        v-model="group.element.monitorList"
                        class="row"
                        group="same-group"
                        :disabled="!editMode"
                        :animation="100"
                        item-key="id"
                    >
                        <template #item="monitor">
                            <div class="col-12 col-md-4" data-testid="monitor" @click="toggleGroup(monitor.element)">
                                <div class="service">
                                    <font-awesome-icon v-if="editMode" icon="arrows-alt-v" class="action drag me-3" />
                                    <font-awesome-icon v-if="editMode" icon="times" class="action remove me-3" @click="removeMonitor(group.index, monitor.index)" />

                                    <font-awesome-icon v-if="monitor.element.type === 'group' && !monitor.element.expanded" icon="plus" class="me-3 icon-expand" />
                                    <font-awesome-icon v-if="monitor.element.type === 'group' && monitor.element.expanded" icon="minus" class="me-3 icon-expand" />
                                    <span class="service-name">{{ monitor.element.name }}</span>
                                    <IconStatus :display="!monitor.element.expanded" class="icon" :monitor="monitor.element" />
                                </div>
                                <Draggable
                                    v-if="monitor.element.expanded"
                                    v-model="monitor.element.children"
                                    group="same-group"
                                    :disabled="!editMode"
                                    :animation="100"
                                    item-key="id"
                                >
                                    <template #item="child">
                                        <div class="item sub-monitor-item" data-testid="child">
                                            <div class="sub-service">
                                                <span class="service-name">{{ child.element.name }}</span>
                                                <IconStatus :display="!child.element.expanded" class="icon" :monitor="child.element" />
                                            </div>
                                        </div>
                                    </template>
                                </Draggable>
                            </div>
                        </template>
                    </Draggable>
                </div>
            </div>
        </template>
    </Draggable>
    <MonitorSettingDialog ref="monitorSettingDialog" />
</template>

<script>
import MonitorSettingDialog from "./MonitorSettingDialog.vue";
import Draggable from "vuedraggable";
import IconStatus from "./IconStatus.vue";
import HeartbeatBar from "./HeartbeatBar.vue";

export default {
    components: {
        MonitorSettingDialog,
        Draggable,
        IconStatus,
    },
    props: {
        /** Are we in edit mode? */
        editMode: {
            type: Boolean,
            required: true,
        },
        /** Should tags be shown? */
        showTags: {
            type: Boolean,
        },
        /** Should expiry be shown? */
        showCertificateExpiry: {
            type: Boolean,
        }
    },
    data() {
        return {

        };
    },
    computed: {
        showGroupDrag() {
            return (this.$root.publicGroupList.length >= 2);
        }
    },
    created() {

    },
    methods: {
        /**
         * Remove the specified group
         * @param {number} index Index of group to remove
         * @returns {void}
         */
        removeGroup(index) {
            this.$root.publicGroupList.splice(index, 1);
        },

        /**
         * Remove a monitor from a group
         * @param {number} groupIndex Index of group to remove monitor
         * from
         * @param {number} index Index of monitor to remove
         * @returns {void}
         */
        removeMonitor(groupIndex, index) {
            this.$root.publicGroupList[groupIndex].monitorList.splice(index, 1);
        },

        /**
         * Should a link to the monitor be shown?
         * Attempts to guess if a link should be shown based upon if
         * sendUrl is set and if the URL is default or not.
         * @param {object} monitor Monitor to check
         * @param {boolean} ignoreSendUrl Should the presence of the sendUrl
         * property be ignored. This will only work in edit mode.
         * @returns {boolean} Should the link be shown
         */
        showLink(monitor, ignoreSendUrl = false) {
            // We must check if there are any elements in monitorList to
            // prevent undefined errors if it hasn't been loaded yet
            if (this.$parent.editMode && ignoreSendUrl && Object.keys(this.$root.monitorList).length) {
                return this.$root.monitorList[monitor.element.id].type === "http" || this.$root.monitorList[monitor.element.id].type === "keyword" || this.$root.monitorList[monitor.element.id].type === "json-query";
            }
            return monitor.element.sendUrl && monitor.element.url && monitor.element.url !== "https://";
        },

        /**
         * Returns formatted certificate expiry or Bad cert message
         * @param {object} monitor Monitor to show expiry for
         * @returns {string} Certificate expiry message
         */
        formattedCertExpiryMessage(monitor) {
            if (monitor?.element?.validCert && monitor?.element?.certExpiryDaysRemaining) {
                return monitor.element.certExpiryDaysRemaining + " " + this.$tc("day", monitor.element.certExpiryDaysRemaining);
            } else if (monitor?.element?.validCert === false) {
                return this.$t("noOrBadCertificate");
            } else {
                return this.$t("Unknown") + " " + this.$tc("day", 2);
            }
        },

        /**
         * Returns certificate expiry color based on days remaining
         * @param {object} monitor Monitor to show expiry for
         * @returns {string} Color for certificate expiry
         */
        certExpiryColor(monitor) {
            if (monitor?.element?.validCert && monitor.element.certExpiryDaysRemaining > 7) {
                return "#059669";
            }
            return "#DC2626";
        },

        toggleGroup(monitor) {
            console.log(monitor);
            if (typeof monitor.expanded === "undefined") {
                monitor.expanded = true;
            } else {
                monitor.expanded = !monitor.expanded;
            }
        }
    }
};
</script>

<style lang="scss" scoped>
@import "../assets/vars";

.no-monitor-msg {
    position: absolute;
    width: 100%;
    top: 20px;
    left: 0;
}

.monitor-list {
    min-height: 46px;
}

.item-name {
    padding-left: 5px;
    padding-right: 5px;
    margin: 0;
    display: inline-block;
}

.no-padding {
    padding: 0 !important;
}

.btn-link {
    color: #bbbbbb;
    margin-left: 5px;
}

.link-active {
    color: $primary;
}

.flip-list-move {
    transition: transform 0.5s;
}

.no-move {
    transition: transform 0s;
}

.drag {
    color: #bbb;
    cursor: grab;
}

.remove {
    color: $danger;
}

.group-title {
    span {
        display: inline-block;
        min-width: 15px;
    }
}

.mobile {
    .item {
        padding: 13px 0 0px;
    }
}

.icon-expand{
    width: 8px;
    vertical-align: middle;
}

.bg-maintenance {
    background-color: $maintenance;
}

.icon{
    font-size: 20px;
    float: right;
}

.service-name{
    vertical-align: middle;
    overflow: hidden;
    white-space: nowrap;
    text-overflow: ellipsis;
    display: inline-block;
    font-weight: 400;
    max-width: 75%;
}

.service {
  justify-content: space-between;
  padding: 8px 12px !important;
  border: 1px solid #212121;
  border-radius: 5px;
  margin-bottom: 5px;
  text-align: left;
  margin: 5px 0px;
  align-items: center;
  width: 100%;
}

.sub-service {
  justify-content: space-between;
  padding: 5px;
  border-top: 1px solid #212121;
  text-align: left;
  border-radius: 0px;
  width: 90%;
  margin: 0 auto;
  padding-left: 20px;
  align-items: center;
}

.monitor-grid {
  display: grid;
  grid-template-columns: repeat(3, 1fr);
  gap: 5px;
}

.monitor-item {
  background-color: #fff;
  padding: 5px;
  border-radius: 5px;
  text-align: center;
}

.sub-monitor-item{
  background-color: #fff;
  padding: 5px;
  margin: 0 auto;
  border-radius: 0px;
  text-align: center;
  border-left: 1px solid #212121;
  border-right: 1px solid #212121;
  border-bottom: 1px solid #212121;
  border-bottom-left-radius: 5px;
  border-bottom-right-radius: 5px;
  margin-top: -10px;
}

.action.toggle {
  cursor: pointer;
  font-size: 20px;
  color: #555;
}

</style>
