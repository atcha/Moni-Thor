<template>
    <div class="home">
        <md-app-toolbar class=" md-toolbar md-primary" md-elevation="0">
            <span>
                <h2>Home</h2>
            </span>
        </md-app-toolbar>
        <div class="md-layout">
          <div class="md-layout-item md-size-50">
            <md-card>
              <md-card-header>
                Tutorial
              </md-card-header>
              <md-card-content>
                  <vue-markdown :source="test"></vue-markdown>
              </md-card-content>
            </md-card>
          </div>
        </div>
    </div>
</template>

<script>
    import axios from 'axios';
    import Vue from 'vue'
    import VueMarkdown from 'vue-markdown'

    export default {
        components : {
          'vue-markdown': VueMarkdown
        },
        name: 'home',
        data() {
            return {
              test : `
# [Java8][1st Tutorial](U1st_tutorial)
*Test tutorial Markdown*
**test**
i am a ~~tast~~ **test**.
\`\`\` java
Interface1 i1 = (s) -> System.out.println(s);
i1.method1("abc");
\`\`\`
              `
            }
        },
        computed: {
        },
        methods: {
        },
        mounted() {
        }
    }
</script>
<style scoped>
    h1, h2 {
        font-weight: normal;
    }

  .home .md-card.md-theme-default{
    margin: 0;
  }

</style>
