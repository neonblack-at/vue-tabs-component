<template>
    <div class="tabs-component">
        <ul role="tablist" class="tabs-component-tabs" :class="navClass">
            <li v-for="tab in tabs"
                :class="{ [navTabClass]: true, 'is-active': tab.isActive }"
                class="tabs-component-tab"
                role="presentation"
            >

                <a v-html="tab.header"
                   :aria-controls="tab.hash"
                   :aria-selected="tab.isActive"
                   @click="selectTab(tab.hash)"
                   :href="tab.hash"
                   class="tabs-component-tab-a"
                   role="tab"
                ></a>

            </li>
        </ul>
        <div class="tabs-component-panels">
            <slot/>
        </div>
    </div>
</template>

<script>
    import expiringStorage from '../expiringStorage';

    export default {
        props: {
            cacheLifetime: { default: 5 },
            navClass: {
                type: String,
                default: ''
            },
            navTabClass: {
                type: String,
                default: ''
            }
        },

        data: () => ({
            tabs: [],
            activeTabHash: '',
        }),

        computed: {
            storageKey() {
                return `vue-tabs-component.cache.${window.location.host}${window.location.pathname}`;
            },
        },

        created() {
            this.tabs = this.$children;
        },

        mounted() {
            window.addEventListener('hashchange', () => this.selectTab(window.location.hash));

            if (this.findTab(window.location.hash)) {
                this.selectTab(window.location.hash);
                return;
            }

            const previousSelectedTabHash = expiringStorage.get(this.storageKey);

            if (this.findTab(previousSelectedTabHash)) {
                this.selectTab(previousSelectedTabHash);
                return;
            }

            if (this.tabs.length) {
                this.selectTab(this.tabs[0].hash);
            }
        },

        methods: {
            findTab(hash) {
                return this.tabs.find(tab => tab.hash === hash);
            },

            selectTab(selectedTabHash) {
                const selectedTab = this.findTab(selectedTabHash);

                if (! selectedTab) {
                    return;
                }

                this.tabs.forEach(tab => {
                    tab.isActive = (tab.hash === selectedTab.hash);
                });

                this.$emit('changed', { tab: selectedTab });

                this.activeTabHash = selectedTab.hash;

                expiringStorage.set(this.storageKey, selectedTab.hash, this.cacheLifetime);
            },
        },
    };
</script>
