<template lang="html">
    <main
        id="main"
        class="page page-ask-us"
    >
        <nav-breadcrumb
            to="/help/services-resources"
            title="Ask Us"
            parent-title="Services & Resources"
        />
        <banner-text
            class="banner-text"
            :title="page.title"
            :text="page.summary"
        />
        <!-- LibChat Widget -->
        <section-wrapper class="section-ask-us">
            <script src="https://ucla.libanswers.com/load_chat.php?hash=e6e621712e7b0ed0193f065d84d4e0c9" />
            <div id="libchat_e6e621712e7b0ed0193f065d84d4e0c9" />
        </section-wrapper>

        <section-wrapper theme="divider">
            <divider-way-finder
                v-if="page.blocks.length > 0"
                color="help"
                class="divider-way-finder"
            />
        </section-wrapper>

        <!-- Flexible Page Blocks -->
        <flexible-blocks
            class="flexible-content"
            :blocks="page.blocks"
        />
    </main>
</template>
<router>
  {
    alias: '/help',
  }
</router>
<script>
// HELPERS
import _get from "lodash/get"
import removeTags from "~/utils/removeTags"

// GQL
import ASK_US from "~/gql/queries/AskUs"

export default {
    async asyncData({ $graphql }) {
        const data = await $graphql.default.request(ASK_US)
        return {
            page: _get(data, "entry", {}),
        }
    },
    head() {
        let title = this.page ? this.page.title : "... loading"
        let metaDescription = removeTags(this.page.text)

        return {
            title: title,
            meta: [
                {
                    hid: "description",
                    name: "description",
                    content: metaDescription,
                },
            ],
        }
    },
}
</script>

<style lang="scss" scoped>
.page-ask-us {
    .banner-text {
        margin-bottom: var(--space-l);
    }
    .section-wrapper.section-ask-us {
        margin-top: 0;
    }
}
</style>
