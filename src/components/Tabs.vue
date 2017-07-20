<template>
    <div class="tabs-component">
        <ul role="tablist" :class="navClass">
            <li v-for="tab in tabs"
                :class="{ [navTabClass]: true, 'is-active': tab.isActive }"
                role="presentation"
            >

                <a v-html="tab.header"
                   :aria-controls="tab.hash"
                   :aria-selected="tab.isActive"
                   @click="selectTab(tab.hash)"
                   :href="tab.hash"
                   :class="navLinkClass"
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
                default: 'tabs-component-tabs'
            },
            navTabClass: {
                type: String,
                default: 'tabs-component-tab'
            },
            navLinkClass: {
                type: [Object, String],
                default: 'tabs-component-tab-a'
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
