<template>
    <div
        class="topbar-notification"
        v-if="notification.visible"
        :data-timestamp="notification.timestamp" >
        <span
            ref="content"
            v-html="notification.text">
        </span>

        <span
            class="topbar-notification-close"
            title="Hide this notification"
            @click="closeNotification">
            &times;
        </span>
    </div>
</template>

<script>
import { ipcRenderer, shell } from 'electron';

export default {
    name: 'topbar-notification',
    data () {
        return {
            contentEventsAdded: false
        }
    },
    computed: {
        notification: function() {
            return this.$store.state.app.notification;
        }
    },
    created: function() {
        this.getNotifications();
    },
    mounted: function() {
        let self = this;
    },
    methods: {
        getNotifications() {
            if(this.$store.state.app.notification) {
                return;
            }

            let self = this;

            ipcRenderer.send('app-notifications-retrieve', this.shouldRetrieveNotifications());

            ipcRenderer.once('app-notifications-retrieved', function(event, data) {
                if(data.status === true) {
                    self.setNotification(data);
                }

                localStorage.setItem('publii-notification-retrieve-timestamp', new Date().getTime());
            });
        },
        shouldRetrieveNotifications() {
            let currentTime = new Date().getTime();
            let lastRetrieveTime = localStorage.getItem('publii-notification-retrieve-timestamp');

            if(lastRetrieveTime !== null) {
                if(currentTime > parseInt(lastRetrieveTime, 10) + (2 * 60 * 60 * 1000)) {
                    return true;
                }
            } else {
                return true;
            }

            return false;
        },
        setNotification(data) {
            let closeTimestamp = localStorage.getItem('publii-notification-close-timestamp');

            if(closeTimestamp === null || data.notification.timestamp > parseInt(closeTimestamp, 10)) {
                this.$store.commit('setNotification', {
                    timestamp: data.notification.timestamp,
                    text: data.notification.text,
                    visible: true
                });
            } else {
                this.$store.commit('setNotification', {
                    timestamp: data.notification.timestamp,
                    text: data.notification.text,
                    visible: false
                });
            }

            setTimeout(() => {
                if(!this.$refs.content) {
                    return;
                }

                if (this.contentEventsAdded) {
                    return;
                }

                this.$refs.content.addEventListener('click', e => {
                    if(e.target.tagName === 'A') {
                        e.preventDefault();
                        shell.openExternal(e.target.getAttribute('href'));
                        this.closeNotification();
                    }
                });

                this.contentEventsAdded = true;
            }, 500);
        },
        closeNotification(event) {
            localStorage.setItem('publii-notification-close-timestamp', this.notification.timestamp);
            this.$store.commit('setNotification', false);
        }
    }
}
</script>

<style lang="scss" scoped>
@import '../scss/variables.scss';

.topbar {
    &-notification {
        -webkit-app-region: no-drag; // Make the links clickable again   
        align-items: center;
        border-right: 1px solid $color-helper-8;
        display: inline-flex;
        font-size: 1.4rem;
        font-weight: 400;
        margin: 0 2rem 0 0;
        order: 1;
        padding: .9rem 2rem;
        position: relative;

        &-close {
            -webkit-app-region: no-drag; // Make the button clickable again   
            background: $color-9;
            border-radius: 50%;
            color: $color-7;
            cursor: pointer;
            font-size: 2.1rem;
            font-weight: 300;
            height: 2.4rem;
            left: auto;
            line-height: 1.1; 
            margin: 0 1.9rem;
            text-align: center;       
            transition: all .3s ease-out;            
            width: 2.4rem;
                                
            &:active,
            &:focus,
            &:hover {
                color: $color-4;
            }
        
            &:hover {
                background: $color-helper-8;
            }  
        }
    }
}
</style>
