<template lang="html">
    <main
        id="main"
        class="page page-help-topic"
    >
        <mastheadSecondary
            :title="page.title"
            :text="page.text"
        />
        <section-wrapper v-if="page.richText">
            <RichText :rich-text-content="page.richText" />
        </section-wrapper>

        <div
            v-for="(block, index) in parsedHelpTopicBlocks"
            :key="`HelpTopicBlocksKey${index}`"
        >
            <section-wrapper>
                <simple-cards
                    :section-title="block.sectionTitle"
                    :section-summary="block.sectionSummary"
                    :items="block.parsedAssociatedEntries"
                />
            </section-wrapper>
            <section-wrapper theme="divider">
                <divider-way-finder
                    class="divider-way-finder"
                    color="help"
                />
            </section-wrapper>
        </div>

        <flexible-blocks
            class="content"
            :blocks="page.blocks"
        />

        <section-wrapper
            v-if="page.blocks.length > 0"
            theme="divider"
        >
            <divider-way-finder
                class="divider-way-finder"
                color="help"
            />
        </section-wrapper>

        <section-wrapper>
            <block-call-to-action
                class="block-call-to-action"
                :is-global="true"
            />
        </section-wrapper>
    </main>
</template>

<script>
// HELPERS
import _get from "lodash/get"
import removeTags from "~/utils/removeTags"

// GQL
import HELP_TOPIC_DETAIL from "~/gql/queries/HelpTopicDetail"

export default {
    async asyncData({ $graphql, params, $elasticsearchplugin, error }) {
        const data = await $graphql.default.request(HELP_TOPIC_DETAIL, {
            slug: params.slug,
        })
        if (!data.entry) {
            error({ statusCode: 404, message: "Page not found" })
        }
        if (data && params.slug !== undefined) {
            //console.log("Helptopics slugs Indexing slug: " + params.slug)
            await $elasticsearchplugin.index(data.entry, params.slug)
        }
        // //console.log("Data fetched: " + JSON.stringify(data))

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
    computed: {
        parsedHelpTopicBlocks() {
            return this.page.helpTopicBlocks.map((obj) => {
                return {
                    ...obj,
                    parsedAssociatedEntries: obj.associatedEntries.map(
                        (entry) => {
                            return {
                                ...entry,
                                to: entry.externalResourceUrl
                                    ? entry.externalResourceUrl
                                    : `/${entry.uri}`,
                            }
                        }
                    ),
                }
            })
        },
    },
}
</script>

<style lang="scss" scoped>
.page-help-topic {
}
</style>
